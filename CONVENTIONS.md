# SpecWorks Factory Conventions

This document defines the conventions and patterns used across all SpecWorks components ("Parts"). These conventions ensure consistency, maintainability, and alignment with the [SpecWorks Factory pattern](README.md).

## Table of Contents
- [Project Structure](#project-structure)
- [Naming Conventions](#naming-conventions)
- [.NET Conventions](#net-conventions)
- [Documentation Standards](#documentation-standards)
- [Testing Requirements](#testing-requirements)
- [Specification Linksets](#specification-linksets)
- [Architecture Decision Records](#architecture-decision-records)
- [License and Legal](#license-and-legal)

---

## Project Structure

### Multi-Language Projects

For components implemented in multiple languages (e.g., vCard):

```
<project>/
├── README.md              # Overview of all implementations
├── specs.json            # Specification linkset (RFC 9264)
├── adr/                  # Architecture Decision Records
│   ├── 0001-<topic>.md
│   └── 0002-<topic>.md
├── dotnet/               # .NET implementation
│   ├── .gitignore
│   ├── README.md
│   ├── <Project>.sln
│   ├── src/<Project>/
│   └── tests/<Project>.Tests/
├── python/               # Python implementation
│   ├── README.md
│   ├── src/<package>/
│   └── tests/
├── rust/                 # Rust implementation
│   ├── README.md
│   ├── Cargo.toml
│   └── src/
└── testcases/           # Shared test fixtures
    └── README.md
```

**Key principles:**
- Root README provides overview of ALL language implementations
- Each language has its own subdirectory with dedicated README
- Shared test cases in `testcases/` directory
- ADRs at root level (cross-language decisions)

### Single-Language Projects

For .NET-only components (most common):

```
<project>/
├── README.md            # Optional: high-level overview
├── specs.json          # At project root
├── dotnet/
│   ├── README.md        # Primary documentation
│   ├── .gitignore
│   ├── <Project>.sln
│   ├── src/
│   │   └── <Project>/
│   │       └── <Project>.csproj
│   └── tests/
│       └── <Project>.Tests/
│           └── <Project>.Tests.csproj
```

**Key principles:**
- Primary documentation in `dotnet/README.md`
- Solution file at root of `dotnet/` directory
- Standard `src/` and `tests/` subdirectories

---

## Naming Conventions

### Repository Names
- **PascalCase** (e.g., `JsonDiff`, `MarkMyWord`, `RateLimiter`)
- Use standard/specification names where appropriate (e.g., `vCard`, `iCalendar`)
- No prefixes or suffixes (clean names)

### .NET Namespaces
All components should be SpecWorks branded.

**SpecWorks-branded** (for factory-specific components):
- `SpecWorks.JsonDiff`
- `SpecWorks.Linkset`

### .NET Project Files
- Source: `src/<ProjectName>/<ProjectName>.csproj`
- Tests: `tests/<ProjectName>.Tests/<ProjectName>.Tests.csproj`
- Project name matches namespace (e.g., `VCard.csproj` for `VCard` namespace)

### NuGet Package Names
- Match the namespace: `SpecWorks.JsonDiff`

---

## .NET Conventions

### Target Frameworks
All .NET projects must multi-target:

```xml
<TargetFrameworks>net10.0;net8.0</TargetFrameworks>
```

**Rationale:** Support both latest .NET (10.0) and current LTS version (8.0).

**ADR Reference:** See `specification/adr/` for organization-level decision.

### Project Settings (Required)

```xml
<PropertyGroup>
  <TargetFrameworks>net10.0;net8.0</TargetFrameworks>
  <Nullable>enable</Nullable>
  <ImplicitUsings>enable</ImplicitUsings>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <LangVersion>latest</LangVersion>
</PropertyGroup>
```

**Required settings:**
- **Nullable reference types** - Must be enabled
- **Implicit usings** - Enabled for modern C# experience
- **XML documentation** - Always generate for IntelliSense
- **Latest language version** - Use newest C# features

### NuGet Package Metadata (Required)

```xml
<PropertyGroup>
  <PackageId><!-- Namespace --></PackageId>
  <Version><!-- SemVer --></Version>
  <Authors>SpecWorks</Authors>
  <Company>SpecWorks</Company>
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
  <PackageReadmeFile>README.md</PackageReadmeFile>
  <PackageProjectUrl>https://github.com/spec-works/<repo></PackageProjectUrl>
  <RepositoryUrl>https://github.com/spec-works/<repo></RepositoryUrl>
  <RepositoryType>git</RepositoryType>
  <Description><!-- Brief description --></Description>
  <PackageTags><!-- relevant tags --></PackageTags>
</PropertyGroup>
```

### Source Link (Required)

Enable Source Link for debugging:

```xml
<PropertyGroup>
  <PublishRepositoryUrl>true</PublishRepositoryUrl>
  <EmbedUntrackedSources>true</EmbedUntrackedSources>
  <IncludeSymbols>true</IncludeSymbols>
  <SymbolPackageFormat>snupkg</SymbolPackageFormat>
</PropertyGroup>

<ItemGroup>
  <PackageReference Include="Microsoft.SourceLink.GitHub" Version="*" PrivateAssets="All"/>
</ItemGroup>
```

### Code Style
- **File-scoped namespaces** - Use modern C# style
- **PascalCase** for all public APIs
- **Explicit types** preferred over `var` in public examples
- **XML documentation** on all public types and members

---

## Documentation Standards

### Root README.md Structure

Every project must have a comprehensive README with:

1. **Title and Badges**
   ```markdown
   # Project Name

   ![Version](badge-url)
   ![Languages](badge-url)
   ![Status](badge-url)
   ```

2. **Overview**
   - Brief description (1-2 sentences)
   - Link to primary specification (RFC, W3C, ISO)

3. **Features**
   - Bullet list with checkmarks
   - Highlight specification compliance
   - Note test coverage (e.g., "42+ tests")

4. **Quick Start** (for each language)
   ```markdown
   ## Quick Start (.NET)

   ### Installation
   ### Usage
   ### Example
   ```

5. **Project Structure**
   - Visual tree diagram of repository layout
   - Brief explanation of each directory

6. **Specification Compliance**
   - Table mapping features to specification sections
   - Note any unsupported optional features

7. **Testing**
   - Test count (prominently displayed)
   - Coverage information
   - How to run tests

8. **License**
   - MIT License
   - Link to LICENSE file

### Language-Specific READMEs

Each language subdirectory should have detailed documentation:

- Installation instructions (platform-specific)
- API reference or examples
- Platform-specific configuration
- Build instructions
- Testing instructions

### Documentation Tone
- **Specification-focused** - Reference RFC/standard sections
- **Precise** - Avoid vague terms like "inspired by"
- **Complete** - Document all public APIs
- **Example-rich** - Show real usage

---

## Testing Requirements

### Test Coverage
- **Comprehensive** - Aim for high coverage of specification requirements
- **Advertise** - Display test count prominently in README
- **Real-world** - Include actual payloads from specifications

### Test Organization
- Separate test projects (e.g., `<Project>.Tests`)
- Use standard testing framework for platform:
  - .NET: xUnit, NUnit, or MSTest
  - Python: pytest
  - Rust: built-in test framework

### Test Documentation
Show test counts in README:
```markdown
## Testing
This library includes **42+ tests** covering:
- Specification compliance tests
- Edge case handling
- Real-world payload examples
```

### Shared Test Cases
For multi-language projects:
- Place shared fixtures in `testcases/` directory
- Document test case format in `testcases/README.md`
- Ensure all language implementations pass same tests

---

## Specification Linksets

Every project must include a `specs.json` file at the root level using RFC 9264 linkset format.

### Required Links

**Minimum required:**
- Primary specification (with `rel: describedby`)

**Recommended:**
- Related specifications
- IANA registrations
- Errata pages
- Reference implementations
- Official websites

### Format

```json
{
  "linkset": [
    {
      "href": "https://www.rfc-editor.org/rfc/rfcXXXX.html",
      "type": "text/html",
      "rel": "describedby",
      "title": "RFC XXXX - Specification Name"
    },
    {
      "href": "https://www.iana.org/assignments/...",
      "type": "text/html",
      "rel": "related",
      "title": "IANA Registry"
    }
  ]
}
```

### Placement
- **Multi-language projects:** Root directory
- **Single-language projects:** Root directory (not in language subdirectory)

---

## Architecture Decision Records

### When to Create ADRs

**Organization-level decisions** → `specification/adr/`:
- Framework/language version policies
- Organization-wide patterns
- Naming conventions
- Cross-project architectural choices

**Project-level decisions** → `<project>/adr/`:
- API design choices
- Specification version support
- Implementation trade-offs
- Library dependencies
- Feature inclusion/exclusion

### ADR Format

Use this standard template:

```markdown
# ADR NNNN: <Title>

## Status
Accepted | Proposed | Deprecated | Superseded

## Context
<Problem description, background, constraints>

## Decision
<The decision made>

## Consequences

### Positive
- Benefit 1
- Benefit 2

### Negative
- Trade-off 1
- Trade-off 2

### Mitigation
- How to address negative consequences

## References
- [RFC XXXX](url)
- [Related ADR](url)

## Date
YYYY-MM-DD
```

### ADR Numbering
- Four digits with leading zeros: `0001`, `0002`, etc.
- Sequential within repository
- Never reuse numbers

### Common ADR Topics
- Specification version support
- API design patterns
- Type safety trade-offs
- Performance vs. specification purity
- Migration paths for breaking changes

---

## License and Legal

### License
All SpecWorks projects use **MIT License**.

### Files Required
- `LICENSE` file at repository root (for single-language)
- `LICENSE` file at project root (for multi-language)

### Package Metadata
```xml
<PackageLicenseExpression>MIT</PackageLicenseExpression>
```

### Copyright
Copyright holder: SpecWorks

---

## Design Principles

While implementing SpecWorks components, adhere to these principles:

1. **Specification Purity**
   - Implement specifications completely and correctly
   - Avoid proprietary extensions
   - Document deviations in ADRs

2. **Interoperability Over Features**
   - Standards compliance is more valuable than convenience features
   - When in doubt, follow the specification exactly

3. **Type Safety**
   - Use strong types where possible
   - Prevent invalid states at compile time
   - Make correct usage discoverable

4. **Test-Driven Specification Implementation**
   - Write tests based on specification examples
   - High test coverage demonstrates compliance
   - Prominently display test counts

5. **Documentation as Specification Translation**
   - Help developers understand specifications
   - Link code to specification sections
   - Provide examples that mirror spec examples

6. **AI-Assisted Development**
   - Document patterns to enable AI code generation
   - Use consistent structures for predictability
   - Maintain human-readable conventions

---

## Compliance Checklist

Use this checklist when creating a new SpecWorks component:

**Project Structure:**
- [ ] Correct directory structure (multi-language or single-language)
- [ ] `specs.json` at project root
- [ ] README.md at appropriate level(s)
- [ ] LICENSE file present

**.NET Projects:**
- [ ] Multi-target: net10.0;net8.0
- [ ] Nullable reference types enabled
- [ ] XML documentation generated
- [ ] Source Link configured
- [ ] NuGet metadata complete

**Documentation:**
- [ ] README includes specification link
- [ ] Features mapped to specification
- [ ] Test count displayed prominently
- [ ] Installation instructions present
- [ ] Examples demonstrate core functionality

**Testing:**
- [ ] Test project created
- [ ] Specification compliance tests
- [ ] Real-world payloads tested
- [ ] Test count > 20 (target)

**Specification Linkset:**
- [ ] `specs.json` present
- [ ] Primary specification linked
- [ ] Related standards linked (if any)

**ADRs:**
- [ ] Major design decisions documented
- [ ] API design choices explained
- [ ] Trade-offs identified

**Quality:**
- [ ] Builds without warnings
- [ ] All tests pass
- [ ] Public APIs documented
- [ ] Package metadata complete

---

## Examples and Templates

See the `examples/` directory in this repository for:
- `template-readme.md` - README structure template
- `template-adr.md` - ADR format template
- `template-csproj.xml` - Standard .NET project file
- `template-specs.json` - Specification linkset template

## Questions or Suggestions?

For questions about these conventions or suggestions for improvements:
- Open an issue in the `specification` repository
- Reference specific convention by section name
- Propose changes via ADR format

---

**Document Version:** 1.0.0
**Last Updated:** 2026-01-09
**Status:** Active
