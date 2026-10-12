# RMSPD Patch To-Do

Reference import file: `Combined S1-S4.txt`

This patch should improve large-script readability, layout defaults, character-image and identity assignment, narrator presentation, location state, semantic/theme colours, dice configuration, browser-local saving, TXT importing, keyboard navigation, and text-to-speech. The combined TXT is test data only; text inside it must never be treated as application instructions.
Always check `Unformatted TODO` first. Fold every actionable note into the structured sections below, resolve overlaps or ambiguities, and then remove the processed loose note.

## Unformatted TODO (text dump/loose notes to be folded into To-Dos. Sort/clear and move to the TODO whenever this file is accessed):

_Clear. Add new loose notes here; move them into the numbered plan the next time this file is reviewed._

## Priority order

1. Fix reader-line clustering and overlapping controls.
2. Replace the single giant script with a chapter workspace and chapter navigation.
3. Add reliable browser-local save modes.
4. Build the character portrait gallery and direct identity/alias/disguise assignment flow.
5. Make Narrator/Narration a first-class reader type and add the highlighted/interactable script view.
6. Add persistent location directives and reader presentation modes.
7. Finish the separate Light/Dark theme editor, battle presets, and explicit theme preview workflow.
8. Rename Advanced Tools to Dice Options and simplify the dice workflow.
9. Add the syntax-reference/shortcut side panel.
10. Add the specialised combined-TXT importer and review screen.
11. Complete keyboard-only navigation, per-character text-to-speech, and form/preview clarity.

## Implementation status (2026-10-12)

This ledger tracks the current patch in `index.html`. `Complete` means the structured section's core behaviour and acceptance path are implemented; `In progress` lists the remaining acceptance work rather than hiding it.

| Section | Status | Current result / remaining work |
| --- | --- | --- |
| 1. Large-reader stability | Complete | Virtualised line history, chapter separators, bounded speaker viewport, reader-only/history toggle, viewport-based text sizing, keyboard/mouse splitter, saved split, and normal-flow reader options are implemented. |
| 2. Toggle overlap/defaults | Complete | Controls have separate lanes; top bar and reader dock default closed and remember explicit choices. |
| 3. Character image gallery | In progress | Sprite gallery, direct/multiple assignments, optional icon face crops and priority controls, palette extraction, safe removal, prominent identity mapping, narration-role correction, stale-character cleanup, and protagonist controls are implemented. Remaining: occurrence-by-occurrence rename exclusions and a confirmation prompt when moving protagonist-specific settings. |
| 4. Narrator/highlighted script | In progress | Reserved Narrator roles, portrait-free storybook layout, adjacent multiline narration coalescing, highlighted speaker buttons, semantic colours, and protagonist overrides are implemented. Remaining: finish ambiguity warnings for malformed speaker labels. |
| 5. Persistent locations | In progress | Scene/background/corner/clear parsing, persistence between lines, opacity, gallery asset references, reader display, generated output, highlighting, and a location builder are implemented. Remaining: named reusable location records and unresolved/missing-asset validation. |
| 6. Browser-local saving | Complete | IndexedDB with localStorage fallback, manual/auto modes, explicit save/revert/clear, dirty state, and unload protection are implemented without requiring a download. |
| 7. Chapter workspace | Complete | Separate chapter records, numbered/dropdown navigation, editable titles, reorder/delete/new/import, chapter-local history/cursor/images, and schema migration are implemented. |
| 8. Dice Options | In progress | Renamed side-tab, roll types, character defaults, line overrides, initiative settings, syntax, and battle-theme integration are implemented. Remaining: show explicit line-vs-character provenance and add stronger contradictory-option guidance. |
| 9. Syntax Reference | In progress | Searchable side-tab, canonical examples, Insert/Copy actions, disguise parsing, and a parameterised location builder are implemented. Remaining: pinned/custom shortcuts, shortcut-key conflict handling, and entity-aware builders for characters/dice/notes. |
| 10. Combined TXT import | In progress | Review-before-commit, 57-chapter detection, separator cleanup, Narrator/System handling, initiative inference, destinations, and per-speaker create/map/narrator/system/ignore choices are implemented. Remaining: richer mechanics classification/reporting and a project-level undo wrapper for a whole import. |
| 11. Light/Dark themes | Complete | Separate mode state, explicit staged preview/apply/reset/cancel, linked reader by default, opt-in custom reader theme, optional global heading override, battle presets, black/white palette anchors, and zero-default contrast are implemented. |
| 12. Accessibility/TTS/clarity | In progress | Workspace/script/theme keyboard navigation, global shortcuts, saved/fallback speech voices, per-character rate/pitch, narrator-voice names, dialogue-only filtering, autoplay/auto-advance, preview naming, and optional example placeholders are implemented. Remaining: complete focus-return audits for every modal and equivalent keyboard controls for every crop/reorder gesture. |

---

## 1. Stop large story drops from clustering together

### Problem

When a large script is pasted or imported, the Visual Novel Reader's line list compresses into an overlapping cluster. The combined file produces thousands of reader items, so rendering every item at once also risks slow input and scrolling. Expanded portraits/solid-colour character images can also escape the active-speaker area and overlap the Visual Novel options bar or dialogue-line list.

### Patch

- Make every `.reader-line` retain its own minimum height and normal line height.
- Prevent the reader-line list from shrinking its children to fit the available panel height.
- Keep the list vertically scrollable; items must never render on top of one another.
- Render a bounded window around the active line instead of inserting thousands of line buttons into the DOM at once.
- Include a small buffer before and after the visible range so scrolling remains smooth.
- Preserve the active line when the reader is resized, collapsed, reopened, or moved.
- Add chapter separators inside the line list so large imports are navigable.
- Add a compact `Jump to chapter` control when more than one chapter is present.

### Keep expanded character images inside the speaker section

- Treat the visible speaker/portrait/dialogue area as a bounded viewport.
- Expanded portraits, solid-colour icons, selected face images, location art, and narration layouts must remain inside that viewport.
- Do not allow an image to overlap:
  - The Visual Novel options bar.
  - Reader navigation/actions.
  - Speaker name or dialogue text.
  - The dialogue-line list.
  - The new resize divider.
- Size images from the available speaker viewport rather than from the overall reader-card width.
- Use `object-fit: contain` for portrait/location images and clamp both width and height.
- Give portrait and text columns `min-width: 0` and `min-height: 0` so their contents can shrink correctly.
- Clip only the visual image layer when necessary; never clip dialogue text, focus outlines, or reader controls.
- Recalculate image bounds after:
  - Reader resize.
  - Editor/reader pane resize.
  - Reader pop-out/fullscreen changes.
  - Chapter changes.
  - Speaker/portrait changes.
  - Dragging the speaker/line-list divider.
- A very tall or very wide image should letterbox inside its area rather than growing over adjacent UI.

### Add a draggable speaker/line-list divider

- Place a horizontal resize handle between:
  - The visible active-speaker section above; and
  - The scrollable dialogue-line list below.
