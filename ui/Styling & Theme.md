---
title: Styling & Theme
tags: [domain/ui, status/implemented]
status: implemented
sources: ["app/globals.css", "tailwind.config.ts", "components/styles/FullCalendar.css", "lib/utils.ts"]
updated: 2026-06-01
---

> **Status:** ✅ Implemented

# Styling & Theme

The project uses **Tailwind CSS v4** with a custom dark neon theme. v4 differs from v3: the CSS file uses `@import "tailwindcss"` and `@theme` to define design tokens, rather than `tailwind.config.ts` alone.

## Color Palette

Defined in `app/globals.css` (`@theme`) and duplicated in `tailwind.config.ts` (`theme.extend.colors`):

| Token | Hex | Name | Usage |
|-------|-----|------|-------|
| `gym-black` | `#121212` | Dark Background | Page/body background |
| `gym-dark` | `#0a0a0a` | Darker Background | Email template, deep cards |
| `gym-gray` | `#1e1e1e` | Card Surface | glass/glass-card base color |
| `gym-green` | `#ccff00` | Neon Lime | Brand accent: active states, CTA buttons, highlights |
| `gym-blue` | `#00f3ff` | Electric Blue | Secondary accent: icons, Revenue stat |
| `gym-purple` | `#9d00ff` | Accent Purple | Tertiary: Classes stat, progress bars |

**Verb pattern:** gym-green is the dominant accent — active nav items, selected calendar tiles, CTA buttons, progress bars, glow shadows. gym-blue and gym-purple appear on specific stat cards and icon accents.

## Glassmorphism Utilities

Defined as Tailwind v4 `@utility` in `app/globals.css`:

```css
@utility glass {
  @apply bg-gym-gray/40 backdrop-blur-lg border border-white/10 shadow-lg;
}

@utility glass-card {
  @apply bg-gym-gray/30 backdrop-blur-md border border-white/5 rounded-2xl shadow-xl;
}
```

`glass-card` is the primary card style — used by StatCard, RevenueChart, UpcomingClasses, and dropdown menus. `glass` is a heavier blur variant for overlay panels.

## Opacity Overlays

The dark theme uses semi-transparent white overlays extensively:
- Surfaces: `bg-white/5` (very subtle), `bg-white/10` (hover)
- Borders: `border-white/5`, `border-white/10`, `border-white/20`
- These create depth on the `gym-black` background without harsh color contrast

## Glow Shadows

Active navigation items and interactive elements use custom shadow utilities:
```
shadow-[0_0_20px_rgba(204,255,0,0.1)]   ← gym-green glow
```
Calendar selected tile: `box-shadow: 0 0 15px rgba(204, 255, 0, 0.3)`.

## The `cn()` Utility

**File:** `lib/utils.ts`
```ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```
Used in every component that has conditional class logic. `clsx` handles the conditional → string conversion; `tailwind-merge` deduplicates conflicting Tailwind classes (e.g. two `p-*` values).

## FullCalendar.css (react-calendar overrides)

**File:** `components/styles/FullCalendar.css`  
**Note:** The filename says "FullCalendar" but it overrides `.react-calendar` classes (the `react-calendar` npm package). Not related to the FullCalendar.io library.

Key overrides:
- Transparent background, no border, full-width layout
- Navigation buttons: white text, `min-width: 44px` touch targets
- Weekend days: `color: #ef4444` (red-500)
- Today tile: `background: rgba(204, 255, 0, 0.1)`, `color: #ccff00`, `border: 1px solid #ccff00`
- Selected tile: `background: #ccff00`, `color: black`, `font-weight: bold`, `box-shadow: 0 0 15px rgba(204, 255, 0, 0.3)`
- Hover selected: `#b3e600`

## Tailwind v4 Notes

The project has a dual-definition situation: colors are defined in both `globals.css` (`@theme`) and `tailwind.config.ts` (`theme.extend.colors`). This is redundant but harmless — the v4 `@theme` takes precedence for the v4 build pipeline, while the config file keeps compatibility hints.

`postcss.config.mjs` uses `@tailwindcss/postcss` (the v4 PostCSS plugin). Not the v3 `tailwindcss` postcss plugin.

## Mobile Touch Targets

Interactive elements in the header and sidebar are standardized at `h-11 w-11` (44px × 44px) to meet WCAG touch target guidelines. Calendar nav buttons are `min-height: 44px` in CSS.

## Related pages

- [[ui/Component Library]] — which components use glass-card, what colors each uses
- [[ui/Layouts & Navigation]] — Sidebar active state with glow
- [[reference/Conventions & Gotchas]] — the FullCalendar.css naming oddity
