---
name: figma-market-screenshot-localizer
description: Create and revise localized Google Play or app-store market screenshot sections in Figma. Use when the user asks to copy an existing English screenshot section, translate it into another locale, keep the layout stable, apply locale-specific fonts, handle RTL localization, rename screenshot frames, fix component fonts, replace localized signature/name assets, or make follow-up edits to existing localized Figma screenshot sections.
---

# Figma Market Screenshot Localizer

## Core Rule

Use the `figma-use` skill before every Figma `use_figma` write or inspection call. Pass `skillNames: "figma-use"` in every `use_figma` call.

Operate directly in the Figma file when the user asks for creation or modification. Do not stop at a plan unless the user asks for one.

Only create or modify localized sections inside Figma. Do not export final PNG/JPEG screenshots unless the user explicitly asks for export; screenshots generated during QA are inspection artifacts only.

## Supported Locales

Default to these market screenshot locales when the user asks for localization without a custom locale list:

- Spanish: `es`
- Portuguese: `pt`
- Traditional Chinese: `zh-tw`
- Japanese: `ja`
- Korean: `ko`
- French: `fr`
- German: `de`
- Italian: `it`
- Thai: `th`
- Arabic: `ar`

When localizing multiple languages, create one duplicated section per locale from the English/source section. Do not translate on top of the original English/source section.

## Workflow

1. Locate the source section and existing localized sections.
2. Duplicate the English/source section for a new locale, or target the named existing locale section for follow-up edits.
3. Translate visible editable text nodes with concise app-store-safe copy, while preserving protected names and UI terms.
4. Load current fonts and target fonts before any text mutation.
5. Apply the locale font table below, using fallbacks only when the preferred font is unavailable in the file.
6. Preserve layout by keeping fixed frame sizes, shortening translations where needed, and checking dense areas such as banners, nav labels, file lists, document body text, and tool grids.
7. For RTL locales, follow the Arabic RTL stability rules below.
8. Run final QA:
   - count visible text nodes
   - check target font coverage
   - check likely untranslated editable text
   - generate Figma screenshots for the whole section and any risky individual frames
   - visually inspect the screenshots

## Locale Font Table

Use this table as the default font policy. Preserve weight/style as closely as available; if a requested family lacks the needed style, load the nearest available style and report the substitution.

| Locale | Preferred family | Fallbacks |
| --- | --- | --- |
| `es` | Source Latin font | `Noto Sans`, `Inter`, `Arial` |
| `pt` | Source Latin font | `Noto Sans`, `Inter`, `Arial` |
| `fr` | Source Latin font | `Noto Sans`, `Inter`, `Arial` |
| `de` | Source Latin font | `Noto Sans`, `Inter`, `Arial` |
| `it` | Source Latin font | `Noto Sans`, `Inter`, `Arial` |
| `zh-tw` | `Noto Sans TC` | `Noto Sans CJK TC`, `PingFang TC`, `Microsoft JhengHei` |
| `ja` | `Noto Sans JP` | `Noto Sans CJK JP`, `Hiragino Sans`, `Yu Gothic` |
| `ko` | `Noto Sans KR` | `Noto Sans CJK KR`, `Apple SD Gothic Neo`, `Malgun Gothic` |
| `th` | `Noto Sans Thai` | `Noto Sans`, `Tahoma` |
| `ar` | `Noto Sans Arabic` | `Noto Kufi Arabic`, `Arial` |

For CJK locales, prefer the Noto family even if the English/source section uses a different display font. For Latin-script locales, preserve the source Latin font unless it lacks locale glyph coverage or the user provides a different font rule.

## Translation Rules

- Keep product names, brand names, app names, company names, feature names, and branded UI terms in English by default.
- Preserve capitalization, spacing, and punctuation for protected English terms exactly unless the user provides a glossary that says otherwise.
- Translate descriptive marketing copy, generic UI labels, captions, benefits, and instructional text.
- Keep translations short enough for the existing text boxes; prioritize natural concise copy over literal length expansion.
- If a locale needs materially longer wording, shorten the copy first. Only resize text boxes or reduce font size after confirming the parent frame remains visually consistent.
- For ambiguous source text, infer the app-store screenshot intent from neighboring frames and existing visual hierarchy before translating.

## Protected Brand Lockups