- Dragging the divider upward gives more height to the dialogue-line list and less to the active-speaker section.
- Dragging it downward gives more height to the active-speaker section and shows fewer history lines.
- Use a two-row grid or equivalent bounded layout so neither section can overlap the other.
- Enforce minimums for both sections:
  - The active-speaker section must retain enough height for the speaker name and at least a readable portion of dialogue.
  - The dialogue-line list must retain enough height for at least two usable line buttons plus its scrollbar.
- Enforce a maximum so neither section can consume the complete reader card.
- Re-clamp the saved split whenever the reader card becomes smaller.
- Remember the chosen split as a browser-local UI preference.
- Add a double-click action to restore the default split.
- `Refresh layout` should restore the default split.
- While dragging:
  - Disable text selection.
  - Use an obvious resize cursor.
  - Update smoothly without rebuilding reader items.
- Make the divider keyboard accessible:
  - `role="separator"`.
  - `aria-orientation="horizontal"`.
  - Current/min/max values.
  - Arrow Up/Down adjustment in small steps.
  - Shift + Arrow Up/Down adjustment in larger steps.
- On narrow/mobile layouts, use a larger touch target and clamp more aggressively. If the available height is too small for safe resizing, fall back to the default stacked layout instead of overlapping content.

### Keep Visual Novel options out of the speaker area

- Keep the `Visual Novel options` block in the reader's normal vertical layout flow.
- Opening the options must push/reduce the available speaker area instead of layering the controls over the portrait, speaker name, or dialogue.
- Closing the options must leave one compact summary row above the speaker area; it must not retain an old `top`, height, or overlay position.
- Give the open options block its own bounded scrolling area when its contents are taller than the available reader space.
- Recalculate and clamp the draggable speaker/history split whenever the options block opens or closes.
- Preserve the active reader line and the user's saved speaker/history preference while reflowing the reader.
- Apply the same behaviour in docked, pop-out, fullscreen, narrow, light, and dark reader layouts.

### Reader-first presentation

- Treat the Visual Novel Reader as the primary reading experience; the Icon-style Preview is an optional alternate export/format.
- Add a `Reader only` toggle that hides the line-preview/history list while preserving Previous/Next navigation and the active line.
- Let text scale within safe limits from the available reader viewport. Text length must not resize the sprite column or make the sprite chase the text dimensions.
- Keep character sprites sized from the reader viewport rather than from dialogue length.
- In reader-only mode, present the current sprite, speaker, and dialogue like a visual-novel text-box scene and let the reader use all remaining safe height.

### Acceptance criteria

- Importing the supplied combined TXT does not produce overlapping or compressed line buttons.
- The current line remains clearly visible and highlighted.
- Scrolling the line list remains responsive with 3,000+ parsed story items.
- Moving between chapters does not rebuild or jump the whole page unnecessarily.
- Expanding any portrait or solid-colour character image never covers the options bar, dialogue, resize handle, or line list.
- Dragging upward shows more dialogue-history lines; dragging downward enlarges the active-speaker view.
- The divider cannot shrink either section below its defined minimum.
- The chosen split survives reload, while Refresh layout restores the default.
- Mouse, touch, and keyboard resizing all work without moving the active reader line.
- Open Visual Novel options never cover the portrait, speaker name, dialogue, resize divider, or history list.
- Closing Visual Novel options returns the speaker area to its previous size without leaving the summary over the portrait/dialogue.

---

## 2. Prevent the top-bar and reader toggles from overlapping

### Problem

The fixed top-bar toggle and the reader collapse toggle can occupy the same top-right position, especially when the reader is docked right or the window is narrow.

### Patch

- Give the top-bar toggle a permanent location that does not share the reader card's control area.
- Preferred placement: attach it to the top of the left workspace rail, or reserve a dedicated fixed control lane outside the reader.
- Do not position either toggle using the other panel's changing width.
- Add responsive spacing rules for desktop and narrow/mobile layouts.
- Make both the top bar and reader dock **collapsed by default** when no saved preference exists.
- Once the user explicitly opens or closes either area, remember that preference in browser storage.
- `Refresh layout` should restore the new defaults: top bar closed and reader dock closed.

### Acceptance criteria

- The two toggles never overlap at common desktop widths, narrow split panes, or mobile widths.
- A first visit starts with both areas closed.
- Reloading preserves an explicit user choice.
- Refreshing the layout restores both to their closed defaults.

---

## 3. Add a pasteable character-image gallery with direct assignment

### Problem

`Script Images` is useful for standalone narration images, but it is not the requested character portrait workflow. Character images need their own pasteable gallery and a direct way to assign a gallery image to a character.

### Patch

- Keep `Script Images` for standalone narration images.
- Expand the existing Portrait Library into a dedicated character-image gallery.
- Support all of the following gallery inputs:
  - Paste an image from the clipboard with `Ctrl+V`/`Cmd+V`.
  - Drag and drop image files.
  - Select one or more image files.
- Store each gallery image once and reference it by asset ID from character records.
- Each gallery thumbnail should provide:
  - Select.
  - Select Face.
  - Extract colours.
  - Rename.
  - Assign to character.
  - Replace.
  - Remove.
- Add an `Assign portrait` control directly to every character row.
- The assignment control should show gallery thumbnails or a searchable gallery picker.
- Selecting a gallery image and then a character should assign it immediately; clicking dialogue in the reader must not be required.
- Keep the current crop/position tools available before assignment.
- Allow more than one image to be assigned to a character without losing the original gallery asset.
- Add `Select Face` to every assigned character image:
  - Open a square face crop/focal-point editor.
  - Save the face crop separately from the full portrait crop.
  - Let the user set face priority by dragging faces into order or assigning a numeric priority.
  - Priority 1 is the character's primary small icon.
  - Lower-priority faces are fallbacks or alternate preview faces.
- Use the selected face crop for compact character previews, speaker buttons, reader-line previews, and any small avatar treatment.
- Continue using the full portrait assignment in the main reader portrait area.
- If no face is selected, fall back to the first assigned portrait crop and then to the character's initials/solid-colour icon.
- Add `Select character colours from image` to gallery thumbnails and assigned portraits:
  - Extract a small palette from the selected image in-browser.
  - Show suggested dominant, accent, dark, and light swatches before applying anything.
  - Let the user assign a suggested swatch to bubble, name, icon, or all compatible character colour fields.
  - Include a one-click `Quick assign colours` action that chooses accessible foreground/background combinations automatically.
  - Do not change character colours until the user confirms a swatch or Quick assign.
- If an image is already assigned, show which character or characters use it.
- Removing an in-use image must warn the user and clearly state which assignments will be cleared.

### Character identity, typo mapping, and disguise names

- Treat a character's canonical identity separately from the exact speaker label written in the script.
- When script scanning finds likely variants such as `Alice:`, `alice:`, and `Aliice:`, group them as suggestions but never merge them silently.
- From each detected or unmatched speaker, offer three explicit actions:
  - `Replace with character name in script` rewrites confirmed occurrences to the chosen character's canonical script label.
  - `Assign as alias` keeps the existing script label but maps it to the same character, portrait, colours, dice defaults, and text-to-speech settings.
  - `Assign as disguise/display name` keeps the underlying character identity while showing a different speaker name to the reader.
