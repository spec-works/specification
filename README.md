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
- **Discoverability**: Components can be found programmatically
- **Traceability**: Clear links between components and their source specifications
- **Consumability**: Components are published to standard package ecosystems

### 1.3 Non-Goals

This specification does NOT prescribe:
- How components are generated (manual, AI-assisted, or otherwise)
- Programming languages or frameworks
- Project structures or build systems
- Testing methodologies
- Code style or quality metrics

## 2. Architecture

A SpecWorks Factory consists of three primary elements:

```
┌─────────────────────────────────────────┐
│         SpecWorks Factory                │
│                                          │
│  ┌────────────────────────────────┐    │
│  │      xRegistry Instance         │    │
│  │   (Factory Inventory/Catalog)   │    │
│  │                                  │    │
│  │  Resource Type: "part"          │    │
│  │  Format: application/linkset+json│    │
│  └────────────────────────────────┘    │
│              │                           │
│              ↓                           │
│  ┌────────────────────────────────┐    │
│  │     Factory Parts (Components)  │    │
│  │                                  │    │
│  │  Each part is described by a    │    │
│  │  linkset document that relates: │    │
│  │  - Specifications                │    │
│  │  - Published packages            │    │
│  │  - Tests                         │    │
│  │  - Documentation                 │    │
│  └────────────────────────────────┘    │
│                                          │
└─────────────────────────────────────────┘
```

### 2.1 xRegistry Instance

The factory MUST maintain an xRegistry instance that serves as the catalog of all factory parts.

### 2.2 Factory Parts

Each component produced by the factory is called a "Part". Parts are cataloged as resources in the xRegistry.

### 2.3 Part Descriptors

Each Part MUST be described by a linkset document (application/linkset+json, RFC 9264) that uses the link relations defined in Section 4.

## 3. Requirements

### 3.1 Factory Requirements

A SpecWorks Factory MUST:

1. **Maintain an xRegistry instance** that catalogs all parts
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

## Appendix A: Example Factory

See the SpecWorks reference implementation at:
- Registry: https://registry.specworks.org/
- Source: https://github.com/spec-works/.github

## Appendix B: Creating Your Own Factory

To create your own factory:

1. Choose a domain name (e.g., `mycompany-specs.org`)
2. Deploy an xRegistry instance
3. Configure the resource type for Parts
4. Create components that implement specifications
5. Register Parts in your xRegistry
6. Ensure each Part satisfies the requirements in Section 3.2

Your factory can focus on any domain:
- Industry-specific standards
- Internal company specifications
- Protocol implementations
- Data format libraries
