# ADR 0001: Multi-Repository Organization for Factory Parts

## Status

Accepted

## Context

The SpecWorks Factory produces software components ("Parts") that implement publicly available specifications (RFCs, W3C standards, etc.). Each Part may be implemented in multiple programming languages (e.g., .NET, Python, Rust), and the factory is designed to be operated by AI agents that can work in parallel to generate, test, and maintain these implementations.

We need to decide on the organizational structure for hosting these Parts:
1. **Multi-repo**: Each Part has its own repository (e.g., `spec-works/vCard`, `spec-works/iCalendar`)
2. **Mono-repo**: All Parts live in a single repository (e.g., `spec-works/factory` with subdirectories)

### Key Requirements

1. **Specification-Centric Discovery**: Developers should discover Parts by searching for solutions to problems (e.g., "contact information standard"), not by implementation language
2. **AI Agent Parallelism**: Multiple AI agent instances must be able to work on different Parts simultaneously without conflicts
3. **Independent Part Lifecycle**: Each Part evolves based on its specification's lifecycle, not the factory's lifecycle
4. **Factory Replicability**: Others should be able to create their own factories using our patterns
5. **Minimal Coordination Overhead**: Parts implementing different specifications should not require coordination

### Operational Context

The factory is designed for AI-agent-driven parallel operations:
- Multiple agents work concurrently on different Parts
- Each agent needs isolated workspace (git repository, file system, build outputs)
- Agents create branches, run tests, commit changes, and push independently
- No inter-agent coordination should be required for independent Parts

### Discovery Model

Developers interact with the factory through IDE agents/tools, not directly via GitHub:
```
Developer: "I need to represent contact information"
  → IDE Agent queries xRegistry
  → Finds Part: "vCard (RFC 6350)"
  → Shows available implementations: .NET, Python, Rust
  → Developer selects/requests implementation
```

## Decision

We will organize the factory as a **GitHub organization with multiple repositories**, where:
- **Each Part = One Repository** (e.g., `spec-works/vCard`)
- **Repository name = Problem space** addressed by the specification(s)
- **One xRegistry entry per Part** with multiple library links
- **Shared infrastructure** in separate repositories:
  - `spec-works/factory` (formerly `specification`) - Factory pattern documentation and templates
  - `spec-works/.github` - Reusable workflow templates

### Repository Structure

Each Part repository contains:
```
spec-works/vCard/
├── README.md              # Part overview (problem space, specifications)
├── specs.json            # Linkset with specification and library links
├── adr/                  # Part-specific Architecture Decision Records
├── testcases/            # Shared test fixtures across languages
├── dotnet/               # .NET implementation
│   ├── src/
│   └── tests/
├── python/               # Python implementation (if exists)
│   ├── src/
│   └── tests/
└── rust/                 # Rust implementation (if exists)
    ├── src/
    └── tests/
```

### Factory Infrastructure

```
spec-works/factory/       # Factory pattern documentation (renamed from specification)
├── README.md            # Factory pattern specification
├── CONVENTIONS.md       # Coding and organizational conventions
├── examples/            # Templates for creating new Parts
└── adr/                 # Factory-level Architecture Decision Records

spec-works/.github/      # Organization-wide infrastructure
├── profile/
│   └── README.md        # Organization description
└── workflows/           # Reusable workflow templates
    ├── dotnet-test.yml
    ├── python-test.yml
    └── rust-test.yml
```

## Consequences

### Positive

- **Agent Isolation**: Multiple AI agents can work in parallel without git lock contention, branch conflicts, or build interference
- **Focused Cloning**: Each agent clones only the Part it needs (~10MB), not entire factory history
- **Independent Lifecycles**: Parts version and release independently based on their specification's evolution
- **Clear Problem Spaces**: Each repository represents one problem domain (contact info, calendaring, JSON patching)
- **Simplified Agent Instructions**: "Clone vCard, implement feature X, test, push" - no coordination needed
- **Granular CI/CD**: Changes to vCard trigger only vCard CI, not entire factory
- **Easier Failure Isolation**: Agent working on vCard fails independently; other agents unaffected
- **Natural Work Units**: Each Part is an atomic unit of work for agents
- **Resource Optimization**: Faster clone times, less disk space per agent instance

### Negative