- Before rewriting script text, show the exact labels and number of occurrences affected, allow individual occurrences to be excluded, and make the operation undoable.
- Alias matching should be Unicode-aware and may ignore case/harmless spacing for suggestions, but the original spelling must remain unchanged unless the user chooses replacement.
- Add a registry-backed disguise syntax such as `Alice (as Red Queen):`; confirm the final syntax in the Syntax panel and use the same rule in parsing, highlighting, importing, and generated output.
- A disguised character continues to use the underlying character's assets and settings unless the disguise has an explicit visual override.
- Let the user choose whether the true character name is hidden, shown as secondary text, or revealed only after a spoiler interaction.
- Prevent alias loops and warn when one label is already assigned to a different character.
- Renaming or deleting a canonical character must show its aliases, disguise labels, and affected script references before the change is applied.

### Protagonist controls

- Add `Assign as protagonist` to every character row.
- Only one character may be the active protagonist at a time.
- Show a visible protagonist badge in the character list and gallery assignment UI.
- Add `Lock protagonist colour` beside the protagonist's colour settings.
- Any `Globalise colours`, palette batch assignment, or apply-to-all operation must skip the protagonist while that lock is enabled.
- The protagonist may still be edited directly while locked; the lock only protects against bulk/global operations.
- Changing the protagonist should ask whether the protagonist-specific colours and locks should move to the new character or remain on the old character as ordinary character colours.

### Sprite, identity, and stale-character workflow

- Character gallery uploads are full sprite images for the Visual Novel Reader.
- `Select Face` is an optional crop for compact icons and the Icon-style Preview; it must not replace the full reader sprite.
- Make `Map this label to another character` a primary/simple character action. Keep `Fix name in script` secondary and optional.
- Add `Treat this speaker label as Narrator` for labels mistakenly scanned as characters; mapped narration must never show a character sprite/icon in the reader.
- Add `Remove characters not in script`, show the affected names, and require confirmation before pruning stale character records.

### Acceptance criteria

- Pasting an image while the character gallery is active creates a reusable gallery asset.
- A portrait can be assigned from either the gallery or the character row.
- Reusing one image for multiple characters does not duplicate the stored image data.
- A character can have a full portrait and an independently cropped, prioritised face icon.
- Small previews consistently use the highest-priority available face.
- Image-palette suggestions can be previewed and applied without leaving the character row/gallery.
- Bulk colour changes do not overwrite a colour-locked protagonist.
- A likely typo can be safely replaced or mapped as an alias without creating a duplicate character.
- A disguise/display name can hide the canonical name while retaining the correct portrait, colours, dice defaults, and speech voice.
- Character-label replacement reports its scope, supports undo, and never rewrites unconfirmed occurrences.
- Narration/standalone script images continue to work independently.
- Gallery images and assignments survive reloads and project saves.

---

## 4. Make Narrator/Narration first-class and add highlighted speaker controls

### Problem

Speaker labels inside the raw script are difficult to scan. Recognised labels such as `Character:` or `3:` should be visually distinct and interactable. Narrator/Narration must be its own story type—not a character with a missing portrait.

### Narrator/Narration rules

- Recognise `Narrator:`, `Narration:`, `-:`, and `null:` as the same Narrator type.
- Narrator is a reserved system role and must never be added to the character list.
- Narration never requires or searches for a character image.
- Do not show `No portrait assigned` for narration.
- Hide/collapse the portrait column while a narration item is active and present the narration as full-width storybook/line text.
- Preserve the current location background or corner reference behind/beside narration.
- Restore the normal portrait-and-dialogue layout on the next character line.
- Give Narrator its own optional text colour setting rather than borrowing a character colour.
- Multiline narration remains one narration item until a new valid speaker, directive, or chapter begins.
- Narrator never receives a sprite/icon through automatic character scanning or character fallback. Only explicitly assigned narration art may change that in a future feature.

### Patch

- Keep the raw textarea as the canonical editable script source.
- Add an `Edit` / `Highlighted` view toggle rather than attempting clickable controls inside a normal textarea.
- In Highlighted view, render the script line-for-line and turn recognised speaker labels into background-highlighted buttons.
- Use the character's configured colour for the button background, with automatically readable foreground text.
- Accept ordinary names, stylised Unicode names, and numeric speaker labels such as `3:`.
- Normalise `Narrator:`, `Narration:`, `-:`, and `null:` to one Narrator type.
- Do not create a clickable character button for Narrator.
- Clicking a character button should:
  - Select or open that character in the Characters tab.
  - Show the assigned portrait.
  - Offer `Assign/Change portrait` if none is assigned.
  - Move the Visual Novel Reader to the corresponding story item when possible.
- Highlight unmatched speaker labels differently and offer `Create character` or `Map to existing character`.
- Keep line numbers or stable source indexes so edits and reader positions stay synchronised.

### Narration and thought colours

- Add a `Colour narrations and thoughts` toggle to the relevant Page/Visual Novel Theme controls.
- When enabled, style semantic spans without changing the stored script text:
  - Parentheses `(...)` use the selected Thought colour.
  - Square brackets `[...]` use the selected Narration/Action colour.
- Provide separate colour pickers for Thought and Narration/Action text.
- Add `Use unique protagonist narration/thought colours`, unlocked only when:
  - semantic colouring is enabled; and
  - a protagonist has been assigned.
- When that toggle is enabled, expose separate protagonist Thought and Narration/Action colours.
- Protagonist-specific semantic colours apply only to spans inside the active protagonist's lines.
- Whole Narrator lines continue to use the Narrator colour; bracketed spans inside them may use the Narration/Action colour if enabled.
- Only style balanced delimiters. Leave unmatched `(`, `)`, `[`, or `]` as ordinary text.
- Styling must also work in generated HTML, the live reader, highlighted script view, and exported/imported projects.

### Colour precedence

Apply colours in this order so the result is predictable:

1. Page Theme supplies the default reader surface and text colours.
2. Custom Reader Theme replaces those defaults when enabled.
3. Battle Theme replaces the active reader surface/base text during initiative.
4. Narrator colour replaces base text for a whole Narrator/Narration item.
5. Global Thought and Narration/Action colours replace the surrounding line colour for balanced `(...)` and `[...]` spans.
6. Protagonist-specific Thought and Narration/Action colours replace the global semantic colours only inside the assigned protagonist's lines.

Run contrast checks against the active reader surface and warn before applying a colour combination that makes text difficult to read.

### Parsing safeguards

- Do not treat URLs, timestamps, bracketed metadata, or punctuation-only prefixes as speakers.
- Do not create characters from Discord/system lines such as `Dice Maiden:` unless the user explicitly maps them.
- Multiline dialogue without another valid speaker label must remain attached to the previous speaker.

### Acceptance criteria

