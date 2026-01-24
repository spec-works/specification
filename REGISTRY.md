# xRegistry Usage Guide

This document explains how to use, query, and maintain the SpecWorks xRegistry catalog.

## Overview

The SpecWorks xRegistry is a static catalog of specification-based software components hosted at:

**Registry URL**: https://spec-works.github.io/registry/

The registry follows the [xRegistry 0.5 specification](https://github.com/xregistry/spec) and enables specification-centric component discovery.

## Querying the Registry

### Using curl

**Get registry root document:**
```bash
curl https://spec-works.github.io/registry/ | jq .
```

**List all parts:**
```bash
curl https://spec-works.github.io/registry/parts/ | jq .
```

**Get specific part metadata:**
```bash
curl https://spec-works.github.io/registry/parts/vcard/ | jq .
```

**Get part linkset (specs.json):**
```bash
curl https://spec-works.github.io/registry/parts/vcard/versions/1.0.0/part.json | jq .
```

### Using PowerShell

**Query registry:**
```powershell
# Get all parts
$parts = Invoke-RestMethod -Uri "https://spec-works.github.io/registry/parts/"
$parts.parts | Format-Table id, name, description

# Get specific part
$vcard = Invoke-RestMethod -Uri "https://spec-works.github.io/registry/parts/vcard/"
$vcard | ConvertTo-Json -Depth 10

# Get part linkset
$linkset = Invoke-RestMethod -Uri "https://spec-works.github.io/registry/parts/vcard/versions/1.0.0/part.json"
$linkset.linkset | Where-Object { $_.rel -eq "describedby" }
```

### Using Python

```python
import requests

# Get all parts
response = requests.get("https://spec-works.github.io/registry/parts/")
parts = response.json()
for part_id, part in parts["parts"].items():
    print(f"{part_id}: {part['description']}")

# Get part linkset
response = requests.get("https://spec-works.github.io/registry/parts/vcard/versions/1.0.0/part.json")
linkset = response.json()
for link in linkset["linkset"]:
    if link["rel"] == "describedby":
        print(f"Specification: {link['href']}")
```

## Registry Structure

```
https://spec-works.github.io/registry/
├── index.json                    # xRegistry root document
├── parts/
│   ├── index.json               # Parts collection
│   ├── vcard/
│   │   ├── index.json          # Part metadata
│   │   └── versions/
│   │       └── 1.0.0/
│   │           └── part.json   # Linkset (specs.json)
│   ├── jsondiff/
│   ├── icalendar/
│   ├── markmyword/
│   ├── ratelimiter/
│   └── linkset/
└── schemas/
    └── xregistry-v0.5.json     # xRegistry schema (future)
```

## Discovery Patterns

### Find Component by Problem Space

**Scenario**: "I need to parse calendar data"

1. **Query parts collection:**
   ```bash
   curl https://spec-works.github.io/registry/parts/ | jq '.parts | to_entries[] | select(.value.description | contains("calendar"))'
   ```

2. **Result**: `icalendar` part implementing RFC 5545

3. **Get implementation links:**
   ```bash
   curl https://spec-works.github.io/registry/parts/icalendar/versions/1.0.0/part.json | jq '.linkset[] | select(.rel | contains("library"))'
   ```

### Find Component by Specification

**Scenario**: "I need an RFC 6350 implementation"

1. **Search all part linksets:**
   ```bash
   for part in vcard jsondiff icalendar markmyword ratelimiter linkset; do
     curl -s "https://spec-works.github.io/registry/parts/$part/versions/1.0.0/part.json" | \
     jq --arg part "$part" 'select(.linkset[].href | contains("6350")) | $part'
   done
   ```

2. **Result**: `vcard` part

### List All Available Implementations

**Get all .NET implementations:**
```bash
curl https://spec-works.github.io/registry/parts/ | \
jq '.parts | to_entries[] | 
    {id: .key, name: .value.name, description: .value.description}'
```

Then check each part's linkset for library links.

## Adding a New Part

### 1. Create Part Repository

Follow the SpecWorks Factory pattern:
- Create repository: `spec-works/{PartName}`
- Add `specs.json` with linkset
- Include README.md with description
- Implement the specification

### 2. Add specs.json

Create `specs.json` at repository root:

```json
{
  "linkset": [
    {
      "href": "https://www.rfc-editor.org/rfc/rfcXXXX.html",
      "rel": "describedby",
      "title": "Specification Title"
    },
    {
      "href": "https://www.nuget.org/packages/SpecWorks.PartName",
      "rel": "library",
      "title": "PartName library for .NET"
    }
  ]
}
```

Required link relations:
- `describedby`: Link to specification
- `library`: Links to published packages (NuGet, PyPI, npm, etc.)

Optional but recommended:
- `related`: IANA registries, errata
- `documentation`: API docs, user guides
- Test results, examples, etc.

### 3. Trigger Registry Update

**Option A: Automatic (weekly scheduled)**
- Registry updates automatically every Sunday at 2am UTC

**Option B: Manual trigger**
```bash
# From specification repository
gh workflow run update-registry.yml
```

**Option C: Repository dispatch** (future)
- Push to part repository triggers registry update

### 4. Verify Registration

After update completes:
```bash
curl https://spec-works.github.io/registry/parts/{partname}/ | jq .
```

## Updating Part Metadata

### Update specs.json

1. Edit `specs.json` in part repository
2. Add/update links in linkset array
3. Commit and push changes
4. Trigger registry update (manual or wait for scheduled)

### Update Part Description

The part description is extracted from the first paragraph of the part's README.md. To update:

1. Edit README.md in part repository
2. Ensure first paragraph clearly describes the component
3. Trigger registry update

### Update Part Version

Currently all parts use version `1.0.0`. To add a new version:

1. Publish new package version (NuGet, PyPI, etc.)
2. Update specs.json with new package links
3. **Future**: Script will support multiple versions

## Automation

### GitHub Action Workflow

The registry is automatically updated by `.github/workflows/update-registry.yml` in the specification repository.

**Triggers:**
- **Manual**: `gh workflow run update-registry.yml`
- **Scheduled**: Weekly on Sundays at 2am UTC
- **Repository dispatch**: When part specs.json changes (future)

**Process:**
1. Clone all part repositories
2. Run `specification/tools/generate-registry.ps1`
3. Commit changes to `spec-works.github.io/registry/`
4. GitHub Pages automatically deploys

### Manual Update

```powershell
# Clone all part repositories to a common directory
cd C:\src\github\spec-works

# Run generation script
cd specification\tools
.\generate-registry.ps1

# Commit and push
cd ..\..\spec-works.github.io
git add registry/
git commit -m "Update xRegistry catalog"
git push
```

### Validation

The generation script includes validation:
- ✅ Checks for `specs.json` in each part
- ✅ Extracts specification links
- ✅ Generates valid xRegistry 0.5 structure
- ✅ Creates proper linkset documents

## Registry Format Details

### Root Document

`https://spec-works.github.io/registry/index.json`

```json
{
  "specversion": "0.5",
  "id": "specworks-factory",
  "name": "SpecWorks Factory Inventory",
  "description": "Catalog of specification-based software components",
  "self": "https://spec-works.github.io/registry/",
  "partsurl": "https://spec-works.github.io/registry/parts",
  "partscount": 6,
  "parts": { ... }
}
```

### Part Document

`https://spec-works.github.io/registry/parts/{id}/index.json`

```json
{
  "id": "vcard",
  "name": "vCard Component",
  "description": "...",
  "defaultversionid": "1.0.0",
  "defaultversionurl": "https://spec-works.github.io/registry/parts/vcard/versions/1.0.0",
  "versions": {
    "1.0.0": {
      "parturl": "https://spec-works.github.io/registry/parts/vcard/versions/1.0.0/part.json"
    }
  }
}
```

### Part Linkset

`https://spec-works.github.io/registry/parts/{id}/versions/{version}/part.json`

This is the `specs.json` from the part repository. Contains:
- Specification links (`rel: describedby`)
- Library links (`rel: library` or custom)
- Test links, documentation, examples

## IDE Integration (Future)

### Copilot CLI

```bash
# Find component by specification
copilot find --spec "RFC 6350"

# List all parts
copilot parts list

# Get part details
copilot parts show vcard
```

### VS Code Extension

Future: VS Code extension to browse registry and insert package references.

### Language Package Managers

Future: Registry could be integrated with package search tools.

## Troubleshooting

### Registry Not Updating

1. **Check GitHub Actions**: https://github.com/spec-works/specification/actions
2. **Verify workflow ran**: Look for "Update xRegistry" workflow
3. **Check for errors**: Review workflow logs
4. **Manual trigger**: `gh workflow run update-registry.yml`

### Part Not Appearing

1. **Verify specs.json exists**: Must be at repository root
2. **Check linkset format**: Must be valid JSON with `linkset` array
3. **Verify repository name**: Must not be in excluded list
4. **Trigger update**: Manual update to force regeneration

### Linkset Issues

1. **Invalid JSON**: Validate with `jq` or JSON validator
2. **Missing required links**: Must have `describedby` link
3. **Check link relations**: Follow RFC 8288 for link relations

## References

- [xRegistry Specification](https://github.com/xregistry/spec)
- [RFC 9264: Linkset Media Types](https://www.rfc-editor.org/rfc/rfc9264.html)
- [RFC 8288: Web Linking](https://www.rfc-editor.org/rfc/rfc8288.html)
- [SpecWorks Factory Pattern](README.md)

## Support

For issues or questions:
- **Registry issues**: https://github.com/spec-works/specification/issues
- **Part-specific issues**: https://github.com/spec-works/{PartName}/issues
- **Discussions**: https://github.com/orgs/spec-works/discussions

---

**Last Updated**: 2026-01-24  
**Registry Version**: xRegistry 0.5  
**Automation**: `.github/workflows/update-registry.yml`
