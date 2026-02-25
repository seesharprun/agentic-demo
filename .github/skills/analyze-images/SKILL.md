---
name: analyze-images
description: Analyze image files from a JSON array and return minified JSON-only results with accessibility properties.
license: MIT
---

You analyze one or more image file paths provided as a JSON array.

## Accessibility context

Apply guidance aligned with WCAG 2.0, Section 508, and Microsoft Learn image accessibility practices.

### Classification aligned to Microsoft Learn image types

- `diagram` corresponds to complex visuals (architecture diagrams, flowcharts, decision trees, conceptual diagrams).
- `screenshot` corresponds to content images (UI screenshots, terminal/CLI captures, product surfaces).
- `decorative` corresponds to icon/decorative visuals (status checks, logos, small inline symbols that add no unique meaning).

### How to identify decorative images

Treat an image as `decorative` when it is primarily a logo, status indicator, or visual ornament and does not add unique instructional meaning.
Common filename hints include patterns like `*-icon.*`, `*-logo.*`, `check.*`, `yes.*`, `no.*`, `status-*`.

### Alt-text quality rules

When `alt-text` is required:
- Length must be 40-150 characters.
- Start with an informative type phrase such as "Diagram ..." or "Screenshot ...".
- Do not start with "Image" or "Graphic".
- End with a period.
- Be specific and contextual: name key products/services/components shown.
- Describe purpose and meaning, not just appearance.

### Long-description quality rules

When `description` is required (`diagram` only):
- Provide one concise paragraph.
- Expand beyond alt text; do not repeat it verbatim.
- Describe relationships, flow direction, and key components.
- Include important labels or values that are required to understand the diagram.

For `screenshot`, this skill intentionally omits `description` to match the required compact JSON schema.

## Required behavior

1. Input is a JSON array of objects. Each object has:
	- `file` (string, required): the image file path
	- `existing-alt-text` (string, optional): alt-text already present in the Markdown source
	
	Example input:
	```json
	[
		{"file": "media/example/image-one.png"},
		{"file": "media/example/image-two.jpg", "existing-alt-text": "Screenshot of the settings page."}
	]
	```
2. For each file, classify image type as exactly one of:
	- `diagram`
	- `screenshot`
	- `decorative`
3. If type is `diagram` or `screenshot`, generate `alt-text` with 40-150 characters.
4. If type is `diagram`, also generate `description` as one paragraph.
5. If type is `decorative`, do not include `alt-text` or `description`.

### Evaluating existing alt-text

When `existing-alt-text` is provided for an image:

1. Evaluate it against all alt-text quality rules (40-150 characters, starts with type phrase, names specific products, ends with period, describes purpose not just appearance).
2. If the existing alt-text passes **all** checks:
   - Set `"status": "sufficient"`.
   - Copy the existing alt-text as the `alt-text` value.
   - Still generate `description` if the type is `diagram`.
3. If the existing alt-text fails **any** check:
   - Set `"status": "insufficient"`.
   - Generate improved `alt-text` that preserves the intent and key details from the original.
   - Include `"existing-alt-text"` in the output for human comparison.
   - Still generate `description` if the type is `diagram`.

When `existing-alt-text` is **not** provided:
- Set `"status": "generated"`.
- Generate `alt-text` and `description` per the standard rules above.

## Output format (strict JSON only)

Return exactly one JSON array and nothing else.

Each item must include:
- `file` (string)
- `type` (string: `diagram`, `screenshot`, `decorative`)
- `status` (string: `generated`, `sufficient`, `insufficient`)

Conditional properties:
- For `diagram`: include `alt-text` and `description`
- For `screenshot`: include `alt-text` only
- For `decorative`: include no additional properties; set `status` to `generated`
- When `status` is `insufficient`: include `existing-alt-text` (the original text that was evaluated)

Rules:
- Output must be valid minified JSON (single line, no extra whitespace/newlines).
- Do not wrap JSON in code fences.
- Do not add commentary, labels, or prose before/after JSON.
- Do not include status updates like "running", "analyzing", or "invoking skill".
- Output must start with `[` and end with `]`.
- Keep output deterministic and syntactically valid.

## JSON lint self-check

Before returning output, verify:
- JSON parses successfully.
- Top-level value is an array.
- Every item has `file`, `type`, and `status`.
- `type` values are only `diagram`, `screenshot`, or `decorative`.
- `status` values are only `generated`, `sufficient`, or `insufficient`.
- `alt-text` length is 40-150 characters when present.
- `description` is present only for `diagram`.
- `existing-alt-text` is present only when `status` is `insufficient`.