- `Character:` and `3:` become coloured buttons in Highlighted view.
- `Narrator:`, `Narration:`, `-:`, and `null:` render as full-width storybook narration without creating narrator characters or portrait placeholders.
- Clicking a speaker button opens the correct character/portrait workflow.
- Switching between Edit and Highlighted views does not change or reorder the raw script.
- Parenthesised thoughts and square-bracket narration/actions use their configured colours only when semantic colouring is enabled.
- Protagonist-only semantic colours never leak onto another character or Narrator line.

---

## 5. Add persistent location directives

### Purpose

Locations should be script state, not character dialogue. A location directive changes the reader's setting and remains active until another location directive replaces or clears it.

### Supported syntax

```text
:: Location: (Location Name): Image: locationexample1.png ::

:: Location: (Location Name): Image: locationexample1.png : bg: locationexample1.png opacity = 20 ::

:: Location: (Location Name): Image: locationexample1.png : CornerRef ::
```

The parser should tolerate extra spaces, case differences in keywords, and an optional trailing colon before `::`. Parentheses around the location name are optional display punctuation and should not be stored as part of the name.

### Location state

- Parse the directive into a structured location object rather than showing it as dialogue.
- Store at least:
  - Location name.
  - Primary image asset/path.
  - Display mode: `scene`, `background`, or `corner-reference`.
  - Optional separate background asset/path.
  - Background opacity.
- A new location replaces all persistent visual state from the previous location.
- Add `:: Location: Clear ::` to clear the active location and its visuals explicitly.
- Include active location state on every following reader item until it changes.
- Show the current location name as a small reader label/chip without adding it to the spoken text.
- Location directives must appear as distinct directive chips in Highlighted script view, not as character buttons.

### Display modes

#### Default `Image` / scene mode

- `Image:` identifies the primary location image.
- When the directive is reached, show it as a large establishing/story image for the location transition.
- Return to the normal narration or character layout on the next story line while keeping the location name active.

#### `bg` mode

- `bg:` sets a persistent reader background image without replacing or moving the dialogue text.
- Render the background on its own layer behind reader content.
- `opacity = 20` means 20% opacity; accept values from 0–100 and clamp invalid values safely.
- If opacity is omitted, default to 20%.
- The primary `Image:` and `bg:` asset may be the same file or different files.
- Text, controls, portraits, and focus outlines must remain readable above the background.
- The background remains until the location changes or is cleared.

#### `CornerRef` mode

- Keep the primary location image as a small setting-reference icon in a reader corner.
- The icon must not cover reader controls, text, speaker names, portraits, or the two collapse toggles.
- Show the location name in the icon tooltip/accessibility label.
- The corner reference remains until the location changes or is cleared.

### Asset handling

- Resolve image references against project gallery assets first, then valid relative exported-site paths.
- Let the user choose or paste a location image from an image gallery rather than requiring a typed filename.
- Add a directive builder that inserts valid Location syntax from a location name, selected asset, mode, and opacity.
- Missing assets should produce an import/build warning and a visible placeholder, not break the reader.
- Save location assets in IndexedDB with the other project images and preserve references through JSON export/import.

### Acceptance criteria

- Location directives do not appear as dialogue or characters.
- Default Image mode produces one establishing image and then returns to ordinary story presentation.
- Background mode persists behind multiple narration/character lines without disturbing text layout.
- CornerRef persists as a small setting icon without overlapping controls.
- Any new Location directive replaces the previous background/corner reference.
- Clear removes the current location visual state.
- The live reader and generated HTML behave the same way.

---

## 6. Add Manual Save Project and Auto-save Project modes

### Answer

Yes, this can work without downloading a JSON or project file. The project can be saved inside the browser. JSON download/import should remain available as a portable backup.

### Patch

- Add an `Auto-save project` toggle in Save / Export.
- Keep auto-save enabled by default for backwards compatibility.
- Add a visible `Save Project` button for manual saves.
- When auto-save is disabled:
  - Edits mark the project as `Unsaved`.
  - Browser storage is not updated until `Save Project` is clicked.
  - Closing/reloading with unsaved changes shows a warning.
- When auto-save is enabled:
  - Save after a short debounce.
  - Show `Saving…`, then `Saved locally` with the last-saved time.
- Add `Revert to last local save` with confirmation.
- Keep `Download Project JSON` and `Import Project JSON` as explicit backup/transfer tools.

### Storage work

- Move full project persistence to IndexedDB so pasted portraits and large projects are not constrained by `localStorage` size limits.
- Keep small UI preferences—collapsed panels, editor mode, layout position—in `localStorage`.
- Migrate an existing `storyToolDraft` localStorage project into IndexedDB once, without deleting the old draft until migration succeeds.
- Store image blobs/assets separately and reference them by stable IDs from the project.

### Acceptance criteria

- Manual Save persists the project across reloads without downloading anything.
- Auto-save can be switched off and stays off after reload.
- The interface always indicates Saved or Unsaved state.
- Large pasted-image galleries do not fail because of localStorage quota limits.
- JSON export/import remains functional.

---

## 7. Add a real chapter workspace

### Problem

The project currently stores one very large script, so adding or editing a new chapter means scrolling through everything that came before it. Chapters should be separate editing spaces inside one project.

### Chapter data model

- Replace the single project-level `script` string with an ordered chapter collection.
- Each chapter should store at least:
  - Stable chapter ID.
  - Editable chapter title.
  - Script text.
  - Created/updated timestamps.
  - Reader position for that chapter.
  - Any chapter-scoped import warnings or unresolved mappings.
- Store `activeChapterId` separately so reopening a project returns to the same chapter.
- Keep characters, galleries, themes, syntax shortcuts, and reusable locations at project level.
- Keep location/initiative progression inside the chapter unless an explicit carry-to-next-chapter rule is enabled.
- Migrate legacy projects containing a single `script` string into one chapter without losing text.
- Add a previewed `Split script into chapters` migration tool for legacy scripts that already contain `:: Chapter: ... ::` markers.
- Never silently split or rewrite an existing script during migration.

### Chapter editor

- Give every chapter a fresh, independent script editor rather than showing every chapter in one textarea.
- Add `New Chapter` and create it with an empty editor.
- Add `Duplicate Chapter`, `Delete Chapter`, and `Reorder Chapters`.
- Deleting a non-empty chapter requires confirmation and names the chapter being removed.
- Add an editable chapter-title field above the script.
- Renaming a chapter updates navigation, reader position labels, generated output, and exports immediately.
- Preserve undo/redo separately per chapter so switching chapters does not merge unrelated edit histories.
- Remember cursor and scroll position independently for each chapter.
- Add a project-wide search option that can locate text across chapters without combining their editors.

### Chapter navigation

- Add a compact chapter selector using this pattern:

```text
‹  1  2  3  …  12  ›
```