- Treat app names, product names, logos, wordmarks, and combined brand lockups as protected by default unless the user explicitly asks to localize the brand.
- Preserve protected brand text exactly: characters, font family/style, size, line breaks, fills, effects, relative position, parent order, and visibility should match the source section.
- Do not replace a protected brand text node with a locale-font text node just to satisfy target font coverage. If the brand uses a missing or unavailable source font, keep the original copied node visible and report it as intentionally preserved.
- Do not count protected brand names as likely untranslated text during QA.
- In RTL locales, do not mirror or reorder logo/wordmark lockups themselves. Only move the surrounding semantic UI chrome when needed; the brand artwork/text remains visually unchanged.
- Before mutating text in a banner or app header, identify repeated brand strings such as the app name and exclude those nodes from translation/font replacement maps.
- For final QA, compare protected brand nodes in each localized section against the source: same visible text, same font segments where readable, same relative position within the same parent lockup, and no generated replacement node visible in its place.

## Protected Numeric Text

- Treat pure numbers and number-with-unit strings as protected layout text by default.
- Preserve protected numeric text metrics exactly: characters, font family/style, size, line breaks, fills, effects, width/height, and visibility should match the source section.
- Do not translate, localize date order, localize unit words, change numeral system, insert locale-specific spacing, or change fonts for protected numeric text unless the user explicitly asks for numeric localization.
- Do not replace protected numeric text with locale-font text nodes just to satisfy target font coverage. Keep the original copied node visible and report it as intentionally preserved.
- Do not count protected numeric text as likely untranslated text or as a target-font coverage issue during QA.
- Protected numeric text includes:
  - standalone numbers such as `75`, `200`, `20`, and `9:30`
  - numbers with storage or file units such as `296.7 MB`, `/ 256.0 GB`, `230 GB`, `256 GB`, and `118 GB`
  - dates and timestamps such as `5/21/26`, `5/12/26`, and `19-04 15:44`
  - compact values embedded in labels when the value is the layout-critical part, such as `230 GB Used` or `256 GB Total`
- Before mutating text in meters, charts, counters, badges, storage cards, progress rows, dates, timestamps, and file-size labels, identify and exclude protected numeric nodes from translation/font replacement maps.
- For RTL locales, protected numeric nodes may move as part of row-level mirroring, but their text content, numeral order, unit string, font, and box metrics stay source-matched.
- For final QA, compare protected numeric nodes in each localized section against the source for content/font/metrics and verify that no generated replacement node is visible in their place.

## Protected Icon Glyph Text

- Treat icon-only text nodes as protected layout glyphs by default, including SF Symbols, icon-font glyphs, emoji-like symbols, and private-use characters in both BMP and supplementary private-use Unicode ranges.
- Do not translate, rename, font-replace, resize, or count icon-only text nodes as untranslated copy.
- Detect icon-only text by checking whether the trimmed string contains no locale text, Latin letters, Arabic letters, CJK characters, or digits; do not rely only on the `U+E000-F8FF` private-use range because some SF Symbols live outside it.
- In RTL locales, icon-only text nodes may move when their parent semantic group is mirrored, but their glyph text, font, size, and box metrics stay source-matched.

## Arabic RTL Stability Rules

