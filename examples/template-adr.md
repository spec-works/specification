# ADR NNNN: [Short Title of Decision]

## Status

[Proposed | Accepted | Deprecated | Superseded]

*If superseded, reference the superseding ADR: "Superseded by ADR XXXX"*

## Context

[Describe the problem or situation that requires a decision]

[Include relevant background information:]
- What technical challenge are we facing?
- What are the constraints (performance, compatibility, specification requirements)?
- What is the current state (if this is changing something)?
- Why is this decision necessary now?

[If this relates to a specification, reference specific sections:]
- RFC XXXX, Section N.N states: "[relevant quote]"
- This creates tension with [other requirement/goal]

## Decision

[State the decision clearly and concisely]

We will [action/approach].

[Explain the decision in detail:]
- How will this be implemented?
- What specific approach will be taken?
- Are there any conditions or exceptions?

[If choosing between alternatives, state why this option was selected:]
- Considered alternatives: [Alternative A], [Alternative B]
- Selected [this approach] because [rationale]

## Consequences

### Positive

- [Benefit 1: e.g., "Improves type safety by..."]
- [Benefit 2: e.g., "Aligns with RFC XXXX requirement..."]
- [Benefit 3: e.g., "Simplifies API usage by..."]
- [Benefit 4: e.g., "Enables better tooling support..."]

### Negative

- [Trade-off 1: e.g., "Increases memory usage by..."]
- [Trade-off 2: e.g., "Breaks backward compatibility with..."]
- [Trade-off 3: e.g., "Adds complexity to..."]
- [Trade-off 4: e.g., "Prevents optimization of..."]

### Mitigation

[How will we address the negative consequences?]

- [For trade-off 1]: [Mitigation strategy]
- [For trade-off 2]: [Mitigation strategy]
- [For trade-off 3]: [Mitigation strategy]

[If no mitigation is possible or needed, state: "No mitigation planned - the positive consequences outweigh the negatives."]

## Alternatives Considered

### Alternative 1: [Name]

**Description:** [Brief description of this alternative approach]

**Pros:**
- [Advantage 1]
- [Advantage 2]

**Cons:**
- [Disadvantage 1]
- [Disadvantage 2]

**Rejected because:** [Clear reason for rejection]

### Alternative 2: [Name]

**Description:** [Brief description of this alternative approach]

**Pros:**
- [Advantage 1]
- [Advantage 2]

**Cons:**
- [Disadvantage 1]
- [Disadvantage 2]

**Rejected because:** [Clear reason for rejection]

## Implementation Notes

[Optional section - include if there are specific technical details needed for implementation]

- [Implementation detail 1]
- [Implementation detail 2]
- [Migration path if this is a breaking change]

## References

- [RFC XXXX - Title](https://www.rfc-editor.org/rfc/rfcXXXX.html)
- [Related ADR YYYY](../YYYY-topic.md)
- [External article or documentation](https://example.com)
- [GitHub Issue #NNN](https://github.com/spec-works/project/issues/NNN)
- [IANA Registry](https://www.iana.org/assignments/...)

## Date

YYYY-MM-DD

## Author

[Name or "SpecWorks Team"]

---

## Template Usage Notes

**Remove this section when creating actual ADR**

### Numbering
- Use four-digit numbers with leading zeros: 0001, 0002, 0003, etc.
- Sequential within repository (project-level ADRs)
- Organization-level ADRs in `specification/adr/` have separate numbering

### File Naming
- Format: `NNNN-short-kebab-case-title.md`
- Examples: `0001-rfc-6902-only.md`, `0002-strongly-typed-api.md`

### When to Create ADRs
Create an ADR when making decisions about:
- API design patterns
- Specification version support
- Technology/library choices
- Performance vs. correctness trade-offs
- Breaking changes
- Feature inclusion/exclusion based on specification
- Architecture patterns
- Cross-cutting concerns

### Keep It Focused
- One decision per ADR
- If a decision has multiple parts, consider whether they should be separate ADRs
- Be specific and actionable

### Context Section Tips
- Assume reader has general technical knowledge but may not know project specifics
- Include quotes from specifications when relevant
- Explain "why now" - what triggered this decision?
- Link to related issues or discussions

### Decision Section Tips
- Be clear and unambiguous
- Use active voice: "We will..." not "It would be good to..."
- Include enough detail that someone can implement the decision
- Reference specification sections when applicable

### Consequences Section Tips
- Be honest about trade-offs
- Include both obvious and subtle consequences
- Think about: developers, users, maintainers, performance, compatibility
- Don't oversell positives or hide negatives

### Status Lifecycle
1. **Proposed** - Draft for discussion, not yet accepted
2. **Accepted** - Team has agreed, is/will be implemented
3. **Deprecated** - Decision is outdated but still in effect (use sparingly)
4. **Superseded** - Replaced by another ADR (link to new ADR)

### Examples from SpecWorks Projects
- **vCard/adr/0001-parse-returns-list.md** - API design decision
- **vCard/adr/0004-vcard-version-4-only.md** - Specification version support
- **JsonDiff/adr/0001-rfc-6902-only.md** - Feature scope decision
- **JsonDiff/adr/0002-wrapper-architecture.md** - Architecture pattern decision