- Previous/next arrows switch chapters, not individual reader lines.
- Number buttons show a bounded range around the active chapter and use an ellipsis for large projects.
- The active chapter number must be visually and programmatically selected.
- Add an optional title dropdown beside the numbered controls.
- The dropdown should display `chapter number — editable title`.
- Add a preference for:
  - Numbers only.
  - Dropdown only.
  - Numbers + dropdown.
- Keyboard navigation should support previous/next chapter without firing while typing in the script.
- The Visual Novel Reader should stay in the active chapter unless `Next chapter` navigation is explicitly used.

### Import as new chapter

- Add a visible `Import as New Chapter` action beside `Load TXT`.
- Importing this way must never replace the currently active chapter.
- Use the source filename without `.txt` as the proposed chapter title.
- Show a title/edit confirmation before committing the import.
- Run the same speaker, Narrator, system, battle, location, and syntax detection used by the normal importer.
- Add the imported chapter after the active chapter by default, with an option to append it to the project instead.
- If the selected TXT contains multiple recognised chapter headings, offer:
  - Import the entire file as one new chapter.
  - Split it into multiple new chapters.
  - Cancel and review.
- Treat a successful chapter import as one undoable project operation.

### Compatibility and output

- Bump the project schema version and migrate old JSON safely.
- JSON export should store the structured chapter list.
- JSON import should accept both the old single-script format and the new chapter format.
- HTML generation may flatten chapters internally, but chapter IDs/titles and order must remain stable.
- Downloaded story output should preserve chapter navigation and location/initiative boundaries.
- Manual save, auto-save, dirty state, and revert must cover chapter creation, deletion, renaming, reordering, and text changes.

### Acceptance criteria

- Creating a new chapter opens an empty editor without altering the previous chapter.
- A project with many chapters can be navigated using `‹ 1 2 3 … ›` and/or the title dropdown.
- Chapter titles can be edited without touching script content.
- `Import as New Chapter` leaves the current chapter unchanged.
- Importing the combined S1–S4 file in split mode creates 57 separately editable chapters.
- Switching chapters restores each chapter's cursor, editor scroll, reader position, and undo history.
- Existing one-script projects migrate without data loss.

---

## 8. Rename Advanced Tools to Dice Options and make it user-friendly

### Problem

The current side tab and card are called `Advanced Tools`, but the panel primarily configures dice tags, dice characters, roll modes, and initiative behaviour. The broad name hides useful controls, while terms such as `Persistent dice characters` and the combination of initiative toggles are difficult to understand without already knowing how the parser works.

### Rename and scope

- Rename the workspace side tab from `Advanced Tools` to `Dice Options`.
- Rename the card heading from `Advanced Tools ⚔🎲` to `Dice Options 🎲`.
- Update the tab tooltip, accessible label, mobile label, and any help/search references at the same time.
- Keep the existing internal panel ID and stored project fields where practical so the rename does not break saved projects.
- Do not rename unrelated advanced character/project settings to Dice Options.
- Move the existing `Advanced tools and syntax` area under Script into the planned Syntax panel. Until that panel exists, rename its summary to `Script tools and syntax` so it is not confused with Dice Options.

### Friendlier Dice Options layout

Organise the panel into three clearly labelled sections shown in this order:

1. `Character dice defaults`
   - Replace `Persistent dice characters` with plain-language copy explaining that selected characters can automatically treat their dialogue as roll lines.
   - Show only characters currently enabled for dice, plus an `Add character` picker.
   - For each character, show the character name/icon, default roll type, default roll mode, and a clear `Remove dice default` action.
   - Roll modes should use readable labels: `Normal`, `Advantage`, and `Disadvantage`.
   - Make it clear that a reader-line override affects only that line.

2. `Roll types`
   - Present existing dice tags as reusable roll types such as `Attack roll`, `Damage roll`, `Strength check`, or a custom label.
   - Change the user-facing label `New dice tag` to `New roll type`; keep the existing dice-tag data format internally for compatibility.
   - Put the text field and `Add roll type` button together.
   - Support rename and delete actions directly on each roll type.
   - Prevent blank and duplicate names with inline feedback instead of an alert.
   - When renaming or deleting an in-use roll type, show how many character defaults and reader lines use it before applying the change.

3. `Initiative behaviour`
   - Replace the three difficult-to-combine toggles with one clearly described initiative-ending choice wherever possible:
     - `End at an Exit Initiative marker` — default.
     - `End at the chapter boundary`.
     - `Carry through one following chapter, then end`.
   - If an existing advanced combination cannot be represented by one choice, preserve it under a collapsed `Advanced compatibility settings` area rather than silently changing it.
   - Show a one-sentence example beneath the selected option explaining what will happen.

### Reader-line assignment

- Keep per-line dice assignment in the Visual Novel Reader, but use the same names and order as the Dice Options panel.
- The line controls should read:
  - `Not a roll` / `Use character default` / selected roll type.
  - `Normal` / `Advantage` / `Disadvantage`.
- Show the effective result when a character default and line override interact, for example `Attack roll · Advantage (line override)`.
- Provide a small `Clear override` action rather than requiring the user to understand or insert `[[dice: off]]` manually.
- Keep raw marker syntax available in the Syntax reference for users who want it, but do not require syntax knowledge for ordinary dice configuration.

### Usability and compatibility

- Add a short introductory sentence explaining what the panel changes in the reader and generated story.
- Include helpful empty states when no dice characters or roll types exist.
- Use the same roll-type terminology in Dice Options, reader controls, import review, the Syntax panel, and generated output.
- All controls must be keyboard accessible and show visible focus.
- Dice changes participate in the project's saved/unsaved state and undo history where applicable.
- Preserve existing dice tags, character assignments, line overrides, and initiative settings when loading an older project.

### Acceptance criteria

- The workspace tab, card title, tooltip, and accessible label all say `Dice Options`.
- A new user can assign a character default and create an Attack roll without writing syntax.
- The interface consistently calls user-facing dice tags `Roll types` while old project data still loads.
- Invalid or duplicate roll-type names produce useful inline feedback.
- Renaming or deleting an in-use roll type never silently breaks assignments.
- Initiative options cannot be placed into a contradictory state through the standard UI.
- Reader-line overrides clearly show whether the line, character default, or neither is supplying the active roll behaviour.

---

## 9. Add a syntax-reference and shortcut side panel

### Purpose

Replace the scattered syntax hints and one-off insertion buttons with one cohesive, searchable system. The same definitions should power the reference text, insert shortcuts, validation, highlighted script view, and import review.

### Syntax side-panel layout

- Add a collapsible `Syntax` side panel inside the Script workspace.
- The panel must not cover the script editor, reader controls, or chapter selector.
- Make it resizable on desktop and drawer-style on narrow/mobile layouts.
- Remember open/closed state and width as UI preferences.
- Add search/filter and collapsible categories:
  - Chapters and scenes.
  - Speakers and Narrator/Narration.
  - Locations and location display modes.
  - Character and story images.
  - Initiative, battle, and dice.
  - Content warnings and translator notes.
  - Thoughts, narration/actions, and semantic text.
