# Find-My-Line — Knowledge Management & Decision Traceability

## Purpose

Find-My-Line must preserve the project's reasoning and decision history as well as its current technical and business requirements.

Brainstorming, research, alternatives, decisions, reversals and lessons learned are project knowledge. They must remain recoverable without allowing historical discussion to become confused with the current implementation requirements.

## Permanent project requirement

The project will maintain two distinct layers of knowledge:

1. **Brainstorming Sessions — historical source record**
2. **Project Plans and Requirements — authoritative current record**

The historical record preserves what was discussed and why. The project documents preserve what is currently agreed and should be implemented.

## Master archive

A private master archive will be maintained as:

**`brainstorming sessions.zip`**

The archive should be password protected using strong encryption.

It should contain, to the extent the source material is actually available:

- Complete chronological conversation records.
- Session boundaries.
- Dates/timestamps where available.
- Original discussion context.
- Decisions and alternatives considered.
- Superseded decisions where historically relevant.
- References to resulting project documents/issues where available.

The archive is a historical record, not the implementation source of truth.

### Important completeness rule

The project must never represent a partial export, reconstruction or manually assembled collection as a complete archive of all ChatGPT conversations. If the full historical source cannot be obtained, the archive must be labeled accurately as partial or reconstructed.

## Separation of authority

The project follows this chain:

```text
Brainstorming session
        |
        v
Decision / requirement
        |
        v
Authoritative project document
        |
        v
Development issue / task
        |
        v
Implementation
```

A historical conversation can explain **why** something exists. The current project document determines **what is currently required**.

If a decision changes:

1. Preserve the historical discussion.
2. Record the change and rationale in the appropriate current project document.
3. Update affected development tasks/issues.
4. Do not rewrite history to make the old discussion appear to have reached the new decision.

## Distribution into project documentation

Relevant portions of brainstorming must be copied or distilled into the appropriate authoritative location rather than remaining only in the archive.

Examples:

| Knowledge | Appropriate project location |
|---|---|
| Product definition / boundaries | `DEVELOPMENT_PLAN.md` |
| Phase resources / capacity | `PHASE_RESOURCES.md` |
| Business / operations / GTM decisions | Business and GTM plan |
| Website requirements | Website plan |
| Positioning / mission / messaging | Positioning and mission plan |
| Marketing / user-growth strategy | `MARKETING_PLAN.md` |
| Architecture decisions | Development/architecture documentation |
| Data-source and licensing decisions | Data/resource documentation |
| Privacy, security and community rules | Relevant policy/operations documentation |
| Testing decisions and acceptance criteria | Development/testing documentation and issues |

The exact destination may change as the project grows. The principle does not.

## Traceability requirements

Important decisions should retain, where practical:

- Decision statement.
- Date or phase in which it was made, when known.
- Reason for the decision.
- Alternatives considered.
- Evidence or testing that influenced it.
- Current status: active, superseded, experimental or rejected.
- Affected project areas.
- Related issue/task when one exists.

This is intended to prevent repeated debates, lost context and accidental reintroduction of previously rejected approaches while still allowing decisions to be revisited when new evidence warrants it.

## Access and security

The master brainstorming archive is private project material.

- Do not place the encrypted master archive in the public GitHub repository.
- Do not commit passwords, encryption keys or recovery secrets to Git.
- Store the archive and encryption credentials separately.
- Project documents in the public repository should contain only information that is appropriate for public repository visibility.
- Sensitive personal information, credentials, private user data and other restricted material must not be copied into public project documentation.

## Moving forward

For future project discussions, significant decisions should be captured in the appropriate project document rather than relying on conversational history alone.

When a discussion produces an implementation decision, the resulting requirement should be traceable back to the relevant brainstorming session when the source is available.

When the same topic is revisited, the current authoritative project document should be consulted first, while the historical archive provides context when necessary.

## Relationship to development planning

This knowledge-management system is cross-phase infrastructure. It is not a separate product-development phase and should not create a calendar deadline.

It should be established early, maintained throughout development, and used whenever requirements, architecture, business strategy, marketing, legal/operational planning, community policy or other significant project decisions are created or changed.

## Guiding principle

**Brainstorming preserves the path we took. Project plans define where we are going.**
