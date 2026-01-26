# Software Component Registry Service - Version 1.0-wip

## Abstract

This specification defines a Software Component Registry extension to the xRegistry
document format and API [specification](https://github.com/xregistry/spec/blob/main/core/spec.md). 
A Software Component Registry allows for the storage, management and discovery of software 
component definitions and their associated artifacts produced by Software Component Factories 
like SpecWorks.

## Table of Contents

- [Software Component Registry Service - Version 1.0-wip](#software-component-registry-service---version-10-wip)
  - [Abstract](#abstract)
  - [Table of Contents](#table-of-contents)
  - [1. Overview](#1-overview)
    - [1.1. Software Components](#11-software-components)
    - [1.2. Component Artifacts](#12-component-artifacts)
    - [1.3. Component Dependencies](#13-component-dependencies)
    - [1.4. Versioning](#14-versioning)
    - [1.5. Document Store](#15-document-store)
  - [2. Notations and Terminology](#2-notations-and-terminology)
    - [2.1. Notational Conventions](#21-notational-conventions)
    - [2.2. Terminology](#22-terminology)
      - [2.2.1. Software Component](#221-software-component)
      - [2.2.2. Component Factory](#222-component-factory)
      - [2.2.3. Component Artifact](#223-component-artifact)
  - [3. Component Registry Model](#3-component-registry-model)
  - [4. Component Registry](#4-component-registry)
    - [4.1. Component Groups](#41-component-groups)
      - [4.1.1. `owner`](#411-owner)
      - [4.1.2. `namespace`](#412-namespace)
    - [4.2. Component Resources](#42-component-resources)
      - [4.2.1. `componenttype`](#421-componenttype)
      - [4.2.2. `language`](#422-language)
      - [4.2.3. `framework`](#423-framework)
      - [4.2.4. `license`](#424-license)
      - [4.2.5. `repository`](#425-repository)
      - [4.2.6. `homepage`](#426-homepage)
      - [4.2.7. `keywords`](#427-keywords)
      - [4.2.8. `dependencies`](#428-dependencies)
      - [4.2.9. `artifacts`](#429-artifacts)
      - [4.2.10. `buildinfo`](#4210-buildinfo)
      - [4.2.11. `status`](#4211-status)
      - [4.2.12. `maturity`](#4212-maturity)
    - [4.3. Component Types](#43-component-types)
      - [4.3.1. Library](#431-library)
      - [4.3.2. Service](#432-service)
      - [4.3.3. Application](#433-application)
      - [4.3.4. Specification](#434-specification)
      - [4.3.5. Schema](#435-schema)
      - [4.3.6. API](#436-api)
      - [4.3.7. Tool](#437-tool)
      - [4.3.8. Framework](#438-framework)

## 1. Overview

A Software Component Registry provides a repository for managing metadata about 
software components and the resources they produce. This includes libraries, services, 
applications, specifications, schemas, APIs, tools, and frameworks. The registry enables 
discovery, dependency management, versioning, and traceability of software components 
across an organization or ecosystem.

### 1.1. Software Components

Software components are reusable units of software that can be independently developed, 
versioned, and deployed. In the context of a Software Component Factory like SpecWorks, 
components represent the formal definitions and outputs of the factory process.

A component in this registry includes:
- **Metadata**: Descriptive information about the component (name, description, type, language, etc.)
- **Artifacts**: The actual deliverables produced (packages, containers, documents, schemas, etc.)
- **Dependencies**: References to other components this component depends on
- **Versioning**: Multiple versions of the same logical component with compatibility tracking
- **Provenance**: Build and factory information about how the component was created

### 1.2. Component Artifacts

Each component version MAY have one or more artifacts associated with it. Artifacts represent 
the concrete deliverables of the component, such as:

- **Packages**: npm packages, Python wheels, NuGet packages, Maven artifacts, etc.
- **Container Images**: Docker images, OCI images
- **Documents**: Specifications, API documentation, user guides
- **Schemas**: JSON Schema, XML Schema, OpenAPI definitions, Protobuf definitions
- **Source Code**: Git repositories, source archives
- **Binaries**: Executables, libraries, compiled artifacts

Artifacts are represented as a collection with each artifact having:
- `name`: The name/identifier of the artifact
- `type`: The type of artifact (package, container, document, schema, binary, source)
- `url`: The URL where the artifact can be retrieved
- `contenttype`: The MIME type of the artifact
- `size`: The size in bytes (optional)
- `checksums`: A map of checksum algorithm to value (e.g., `{"sha256": "abc123..."}`)
- `labels`: Additional metadata specific to the artifact type

### 1.3. Component Dependencies

Components often depend on other components. The `dependencies` attribute captures these 
relationships, enabling:

- Dependency resolution and graph analysis
- Impact analysis (what depends on this component?)
- Security vulnerability tracking
- License compliance checking

Each dependency includes:
- `componentxid`: An xRegistry XID reference to the dependent component
- `versionconstraint`: A version constraint (e.g., "^1.2.0", ">=2.0.0", "[1.0.0,2.0.0)")
- `scope`: The scope of the dependency (runtime, development, test, build, optional)
- `transitive`: Whether transitive dependencies should be included

### 1.4. Versioning

Software components evolve over time. The Component Registry leverages the xRegistry 
versioning model to manage multiple versions of components. Each version represents a 
specific state of the component with its own metadata, artifacts, and dependencies.

The [`compatibility`](https://github.com/xregistry/spec/blob/main/core/spec.md#compatibility-attribute) 
attribute determines what kinds of changes are allowed between versions:
- `semver`: Semantic versioning rules (major.minor.patch)
- `forward`: Newer versions can read older data
- `backward`: Older versions can read newer data
- `full`: Bidirectional compatibility
- `none`: No compatibility guarantees

### 1.5. Document Store

The Component Registry MAY be configured as a document store by setting the 
[`hasdocument`](https://github.com/xregistry/spec/blob/main/core/spec.md#hasdocument-attribute) 
attribute to `true` for component Resources where the component itself is primarily a document 
(e.g., specifications, schemas, API definitions).

For components where `hasdocument` is `true`, a GET request to the component's 
[`self`](https://github.com/xregistry/spec/blob/main/core/spec.md#self-attribute) URL 
returns the document content directly with its content-type, while metadata is returned 
in HTTP headers. The component's artifacts may still list additional related documents.

For components where `hasdocument` is `false`, the component version contains only metadata 
and references to artifacts, which are accessed via the `artifacts` collection.

## 2. Notations and Terminology

### 2.1. Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

For clarity, OPTIONAL attributes (specification-defined and extensions) are
OPTIONAL for clients to use, but the servers' responsibility will vary.
Server-unknown extension attributes MUST be silently stored in the backing
datastore. Specification-defined, and server-known extension, attributes MUST
generate an error if the corresponding feature is not supported or enabled.
However, as with all attributes, if accepting the attribute would result in a
bad state (such as exceeding a size limit, or results in a security issue),
then the server MAY choose to reject the request.

In the pseudo JSON format snippets `?` means the preceding attribute is
OPTIONAL, `*` means the preceding attribute MAY appear zero or more times,
and `+` means the preceding attribute MUST appear at least once. The presence
of the `#` character means the remaining portion of the line is a comment.
Whitespace characters in the JSON snippets are used for readability and are
not normative.

### 2.2. Terminology

This specification defines the following terms:

#### 2.2.1. Software Component

A **software component** (or component Resource) in this specification is a 
logical grouping of **component Versions**. A **component Version** represents 
a specific released state of the component with associated metadata, artifacts, 
and dependencies.

Per the definition of the [`compatibility`](https://github.com/xregistry/spec/blob/main/core/spec.md#compatibility-attribute) 
attribute, all Versions of a single **component** MUST adhere to the rules defined 
by the `compatibility` attribute. Any breaking change SHOULD result in careful 
consideration of version numbering or potentially a new **component** Resource.

#### 2.2.2. Component Factory

A **component factory** is a system or process that produces software components 
following defined patterns, templates, and best practices. SpecWorks is an example 
of a component factory that generates specification-based software components.

#### 2.2.3. Component Artifact

A **component artifact** is a concrete deliverable produced by a component version. 
This can include packages, containers, documents, schemas, binaries, or source code 
archives. Each artifact is addressable via a URL and has associated metadata.

## 3. Component Registry Model

For easy reference, the JSON serialization of a Component Registry adheres to
this form:

```yaml
{
  "specversion": "<STRING>",                       # xRegistry core attributes
  "registryid": "<STRING>",
  "self": "<URL>",
  "xid": "<XID>",
  "epoch": <UINTEGER>,
  "name": "<STRING>", ?
  "description": "<STRING>", ?
  "documentation": "<URL>", ?
  "labels": {
    "<STRING>": "<STRING>" *
  }, ?
  "createdat": "<TIMESTAMP>",
  "modifiedat": "<TIMESTAMP>",

  "model": { ... }, ?

  "componentgroupsurl": "<URL>",                   # ComponentGroups collection
  "componentgroupscount": <UINTEGER>,
  "componentgroups": {
    "KEY": {                                       # componentgroupid
      "componentgroupid": "<STRING>",              # xRegistry core attributes
      "self": "<URL>",
      "xid": "<XID>",
      "epoch": <UINTEGER>,
      "name": "<STRING>", ?
      "description": "<STRING>", ?
      "documentation": "<URL>", ?
      "labels": { "<STRING>": "<STRING>" * }, ?
      "createdat": "<TIMESTAMP>",
      "modifiedat": "<TIMESTAMP>",
      "deprecated": { ... }, ?

      "owner": "<STRING>", ?                       # ComponentGroup extensions
      "namespace": "<STRING>", ?

      "componentsurl": "<URL>",                    # Components collection
      "componentscount": <UINTEGER>,
      "components": {
        "KEY": {                                   # componentid
          "componentid": "<STRING>",               # xRegistry core attributes
          "versionid": "<STRING>",
          "self": "<URL>",
          "xid": "<XID>",

          #  Start of default Version's attributes
          "epoch": <UINTEGER>,
          "name": "<STRING>", ?                    # Version level attrs
          "description": "<STRING>", ?
          "documentation": "<URL>", ?
          "labels": { "<STRING>": "<STRING>" * }, ?
          "createdat": "<TIMESTAMP>",
          "modifiedat": "<TIMESTAMP>",
          "ancestor": "<STRING>",

          "componenttype": "<STRING>", ?           # Component extension attrs
          "language": "<STRING>", ?
          "framework": "<STRING>", ?
          "license": "<STRING>", ?
          "repository": "<URL>", ?
          "homepage": "<URL>", ?
          "keywords": [ "<STRING>" * ], ?
          
          "dependencies": [                        # Component dependencies
            {
              "componentxid": "<XID>",
              "versionconstraint": "<STRING>", ?
              "scope": "<STRING>", ?               # runtime|dev|test|build|optional
              "transitive": <BOOLEAN> ?
            } *
          ], ?

          "artifacts": [                           # Component artifacts
            {
              "name": "<STRING>",
              "type": "<STRING>",                  # package|container|document|schema|binary|source
              "url": "<URL>",
              "contenttype": "<STRING>", ?
              "size": <UINTEGER>, ?
              "checksums": {
                "<STRING>": "<STRING>" *           # algorithm: value (e.g. sha256: abc...)
              }, ?
              "labels": {
                "<STRING>": "<STRING>" *
              } ?
            } *
          ], ?

          "buildinfo": {                           # Build/factory information
            "builder": "<STRING>", ?               # e.g., "SpecWorks", "Jenkins"
            "buildurl": "<URL>", ?
            "buildtime": "<TIMESTAMP>", ?
            "commit": "<STRING>", ?                # Git commit SHA
            "branch": "<STRING>", ?                # Git branch
            "factory": "<STRING>", ?               # Factory/template used
            "labels": {
              "<STRING>": "<STRING>" *
            } ?
          }, ?

          "status": "<STRING>", ?                  # active|deprecated|archived
          "maturity": "<STRING>", ?                # alpha|beta|stable|mature

          #  End of default Version's attributes

          "metaurl": "<URL>",                      # Resource level attrs
          "meta": {
            ... core spec metadata attributes ...
          }, ?

          "versionsurl": "<URL>",
          "versionscount": <UINTEGER>,
          "versions": { ... } ?
        } *
      } ?
    } *
  } ?
}
```

## 4. Component Registry

The Component Registry is a metadata store for organizing software components and 
component Versions. It tracks the resources produced by software component factories 
and enables discovery, dependency management, and lifecycle tracking.

Implementations of this specification MAY include additional extension attributes.

Since the Component Registry is an application of the xRegistry specification, 
all attributes for Groups, Resources, and Resource Version objects are inherited 
from the core specification.

### 4.1. Component Groups

The Group (`<GROUP>`) name for the Component Registry is `componentgroup` (singular).
The plural, used as the collection name, is `componentgroups`. 

A Component Group is a container for components that are related to each other in
some application-defined way. Common grouping patterns include:

- **Organization**: Components owned by a specific team or organization
- **Product**: Components that make up a specific product or system
- **Namespace**: Components in a programming language namespace (e.g., `com.example`)
- **Repository**: Components from a specific source code repository

Component Groups have the following extension attributes:

#### 4.1.1. `owner`

- Type: String
- Description: The owner of the component group (e.g., team name, organization name, user ID)
- Constraints: None
- Examples: `"platform-team"`, `"acme-corp"`, `"john.doe@example.com"`

#### 4.1.2. `namespace`

- Type: String
- Description: A namespace qualifier for the components in this group (e.g., package namespace, organization domain)
- Constraints: None
- Examples: `"com.example.platform"`, `"@acme-corp"`, `"github.com/myorg"`

### 4.2. Component Resources

The Resource (`<RESOURCE>`) inside of Component Groups is named `component`. The
plural, used as the collection name, is `components`. Any single `component` is a
container for one or more `versions`, which hold the concrete component metadata,
artifacts, and dependencies.

All Versions of a single Component Resource SHOULD adhere to the semantic rules of
the component's [`compatibility`](https://github.com/xregistry/spec/blob/main/core/spec.md#compatibility-attribute) 
attribute.

Component Resources have the following extension attributes (all at the Version level):

#### 4.2.1. `componenttype`

- Type: String (enum)
- Description: The type of software component
- Constraints: MUST be one of: `library`, `service`, `application`, `specification`, `schema`, `api`, `tool`, `framework`
- Examples: `"library"`, `"service"`, `"specification"`

#### 4.2.2. `language`

- Type: String
- Description: The primary programming language of the component
- Constraints: SHOULD use lowercase language names
- Examples: `"javascript"`, `"python"`, `"java"`, `"go"`, `"typescript"`, `"csharp"`

#### 4.2.3. `framework`

- Type: String
- Description: The primary framework or runtime the component is built with or for
- Constraints: None
- Examples: `"react"`, `"express"`, `"django"`, `"spring-boot"`, `"dotnet"`

#### 4.2.4. `license`

- Type: String
- Description: The software license under which the component is distributed
- Constraints: SHOULD use SPDX license identifiers when possible
- Examples: `"MIT"`, `"Apache-2.0"`, `"GPL-3.0"`, `"BSD-3-Clause"`

#### 4.2.5. `repository`

- Type: URL
- Description: The URL of the source code repository
- Constraints: MUST be a valid URL
- Examples: `"https://github.com/myorg/mycomponent"`, `"git@github.com:myorg/mycomponent.git"`

#### 4.2.6. `homepage`

- Type: URL
- Description: The URL of the component's homepage or documentation site
- Constraints: MUST be a valid URL
- Examples: `"https://mycomponent.example.com"`, `"https://docs.example.com/components/mycomponent"`

#### 4.2.7. `keywords`

- Type: Array of Strings
- Description: Keywords or tags for categorizing and discovering the component
- Constraints: None
- Examples: `["rest-api", "authentication", "oauth2"]`, `["data-processing", "etl"]`

#### 4.2.8. `dependencies`

- Type: Array of Objects
- Description: Other components this component depends on
- Constraints: Each dependency object MUST have a `componentxid` field
- Structure:
  ```yaml
  {
    "componentxid": "<XID>",           # REQUIRED: XID reference to component
    "versionconstraint": "<STRING>", ? # Version constraint (e.g., "^1.2.0")
    "scope": "<STRING>", ?             # runtime|dev|test|build|optional
    "transitive": <BOOLEAN> ?          # Include transitive deps (default: true)
  }
  ```
- Examples:
  ```json
  [
    {
      "componentxid": "/componentgroups/platform/components/auth-lib",
      "versionconstraint": "^2.1.0",
      "scope": "runtime"
    },
    {
      "componentxid": "/componentgroups/platform/components/test-utils",
      "versionconstraint": ">=1.0.0",
      "scope": "test",
      "transitive": false
    }
  ]
  ```

#### 4.2.9. `artifacts`

- Type: Array of Objects
- Description: The concrete deliverables produced by this component version
- Constraints: Each artifact object MUST have `name`, `type`, and `url` fields
- Structure:
  ```yaml
  {
    "name": "<STRING>",              # REQUIRED: Artifact name/identifier
    "type": "<STRING>",              # REQUIRED: package|container|document|schema|binary|source
    "url": "<URL>",                  # REQUIRED: Where to retrieve the artifact
    "contenttype": "<STRING>", ?     # MIME type
    "size": <UINTEGER>, ?            # Size in bytes
    "checksums": {                   # Checksums for verification
      "<STRING>": "<STRING>" *       # algorithm: value
    }, ?
    "labels": {                      # Artifact-specific metadata
      "<STRING>": "<STRING>" *
    } ?
  }
  ```
- Examples:
  ```json
  [
    {
      "name": "mycomponent-1.2.0.tgz",
      "type": "package",
      "url": "https://registry.npmjs.org/mycomponent/-/mycomponent-1.2.0.tgz",
      "contenttype": "application/gzip",
      "size": 1048576,
      "checksums": {
        "sha256": "abc123...",
        "sha512": "def456..."
      },
      "labels": {
        "registry": "npm",
        "scope": "@myorg"
      }
    },
    {
      "name": "mycomponent:1.2.0",
      "type": "container",
      "url": "docker.io/myorg/mycomponent:1.2.0",
      "contenttype": "application/vnd.oci.image.manifest.v1+json",
      "checksums": {
        "sha256": "789abc..."
      },
      "labels": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
    {
      "name": "openapi.yaml",
      "type": "schema",
      "url": "https://example.com/schemas/mycomponent/1.2.0/openapi.yaml",
      "contenttype": "application/vnd.oai.openapi+yaml"
    }
  ]
  ```

#### 4.2.10. `buildinfo`

- Type: Object
- Description: Information about how this component version was built/produced
- Constraints: All fields are optional
- Structure:
  ```yaml
  {
    "builder": "<STRING>", ?         # Builder system (e.g., "SpecWorks", "Jenkins")
    "buildurl": "<URL>", ?           # URL to build job/run
    "buildtime": "<TIMESTAMP>", ?    # When the build occurred
    "commit": "<STRING>", ?          # Git commit SHA
    "branch": "<STRING>", ?          # Git branch name
    "factory": "<STRING>", ?         # Factory/template identifier
    "labels": {                      # Additional build metadata
      "<STRING>": "<STRING>" *
    } ?
  }
  ```
- Examples:
  ```json
  {
    "builder": "SpecWorks",
    "buildurl": "https://ci.example.com/builds/12345",
    "buildtime": "2024-01-15T10:30:00Z",
    "commit": "a1b2c3d4e5f6",
    "branch": "main",
    "factory": "rest-api-template-v2",
    "labels": {
      "ci-system": "github-actions",
      "runner": "ubuntu-latest"
    }
  }
  ```

#### 4.2.11. `status`

- Type: String (enum)
- Description: The lifecycle status of the component version
- Constraints: SHOULD be one of: `active`, `deprecated`, `archived`
- Examples: `"active"`, `"deprecated"`
- Notes:
  - `active`: Component is actively maintained and recommended for use
  - `deprecated`: Component is no longer recommended; use `deprecated` attribute to point to replacement
  - `archived`: Component is no longer maintained and should not be used for new projects

#### 4.2.12. `maturity`

- Type: String (enum)
- Description: The maturity level of the component version
- Constraints: SHOULD be one of: `alpha`, `beta`, `stable`, `mature`
- Examples: `"stable"`, `"beta"`
- Notes:
  - `alpha`: Early development, unstable API, may have significant changes
  - `beta`: Feature-complete but may have bugs, API may still change
  - `stable`: Production-ready, stable API, suitable for general use
  - `mature`: Well-established, battle-tested, minimal changes expected

### 4.3. Component Types

This section describes the intended use of each component type value.

#### 4.3.1. Library

A library is a reusable collection of code that provides specific functionality and is 
intended to be used by other software components. Libraries are typically distributed 
as packages (npm, PyPI, Maven, NuGet, etc.) and linked/imported into applications.

Examples: utility libraries, UI component libraries, SDK libraries

#### 4.3.2. Service

A service is a standalone software component that runs independently and provides 
functionality via a network interface (REST API, gRPC, GraphQL, etc.). Services are 
typically containerized and deployed to runtime environments.

Examples: microservices, REST APIs, gRPC services, background workers

#### 4.3.3. Application

An application is a complete software system intended for end-users. Applications may 
be composed of multiple services and libraries and have user interfaces.

Examples: web applications, mobile apps, desktop applications, CLI tools

#### 4.3.4. Specification

A specification is a formal definition or standard document. This type is used when 
the component itself is primarily a specification document rather than executable code.

Examples: API specifications, data format specifications, protocol definitions

#### 4.3.5. Schema

A schema is a structured definition of data formats or structures. Schemas are used 
for validation, code generation, and documentation.

Examples: JSON Schema, XML Schema, Avro schema, Protobuf definitions, database schemas

#### 4.3.6. API

An API component represents an API definition or contract, typically using standards 
like OpenAPI, AsyncAPI, or GraphQL SDL. This is distinct from the service that 
implements the API.

Examples: OpenAPI definitions, AsyncAPI definitions, GraphQL schemas, gRPC proto files

#### 4.3.7. Tool

A tool is a software component designed to aid in development, deployment, or 
operational tasks. Tools are typically command-line utilities or development utilities.

Examples: build tools, code generators, linters, formatters, deployment tools

#### 4.3.8. Framework

A framework is a comprehensive platform that provides structure and conventions for 
building applications. Frameworks typically include libraries, tools, and architectural 
patterns.

Examples: web frameworks, testing frameworks, application frameworks

---

## Examples

### Example 1: REST API Service Component

```json
{
  "componentid": "user-service",
  "versionid": "2.1.0",
  "self": "https://registry.example.com/componentgroups/platform-services/components/user-service/versions/2.1.0",
  "name": "User Management Service",
  "description": "Microservice for user account management and authentication",
  "componenttype": "service",
  "language": "typescript",
  "framework": "express",
  "license": "MIT",
  "repository": "https://github.com/example/user-service",
  "homepage": "https://docs.example.com/services/user-service",
  "keywords": ["authentication", "user-management", "rest-api"],
  "dependencies": [
    {
      "componentxid": "/componentgroups/platform/components/auth-lib",
      "versionconstraint": "^3.0.0",
      "scope": "runtime"
    },
    {
      "componentxid": "/componentgroups/platform/components/db-client",
      "versionconstraint": ">=2.5.0 <3.0.0",
      "scope": "runtime"
    }
  ],
  "artifacts": [
    {
      "name": "@example/user-service:2.1.0",
      "type": "container",
      "url": "docker.io/example/user-service:2.1.0",
      "contenttype": "application/vnd.oci.image.manifest.v1+json",
      "checksums": {
        "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
      },
      "labels": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
    {
      "name": "openapi.yaml",
      "type": "api",
      "url": "https://registry.example.com/schemas/user-service/2.1.0/openapi.yaml",
      "contenttype": "application/vnd.oai.openapi+yaml;version=3.0"
    }
  ],
  "buildinfo": {
    "builder": "SpecWorks",
    "buildurl": "https://github.com/example/user-service/actions/runs/123456",
    "buildtime": "2024-01-15T14:23:45Z",
    "commit": "a1b2c3d4e5f6g7h8i9j0",
    "branch": "main",
    "factory": "rest-api-template-v3"
  },
  "status": "active",
  "maturity": "stable",
  "createdat": "2024-01-15T14:23:45Z",
  "modifiedat": "2024-01-15T14:23:45Z"
}
```

### Example 2: JavaScript Library Component

```json
{
  "componentid": "data-validator",
  "versionid": "1.5.2",
  "self": "https://registry.example.com/componentgroups/utils/components/data-validator/versions/1.5.2",
  "name": "Data Validator",
  "description": "Extensible data validation library with schema support",
  "componenttype": "library",
  "language": "javascript",
  "framework": "none",
  "license": "Apache-2.0",
  "repository": "https://github.com/example/data-validator",
  "homepage": "https://data-validator.example.com",
  "keywords": ["validation", "schema", "data-quality"],
  "dependencies": [],
  "artifacts": [
    {
      "name": "@example/data-validator-1.5.2.tgz",
      "type": "package",
      "url": "https://registry.npmjs.org/@example/data-validator/-/data-validator-1.5.2.tgz",
      "contenttype": "application/gzip",
      "size": 524288,
      "checksums": {
        "sha512": "abcd1234..."
      },
      "labels": {
        "registry": "npm",
        "scope": "@example"
      }
    }
  ],
  "buildinfo": {
    "builder": "npm",
    "buildtime": "2024-01-10T09:15:30Z",
    "commit": "z9y8x7w6v5u4",
    "branch": "release/1.5"
  },
  "status": "active",
  "maturity": "mature",
  "createdat": "2024-01-10T09:15:30Z",
  "modifiedat": "2024-01-10T09:15:30Z"
}
```

### Example 3: API Specification Component

```json
{
  "componentid": "payment-api",
  "versionid": "3.0.0",
  "self": "https://registry.example.com/componentgroups/apis/components/payment-api/versions/3.0.0",
  "name": "Payment Processing API",
  "description": "OpenAPI specification for payment processing endpoints",
  "componenttype": "api",
  "license": "CC-BY-4.0",
  "repository": "https://github.com/example/payment-api-spec",
  "documentation": "https://developers.example.com/payment-api",
  "keywords": ["payments", "openapi", "rest"],
  "dependencies": [
    {
      "componentxid": "/componentgroups/schemas/components/payment-schema",
      "versionconstraint": "^2.0.0",
      "scope": "runtime"
    }
  ],
  "artifacts": [
    {
      "name": "openapi.yaml",
      "type": "schema",
      "url": "https://registry.example.com/artifacts/payment-api/3.0.0/openapi.yaml",
      "contenttype": "application/vnd.oai.openapi+yaml;version=3.1",
      "size": 65536,
      "checksums": {
        "sha256": "1234abcd..."
      }
    },
    {
      "name": "openapi.json",
      "type": "schema",
      "url": "https://registry.example.com/artifacts/payment-api/3.0.0/openapi.json",
      "contenttype": "application/vnd.oai.openapi+json;version=3.1"
    }
  ],
  "buildinfo": {
    "builder": "SpecWorks",
    "factory": "openapi-spec-template-v2",
    "buildtime": "2024-02-01T11:00:00Z",
    "commit": "f1e2d3c4b5a6"
  },
  "status": "active",
  "maturity": "stable",
  "createdat": "2024-02-01T11:00:00Z",
  "modifiedat": "2024-02-01T11:00:00Z"
}
```

---

## Acknowledgements

This specification extends the [xRegistry Core Specification](https://github.com/xregistry/spec) 
and follows the patterns established by the [Schema Registry](https://github.com/xregistry/spec/blob/main/schema/spec.md) 
and [Message Definitions Registry](https://github.com/xregistry/spec/blob/main/message/spec.md) 
specifications.

## Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", 
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in 
[RFC 2119](https://tools.ietf.org/html/rfc2119).