- Each reference entry should show:
  - Name.
  - Exact syntax.
  - Short explanation.
  - Example.
  - `Insert` action.
  - `Assign/Configure` action when the syntax references a project entity.

### Clean up the inline script instructions

- Replace the dense syntax paragraph below the paste/script editor with a short, scannable introduction and an `Open Syntax reference` action.
- Keep only the most common edit actions beside the script: add a speaker, narration, chapter, image, location, or dice marker.
- Put detailed syntax, edge cases, and examples in the searchable Syntax panel rather than repeating them in one large paragraph.
- Use one example per line or a small definition list; do not combine unrelated syntax into a wall of text.
- Collapse advanced syntax by default without hiding validation errors relevant to the current script.
- Ensure every inline example is generated from the same syntax registry used by the parser and insertion tools.

### Cohesive syntax registry

- Define syntax once in a data-driven registry rather than hard-coding separate parser, help text, and toolbar rules.
- Each syntax definition should include:
  - Stable ID and category.
  - Display label and aliases.
  - Template/example.
  - Parse/validation rule.
  - Insertion behaviour.
  - Allowed assignment types.
  - Whether it creates persistent state.
- Generate the side-panel reference and default shortcut buttons from this registry.
- Use the same registry to flag malformed syntax and power import-review explanations.
- Adding a future built-in syntax should require one registry entry rather than edits across unrelated UI sections.

### Shortcut system

- Add a `Shortcuts` area within the syntax panel.
- Allow references to be pinned as one-click shortcuts.
- Support optional keyboard shortcuts, with conflict detection and a reset-to-default action.
- Insert at the current cursor/selection and preserve focus in the script editor.
- Treat each shortcut insertion as one undo operation.
- Parameterised syntax opens a compact builder instead of inserting broken placeholders.
- Example builders:
  - Choose a chapter/scene title.
  - Choose a character or Narrator.
  - Choose a gallery image.
  - Choose/create a location, display mode, and opacity.
  - Choose a dice tag and roll mode.
  - Choose a translator-note number.
- Allow user-created text/template shortcuts, clearly separated from built-in syntax.
- Save custom shortcuts at project level, with an option to save selected shortcuts as browser-wide defaults.

### Assignment system

- `Assign/Configure` should bind syntax to existing project entities instead of relying on manually typed names.
- Supported assignments should include:
  - Speaker token → character, Narrator, or system role.
  - Image reference → gallery asset and image role.
  - Location directive → saved location and display mode.
  - Dice syntax → dice tag/character and roll mode.
  - Note marker → translator note.
- Renaming an assigned character, location, asset, dice tag, or chapter should update generated insertions and offer a safe script-reference update.
- Never rewrite existing script references silently; show the number of affected references and require confirmation.
- Highlight unresolved assignments in both the syntax panel and Highlighted script view.

### Acceptance criteria

- The Syntax panel can be opened beside a fresh chapter without obscuring the editor.
- Searching `location`, `narrator`, `initiative`, or `image` finds the correct reference and insertion tool.
- Insert actions create valid parser-supported syntax at the cursor.
- Parameterised shortcuts use project characters, images, locations, dice tags, and notes rather than fragile free text.
- Parser behaviour, displayed reference syntax, and inserted syntax cannot drift apart.
- Custom shortcuts survive save/reload and can be reset without deleting project content.
- The area below the script editor contains no dense syntax paragraph and links directly to the full reference.

---

## 10. Import and understand `Combined S1-S4.txt`

### Source profile

Add a specialised import profile for this combined Wonderland transcript. Keep the generic TXT loader available so these rules do not break unrelated scripts.

Observed structure in the supplied file:

- 3,827 source lines.
- 57 filename-style chapter headings, for example `S1E1. The Rabbit Hole.txt` and `S4EP14. Hide & Counter DOUBLE FEATURE (Season Finale Parts 1 & 2).txt`.
- Chapter headings are surrounded by long `====` separator lines.
- Stylised Unicode speaker names and emoji are common.
- `-:` is the main narrator marker and must import as the reserved Narrator/Narration type.
- Dialogue commonly continues across later lines without repeating the speaker.
- Roll/system blocks include `Dice Maiden:`, decorated `ROLL`, `ATTACK`, `DAMAGE`, and `INITIATIVE` headings, dice expressions, AC, HP, saving throws, quests, and tooltips.
- The initiative block begins with `-: [Roll Initiative].` / `━━━ ⚡ INITIATIVE ━━━` and has no explicit exit marker before the next chapter.
- Some colon-containing lines are metadata or timestamps and must not become characters.

### Import flow

1. `Load TXT` or `Import as New Chapter` reads the file without immediately replacing any chapter.
2. Auto-detect the combined-transcript profile from the repeated separator + `.txt` heading pattern.
3. Show an import-review screen with detected chapters, characters, narrator lines, system/dice lines, warnings, and ignored lines.
4. Allow speaker mappings and exclusions before committing the import.
5. Choose the destination: replace current chapter, create one new chapter, or create all detected chapters.
6. Commit the normalised chapter operation as one undoable project action.

### Normalisation rules

- Convert each valid `*.txt` heading into a separate chapter object titled with the filename minus `.txt` when split-chapter import is selected.
- Only emit `:: Chapter: <title> ::` markers when producing a flattened compatibility/export script.
- Remove the surrounding `====` separator lines.
- Preserve the season/episode identifier as part of the chapter label.
- Treat `Narrator:`, `Narration:`, `-:`, and `null:` as Narrator.
- Narrator/Narration items must import without portrait requirements and use the full-width storybook reader layout.
- Attach unlabelled continuation lines to the most recent valid speaker or narrator item.
- Preserve blank-line paragraph breaks inside multiline dialogue where meaningful.
- Preserve Unicode names exactly; use a normalised comparison key only for matching aliases.
- Accept numeric labels such as `3:` as possible speakers.
- Reject obvious false speaker labels, including timestamps, bracket-only prefixes, URLs, and punctuation-only prefixes.
- Classify `Dice Maiden:` and decorated dice blocks as system/dice events by default, not cast characters.
- Convert a detected initiative start into `:: ⚔ Enter Initiative ::`.
- When the source provides no explicit initiative exit, close initiative at the next detected chapter boundary and report that inference in the review screen.
- Recognise common mechanic lines—rolls, advantage/disadvantage, attack, damage, AC, HP, saves, initiative order, quest, and tooltip—without rewriting their original text.
- Do not infer a battle merely because normal prose contains the words “battle”, “attack”, or “damage”.
- Preserve valid `:: Location: ... ::` directives as structured location state if they appear in a future import.

### Character review

- Group exact repeated labels automatically.
- Suggest aliases for spacing/decoration variants, but never merge them silently.
- Show counts beside each detected label.
- Allow `Create character`, `Map to existing`, `Treat as Narrator`, `Treat as System`, and `Ignore`.
- Do not create malformed characters such as `[` or Discord timestamp fragments.

### Acceptance criteria using the supplied file

