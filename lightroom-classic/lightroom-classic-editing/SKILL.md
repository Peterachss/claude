---
name: lightroom-classic-editing
description: Use whenever Peter asks Claude to edit, adjust, grade, rate, flag, keyword or organise photos in Adobe Lightroom Classic, or says "edit my selected photo", "make it warmer", "fix the exposure" and similar while Lightroom Classic is open. Edits go straight onto Classic's Develop sliders through the lightroom-cli MCP tools (lr_ prefix), so they stay non-destructive and reversible in History. Not for the cloud Lightroom app (use lightroom-low-usage-editing for that).
---

# Lightroom Classic editing via lightroom-cli

Peter runs Lightroom Classic on Windows with the lightroom-cli MCP server installed in Claude Desktop. All tools start with `lr_` (for example `lr_system_ping`, `lr_develop_apply_settings`).

## 1. Connect first

1. Call `lr_system_ping`.
2. If it fails, stop and tell Peter, in one short message:
   - Open Lightroom Classic.
   - Go to **File > Plug-in Extras > Start CLI Bridge** (it does not start on its own).
   - Then say "ready".
3. Do not retry more than once before asking.

## 2. Find the photo

- Call `lr_catalog_get_selected_photos` to get the selected photo ID(s).
- If nothing is selected, ask Peter to click a photo in Lightroom Classic.
- Call `lr_develop_get_settings` once to see the current slider values before changing anything.

## 3. Seeing the photo

- Do NOT call the `lr_preview_*` tools. They return the image as a huge base64 text blob, which burns usage and does not show up as a picture.
- To judge the look, use the current slider values plus what Peter describes. If Peter wants Claude to judge the image itself, ask for a screenshot or an exported JPEG pasted into the chat.

## 4. Editing

- Before the first edit on a photo, call `lr_catalog_create_develop_snapshot` with a name like `Before Claude` so Peter can jump back.
- Apply changes in as few calls as possible: prefer one `lr_develop_apply_settings` with several sliders over many `lr_develop_set_value` calls.
- Use Lightroom's own slider names and ranges, for example:
  - Light: `Exposure` (-5 to 5), `Contrast`, `Highlights`, `Shadows`, `Whites`, `Blacks` (-100 to 100)
  - Colour: `Temperature` (2000 to 50000), `Tint`, `Vibrance`, `Saturation`
  - Presence: `Texture`, `Clarity`, `Dehaze`
- Keep edits moderate unless Peter asks for something strong. Small steps (Exposure ±0.3, other sliders ±10 to 20) are easier to fine-tune.
- For sky, subject or background tweaks, use `lr_develop_create_ai_mask_with_adjustments`.
- For several photos with the same look, use the batch tools (`lr_develop_batch_apply_settings`) or copy/paste settings instead of repeating calls.
- Use `dry_run=true` if Peter wants to preview what will change before applying.

## 5. Organising

- Ratings: `lr_catalog_set_rating` (0 to 5)
- Flags: `lr_catalog_set_flag` (1 = pick, -1 = reject, 0 = none)
- Keywords: `lr_catalog_add_keywords`
- Collections (Classic's version of albums): `lr_catalog_create_collection` only makes an EMPTY collection. There is no tool to add photos to a normal collection.
  - To fill an album automatically: add a keyword to the right photos with `lr_catalog_add_keywords`, then make a smart collection on that keyword with `lr_catalog_create_smart_collection`.
  - Otherwise tell Peter to drag the selected photos onto the collection in the left panel.

## 6. Safety

- Never call reset tools (`lr_develop_reset_all_develop_adjustments`, `lr_develop_reset_to_default`, etc.) or `lr_catalog_remove_from_catalog` unless Peter clearly asks for it in that message.
- Never edit photos Peter did not select or name.

## 7. Reporting back

After each edit, reply with a short table of the sliders changed, old value to new value, so Peter can fine-tune them or note them for the Framing the Unframed project. Mention once per session that every change is in the **History** panel and the "Before Claude" snapshot. No em dashes in replies.
