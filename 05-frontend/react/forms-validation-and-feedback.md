# React Forms, Validation, Feedback and File-Upload UX Standard

## Form ownership and library decision

Use native semantic HTML `<form>` elements. A simple form may use React state and native browser controls. For repeated, nested, multi-step or high-field-count forms, use one project-approved form library plus one schema-validation approach; record the choice in the project ADR. Do not mix several form libraries/patterns within a product without a reason.

Each form owns:

```text
initial values, field schema, UI validation state, submit mutation,
server error mapping, pending/success state, and unsaved-change behaviour
```

The backend is the final validation and authorization authority. Client validation gives faster, clearer feedback; it is never a security boundary and must not duplicate complex business rules blindly.

## Field and validation rules

- Use a visible, programmatically associated label for every control. Placeholder text is not a label.
- Use the semantic input type and useful browser attributes: `email`, `tel`, `date`, `number`, `autocomplete`, `inputMode`, `min`, `max`, `maxLength`, `required`, and `accept` where applicable.
- A controlled field needs an `onChange`; do not accidentally change a field between controlled and uncontrolled state.
- Validate simple format/range/required/cross-field constraints in the UI. Use an allow-list for select values, known enums, route-linked identifiers and structured input.
- Parse and normalize intentionally. Do not silently trim/transform meaningful user content, names, identifiers, passwords or rich text without an explicit requirement.
- Do not expose regex internals, security checks, hidden rules, or technical stack errors as a public form message.
- Client-side validation must not be used to decide prices, final totals, permissions, record ownership, workflow state or availability. Display server-calculated values and handle server conflicts.

## When to validate

| Moment | Default behaviour |
| --- | --- |
| First interaction | Do not show an error before the user has a meaningful chance to enter data. |
| Blur | Show concise field-level feedback for clear field errors. |
| Submit | Validate all relevant fields, focus/announce summary, prevent duplicate submission. |
| While typing | Use sparingly for simple, non-disruptive feedback; do not announce noisy errors every keystroke. |
| Server response | Map safe field errors to their fields and safe form/business errors to the form summary. |

For a failed submit, retain non-sensitive user input, present an error summary before the form, link each issue to its field, then focus the summary or first invalid field. Passwords and other sensitive fields may be cleared if product/security policy requires it.

## Submission and feedback

```text
user submits
  -> client validation
  -> disable only relevant duplicate action + show pending state
  -> feature mutation
  -> server validates/authorizes
  -> success confirmation or safe field/form-level failure
```

- Keep the primary action label meaningful (`Save booking`, `Create employee`), then show an in-progress state without losing context.
- Do not disable all navigation merely because one form submits; block only the action/form as needed.
- Success feedback confirms what happened and what comes next. For a destructive or irreversible action, require a clear confirmation whose wording identifies the consequence.
- Error feedback is concise, human-readable and actionable. Do not use only a toast for important failure; keep relevant feedback near the form.
- Toasts are supplementary, not the sole source for critical errors/success. They must be accessible, not disappear too quickly, and not stack endlessly.
- A 422/request validation response maps stable backend field/code errors. A 409 business conflict keeps the draft where safe and explains that data changed/was unavailable. 401/403/429 follow the API-state standard.
- If a command has uncertain completion after network timeout, do not imply failure or invite duplicate submission. Use the idempotency/status strategy defined for the endpoint.

## Accessibility and keyboard behaviour

- Use semantic `<fieldset>` and `<legend>` for related inputs such as radio groups.
- Associate inline error/help text via `aria-describedby` only when native semantic association is insufficient; set `aria-invalid` for invalid fields.
- Error summaries and async submission results are announced through appropriate accessible status/live regions, with no excessive repeated announcements.
- Preserve logical keyboard order. Modals, date pickers, autocompletes and custom selects must manage focus, Escape and keyboard operation correctly, or use an approved accessible component.
- Required fields use text/symbol plus accessible indication—never colour alone.

## Drafts and multi-step forms

- Draft persistence is requirement-driven. Before persisting, define data sensitivity, storage location, tenant/user ownership, TTL, schema version, encryption/security needs, resume/discard UX, logout eviction and server reconciliation.
- Do not persist passwords, authentication factors, payment credentials, government IDs or other sensitive data in browser storage by default.
- Multi-step forms validate the current step before progression and validate the complete payload on final submission. Back/forward navigation must not silently discard entered values.
- Warn before leaving a genuinely dirty unsaved form. Do not use a prompt for read-only filters or automatically saved data.

## File upload UX

Frontend file checks improve feedback only; server-side validation, malware scanning/quarantine and authorization remain mandatory under the backend file-handling standard.

- The PRD defines allowed business purpose, extensions/content types, size, file count, retention, visibility and download authorization before UI development.
- Before upload, display limits and validate selected file count/size/type as a fast pre-check. Do not trust the filename or browser-reported MIME type as proof.
- Use the native file chooser as the base; drag-and-drop is an enhancement with an equally usable keyboard/chooser path.
- Show each selected file's name, size, status, progress (when available), error and remove/retry action. Never claim an upload succeeded until the server confirms it.
- Do not render untrusted file HTML/SVG in the application context. For previews, use approved safe rendering and treat every file as untrusted.
- For large/resumable uploads, use the backend-approved protocol, cancellation and retry rules. A retry must not create accidental duplicate business attachments.
- On logout/tenant switch, cancel/remove in-progress uploads and tenant-scoped draft/upload metadata.

## Required tests

- Labels, keyboard submit, client validation timing, error summary/focus and screen-reader announcements
- Client/server error mapping, pending state and double-submit prevention
- 409/422/429/network-timeout recovery without unintended duplicate command
- Dirty-form navigation and draft privacy/eviction behaviour
- File type/size/count pre-check, progress, failure, retry and tenant/logout cleanup