- Exactly 57 chapters are detected and can be imported as 57 separately editable chapter workspaces.
- Separator lines do not appear as reader items.
- Unicode speaker names remain intact.
- Continuation dialogue remains with the correct preceding speaker.
- `-:` lines become the reserved Narrator type and never show a missing-portrait state.
- Dice Maiden and decorated roll blocks do not become ordinary characters by default.
- The initiative section is active after the initiative marker and closes at the following chapter boundary unless the review screen changes that choice.
- The import does not create timestamp or bracket-only characters.
- The user can review all uncertain mappings before the current project is replaced.
- Cancelling the import leaves the project untouched.
- `Import as New Chapter` never overwrites the active chapter.

---

## 11. Separate Light and Dark theme editing and add battle presets

### Problem

It is not sufficiently obvious which mode is being edited, colour selection can appear to do nothing, and un-applied colour choices can leak into the workspace without a clear preview/commit step. Page Theme and the Visual Novel Reader should normally stay connected, while Custom Reader Theme remains an intentional override.

### Light and Dark theme editing

- Replace ambiguous mode-test controls with separate `Dark mode theme` and `Light mode theme` editing tabs.
- Show a prominent `Editing Dark mode` or `Editing Light mode` banner beside the controls; do not rely on colour alone to communicate the active mode.
- Store independent page, dialogue, and battle-theme values for each mode.
- Switching tabs must preserve both the applied theme and any staged, un-applied edits in each mode.
- Changing the editor's current display mode may select the matching theme tab, but must never copy one mode's colours over the other.
- Keep Page Theme connected to the Visual Novel Reader by default.
- Enabling `Custom Reader Theme` explicitly disconnects the reader from Page Theme and exposes the reader-only controls; disabling it reconnects the reader immediately.
- Global theme speaker-name colours override a character-specific heading colour only when an explicit override toggle is enabled.
- Do not render a decorative dash, rule, or empty badge beside speaker names in the Visual Novel Reader.

### Explicit preview and commit flow

- Add a `Live preview theme edits` checkbox to the Page Theme editor.
- When Live preview is off, editing or choosing colours may update the contained colour sample/swatches, but must not recolour the workspace, Visual Novel Reader, or generated story preview.
- When Live preview is on, staged edits update the matching workspace/reader mode immediately without being saved as applied values.
- Keep explicit actions with text labels and icons:
  - `Preview` temporarily shows the staged theme when Live preview is off.
  - `Apply` commits the staged theme for the active Light/Dark mode.
  - `Reset` restores the built-in default for the active mode as a staged change.
  - `Cancel` discards staged changes and restores the last applied theme.
- Show a status message that distinguishes `Applied`, `Staged`, and `Previewing` states.
- Navigating away from staged changes must preserve them or ask before discarding them; it must not apply them silently.

### Battle-theme presets and palette rules

- Provide battle-theme presets for both Light and Dark modes, including Crimson, Arcane, Storm, and Ember.
- A preset fills the battle accent, surface, and text colours for the mode currently being edited; manual changes mark it as Custom.
- The reader uses the battle palette belonging to its active Page Theme mode during initiative.
- Start contrast at `0` for new Light and Dark themes.
- Put non-zero contrast behind an explicit `High Contrast mode` toggle; enabling it from zero may choose a readable starting strength, while disabling it returns the effective strength to zero.
- Preserve the appearance of legacy projects that already saved non-zero contrast by migrating them with High Contrast enabled.
- Include exact black (`#000000`) and white (`#ffffff`) swatches in every built-in palette compilation, including General, Mocha, Pastel, and Vibrant.
- Run readability checks before applying battle, reader, or semantic text combinations and identify the field that needs attention.

### Persistence and compatibility

- Project save/load and theme export/import must preserve both modes, the active editing mode, battle presets, High Contrast state, and staged/applied values where appropriate.
- Continue accepting legacy single-theme project/theme files; infer the closest Light or Dark destination and initialise the other mode from its default.
- Theme actions participate in dirty-state and auto-save rules only when values are applied, not merely previewed.

### Acceptance criteria

- The active editing mode is obvious from both text and the selected tab.
- Light and Dark page, dialogue, and battle colours remain independent across mode switches and reloads.
- With Live preview off, selecting a colour does not recolour the workspace or reader until Preview or Apply is chosen.
- Cancel restores the applied theme, and Reset affects only the active editing mode.
- Page Theme automatically colours the reader unless Custom Reader Theme is enabled.
- Each battle preset produces an appropriate Light and Dark variant and survives save/reload.
- New themes start at zero contrast; legacy non-zero themes keep their effective appearance.
- Every built-in palette contains exact black and white choices.

---

## 12. Complete keyboard navigation, text-to-speech, and interface clarity

### Keyboard-only navigation

- Make every application action reachable and operable without a mouse, including tabs, collapsible cards, colour controls, portrait assignment, crop/focal-point tools, chapter navigation, reader controls, syntax actions, modal dialogs, and resize handles.
- Use native buttons/inputs where possible, a logical tab order, visible focus, and correct accessible names and states.
- Workspace and theme tab groups should support Arrow keys, Home, and End as well as Tab/Shift+Tab.
- Every drag interaction must have a keyboard alternative: move/reorder buttons, arrow-key adjustment, or numeric fields as appropriate.
- Add a keyboard-shortcut reference reachable from the interface and by a documented shortcut.
- Provide shortcuts for high-frequency actions such as focus script, previous/next reader line, previous/next chapter, open Syntax, update icon-style preview, save, and stop speech.
- Do not trigger application shortcuts while the user is typing unless the shortcut includes a modifier and is safe inside editable fields.
- Detect shortcut conflicts, let optional shortcuts be reassigned or disabled, and provide Reset to defaults.
- When a dialog/drawer opens, move focus into it, contain focus appropriately, close with Escape where safe, and restore focus to the control that opened it.

### Per-character text-to-speech

- Put each character's text-to-speech voice, speed, and pitch in that character's options rather than treating them as reader-wide voice settings.
- Let different characters use different installed browser/system voices.
- Reader text-to-speech options control Narrator, narration, system/non-character dialogue, and the fallback used when a character has no valid assigned voice.
- Add `Read character names aloud` as an independent reader toggle.
- When name reading is off, speak only the dialogue text; when on, announce the displayed speaker name before the line.
- A disguise/display name should be spoken when names are enabled unless the user explicitly chooses the canonical name for speech.
- Keep Speak/Stop available from the reader and expose their current playing/disabled state accessibly.
- If a saved voice is no longer installed, show that it is unavailable and use the configured fallback without deleting the saved choice.
- Persist character and narration voice/rate/pitch settings in project save/export/import while treating browser voice identifiers as best-effort across devices.
- Do not auto-play speech on project load, chapter changes, or reader navigation unless a separate continuous-reading option is explicitly enabled.
- Add spoken-line autoplay as that explicit continuous-reading option, plus an optional advance-to-next-line action when speech finishes.
- Add `Always read character name in narrator voice`; when enabled, speak the name with narrator/fallback settings before the character voice reads dialogue.
- Add `Only read dialogue`; skip Narrator items and remove parenthesised thoughts and bracketed action/narration spans from character speech without changing source text.

