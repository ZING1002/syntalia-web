# Feedback protocol

How the reviewer comments on the overview and approves it from the page, and how Claude reads it back. Spec-agnostic: another repo can copy this file with the template.

## Item IDs

Every reviewable element in the page carries `data-item="<ID>" data-kind="<kind>" data-title="<short title>"`; the template's script adds the comment field under it.

| Kind | ID | Element |
| --- | --- | --- |
| `decision` | `D1`, `D2`... | decision card |
| `question` | `Q1`... | open-question card |
| `diagram` | `G1`... | diagram `<figure>` |
| `data-model` | `M1`... | one `.entity` block per table |

IDs stay stable across revisions. A new item gets the next number; never reuse the ID of a removed item.

## Page setup

On `#review` fill the data attributes: `data-spec` (the spec dir name `<###-name>`), `data-artifact` (the URL, from the `spec.md` header), `data-version` (the publish label), `data-round` (starts at `1`). On each republish or revision update `data-version` and increment `data-round`, so the fields start empty and older rounds stay stored in the db.

## Live channel (primary)

The page requests the `db`, `comments` and `downloads` runtime capabilities (declared at publish) and saves into the artifact db, which survives reloads, republishes and sessions.

`review/round-<n>`, written by the page (`n` is `data-round`):

```json
{ "round": 1, "approved": false, "approvedAt": "<ISO, only when approved>", "items": { "D1": { "kind": "decision", "title": "...", "comment": "..." } }, "general": "", "updatedAt": "<ISO>" }
```

Approve is a two-step click in the page (the button turns into "Confirm approval" for about 6 s, then reverts), because modals such as `confirm()` are blocked in the sandboxed frame.

Only items with a comment appear. **Send feedback** saves, then posts a comment on the review bar, which notifies the publishing session:

- `SPEC-FEEDBACK round <n>: <k> comments, approved=false. Read db review/round-<n>.`
- on Approve: `SPEC-FEEDBACK round <n>: approved=true, <k> comments. Read db review/round-<n>.`

The message is only a pointer. Claude reads `review/round-<n>` (Artifact tool, db read) and the artifact's comment threads for the truth, and treats `approved: true` in the db as the only approval. If the send fails (`claude_unavailable`, `consent_required`, `rate_limited`, `not_granted`) or the db is missing, the page keeps the saved state and points to Export.

The page writes each round to its own doc; Claude only reads it, and older rounds remain as history.

## Export (fallback, always present)

**Export feedback** shows the text in a read-only, pre-selected textarea, tries the clipboard, and offers `<###-name>-feedback.md` (via `downloads.save` in the viewer, a blob link when the page is opened as a local file). Page-started downloads are blocked in the claude.ai viewer, which is why the textarea is always shown. The text is relayed to Claude as is:

````
SPEC-FEEDBACK v1

```json
{ "type": "spec-feedback", "version": 1, "spec": "<###-name>", "artifact": "<url>", "overviewVersion": "<label>", "round": 1, "approved": false, "general": "",
  "items": [ { "id": "D1", "kind": "decision", "title": "...", "comment": "..." } ], "exportedAt": "<ISO>" }
```
````

Same meaning as `review/round-<n>`: `items` lists only the commented items, `approved` is the reviewer's explicit choice.

## Reading it

1. A `SPEC-FEEDBACK round <n>` comment arrived, or a `SPEC-FEEDBACK v1` block was pasted: parse it. For the live message, read `review/round-<n>` and the comment threads first.
2. Match each item by ID to the spec section it came from and act on it. Items without a comment are accepted as they are.
3. `approved` must be literally `true`. Silence, an empty db or a state without `approved: true` is not approval.
