---
name: calicat-cli-operator
description: Use when an AI agent needs to operate the local Calicat CLI to extract design data, interaction data, PRD data, canvas/page structure, or screenshots from Calicat links or file ids. Trigger especially when the user provides a Calicat design URL, asks the agent to parse file_id, canvas id, or layer id correctly, or asks for whole-canvas frontend implementation that must fetch page data progressively instead of loading all design JSON at once. Also use when the agent needs to verify whether the local Calicat CLI is installed, check its version, or decide whether CLI update/install guidance is needed before continuing.
---

# Calicat CLI Design Data

Use this skill when the user wants data from a Calicat file and the local `calicat` CLI is the execution path.

## Read This File For

- skill trigger conditions
- link-form detection
- progressive retrieval rules
- task routing by user intent

## Read Extra References Only When Needed

- If the task depends on making sure the CLI is installed, checking its version, updating it, or telling another agent/user how to install/update it, read [references/cli-lifecycle.md](references/cli-lifecycle.md).

## Link Parsing

Calicat design links commonly appear in three forms.

### File only

```text
https://www.calicat.cn/design/2066708151519244288
```

Parse as:

- `file_id = 2066708151519244288`
- `canvas_id = unknown`
- `selected_layer_id = unknown`

Meaning:

- The user identified the file, but not a specific canvas or layer.
- If the task is canvas-specific, call `get_canvas_list` first.

### Canvas link

```text
https://www.calicat.cn/design/2066708151519244288?node-id=2066708151531827200&node-type=canvas
```

Parse as:

- `file_id = 2066708151519244288`
- `canvas_id = 2066708151531827200`
- `selected_layer_id = unknown`

Meaning:

- Here `node-id` is the canvas id.
- Do not send this `node-id` to `get_meta_data` or `get_design_data` as if it were a layer id.
- For canvas-wide work, go straight to `get_design_page_list` with that `canvas_id`.

### Layer link

```text
https://www.calicat.cn/design/2066708151519244288?node-id=21c84811-08c9-42f6-9dc7-82c6992a6ae8&node-type=group
```

Parse as:

- `file_id = 2066708151519244288`
- `selected_layer_id = 21c84811-08c9-42f6-9dc7-82c6992a6ae8`
- `canvas_id = unknown`

Meaning:

- Here `node-id` is a layer id.
- `node-type=group` is a strong hint that this is a layer-like object, not a canvas.
- Start with `get_meta_data`, then decide whether to read full design data.

## Parsing Rules

- Always extract `file_id` from `/design/{file_id}` first.
- If `node-type=canvas`, treat `node-id` as `canvas_id`.
- If `node-type` exists and is not `canvas`, treat `node-id` as `selected_layer_id`.
- If `node-id` exists but `node-type` is missing, do not guess immediately:
  - if the id looks like a UUID, it is usually a layer id
  - if the id is a long numeric string, it may be a canvas id
  - but when the next step matters, verify by choosing the lightest safe API path

## Safe Resolution Strategy

When the link is ambiguous:

1. parse `file_id`
2. inspect `node-type`
3. if still ambiguous, prefer canvas discovery before large reads
4. do not force a layer workflow on a probable canvas link

## Core Rule

Prefer progressive retrieval.

Large `get_design_data` payloads can easily blow up context. Start with the lightest structure endpoint that answers the next decision:

1. `get_canvas_list`
2. `get_design_page_list`
3. `get_meta_data`
4. `get_design_data`
5. `get_interaction_design_data`

Treat this as directory first, full content later.

## Minimal CLI Surface

List tools:

```bash
calicat tools-list
```

Canvas list:

```bash
calicat tools-call --name get_canvas_list --args '{"file_id":"123456"}'
```

Page list:

```bash
calicat tools-call --name get_design_page_list --args '{"file_id":"123456","canvas_id":"canvas-id"}'
```

Layer meta:

```bash
calicat tools-call --name get_meta_data --args '{"file_id":"123456","selected_layer_id":"layer-uuid"}'
```

Layer design data:

```bash
calicat tools-call --name get_design_data --args '{"file_id":"123456","selected_layer_id":"layer-uuid"}'
```

Interaction design:

```bash
calicat tools-call --name get_interaction_design_data --args '{"file_id":"123456","selected_layer_id":"layer-uuid"}'
```

PRD list:

```bash
calicat tools-call --name get_prd_list --args '{"file_id":"123456"}'
```

PRD full content:

```bash
calicat tools-call --name get_prd_full_content --args '{"file_id":"123456","prd_id":"prd-id"}'
```

## Recommended Workflow By Task

### Single layer design analysis

Use when the user gives a design URL and asks for one layer's design or interaction data.

Workflow:

1. Parse the link shape first
2. Confirm that the link actually points to a layer, not a canvas
3. Call `get_meta_data`
4. Confirm the layer looks like the target
5. Call `get_design_data`
6. If interaction matters, call `get_interaction_design_data`

### Canvas-level work

Use when the user asks for “这个画布”, “当前画布”, or asks to process all pages inside a canvas.

Workflow:

1. Parse the link shape first
2. If the URL already says `node-type=canvas`, treat `node-id` as `canvas_id`
3. Otherwise call `get_canvas_list` before guessing
4. Call `get_design_page_list`
5. Process one page layer at a time
6. For each page, call `get_meta_data` before `get_design_data`

### Requirement card retrieval

Use when the user asks for requirements, PRDs, summaries, or requirement card content.

Workflow:

1. Parse `file_id`
2. Call `get_prd_list`
3. Match by title or summary
4. Call `get_prd_full_content`

Never guess `prd_id` first.

### Full canvas frontend implementation

Use when the user asks to build frontend code for “all designs”, “all pages”, or “the whole canvas”.

Workflow:

1. Resolve `file_id`
2. Resolve `canvas_id`
3. Call `get_design_page_list`
4. Process one page layer at a time
5. For each page:
   - call `get_meta_data`
   - then call `get_design_data`
   - summarize or implement
6. Move to the next page only after finishing the current one

Do not fetch all page design data in a single pass.

Default batch size:

- 1 page per step
- 2 pages only if payloads are clearly small

## Decision Heuristics

If the user says:

- “这个链接里的图层”
  use the layer path
- “这个链接里的画布”
  use the canvas path
- “这个文件里的 PRD”
  use `file_id`, then `get_prd_list`
- “这个画布里的所有页面”
  resolve `canvas_id`, then `get_design_page_list`
- “把整个设计做成前端”
  do not pull everything at once; page-by-page retrieval is mandatory

## Failure Handling

If data access fails:

1. run `calicat status`
2. if auth looks stale, run `calicat logout` then `calicat login`
3. re-check parsed `file_id`, `canvas_id`, and `node-id`
4. if the task depends on install/update/version state, read `references/cli-lifecycle.md`

## Output Style

When reporting back to the user:

- explicitly say which ids you parsed
- explicitly say which CLI calls you made
- explicitly say whether the link pointed to a file, canvas, or layer
- if you use progressive retrieval, explain that you are avoiding loading huge design JSON all at once