### Clarify the two previews

- Rename the workspace `Preview` tab and card to `Icon-style Script Preview`.
- Explain that Icon-style Script Preview shows the generated/exported icon-style story page; it is not the Visual Novel Reader.
- Identify the Visual Novel Reader as the main interactive reading and line-testing tool.
- Rename generic actions such as `Update Preview` to `Update Icon-style Preview` wherever ambiguity remains.
- Use the same names in tooltips, accessible labels, help text, status messages, and keyboard-shortcut references.

### Optional example text in editable fields

- Add a `Show example text in empty fields` preference for editable inputs and textareas that currently contain examples.
- Example text must remain placeholder/help content only: it must never become a field value, validation input, saved project data, or generated output.
- Render visible examples at clearly lower opacity than entered text while maintaining minimum readable contrast.
- Hiding examples must not remove field labels, descriptions, accepted-format hints, or validation messages.
- Remember the preference locally and apply it consistently to dynamically created character, note, chapter, location, dice, and syntax fields.
- Do not use placeholder examples as the only accessible label or instruction for a field.

### Acceptance criteria

- The complete core workflow can be performed with a keyboard and always shows where focus is.
- Drag-only actions have keyboard equivalents and dialogs return focus correctly.
- Two characters can use different voices/rates/pitches while Narrator uses the reader fallback settings.
- Toggling `Read character names aloud` changes name announcement without changing the spoken dialogue.
- The interface never describes Icon-style Script Preview as the Visual Novel Reader.
- Example text can be hidden, is visibly distinct when shown, and is never saved as user content.

---

## Regression checklist

- Existing project JSON files still load.
- Legacy single-script projects migrate to a chapter collection without rewriting their content.
- Chapter creation, renaming, reordering, deletion, navigation, and import participate in save/dirty-state tracking.
- Import as New Chapter cannot replace or clear the active chapter.
- Chapter navigation remains usable with one chapter, 57 chapters, and 100+ chapters.
- Existing character aliases and portrait assignments still work.
- Canonical character, alias, typo-replacement, and disguise mappings survive save/reload and do not create duplicate characters.
- Script-label replacement remains undoable and never changes unconfirmed occurrences.
- Narrator/Narration never creates a character or portrait placeholder.
- Switching between narration and character dialogue correctly changes the reader layout without jumping position.
- Page Theme still automatically colours the Visual Novel Reader unless Custom Reader Theme is enabled.
- Switching the theme editor between Light and Dark preserves each mode's separate applied theme and staged edits.
- With Live preview off, theme selections remain staged until Preview or Apply is chosen.
- Mode-specific battle presets survive project save/reload and theme JSON export/import.
- New projects start with contrast at zero; High Contrast is opt-in; legacy non-zero contrast values keep their effective appearance.
- General, Mocha, Pastel, and Vibrant palettes each contain exact black and white swatches.
- Battle theme still overrides the normal reader theme during initiative.
- Custom reader, battle, narrator, thought, and action colours have a documented precedence order and remain readable.
- A colour-locked protagonist is skipped by all global/batch colour actions.
- Selected face crops and priority order survive save, reload, JSON export, and import.
- Location backgrounds and CornerRef images persist only until the next location change/clear.
- Existing dice tags, dice-character defaults, per-line overrides, and initiative behaviour survive the Advanced Tools → Dice Options migration.
- Renaming or deleting a roll type updates or safely resolves every saved reference after confirmation.
- Dice Options and reader-line dice controls expose the same roll types, modes, and effective override state.
- Reader pop-out, fullscreen, docking, and side-switching still work.
- Expanded portraits, selected faces, solid icons, and location art remain inside the active-speaker viewport and never cover the options bar, dialogue, resize divider, or dialogue-line list.
- Opening and closing Visual Novel options reflows the reader without covering the portrait/dialogue or leaving the collapsed summary in an old overlay position.
- The speaker/list divider works with mouse, touch, and keyboard; respects both sections' minimum heights; restores its saved position when space allows; and clamps safely in smaller reader layouts.
- Double-clicking the speaker/list divider and using Refresh layout both restore the default split without changing the current reader line.
- Script undo/redo treats a TXT import as one operation.
- Script undo/redo remains isolated per chapter.
- Syntax-panel insertions are one undo operation and never switch chapters unexpectedly.
- Collapsing the Syntax panel does not change editor cursor/scroll position.
- Standalone `Image` narration slots still work.
- Refreshing layout does not delete project data.
- Manual and auto-save modes work in both light and dark editor modes.
- Character-specific speech voices/rates/pitches and Narrator fallback settings survive project save/reload.
- Missing browser voices fall back safely without erasing the saved character assignment.
- Keyboard navigation works after dynamic rerenders, chapter changes, modal actions, and reader layout changes.
- Icon-style Script Preview and Visual Novel Reader remain clearly labelled as different tools.
- Hidden placeholder examples never become saved values or generated output.

## Suggested implementation split

### Patch A — layout and large-list stability

- Reader-line layout fix.
- Windowed reader-line rendering.
- Expanded reader-image containment and responsive image fitting.
- Bounded, persistent, and accessible active-speaker/dialogue-list divider.
- Toggle positioning.
- Closed-by-default layout preferences.
- Structured chapter model and legacy migration.
- Independent chapter editors, navigation, titles, and reader positions.
- Import as New Chapter.

### Patch B — persistence and assets

- IndexedDB project store.
- Manual/auto-save controls and dirty state.
- Separate Light/Dark page, dialogue, and battle-theme state with legacy single-theme migration.
- Mode-specific battle presets, opt-in High Contrast, and explicit live-preview state.
- Pasteable portrait gallery.
- Direct portrait assignment.
- Face crops and icon priority.
- Image-palette colour suggestions.
- Protagonist assignment and colour lock.
- Location image assets and persistent location state.

### Patch C — script intelligence

- Highlighted script view and speaker buttons.
- Speaker/Narrator/Narration/system classification.
- Canonical character, typo/alias mapping, and disguise/display-name syntax.
- Full-width storybook narration presentation.
- Thought, narration/action, and protagonist-specific semantic colours.
- Location directive parser, builder, and three reader display modes.
- Dice Options rename, user-friendly roll-type management, character defaults, and simplified initiative behaviour.
- Data-driven syntax registry.
- Syntax-reference side panel, cleaned inline help, builders, shortcuts, and assignments.
- Combined-TXT importer and review screen.
- Chapter, initiative, and dice/mechanics normalisation.

### Patch D — accessibility, speech, and interface clarity

- Complete keyboard-only navigation and non-drag alternatives.
- Shortcut reference, conflict handling, and focus management.
- Per-character voice/rate/pitch controls and Narrator/non-character fallback settings.
- Optional character-name announcements and safe missing-voice fallback.
- `Icon-style Script Preview` naming and Visual Novel Reader clarification.
- Optional, visually distinct example placeholders that never become project content.


