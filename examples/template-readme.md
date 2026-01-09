# [Project Name]

![Version](https://img.shields.io/badge/version-[VERSION]-blue)
![Languages](https://img.shields.io/badge/languages-[LANGUAGES]-green)
![Status](https://img.shields.io/badge/status-[STATUS]-yellow)
![Tests](https://img.shields.io/badge/tests-[COUNT]%2B-brightgreen)

[Brief 1-2 sentence description of the component and its purpose]

This library implements [RFC XXXX / W3C / ISO Standard Name](link-to-specification), providing [key value proposition].

## Features

- [Feature 1 - map to spec section]
- [Feature 2 - map to spec section]
- [Feature 3 - map to spec section]
- [X] [Completed feature]
- [ ] [Planned feature]
- Comprehensive test suite with **[XX]+ tests**
- [Multi-language support / Platform support]
- [Other notable features]

## Specifications

This implementation is based on:
- **[RFC XXXX](link)** - [Title] ([Status: Standard/Draft/Proposed])
- [Related RFC/Standard if applicable]

See [specs.json](specs.json) for complete specification linkset.

## Quick Start

### .NET

#### Installation

```bash
dotnet add package [PackageName]
```

Or via Package Manager:
```powershell
Install-Package [PackageName]
```

#### Usage

```csharp
using [Namespace];

// Basic usage example
var example = new [ClassName]();
var result = example.[Method]([parameters]);

// Show result
Console.WriteLine(result);
```

#### Complete Example

```csharp
// More comprehensive example showing common use case
// Include imports, setup, and expected output
```

### Python

*(Include this section only for multi-language projects)*

#### Installation

```bash
pip install [package-name]
```

#### Usage

```python
from [package] import [Class]

# Basic usage example
example = [Class]()
result = example.[method]([parameters])

print(result)
```

### Rust

*(Include this section only for multi-language projects)*

#### Installation

Add to `Cargo.toml`:
```toml
[dependencies]
[package-name] = "[version]"
```

#### Usage

```rust
use [package_name]::[Module];

fn main() {
    // Basic usage example
    let example = [Module]::new();
    let result = example.[method]([parameters]);

    println!("{:?}", result);
}
```

## Project Structure

```
[project-name]/
├── README.md              # This file
├── specs.json            # Specification linkset (RFC 9264)
├── LICENSE               # MIT License
├── adr/                  # Architecture Decision Records
│   ├── 0001-[topic].md
│   └── 0002-[topic].md
├── dotnet/               # .NET implementation
│   ├── README.md
│   ├── [Project].sln
│   ├── src/
│   │   └── [Project]/
│   │       └── [Project].csproj
│   └── tests/
│       └── [Project].Tests/
│           └── [Project].Tests.csproj
├── python/               # Python implementation (if applicable)
│   ├── README.md
│   ├── src/[package]/
│   └── tests/
├── rust/                 # Rust implementation (if applicable)
│   ├── README.md
│   ├── Cargo.toml
│   └── src/
└── testcases/           # Shared test fixtures (if applicable)
    └── README.md
```

## Specification Compliance

| Feature | Specification Section | Status |
|---------|----------------------|--------|
| [Feature 1] | [RFC XXXX §N.N] | Implemented |
| [Feature 2] | [RFC XXXX §N.N] | Implemented |
| [Feature 3] | [RFC XXXX §N.N] | In Progress |
| [Optional Feature] | [RFC XXXX §N.N] | Not Implemented |

### Unsupported Features

- **[Feature Name]** ([Spec Section]) - [Reason for non-support, link to ADR if applicable]

### Extensions and Deviations

This implementation strictly follows the specification with no proprietary extensions. Any deviations are documented in the [ADR directory](adr/).

## API Documentation

*(For single-feature libraries, include brief API overview here)*

### Core Classes/Functions

#### `[ClassName]`

[Brief description]

**Constructor:**
```csharp
public [ClassName]([parameters])
```

**Methods:**

- `[Method1]([params])` - [Description]
- `[Method2]([params])` - [Description]

**Properties:**

- `[Property1]` - [Description]
- `[Property2]` - [Description]

### Examples

See language-specific READMEs for detailed examples:
- [.NET Examples](dotnet/README.md)
- [Python Examples](python/README.md)
- [Rust Examples](rust/README.md)

## Testing

This library includes **[XX]+ tests** covering:

- Specification compliance tests
- Edge case handling
- Error conditions
- Real-world payload examples from the specification
- Interoperability tests

### Running Tests

#### .NET
```bash
cd dotnet
dotnet test
```

#### Python
```bash
cd python
pytest
```

#### Rust
```bash
cd rust
cargo test
```

## Architecture Decision Records

Major design decisions are documented in the [adr/](adr/) directory:

- [ADR 0001: [Topic]](adr/0001-[topic].md)
- [ADR 0002: [Topic]](adr/0002-[topic].md)

## Contributing

This is a SpecWorks Factory component. Contributions should:

1. Maintain strict specification compliance
2. Include tests for new functionality
3. Update documentation
4. Follow existing code style
5. Document design decisions in ADRs

## Roadmap

- [ ] [Planned feature 1]
- [ ] [Planned feature 2]
- [ ] [Language support addition]

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Implementation based on [RFC XXXX / Standard Name]
- Part of the [SpecWorks Factory](https://github.com/spec-works/specification)

## Links

- [Specification](link-to-spec)
- [IANA Registry](link-if-applicable)
- [NuGet Package](link-when-published)
- [PyPI Package](link-when-published)
- [crates.io Package](link-when-published)
- [Documentation](link-when-published)

---

**Status:** [Production / Beta / Alpha / In Development]
**Version:** [X.Y.Z]
**Last Updated:** [YYYY-MM-DD]
