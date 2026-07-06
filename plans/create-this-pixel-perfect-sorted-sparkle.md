# Plan: Render the Evil Dead: The Musical Figma import pixel-perfect

## Context
The user imported a complete Figma frame — a single-page **Evil Dead: The Musical** landing page (`DektopPrimary1728Min`). The import is fully self-contained: every photo/logo/subtitle is a PNG living in `/src/imports/DektopPrimary1728Min/`, referenced via relative imports, plus one SVG path file (`svg-ytn66h3c0n.ts`) for a decorative triangle. The default export `DektopPrimary1728Min()` already composes the entire page (TopBar → Hero → Seats → Join → About → Victims → Critics → Playlist → Gallery → FinalCta → Credits).

The app currently renders an empty placeholder (`src/app/App.tsx`). The goal is to display this design faithfully ("pixel perfect"). There is **no design system** (`@make-kits`) in this project, so no component substitution is required.

## Approach
Faithfully render the existing import — do **not** rewrite it. Per the design-imports rules, `/src/imports/**` is read-only; adaptation happens in the app layer.

### 1. Wire the import into the app
- Edit `src/app/App.tsx` to import and render the import's default export.
- Because the design is a fixed-width (1728px) desktop composition full of hard pixel dimensions and wide bleed elements (e.g. photo stripes at `w-[3772px]`), keep it at native width and let it scroll/scale rather than restructuring — restructuring would break pixel fidelity. Wrap it so it centers and scrolls cleanly:

```tsx
import DektopPrimary1728Min from "../imports/DektopPrimary1728Min";

export default function App() {
  return (
    <div className="min-h-screen w-full bg-black overflow-x-auto">
      <div className="relative w-[1728px] mx-auto">
        <DektopPrimary1728Min />
      </div>
    </div>
  );
}
```
(Confirm the exact import path/module name resolves — the folder is `src/imports/DektopPrimary1728Min/index.tsx`, so `../imports/DektopPrimary1728Min` from `src/app/App.tsx` is correct.)

### 2. Fonts (user is uploading the files)
The design's live (non-image) text uses three custom fonts, all flagged as blocked/not-in-catalog. The user chose to **upload the font files**. Once uploaded:
- Add `@font-face` declarations to the **top of `src/styles/fonts.css`** (the only permitted place for font imports), using family names that exactly match what the import references so no code changes are needed in `imports/`:
  - `VHS Gothic` — nav links + body copy
  - `Unthinkers Slant DEMO` — red section headings
  - `Transcend` (Bold weight) — critic pull-quotes
- The import's Tailwind classes reference these as `font-['VHS_Gothic:Regular',sans-serif]`, `font-['Unthinkers_Slant_DEMO:Regular',sans-serif]`, and `font-['Transcend:Bold',sans-serif]`. The `Family:Style` token resolves to the CSS family name (`VHS Gothic`, etc.), so matching the `@font-face` `font-family` to that base name is sufficient.
- Until files land, text falls back to sans-serif (imagery is unaffected and remains pixel-perfect).

### 3. Assets & SVG
- No changes needed — the import already imports its PNGs relatively and its SVG via `svg-ytn66h3c0n`. Do not redraw or re-wire any assets.

## Files
- **Modify:** `src/app/App.tsx` — render the import inside a centered, scrollable wrapper.
- **Modify (after upload):** `src/styles/fonts.css` — `@font-face` for the three families.
- **Do not touch:** anything under `src/imports/`.

## Verification
- Load the app in the preview surface; confirm the full page renders top-to-bottom (hero with sticker/logo/subtitles, three seat cards, sign-up, About section with quote triangle, Victims photo grid with red glowing borders, Critics image with quotes, Spotify playlist, Gallery photo stripes, Final CTA, Credits).
- Confirm all PNGs and the decorative triangle SVG appear (no broken images).
- After fonts are uploaded: confirm nav/body render in VHS Gothic, red headings in Unthinkers Slant, and critic quotes in Transcend Bold.
- Horizontal scroll should reveal the full 1728px canvas with no layout collapse; background is black.