- **Coordination for Cross-Part Changes**: Changes affecting multiple Parts require multiple PRs
- **Convention Enforcement**: Need to rely on templates and documentation rather than enforced structure
- **More GitHub Repositories**: 6+ repos instead of 1 (currently: 6 Parts + factory + .github)
- **Potential Convention Drift**: Parts might diverge from conventions over time without central enforcement

### Mitigation

- **Factory Repository**: Provides canonical documentation, conventions, and templates
- **Reusable Workflows**: Shared CI/CD in `.github` repository ensures consistency
- **xRegistry Validation**: Can validate all Parts conform to factory requirements
- **Review Process**: Factory maintainers review Parts for convention compliance
- **Templates**: New Parts start from templates, ensuring initial consistency

## Alternatives Considered

### Alternative 1: Mono-Repo with All Parts

**Description:** Single `spec-works/factory` repository containing all Parts as subdirectories

```
spec-works/factory/
├── vCard/
├── iCalendar/
├── JsonDiff/
└── ...
```

**Pros:**
- Unified codebase and commit history
- Enforced consistent structure
- Single clone gets everything
- Cross-Part refactoring in single PR
- GitHub search across all Parts

**Cons:**
- **Git Lock Contention**: Multiple agents competing for `.git/index.lock` when pushing simultaneously
- **Branch Management Chaos**: All agents creating branches in same repository
- **Build Tool Conflicts**: Shared `bin/`, `obj/`, lock files cause conflicts
- **Full Clone Required**: Each agent must clone entire factory history (100s of MB)
- **CI/CD Thrashing**: Every push triggers full mono-repo CI
- **Context Confusion**: Agent working on vCard sees all other Parts
- **Coordination Overhead**: Agents must coordinate branch names, merge conflicts

**Rejected because:** Fundamentally incompatible with parallel AI-agent operations. Git is designed for sequential coordination, not parallel independent work in same repository.

### Alternative 2: Hybrid (One Repo per Language Implementation)

**Description:** Separate repos for each language implementation (e.g., `vCard-dotnet`, `vCard-python`, `vCard-rust`)

**Pros:**
- Maximum isolation
- Language-specific conventions
- Independent versioning per implementation

**Cons:**
- Breaks specification-centric discovery model
- 3x repositories per Part (vCard becomes 3 repos)
- Shared test cases and ADRs must live elsewhere
- Violates "Part = Problem Space" principle
- Developer searching for "contact info" finds 3 separate results

**Rejected because:** Contradicts the specification-centric design. Developers discover by problem, not by language. A Part addresses one problem space, regardless of implementation languages.

## Implementation Notes

### Migration from Current State

The current local setup already resembles this structure:
```
C:\src\github\spec-works\
├── specification/     → Rename to "factory"
├── .github/          → Keep as-is
├── vCard/            → Already separate repo
├── iCalendar/        → Already separate repo
├── JsonDiff/         → Already separate repo
└── ...               → Already separate repos
```

Action items:
1. Rename `specification` repository to `factory`
2. Update all references in documentation
3. Update CONVENTIONS.md to reflect multi-repo architecture
4. Document the local mono-repo convenience pattern (optional for developers)

### Agent Workflow Pattern

When factory receives request for new feature:
1. Spawn agent instance
2. Agent clones relevant Part repository: `git clone github.com/spec-works/vCard`
3. Agent creates feature branch: `git checkout -b feat/add-tel-property`
4. Agent makes changes, runs tests
5. Agent commits and pushes: `git push origin feat/add-tel-property`
6. Agent creates PR
7. On merge, Part-specific CI runs
8. Package published to NuGet/PyPI/crates.io

Multiple agents operate simultaneously without interference.

### Shared Infrastructure Usage

Parts use shared infrastructure via:
- **Workflow templates**: `.github/workflows/dotnet-test.yml` used by all .NET Parts
- **Convention documentation**: `factory/CONVENTIONS.md` referenced in Part READMEs
- **Templates**: `factory/examples/template-csproj.xml` used when creating new Parts
- **xRegistry**: Central catalog at `spec-works.github.io/registry` indexes all Parts

## References

- [SpecWorks Factory Specification](../README.md)
- [GitHub Actions Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [xRegistry Specification](https://github.com/xregistry/spec)

## Date

2026-01-11

## Author

SpecWorks Team

