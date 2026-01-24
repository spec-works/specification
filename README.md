# SpecWorks Factory Specification

**Version**: 1.0-draft
**Date**: 2026-01-01
**Status**: Draft

## Abstract

This document specifies the SpecWorks Factory pattern: a system for creating, cataloging, and distributing software components that implement publicly available specifications. A SpecWorks Factory uses xRegistry as an inventory system and linksets (RFC 9264) to describe the relationships between components, specifications, and artifacts.

## 1. Introduction

### 1.1 Purpose

The SpecWorks Factory specification defines a reproducible pattern for:

1. Generating software components from publicly available specifications
2. Cataloging those components in a machine-readable registry
3. Publishing components to standard package managers
4. Enabling discovery and reuse of specification-compliant components

### 1.2 Goals

- **Reproducibility**: Anyone can create their own factory following this pattern
- **Discoverability**: Parts can be found programmatically through specification-centric search
- **Traceability**: Clear links between Parts and their source specifications
- **Consumability**: Parts are published to standard package ecosystems
- **Specification-Centric**: Discovery is organized around problems and specifications, not implementation languages

### 1.3 Terminology

- **Part**: A software component that implements one or more specifications to solve a specific problem space (e.g., "vCard" solves contact information representation by implementing RFC 6350)
- **Implementation**: A language-specific realization of a Part (e.g., vCard's .NET implementation, Python implementation)
- **Specification Version**: The version of the specification being implemented (e.g., RFC 6350 v4.0) - this is the primary version identifier for a Part
- **Implementation Version**: The artifact version of a specific language implementation (e.g., NuGet package v1.2.0) - these may differ across languages

### 1.4 Non-Goals

This specification does NOT prescribe:
- How Parts are generated (manual, AI-assisted, or otherwise)
- Programming languages or frameworks
- Repository organization (mono-repo vs. multi-repo)
- Project structures or build systems (beyond general patterns)
- Testing methodologies
- Code style or quality metrics

## 2. Architecture

A SpecWorks Factory consists of three primary elements:

```
┌──────────────────────────────────────────────────────┐
│              SpecWorks Factory                        │
│                                                       │
│  ┌────────────────────────────────────────────┐     │
│  │        xRegistry Instance                   │     │
│  │     (Factory Inventory/Catalog)             │     │
│  │                                              │     │
│  │  Resource Type: "part"                      │     │
│  │  Format: application/linkset+json           │     │
│  │                                              │     │
│  │  Each Part entry represents one problem     │     │
│  │  space with specification version and       │     │
│  │  links to available implementations         │     │
│  └────────────────────────────────────────────┘     │
│                    │                                  │
│                    ↓                                  │
│  ┌────────────────────────────────────────────┐     │
│  │          Factory Parts                      │     │
│  │                                              │     │
│  │  Each Part:                                 │     │
│  │  - Addresses one problem space              │     │
│  │  - Implements one or more specifications    │     │
│  │  - Has linkset describing:                  │     │
│  │    • Specifications (with versions)         │     │
│  │    • Libraries (multiple languages)         │     │
│  │    • Tests                                  │     │
│  │    • Documentation                          │     │
│  │  - May have multiple implementations        │     │
│  │    (.NET, Python, Rust, etc.)               │     │
│  └────────────────────────────────────────────┘     │
│                                                       │
└──────────────────────────────────────────────────────┘

Discovery Model (Specification-Centric):
Developer → IDE Agent → xRegistry Query → Find Part by Problem
                                               ↓
                                    Part: "vCard (RFC 6350 v4.0)"
                                    Available: .NET, Python, Rust
                                               ↓
                                    Select/Request Implementation
```

### 2.1 xRegistry Instance

The factory MUST maintain an xRegistry instance that serves as the catalog of all factory Parts. Parts are indexed by problem space and specification, not by implementation language.

### 2.2 Factory Parts

Each Part produced by the factory addresses a specific problem space by implementing one or more specifications. A Part may have multiple language implementations (e.g., .NET, Python, Rust), but is cataloged as a single entry in xRegistry.

**Key Characteristics:**
- **Problem-Centric**: Named after the problem space or specification (e.g., "vCard", not "vCard-dotnet")
- **Specification-Versioned**: Primary version is the specification version (e.g., RFC 6350 v4.0)
- **Multi-Implementation**: May contain implementations in multiple languages, each with independent artifact versions
- **Independent Lifecycle**: Evolves based on its specification's lifecycle, not the factory's lifecycle

### 2.3 Part Descriptors

Each Part MUST be described by a linkset document (application/linkset+json, RFC 9264) that uses the link relations defined in Section 4. A single Part linkset includes:
- Link(s) to specification(s) (REQUIRED)
- Link(s) to library artifacts in one or more languages (REQUIRED - at least one)
- Link(s) to tests (REQUIRED)
- Link(s) to documentation (REQUIRED)

### 2.4 Factory Organization

While this specification does not mandate a specific repository organization, the following patterns are recommended:

**Multi-Repository Pattern** (Recommended):
- Each Part has its own repository (e.g., `github.com/factory-org/vCard`)
- Shared factory infrastructure in separate repositories:
  - Factory pattern documentation and templates
  - Reusable CI/CD workflows
- Enables parallel AI-agent operations on different Parts
- See [ADR 0001](adr/0001-multi-repo-organization.md) for detailed rationale

**Repository Contents** (per Part):
```
vCard/                  # Part repository
├── README.md          # Problem space and specification overview
├── specs.json         # Part linkset descriptor
├── testcases/         # Shared test fixtures across implementations
├── dotnet/            # .NET implementation
├── python/            # Python implementation (if exists)
└── rust/              # Rust implementation (if exists)
```

## 2.5 Using the xRegistry

The SpecWorks Factory maintains a public xRegistry at **https://spec-works.github.io/registry/**

**Quick Start:**
```bash
# List all parts
curl https://spec-works.github.io/registry/parts/ | jq .

# Get specific part
curl https://spec-works.github.io/registry/parts/vcard/ | jq .

# Get part linkset (specs.json)
curl https://spec-works.github.io/registry/parts/vcard/versions/1.0.0/part.json | jq .
```

For complete usage documentation, see [REGISTRY.md](REGISTRY.md).

## 3. Requirements

### 3.1 Factory Requirements

A SpecWorks Factory MUST:

1. **Maintain an xRegistry instance** that catalogs all parts
   - **SpecWorks Registry**: https://spec-works.github.io/registry/
   - See [REGISTRY.md](REGISTRY.md) for usage guide
2. **Define a resource type** for parts with media type `application/linkset+json`
3. **Use the link relations** defined in Section 4
4. **Be publicly accessible** (the xRegistry instance must be queryable)

### 3.2 Part Requirements

Each Part in a SpecWorks Factory MUST satisfy these minimum requirements:

#### 3.2.1 Specification Traceability

The Part's linkset MUST include at least one link with relation type:
- `https://specworks.org/rels/specification`

This link MUST point to the authoritative specification document(s) that the Part implements.

#### 3.2.2 Consumability

The Part's linkset MUST include at least one link with relation type:
- `https://specworks.org/rels/library`

This link MUST point to a published package in a standard package manager (NuGet, PyPI, npm, Maven Central, crates.io, Go modules, etc.).

#### 3.2.3 Quality Evidence

The Part's linkset MUST include at least one link with relation type:
- `https://specworks.org/rels/tests`

This link SHOULD point to evidence of passing tests (CI/CD results, test reports, etc.).

#### 3.2.4 Documentation

The Part's linkset MUST include at least one link with relation type:
- `describedby` (RFC 8288)

This link MUST point to human-readable documentation (typically a README).

### 3.3 Optional Part Attributes

Parts MAY include additional links with these relation types:
- `https://specworks.org/rels/api-docs` - API reference documentation
- `https://specworks.org/rels/example` - Usage examples
- `https://specworks.org/rels/cli` - Command-line tools
- Standard link relations from RFC 8288 (license, alternate, etc.)

## 4. Link Relations

### 4.1 SpecWorks Extension Link Relations

This specification defines the following extension link relation types:

#### 4.1.1 `specification`

- **Relation Name**: `specification`
- **Full URI**: `https://specworks.org/rels/specification`
- **Description**: Points to the authoritative specification document that the Part implements
- **Expected Target**: Specification documents (RFC, W3C, ISO, etc.)
- **Required**: Yes

#### 4.1.2 `library`

- **Relation Name**: `library`
- **Full URI**: `https://specworks.org/rels/library`
- **Description**: Points to a published software library/package
- **Expected Target**: Package manager URLs (NuGet, PyPI, npm, etc.)
- **Required**: Yes

#### 4.1.3 `tests`

- **Relation Name**: `tests`
- **Full URI**: `https://specworks.org/rels/tests`
- **Description**: Points to test suite or test results
- **Expected Target**: CI/CD dashboards, test reports, test source code
- **Required**: Yes

#### 4.1.4 `api-docs`

- **Relation Name**: `api-docs`
- **Full URI**: `https://specworks.org/rels/api-docs`
- **Description**: Points to API reference documentation
- **Expected Target**: Generated API docs (Sphinx, JSDoc, Doxygen, etc.)
- **Required**: No

#### 4.1.5 `example`

- **Relation Name**: `example`
- **Full URI**: `https://specworks.org/rels/example`
- **Description**: Points to usage examples or sample code
- **Expected Target**: Example code, tutorials, guides
- **Required**: No

#### 4.1.6 `cli`

- **Relation Name**: `cli`
- **Full URI**: `https://specworks.org/rels/cli`
- **Description**: Points to command-line tool packages
- **Expected Target**: CLI tool packages or installation instructions
- **Required**: No

### 4.2 Standard Link Relations

Parts SHOULD use standard link relations from RFC 8288 where appropriate:
- `describedby` - Human-readable documentation (REQUIRED)
- `license` - License information
- `alternate` - Alternative representations
- `version-history` - Version history

## 5. Linkset Structure

### 5.1 Basic Structure

A Part descriptor linkset uses the `anchor` attribute to identify the Part (typically its repository URL):

```json
{
  "linkset": [
    {
      "anchor": "https://github.com/example-factory/component-name",
      "href": "https://www.rfc-editor.org/rfc/rfc9999.html",
      "rel": "https://specworks.org/rels/specification",
      "type": "text/html",
      "title": "The Specification Name"
    }
  ]
}
```

### 5.2 Complete Example

A complete Part descriptor for a component called "linkset":

```json
{
  "linkset": [
    {
      "anchor": "https://github.com/spec-works/linkset",
      "href": "https://www.rfc-editor.org/rfc/rfc9264.html",
      "rel": "https://specworks.org/rels/specification",
      "type": "text/html",
      "title": "Linkset: Media Types and a Link Relation Type for Link Sets"
    },
    {
      "anchor": "https://github.com/spec-works/linkset",
      "href": "https://www.rfc-editor.org/rfc/rfc8288.html",
      "rel": "https://specworks.org/rels/specification",
      "type": "text/html",
      "title": "Web Linking"
    },
    {
      "anchor": "https://github.com/spec-works/linkset",
      "href": "https://www.nuget.org/packages/SpecWorks.Linkset",
      "rel": "https://specworks.org/rels/library",
      "type": "application/vnd.nuget.package",
      "title": "Linkset library for .NET"
    },
    {
      "anchor": "https://github.com/spec-works/linkset",
      "href": "https://github.com/spec-works/linkset/actions",
      "rel": "https://specworks.org/rels/tests",
      "type": "text/html",
      "title": "CI Test Results"
    },
    {
      "anchor": "https://github.com/spec-works/linkset",
      "href": "https://github.com/spec-works/linkset/blob/main/README.md",
      "rel": "describedby",
      "type": "text/markdown",
      "title": "README"
    }
  ]
}
```

## 6. xRegistry Integration

### 6.1 Resource Type Definition

The xRegistry instance MUST define a resource type for Parts:

```json
{
  "specversion": "0.5",
  "id": "specworks-factory",
  "epoch": 1,
  "self": "https://registry.example.com/",
  "model": {
    "schemas": [
      "xRegistry-json"
    ],
    "groups": {
      "*": {
        "plural": "parts",
        "singular": "part",
        "resources": {
          "*": {
            "plural": "versions",
            "singular": "version",
            "versions": 1
          }
        }
      }
    }
  }
}
```

### 6.2 Part Resources

Each Part is registered as a resource with its linkset document as the resource content:

```json
{
  "id": "linkset",
  "name": "Linkset Component",
  "epoch": 1,
  "self": "https://registry.example.com/parts/linkset",
  "contenttype": "application/linkset+json",
  "parturl": "https://registry.example.com/parts/linkset/part"
}
```

The `parturl` returns the linkset document describing the Part.

## 7. Validation

### 7.1 Part Validation

A Part is considered valid if:

1. It has a linkset document in `application/linkset+json` format
2. The linkset includes all REQUIRED link relations (Section 3.2)
3. All links are valid URIs
4. The specification links point to accessible specifications
5. The library links point to published packages
6. The test links provide evidence of quality

### 7.2 Factory Validation

A Factory is considered valid if:

1. It has a publicly accessible xRegistry instance
2. The xRegistry defines a resource type for Parts
3. All registered Parts are valid (Section 7.1)
4. The registry is queryable via xRegistry API

## 8. Discovery

### 8.1 Finding Factories

Factories SHOULD publish their xRegistry endpoint at a well-known location:
- `https://<factory-domain>/.well-known/specworks-registry`

### 8.2 Querying Parts

Clients can query for Parts using xRegistry API:

```http
GET https://registry.example.com/parts
```

Filter by specification:

```http
GET https://registry.example.com/parts?filter=specification:rfc9264
```

## 9. Security Considerations

### 9.1 Secrets

Factory repositories MUST NOT contain:
- API keys
- Credentials
- Tokens
- Private keys

Use `.gitignore` to prevent accidental commits of sensitive files.

### 9.2 Package Security

Published packages SHOULD:
- Be signed
- Include checksums
- Undergo security scanning
- Follow security best practices for their ecosystem

## 10. Extensibility

Factories MAY:
- Define additional link relation types
- Add custom metadata to xRegistry resources
- Support additional resource types beyond "parts"
- Implement additional validation rules

## 11. References

- [RFC 8288] Web Linking
- [RFC 9264] Linkset: Media Types and a Link Relation Type for Link Sets
- [xRegistry] xRegistry Specification
- [IANA Link Relations] IANA Link Relation Types Registry


## Appendix A: Creating Your Own Factory

To create your own factory:

1. **Bootstrap Your Factory**
   - Create a GitHub organization (e.g., `mycompany-factory`)
   - Fork/clone the SpecWorks factory template repository
   - Customize conventions for your domain

2. **Set Up Infrastructure**
   - Create `.github` repository for shared workflow templates
   - Deploy an xRegistry instance to catalog your Parts
   - Configure the xRegistry resource type for Parts (Section 6)

3. **Create Your First Part**
   - Create a repository for the Part (e.g., `mycompany-factory/oauth2`)
   - Use templates from factory repository
   - Implement specification(s) in one or more languages
   - Create linkset descriptor (specs.json)
   - Ensure Part satisfies requirements in Section 3.2

4. **Register in xRegistry**
   - Add Part entry to your xRegistry instance
   - Publish linkset descriptor
   - Verify all required links are present

5. **Publish Implementations**
   - Publish libraries to package managers (NuGet, PyPI, etc.)
   - Configure CI/CD using shared workflow templates
   - Set up automated testing and validation

### Factory Scope Examples

Your factory can focus on any domain:
- **Industry-specific standards**: Financial protocols (ISO 20022), Healthcare (HL7 FHIR)
- **Internal specifications**: Company-specific APIs and data formats
- **Protocol implementations**: Network protocols (RFCs), authentication standards
- **Data format libraries**: File formats (PDF, DOCX), serialization formats

### Multi-Repository Pattern Benefits

The multi-repository pattern is particularly beneficial for:
- **AI-agent-driven development**: Multiple agents working in parallel without conflicts
- **Specification-centric discovery**: Each repository represents one problem space
- **Independent evolution**: Parts evolve based on their specification's lifecycle
- **Focused contributions**: Contributors clone only the Part they need

See [ADR 0001](adr/0001-multi-repo-organization.md) for detailed architectural rationale.