- Adjust the frame order inside the localized section for Arabic reading direction: the first screenshot/page should be the rightmost frame, followed by the remaining frames from right to left.
- Treat Arabic users' visual scan as starting at the top-right. Rebuild top bars, section headers, card groups, and repeated rows around a right-side start edge, then continue right-to-left across each row.
- Do not mirror phone frames, device mockups, raster screenshots, logos, centered hero titles, centered buttons, or background artwork.
- Mirror only semantic horizontal UI groups: top app bars, nav rows, toolbar rows, icon-and-label rows, quick-action/tool card grids, app-list rows, file-list rows, settings rows, notification rows, tabs, bottom navigation/tab bars, segmented controls, and left/right paired controls.
- For every repeated list/table row, do a full row-level RTL pass, not just text alignment: leading content icons move to the right, primary labels sit to their left and right-align, and trailing status/action affordances such as locks, trash icons, checkmarks, timestamps, or badges move to the left when they are part of the row's left/right information structure.
- For home/dashboard screens, explicitly mirror the red-box-prone chrome and grids: swap top app bar leading/trailing actions such as menu, add, grid, or overflow icons; reverse two-column tool-card grids row by row; and reverse visible bottom navigation/tab items so the first/primary tab sits on the right and the last tab sits on the left. Keep genuinely centered controls centered.
- Before changing `x` positions, inspect `layoutMode`. For `HORIZONTAL` auto-layout rows and bottom navigation, reverse child order with `insertChild` instead of setting `x`, because auto-layout will overwrite manual coordinates. For `GRID` containers, reorder children row by row. Use x-coordinate recalculation only for absolute-positioned frames/groups.
- Prefer reversing child order or recalculating x positions inside a bounded parent. Do not use negative scale transforms.
- Keep every parent frame size fixed. Do not let Arabic text resize containers horizontally.
- For text, use `Noto Sans Arabic`, right-align text that was originally left-aligned, and preserve center alignment for originally centered text.
- For mixed Arabic plus Latin/CJK text, keep embedded Latin/CJK substrings in their natural LTR order, but align the containing text node to the right unless it was originally centered.
- Arabic has no uppercase/lowercase distinction and can look visually lighter than all-caps Latin. For titles, labels, and buttons translated from uppercase or heavy Latin, check visual weight after translation and adjust weight or size slightly only if it improves parity without breaking the fixed frame.
- Do not add positive letter spacing to Arabic text. Arabic letterforms connect within words, and extra tracking can break the script.
- Swap left/right padding, constraints, and alignment only within the mirrored group.
- Directional icons such as back/next chevrons, transport direction, and reading-direction glyphs such as text lines or tree structures should be swapped or mirrored. Real-world object icons, character-letter icons, media playback icons/progress, round clocks, and prohibition slashes generally stay unmirrored unless their local meaning clearly depends on left/right direction.
- Avoid mixed Arabic and Latin file extensions in one text node. Use extension-free Arabic names or separate the extension if needed.
- For progress bars, loaders, pagination dots, carousels, timelines, calendars, skeleton loaders, and image sequences, mirror the visual order/direction so the first or newest item starts from the right. Preserve continuous numeric values and their digit order.
- For banners, posters, and feature graphics, do not flip the underlying raster artwork. Reposition text blocks, icon-label rows, and CTA/action buttons for RTL instead.
- For document/editor screenshots, set paragraph/text content direction to RTL and mirror toolbar order, including undo/redo placement.
- If a foreground illustration or device artwork intentionally covers part of a decorative list, do not blindly mirror text into the covered area. Shorten the Arabic copy and constrain the text box to the visible region while keeping the parent frame fixed.
- Avoid culturally sensitive imagery in Arabic/MENA screenshots when it is not essential to the product meaning; for example, replace piggy-bank-style savings icons with neutral finance/storage metaphors.
- Make the RTL operation idempotent: do not mirror an existing Arabic section twice.
- After RTL changes, screenshot the whole section and risky dense frames, then check overflow, overlap, wrong order, accidental mirrored assets, every dense repeated list row's icon/text/status order from right to left, top app bar leading/trailing action placement, tool-card grid column order, and bottom navigation/tab order.

## Follow-Up Edits

For existing localized sections, make the smallest targeted change possible:

- Rename screenshot frames by locating the section banner and renaming the seven same-row frames after it to `01` through `07`. For Arabic/RTL sections, sort by x position descending so `01` is the rightmost frame.
- Change a repeated string by exact text match across the page or across matching sections.
- Fix fonts inside a named component or instance by finding text nodes whose ancestor has that name.
- Replace a signature or localized visual asset by hiding the original image node and placing a localized text/vector replacement in the same visual area.
- After edits, clear selection where possible and screenshot the affected frame.

## Figma API Practices

- Use top-level `await` and `return` in `use_figma` code.
- Switch page with `await figma.setCurrentPageAsync(page)`.
- Return `createdNodeIds` and `mutatedNodeIds` for write operations.
- Do not call `figma.notify()` or `figma.closePlugin()`.
- Load every current text font and every target font before setting `characters`, `fontName`, or other text properties.
- Do not delete source sections. Duplicate from the English/source section for new locales.
- Prefer exact matching by section name, frame name, node ID, text content, or ancestor name. Avoid broad mutations unless the user explicitly asks for page-wide changes.
