# SpecWorks Monorepo Factory Guide

**Version**: 1.0-draft
**Date**: 2026-02-08
**Status**: Draft

## Abstract

This document provides guidance for creating a SpecWorks-compliant software
component factory within a **single monorepo**. It translates every convention,
pattern, and requirement from the multi-repo SpecWorks factory into a structure
that works when an organization can only create a single repository.

The SpecWorks Factory specification (see [README.md](README.md)) is deliberately
neutral on repository organization (Section 1.4 Non-Goals). This guide shows how
to satisfy all factory requirements — xRegistry cataloging, linksets, CI/CD,
testing, documentation, and publishing — inside one repository.

---

## Table of Contents

- [1. When to Use a Monorepo Factory](#1-when-to-use-a-monorepo-factory)
- [2. Repository Structure](#2-repository-structure)
- [3. Factory Infrastructure](#3-factory-infrastructure)
- [4. Part Structure](#4-part-structure)
- [5. Naming Conventions](#5-naming-conventions)
- [6. Specification Linksets](#6-specification-linksets)
- [7. CI/CD Workflows](#7-cicd-workflows)
- [8. Testing](#8-testing)
- [9. Documentation](#9-documentation)
- [10. xRegistry Catalog](#10-xregistry-catalog)
- [11. Publishing](#11-publishing)
- [12. Architecture Decision Records](#12-architecture-decision-records)
- [13. .NET Conventions](#13-net-conventions)
- [14. Multi-Language Parts](#14-multi-language-parts)
- [15. AI Agent Workflow](#15-ai-agent-workflow)
- [16. Migration from Multi-Repo](#16-migration-from-multi-repo)
- [17. Compliance Checklist](#17-compliance-checklist)
- [Appendix A: Complete Example](#appendix-a-complete-example)
- [Appendix B: Monorepo vs Multi-Repo Trade-offs](#appendix-b-monorepo-vs-multi-repo-trade-offs)

---

## 1. When to Use a Monorepo Factory

Use a monorepo factory when:

- Your organization restricts you to **a single repository**
- You want **unified versioning and commit history** across all Parts
- You prefer **centralized governance** with enforced structure
- Your team is small and **sequential coordination** is acceptable
- You want **cross-Part refactoring** in a single PR

The monorepo pattern trades AI-agent parallelism (a key multi-repo advantage)
for structural enforcement and operational simplicity. See
[Appendix B](#appendix-b-monorepo-vs-multi-repo-trade-offs) for a detailed
comparison.

---

## 2. Repository Structure

### Top-Level Layout

```
inner-spec-works/                    # Single monorepo
├── README.md                        # Factory overview (org profile equivalent)
├── LICENSE                          # MIT License (covers entire factory)
├── .gitignore                       # Global gitignore
├── .github/
│   └── workflows/                   # All CI/CD workflows
│       ├── dotnet-test.yml          # .NET test workflow (path-filtered)
│       ├── python-test.yml          # Python test workflow (path-filtered)
│       ├── rust-test.yml            # Rust test workflow (path-filtered)
│       ├── build-and-publish.yml    # Package publishing workflow
│       ├── update-registry.yml      # xRegistry catalog update
│       └── docs.yml                 # Documentation site deployment
├── factory/                         # Factory pattern documentation
│   ├── README.md                    # Factory specification (from specification/README.md)
│   ├── CONVENTIONS.md               # Coding conventions
│   ├── MONOREPO-FACTORY.md          # This document
│   ├── REGISTRY.md                  # xRegistry usage guide
│   ├── component-spec.md            # Component registry spec
│   ├── adr/                         # Factory-level ADRs
│   │   ├── 0001-monorepo-organization.md
│   │   └── ...
│   ├── examples/                    # Templates
│   │   ├── template-readme.md
│   │   ├── template-csproj.xml
│   │   ├── template-specs.json
│   │   └── template-adr.md
│   └── tools/                       # Factory tooling
│       ├── generate-registry.ps1
│       └── validate-factory.ps1
├── parts/                           # ═══ ALL PARTS LIVE HERE ═══
│   ├── vCard/
│   ├── iCalendar/
│   ├── JsonDiff/
│   ├── linkset/
│   ├── RateLimiter/
│   ├── message/
│   ├── MarkMyWord/
│   └── MarkMyDeck/
├── registry/                        # Generated xRegistry catalog
│   ├── index.json
│   └── parts/
│       ├── index.json
│       └── <part-id>/
└── docs/                            # Documentation site (DocFX / GitHub Pages)
    ├── docfx.json
    └── index.md
```

### Key Differences from Multi-Repo

| Aspect | Multi-Repo | Monorepo |
|--------|-----------|----------|
| Parts location | Each Part = separate repo | All Parts under `parts/` directory |
| Factory docs | `specification/` repo | `factory/` directory |
| Org profile | `.github/profile/README.md` | Root `README.md` |
| Shared workflows | `.github` repo with reusable workflows | `.github/workflows/` with path filters |
| xRegistry output | `spec-works.github.io/registry/` | `registry/` directory (deployed to Pages) |
| License | Per-repo LICENSE | Single root LICENSE |

---

## 3. Factory Infrastructure

### 3.1 Root README.md

The root `README.md` serves as both the organization profile and the factory
overview. It MUST include:

1. **Factory name and description**
2. **Goals** (specification-driven, AI-assisted, trusted components)
3. **Parts inventory table** (name, specification, languages, status)
4. **Links** to factory documentation, registry, and conventions
5. **Quick start** for using Parts

Example:

```markdown
# Inner SpecWorks

A specification-driven software component factory in a single repository.

## Parts

| Part | Specification | Languages | Status |
|------|--------------|-----------|--------|
| [vCard](parts/vCard/) | [RFC 6350](https://www.rfc-editor.org/rfc/rfc6350.html) | .NET, Python, Rust | Production |
| [JsonDiff](parts/JsonDiff/) | [RFC 6902](https://www.rfc-editor.org/rfc/rfc6902.html) | .NET | Production |
| [iCalendar](parts/iCalendar/) | [RFC 5545](https://www.rfc-editor.org/rfc/rfc5545.html) | .NET | Production |

## Documentation

- [Factory Specification](factory/README.md)
- [Conventions](factory/CONVENTIONS.md)
- [xRegistry Catalog](registry/)
```

### 3.2 Factory Documentation Directory

The `factory/` directory replaces the `specification` repository. Copy the
following files from the multi-repo factory, adjusting internal links:

| Multi-Repo Source | Monorepo Destination |
|-------------------|---------------------|
| `specification/README.md` | `factory/README.md` |
| `specification/CONVENTIONS.md` | `factory/CONVENTIONS.md` |
| `specification/REGISTRY.md` | `factory/REGISTRY.md` |
| `specification/component-spec.md` | `factory/component-spec.md` |
| `specification/adr/` | `factory/adr/` |
| `specification/examples/` | `factory/examples/` |
| `specification/tools/` | `factory/tools/` |

### 3.3 Global .gitignore

Combine all language-specific ignores into one root `.gitignore`:

```gitignore
# .NET
bin/
obj/
*.user
*.suo
*.nupkg
*.snupkg

# Python
__pycache__/
*.pyc
*.egg-info/
dist/
build/
.venv/

# Rust
target/

# Node.js / TypeScript
node_modules/
dist/

# IDE
.vs/
.vscode/
.idea/

# OS
Thumbs.db
.DS_Store
```

---

## 4. Part Structure

### 4.1 Single-Language Part

Each Part lives under `parts/<PartName>/` and follows the same internal
structure as a multi-repo Part:

```
parts/JsonDiff/
├── README.md              # Part overview
├── specs.json             # Specification linkset (RFC 9264)
├── adr/                   # Part-specific ADRs
│   └── 0001-rfc-6902-only.md
└── dotnet/
    ├── README.md          # .NET-specific docs
    ├── JsonDiff.sln
    ├── src/
    │   └── SpecWorks.JsonDiff/
    │       └── SpecWorks.JsonDiff.csproj
    └── tests/
        └── SpecWorks.JsonDiff.Tests/
            └── SpecWorks.JsonDiff.Tests.csproj
```

### 4.2 Multi-Language Part

```
parts/vCard/
├── README.md              # Overview of ALL implementations
├── specs.json             # Specification linkset
├── adr/                   # Cross-language ADRs
│   ├── 0001-parse-returns-list.md
│   └── 0004-vcard-version-4-only.md
├── testcases/             # Shared test fixtures
│   └── README.md
├── dotnet/
│   ├── README.md
│   ├── VCard.sln
│   ├── src/VCard/
│   └── tests/VCard.Tests/
├── python/
│   ├── README.md
│   ├── src/vcard/
│   └── tests/
└── rust/
    ├── README.md
    ├── Cargo.toml
    └── src/
```

### 4.3 Part with CLI Tool

For Parts that include a command-line tool (e.g., MarkMyWord, MarkMyDeck):

```
parts/MarkMyWord/
├── README.md
├── specs.json
├── dotnet/
│   ├── MarkMyWord.sln
│   ├── src/
│   │   ├── MarkMyWord/               # Library
│   │   │   └── MarkMyWord.csproj
│   │   └── MarkMyWord.Cli/           # CLI tool (PackAsTool=true)
│   │       └── MarkMyWord.Cli.csproj
│   └── tests/
│       └── MarkMyWord.Tests/
```

---

## 5. Naming Conventions

All multi-repo naming conventions apply unchanged:

| Element | Convention | Example |
|---------|-----------|---------|
| Part directory | PascalCase or spec name | `parts/JsonDiff/`, `parts/vCard/` |
| .NET namespace | SpecWorks-branded or standalone | `SpecWorks.JsonDiff`, `VCard` |
| .NET project | Matches namespace | `SpecWorks.JsonDiff.csproj` |
| NuGet package | Matches namespace | `SpecWorks.JsonDiff` |
| CLI tool command | Lowercase | `markmyword` |
| CLI package ID | `SpecWorks.<Name>.Cli` | `SpecWorks.MarkMyWord.Cli` |
| Solution file | At `dotnet/` root | `parts/JsonDiff/dotnet/JsonDiff.sln` |

### Monorepo-Specific Naming

- **Part directories** MUST be directly under `parts/` (no nesting)
- **Solution files** remain inside each Part's `dotnet/` directory — do NOT
  create a top-level solution file spanning all Parts
- **Workflow files** use the pattern `<part>-<language>-test.yml` when
  Part-specific workflows are needed

---

## 6. Specification Linksets

Every Part MUST have a `specs.json` at its root (`parts/<PartName>/specs.json`).
The format is identical to the multi-repo pattern.

### Required Links

```json
{
  "linkset": [
    {
      "anchor": "https://github.com/<org>/<monorepo>/tree/main/parts/JsonDiff",
      "href": "https://www.rfc-editor.org/rfc/rfc6902.html",
      "rel": "https://specworks.org/rels/specification",
      "type": "text/html",
      "title": "RFC 6902 - JavaScript Object Notation (JSON) Patch"
    },
    {
      "anchor": "https://github.com/<org>/<monorepo>/tree/main/parts/JsonDiff",
      "href": "https://www.nuget.org/packages/SpecWorks.JsonDiff",
      "rel": "https://specworks.org/rels/library",
      "type": "application/vnd.nuget.package",
      "title": "JsonDiff library for .NET"
    },
    {
      "anchor": "https://github.com/<org>/<monorepo>/tree/main/parts/JsonDiff",
      "href": "https://github.com/<org>/<monorepo>/actions",
      "rel": "https://specworks.org/rels/tests",
      "type": "text/html",
      "title": "CI Test Results"
    },
    {
      "anchor": "https://github.com/<org>/<monorepo>/tree/main/parts/JsonDiff",
      "href": "https://github.com/<org>/<monorepo>/blob/main/parts/JsonDiff/README.md",
      "rel": "describedby",
      "type": "text/markdown",
      "title": "README"
    }
  ]
}
```

### Key Difference from Multi-Repo

The `anchor` attribute points to the Part's subdirectory within the monorepo
(`/tree/main/parts/<PartName>`) rather than a standalone repository URL. All
other linkset requirements from the factory specification (Section 3.2) apply
unchanged.

---

## 7. CI/CD Workflows

### 7.1 Path-Filtered Workflows

In a monorepo, CI/CD workflows use **path filters** to run only when a specific
Part changes. This replaces the per-repo workflow isolation of the multi-repo
pattern.

#### .NET Test Workflow

Create `.github/workflows/dotnet-test.yml`:

```yaml
name: .NET Test

on:
  push:
    branches: [ main ]
    paths:
      - 'parts/*/dotnet/**'
      - '.github/workflows/dotnet-test.yml'
  pull_request:
    branches: [ main ]
    paths:
      - 'parts/*/dotnet/**'
      - '.github/workflows/dotnet-test.yml'

jobs:
  # Discover which Parts have .NET implementations that changed
  discover:
    runs-on: ubuntu-latest
    outputs:
      parts: ${{ steps.changed.outputs.parts }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Detect changed Parts
        id: changed
        run: |
          if [ "${{ github.event_name }}" == "push" ]; then
            CHANGED=$(git diff --name-only HEAD~1 HEAD | grep '^parts/.*/dotnet/' | \
              cut -d'/' -f2 | sort -u | jq -R -s -c 'split("\n") | map(select(length > 0))')
          else
            CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD | \
              grep '^parts/.*/dotnet/' | cut -d'/' -f2 | sort -u | \
              jq -R -s -c 'split("\n") | map(select(length > 0))')
          fi
          echo "parts=$CHANGED" >> $GITHUB_OUTPUT

  test:
    needs: discover
    if: needs.discover.outputs.parts != '[]' && needs.discover.outputs.parts != ''
    runs-on: ubuntu-latest
    strategy:
      matrix:
        part: ${{ fromJson(needs.discover.outputs.parts) }}
      fail-fast: false

    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: |
            10.0.x
            8.0.x

      - name: Restore
        working-directory: parts/${{ matrix.part }}/dotnet
        run: dotnet restore

      - name: Build
        working-directory: parts/${{ matrix.part }}/dotnet
        run: dotnet build --no-restore --configuration Release

      - name: Test
        working-directory: parts/${{ matrix.part }}/dotnet
        run: dotnet test --no-build --configuration Release --verbosity normal
```

#### Python Test Workflow

Create `.github/workflows/python-test.yml`:

```yaml
name: Python Test

on:
  push:
    branches: [ main ]
    paths:
      - 'parts/*/python/**'
      - '.github/workflows/python-test.yml'
  pull_request:
    branches: [ main ]
    paths:
      - 'parts/*/python/**'
      - '.github/workflows/python-test.yml'

jobs:
  discover:
    runs-on: ubuntu-latest
    outputs:
      parts: ${{ steps.changed.outputs.parts }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Detect changed Parts
        id: changed
        run: |
          if [ "${{ github.event_name }}" == "push" ]; then
            CHANGED=$(git diff --name-only HEAD~1 HEAD | grep '^parts/.*/python/' | \
              cut -d'/' -f2 | sort -u | jq -R -s -c 'split("\n") | map(select(length > 0))')
          else
            CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD | \
              grep '^parts/.*/python/' | cut -d'/' -f2 | sort -u | \
              jq -R -s -c 'split("\n") | map(select(length > 0))')
          fi
          echo "parts=$CHANGED" >> $GITHUB_OUTPUT

  test:
    needs: discover
    if: needs.discover.outputs.parts != '[]' && needs.discover.outputs.parts != ''
    runs-on: ubuntu-latest
    strategy:
      matrix:
        part: ${{ fromJson(needs.discover.outputs.parts) }}
        python-version: ['3.10', '3.11', '3.12']
      fail-fast: false

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        working-directory: parts/${{ matrix.part }}/python
        run: |
          python -m pip install --upgrade pip
          pip install pytest
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Test
        working-directory: parts/${{ matrix.part }}/python
        run: pytest
```

#### Rust Test Workflow

Create `.github/workflows/rust-test.yml`:

```yaml
name: Rust Test

on:
  push:
    branches: [ main ]
    paths:
      - 'parts/*/rust/**'
      - '.github/workflows/rust-test.yml'
  pull_request:
    branches: [ main ]
    paths:
      - 'parts/*/rust/**'
      - '.github/workflows/rust-test.yml'

jobs:
  discover:
    runs-on: ubuntu-latest
    outputs:
      parts: ${{ steps.changed.outputs.parts }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Detect changed Parts
        id: changed
        run: |
          if [ "${{ github.event_name }}" == "push" ]; then
            CHANGED=$(git diff --name-only HEAD~1 HEAD | grep '^parts/.*/rust/' | \
              cut -d'/' -f2 | sort -u | jq -R -s -c 'split("\n") | map(select(length > 0))')
          else
            CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD | \
              grep '^parts/.*/rust/' | cut -d'/' -f2 | sort -u | \
              jq -R -s -c 'split("\n") | map(select(length > 0))')
          fi
          echo "parts=$CHANGED" >> $GITHUB_OUTPUT

  test:
    needs: discover
    if: needs.discover.outputs.parts != '[]' && needs.discover.outputs.parts != ''
    runs-on: ubuntu-latest
    strategy:
      matrix:
        part: ${{ fromJson(needs.discover.outputs.parts) }}
      fail-fast: false

    steps:
      - uses: actions/checkout@v4

      - name: Set up Rust
        uses: actions-rust-lang/setup-rust-toolchain@v1

      - name: Build
        working-directory: parts/${{ matrix.part }}/rust
        run: cargo build --verbose

      - name: Test
        working-directory: parts/${{ matrix.part }}/rust
        run: cargo test --verbose
```

### 7.2 Part-Specific Workflows

If a Part needs a custom workflow beyond the shared language workflows (e.g.,
build-and-publish for MarkMyWord), create a Part-specific workflow:

```yaml
# .github/workflows/markmyword-publish.yml
name: MarkMyWord Build & Publish

on:
  push:
    branches: [ main ]
    paths:
      - 'parts/MarkMyWord/**'
    tags:
      - 'markmyword-v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'
      - name: Pack
        working-directory: parts/MarkMyWord/dotnet
        run: dotnet pack --configuration Release
      - name: Publish to NuGet
        run: dotnet nuget push parts/MarkMyWord/dotnet/**/*.nupkg --source nuget.org --api-key ${{ secrets.NUGET_API_KEY }}
```

### 7.3 Workflow Naming Convention

| Workflow | File Name | Scope |
|----------|----------|-------|
| .NET tests (all Parts) | `dotnet-test.yml` | All Parts with `dotnet/` |
| Python tests (all Parts) | `python-test.yml` | All Parts with `python/` |
| Rust tests (all Parts) | `rust-test.yml` | All Parts with `rust/` |
| Part-specific publish | `<partname>-publish.yml` | Single Part |
| Registry update | `update-registry.yml` | Factory-wide |
| Documentation | `docs.yml` | Factory-wide |

---

## 8. Testing

### 8.1 Test Organization

Testing follows the same conventions as multi-repo. Each Part maintains its own
test projects:

```
parts/<PartName>/
├── dotnet/tests/<ProjectName>.Tests/    # .NET tests
├── python/tests/                         # Python tests
├── rust/ (cargo test)                    # Rust tests
└── testcases/                            # Shared test fixtures
```

### 8.2 Running Tests

#### Single Part

```bash
# .NET
cd parts/JsonDiff/dotnet && dotnet test

# Python
cd parts/vCard/python && pytest

# Rust
cd parts/vCard/rust && cargo test
```

#### All Parts (Factory-Wide)

Create `factory/tools/test-all.ps1`:

```powershell
#!/usr/bin/env pwsh
# Test all Parts in the monorepo

param(
    [string]$Language = "all"  # "dotnet", "python", "rust", or "all"
)

$repoRoot = Resolve-Path "$PSScriptRoot\..\.."
$partsDir = Join-Path $repoRoot "parts"
$failed = @()

foreach ($part in Get-ChildItem -Path $partsDir -Directory) {
    Write-Host "`n📦 Testing $($part.Name)..." -ForegroundColor Cyan

    if ($Language -in @("dotnet", "all")) {
        $dotnetDir = Join-Path $part.FullName "dotnet"
        if (Test-Path $dotnetDir) {
            Write-Host "  🔷 .NET" -ForegroundColor Blue
            Push-Location $dotnetDir
            dotnet test --configuration Release --verbosity minimal
            if ($LASTEXITCODE -ne 0) { $failed += "$($part.Name)/dotnet" }
            Pop-Location
        }
    }

    if ($Language -in @("python", "all")) {
        $pythonDir = Join-Path $part.FullName "python"
        if (Test-Path $pythonDir) {
            Write-Host "  🐍 Python" -ForegroundColor Yellow
            Push-Location $pythonDir
            pytest --tb=short
            if ($LASTEXITCODE -ne 0) { $failed += "$($part.Name)/python" }
            Pop-Location
        }
    }

    if ($Language -in @("rust", "all")) {
        $rustDir = Join-Path $part.FullName "rust"
        if (Test-Path $rustDir) {
            Write-Host "  🦀 Rust" -ForegroundColor DarkRed
            Push-Location $rustDir
            cargo test
            if ($LASTEXITCODE -ne 0) { $failed += "$($part.Name)/rust" }
            Pop-Location
        }
    }
}

Write-Host "`n" + ("=" * 50) -ForegroundColor Cyan
if ($failed.Count -eq 0) {
    Write-Host "✅ All tests passed!" -ForegroundColor Green
} else {
    Write-Host "❌ Failures:" -ForegroundColor Red
    $failed | ForEach-Object { Write-Host "   - $_" -ForegroundColor Red }
    exit 1
}
```

### 8.3 Test Requirements

All requirements from [CONVENTIONS.md](CONVENTIONS.md) apply unchanged:

- Comprehensive specification compliance tests
- Test count prominently displayed in README
- Real-world payloads from specifications
- Shared test fixtures in `testcases/` for multi-language Parts
- Standard testing framework per platform (xUnit/.NET, pytest/Python,
  cargo test/Rust)

---

## 9. Documentation

### 9.1 README Hierarchy

```
README.md                         # Factory overview + Parts inventory
├── factory/README.md             # Factory specification
├── factory/CONVENTIONS.md        # Coding conventions
├── factory/REGISTRY.md           # xRegistry usage guide
└── parts/
    ├── vCard/README.md           # Part overview (all languages)
    │   ├── dotnet/README.md      # .NET-specific docs
    │   ├── python/README.md      # Python-specific docs
    │   └── rust/README.md        # Rust-specific docs
    ├── JsonDiff/README.md        # Part overview
    │   └── dotnet/README.md      # .NET-specific docs
    └── ...
```

### 9.2 Part README Requirements

Identical to multi-repo. Every Part README MUST include:

1. Title and badges
2. Overview with specification link
3. Features mapped to specification sections
4. Quick start (installation, usage, example) per language
5. Project structure diagram
6. Specification compliance table
7. Test count
8. License reference

Use `factory/examples/template-readme.md` as the starting point.

### 9.3 Documentation Site

For a DocFX-based documentation site (equivalent to `spec-works.github.io`):

```
docs/
├── docfx.json           # DocFX configuration
├── index.md             # Landing page
├── toc.yml              # Table of contents
└── articles/
    └── getting-started.md
```

The `docfx.json` should reference API docs from each Part's `dotnet/` directory.

### 9.4 Deployment

Deploy documentation with GitHub Pages from the `docs/` directory or a
`gh-pages` branch:

```yaml
# .github/workflows/docs.yml
name: Deploy Documentation

on:
  push:
    branches: [ main ]
    paths:
      - 'docs/**'
      - 'parts/*/dotnet/src/**'

permissions:
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'
      - name: Install DocFX
        run: dotnet tool install -g docfx
      - name: Build docs
        run: docfx docs/docfx.json
      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

---

## 10. xRegistry Catalog

### 10.1 Registry Location

In the monorepo, the generated xRegistry catalog lives in the `registry/`
directory and is deployed to GitHub Pages.

```
registry/
├── index.json                    # xRegistry root document
└── parts/
    ├── index.json                # Parts collection
    ├── vcard/
    │   ├── index.json            # Part metadata
    │   └── versions/
    │       └── 1.0.0/
    │           └── part.json     # Linkset (specs.json copy)
    ├── jsondiff/
    └── ...
```

### 10.2 Generating the Registry

Adapt the `generate-registry.ps1` script for monorepo paths:

```powershell
#!/usr/bin/env pwsh
# generate-registry.ps1 (monorepo version)

param(
    [string]$RepoRoot = "",
    [string]$RegistryUrl = "https://<org>.github.io/<repo>/registry"
)

if ([string]::IsNullOrEmpty($RepoRoot)) {
    $RepoRoot = (Resolve-Path "$PSScriptRoot\..\..").Path
}

$partsDir = Join-Path $RepoRoot "parts"
$outputPath = Join-Path $RepoRoot "registry"

# Scan parts/ directory instead of sibling repositories
$partDirs = Get-ChildItem -Path $partsDir -Directory |
    Where-Object { Test-Path (Join-Path $_.FullName "specs.json") }

# ... remainder identical to multi-repo script, using $partDirs as source
```

### 10.3 Registry Update Workflow

```yaml
# .github/workflows/update-registry.yml
name: Update xRegistry

on:
  push:
    branches: [ main ]
    paths:
      - 'parts/*/specs.json'
      - 'parts/*/README.md'
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * 0'  # Weekly on Sundays at 2am UTC

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate registry
        shell: pwsh
        run: ./factory/tools/generate-registry.ps1

      - name: Commit registry changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add registry/
          git diff --staged --quiet || git commit -m "Update xRegistry catalog"
          git push
```

### 10.4 Querying the Registry

All query patterns from [REGISTRY.md](REGISTRY.md) apply. The base URL changes
to your monorepo's GitHub Pages URL:

```bash
# List all parts
curl https://<org>.github.io/<repo>/registry/parts/ | jq .

# Get specific part
curl https://<org>.github.io/<repo>/registry/parts/vcard/ | jq .
```

---

## 11. Publishing

### 11.1 Versioning Strategy

In a monorepo, Parts version independently. Use **Git tags with Part prefixes**
to identify releases:

```bash
# Tag format: <partname>-v<semver>
git tag jsondiff-v1.2.0
git tag vcard-v2.0.0
git tag markmyword-v1.0.1
```

### 11.2 NuGet Publishing

```yaml
# .github/workflows/publish-nuget.yml
name: Publish NuGet Package

on:
  push:
    tags:
      - '*-v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Parse tag
        id: tag
        run: |
          TAG="${GITHUB_REF#refs/tags/}"
          PART=$(echo "$TAG" | sed 's/-v[0-9].*//')
          VERSION=$(echo "$TAG" | sed 's/.*-v//')
          echo "part=$PART" >> $GITHUB_OUTPUT
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      - name: Pack
        working-directory: parts/${{ steps.tag.outputs.part }}/dotnet
        run: dotnet pack --configuration Release /p:Version=${{ steps.tag.outputs.version }}

      - name: Publish
        run: |
          dotnet nuget push parts/${{ steps.tag.outputs.part }}/dotnet/**/*.nupkg \
            --source nuget.org --api-key ${{ secrets.NUGET_API_KEY }}
```

### 11.3 Package Metadata

The `.csproj` `RepositoryUrl` points to the monorepo, and `PackageProjectUrl`
can point to the Part's subdirectory:

```xml
<RepositoryUrl>https://github.com/<org>/<monorepo></RepositoryUrl>
<PackageProjectUrl>https://github.com/<org>/<monorepo>/tree/main/parts/JsonDiff</PackageProjectUrl>
```

---

## 12. Architecture Decision Records

### 12.1 ADR Locations

| Scope | Location |
|-------|----------|
| Factory-wide decisions | `factory/adr/` |
| Part-specific decisions | `parts/<PartName>/adr/` |

### 12.2 Monorepo Organization ADR

Your first factory-level ADR should document the decision to use a monorepo.
Create `factory/adr/0001-monorepo-organization.md`:

```markdown
# ADR 0001: Monorepo Organization for Factory Parts

## Status
Accepted

## Context
Our organization restricts us to a single repository. We need to implement
the SpecWorks Factory pattern within this constraint while preserving
specification-centric discovery and Part independence.

## Decision
We will organize all Factory Parts under a `parts/` directory in a single
monorepo, with path-filtered CI/CD to maintain Part isolation.

## Consequences

### Positive
- Unified codebase and commit history
- Enforced consistent structure
- Single clone gets everything
- Cross-Part refactoring in a single PR
- Centralized governance

### Negative
- Cannot run multiple AI agents in parallel without branch conflicts
- All Parts share a single CI/CD pipeline (mitigated by path filters)
- Larger clone size as factory grows
- Convention drift is harder to prevent without structural isolation

### Mitigation
- Path-filtered workflows ensure only affected Parts are built/tested
- Part-prefixed Git tags enable independent versioning
- Factory-level validation scripts enforce conventions
```

### 12.3 ADR Format

Use the standard ADR template from `factory/examples/template-adr.md`. All
conventions from [CONVENTIONS.md](CONVENTIONS.md) apply unchanged.

---

## 13. .NET Conventions

All .NET conventions from [CONVENTIONS.md](CONVENTIONS.md) apply without
modification:

- **Target frameworks**: `net10.0;net8.0`
- **Nullable reference types**: Enabled
- **XML documentation**: Generated
- **Source Link**: Configured (use `Microsoft.SourceLink.GitHub`)
- **NuGet metadata**: Complete (PackageId, Authors, License, etc.)
- **CLI tools**: `PackAsTool=true` with `ToolCommandName`

### Monorepo-Specific .NET Notes

1. **No top-level solution file.** Each Part has its own `.sln` in
   `parts/<PartName>/dotnet/`. Do NOT create a factory-wide `.sln`.

2. **Source Link** works automatically — GitHub resolves paths within the
   monorepo correctly.

3. **Project references** between Parts are discouraged. Parts should be
   independent. If a Part depends on another, reference the published NuGet
   package, not a project reference.

4. **Directory.Build.props** can be used at the `parts/` level for shared
   settings, but use with caution — Part independence is more important than
   DRY configuration:

   ```xml
   <!-- parts/Directory.Build.props (optional) -->
   <Project>
     <PropertyGroup>
       <TargetFrameworks>net10.0;net8.0</TargetFrameworks>
       <Nullable>enable</Nullable>
       <ImplicitUsings>enable</ImplicitUsings>
       <LangVersion>latest</LangVersion>
       <GenerateDocumentationFile>true</GenerateDocumentationFile>
     </PropertyGroup>
   </Project>
   ```

---

## 14. Multi-Language Parts

For Parts implemented in multiple languages, all multi-repo conventions apply:

1. **Root README** covers all implementations
2. **Language subdirectories** (`dotnet/`, `python/`, `rust/`) each have
   dedicated README and build configuration
3. **Shared test fixtures** in `testcases/` directory
4. **ADRs** at Part root for cross-language decisions
5. Each language implementation passes the same shared test cases

### Cross-Language Testing in Monorepo

The path-filtered CI workflows handle this automatically. A change to
`parts/vCard/python/` triggers only the Python workflow for vCard, while a
change to `parts/vCard/testcases/` should trigger all language workflows for
that Part.

To handle shared test fixture changes, add testcase paths to all language
workflows:

```yaml
# In dotnet-test.yml, add to paths:
paths:
  - 'parts/*/dotnet/**'
  - 'parts/*/testcases/**'          # Also trigger on shared test changes
  - '.github/workflows/dotnet-test.yml'
```

---

## 15. AI Agent Workflow

### 15.1 Monorepo Constraints for AI Agents

The multi-repo pattern was designed for parallel AI-agent operations (see
[ADR 0001](adr/0001-multi-repo-organization.md)). In a monorepo, agents must
work **sequentially** or use **feature branches** with careful coordination:

```
Agent workflow (monorepo):
1. Clone the monorepo (or use existing clone)
2. Create a feature branch: git checkout -b feat/vcard-tel-property
3. Work ONLY within parts/vCard/ directory
4. Run Part-specific tests: cd parts/vCard/dotnet && dotnet test
5. Commit and push the feature branch
6. Create PR targeting main
7. Merge after review and CI passes
```

### 15.2 Avoiding Conflicts

- **One agent per Part at a time** — do not have two agents modifying the same
  Part concurrently
- **Scope branches to Parts** — use branch names like `feat/vcard-xxx` or
  `feat/jsondiff-xxx` to make ownership clear
- **Avoid touching factory/ or root files** in Part-specific PRs
- **Never modify multiple Parts in a single PR** unless making a deliberate
  cross-cutting change (e.g., updating a convention across all Parts)

### 15.3 Agent Instructions Template

When instructing an AI agent to work on a Part:

```
Work within the monorepo at: <repo-url>
- Your scope is limited to: parts/<PartName>/
- Do NOT modify any files outside parts/<PartName>/
- Follow conventions in: factory/CONVENTIONS.md
- Use templates from: factory/examples/
- Run tests with: cd parts/<PartName>/dotnet && dotnet test
- Create branch: feat/<partname>-<description>
```

---

## 16. Migration from Multi-Repo

### 16.1 Step-by-Step Migration

To migrate an existing multi-repo SpecWorks factory to a monorepo:

```powershell
# 1. Create the monorepo
mkdir inner-spec-works
cd inner-spec-works
git init

# 2. Create directory structure
mkdir -p factory, parts, registry, docs, .github/workflows

# 3. Copy factory documentation
Copy-Item -Recurse ..\spec-works\specification\* factory\

# 4. Copy each Part (without .git history)
foreach ($part in @('vCard', 'iCalendar', 'JsonDiff', 'linkset',
                     'RateLimiter', 'message', 'MarkMyWord', 'MarkMyDeck')) {
    $src = "..\spec-works\$part"
    if (Test-Path $src) {
        Copy-Item -Recurse $src "parts\$part"
        Remove-Item -Recurse -Force "parts\$part\.git" -ErrorAction SilentlyContinue
        Remove-Item -Recurse -Force "parts\$part\.github" -ErrorAction SilentlyContinue
    }
}

# 5. Create monorepo workflows (using path-filtered versions from this guide)
# 6. Update specs.json anchors to point to monorepo paths
# 7. Update .csproj RepositoryUrl and PackageProjectUrl
# 8. Generate xRegistry catalog
# 9. Create root README.md
# 10. Commit and push
```

### 16.2 What Changes

| Item | Multi-Repo | Monorepo |
|------|-----------|----------|
| `specs.json` anchor URLs | `github.com/spec-works/<Part>` | `github.com/<org>/<repo>/tree/main/parts/<Part>` |
| `.csproj` RepositoryUrl | `github.com/spec-works/<Part>` | `github.com/<org>/<repo>` |
| `.csproj` PackageProjectUrl | `github.com/spec-works/<Part>` | `github.com/<org>/<repo>/tree/main/parts/<Part>` |
| CI workflows | Per-repo in `.github/workflows/` | Centralized with path filters |
| Per-Part `.github/` | Deleted (merged into monorepo workflows) | N/A |
| Per-Part `.git/` | Deleted (single repo) | N/A |
| Per-Part `LICENSE` | Can be removed (use root LICENSE) | Optional (root LICENSE covers all) |

### 16.3 What Does NOT Change

Everything else carries over exactly:

- Part internal structure (`dotnet/`, `python/`, `rust/`, `testcases/`)
- `specs.json` linkset format (only `anchor` values change)
- All `.csproj` build settings (targets, nullable, Source Link, etc.)
- ADR format and numbering (per-Part ADRs stay with their Part)
- README structure and content requirements
- Testing conventions and requirements
- NuGet package naming and metadata (except URLs)
- Solution file structure within each Part
- Code style, naming conventions, design principles

---

## 17. Compliance Checklist

Use this checklist when creating a new Part in the monorepo:

**Repository Structure:**
- [ ] Part directory created at `parts/<PartName>/`
- [ ] `specs.json` at Part root with correct `anchor` URLs
- [ ] `README.md` at Part root
- [ ] Root `README.md` Parts inventory table updated

**CI/CD:**
- [ ] Part's language directory matches workflow path filters
- [ ] Tests pass when run locally
- [ ] Part-specific publish workflow created (if publishing)

**.NET Projects:**
- [ ] Multi-target: `net10.0;net8.0`
- [ ] Nullable reference types enabled
- [ ] XML documentation generated
- [ ] Source Link configured
- [ ] NuGet metadata complete with monorepo URLs
- [ ] CLI tools configured with `PackAsTool` (if applicable)
- [ ] Solution file at `parts/<PartName>/dotnet/<Name>.sln`

**Documentation:**
- [ ] README includes specification link
- [ ] Features mapped to specification sections
- [ ] Test count displayed prominently
- [ ] Installation instructions present
- [ ] Examples demonstrate core functionality

**Testing:**
- [ ] Test project created
- [ ] Specification compliance tests
- [ ] Real-world payloads tested
- [ ] Shared test fixtures in `testcases/` (multi-language Parts)

**Specification Linkset:**
- [ ] `specs.json` present at Part root
- [ ] Primary specification linked (`rel: specification`)
- [ ] Library packages linked (`rel: library`)
- [ ] Tests linked (`rel: tests`)
- [ ] Documentation linked (`rel: describedby`)

**ADRs:**
- [ ] Major design decisions documented in `parts/<PartName>/adr/`
- [ ] Factory-wide decisions in `factory/adr/`

---

## Appendix A: Complete Example

A minimal monorepo with two Parts:

```
inner-spec-works/
├── README.md
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       ├── dotnet-test.yml
│       └── update-registry.yml
├── factory/
│   ├── README.md
│   ├── CONVENTIONS.md
│   ├── adr/
│   │   └── 0001-monorepo-organization.md
│   ├── examples/
│   │   ├── template-readme.md
│   │   ├── template-csproj.xml
│   │   └── template-specs.json
│   └── tools/
│       └── generate-registry.ps1
├── parts/
│   ├── JsonDiff/
│   │   ├── README.md
│   │   ├── specs.json
│   │   ├── adr/
│   │   │   └── 0001-rfc-6902-only.md
│   │   └── dotnet/
│   │       ├── JsonDiff.sln
│   │       ├── src/SpecWorks.JsonDiff/
│   │       │   └── SpecWorks.JsonDiff.csproj
│   │       └── tests/SpecWorks.JsonDiff.Tests/
│   │           └── SpecWorks.JsonDiff.Tests.csproj
│   └── vCard/
│       ├── README.md
│       ├── specs.json
│       ├── testcases/
│       ├── dotnet/
│       │   ├── VCard.sln
│       │   ├── src/VCard/
│       │   └── tests/VCard.Tests/
│       ├── python/
│       │   ├── src/vcard/
│       │   └── tests/
│       └── rust/
│           ├── Cargo.toml
│           └── src/
├── registry/
│   ├── index.json
│   └── parts/
│       ├── index.json
│       ├── jsondiff/
│       └── vcard/
└── docs/
    ├── docfx.json
    └── index.md
```

---

## Appendix B: Monorepo vs Multi-Repo Trade-offs

This table summarizes the trade-offs. The multi-repo pattern is the default
recommendation in the SpecWorks Factory specification; this guide addresses
the monorepo alternative.

| Dimension | Multi-Repo | Monorepo |
|-----------|-----------|----------|
| **AI Agent Parallelism** | ✅ Excellent — agents clone separate repos, no conflicts | ⚠️ Limited — agents share one repo, need branch coordination |
| **CI/CD Isolation** | ✅ Per-Part triggers automatically | ⚠️ Requires path-filter configuration |
| **Clone Size** | ✅ Small per Part (~10MB) | ⚠️ Entire factory history (grows over time) |
| **Cross-Part Refactoring** | ⚠️ Requires multiple PRs | ✅ Single PR |
| **Convention Enforcement** | ⚠️ Relies on templates and review | ✅ Structural enforcement possible |
| **Repository Count** | ⚠️ N repos (one per Part + infrastructure) | ✅ Single repo |
| **Part Independence** | ✅ Natural — separate repos | ⚠️ Must be disciplined about isolation |
| **Versioning** | ✅ Per-repo tags and releases | ⚠️ Part-prefixed tags |
| **Failure Isolation** | ✅ One Part's CI failure doesn't affect others | ⚠️ Shared CI pipeline |
| **GitHub Features** | ✅ Per-Part issues, PRs, wiki, releases | ⚠️ Shared issue tracker, labels needed for Part filtering |
| **Discovery** | ✅ Each repo is one problem space | ⚠️ Requires xRegistry or README for navigation |
| **Permissions** | ✅ Per-repo access control | ⚠️ All-or-nothing repo access |

### When Multi-Repo is Clearly Better

- Multiple AI agents working in parallel
- Large number of Parts (10+)
- Different Parts have different maintainers or teams
- Parts have very different release cadences

### When Monorepo is Clearly Better

- Organization restricts to a single repository
- Small factory (< 10 Parts)
- Single team maintaining all Parts
- Frequent cross-Part changes

---

**Document Metadata:**
- Version: 1.0-draft
- Date: 2026-02-08
- Based on: SpecWorks Factory Specification v1.0-draft
- Parent documents: [README.md](README.md), [CONVENTIONS.md](CONVENTIONS.md)
- Status: Draft
