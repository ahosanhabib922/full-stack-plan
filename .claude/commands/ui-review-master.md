---
description: UI Review Master — pixel-perfect visual audit comparing design intent to implementation across all screens
argument-hint: "[screen|component|all]"
---

# UI Review Master

Visual and UX quality audit. Compares the project's UI against design references (inspiration images, Figma specs, or PRD descriptions), flags deviations, scores quality, and fixes issues.

---

## Quick Reference

```bash
/ui-review-master              # Full UI audit of all screens
/ui-review-master home         # Audit screens matching "home"
/ui-review-master audit        # Audit only, no changes
/ui-review-master fix          # Apply fixes from last audit
```

---

## Phase 1: Gather Design References

Look for design references in this order:
1. `inspiration/` or `designs/` folder — image files (PNG, JPG, WebP)
2. `.claude-project/design-guide.*` or `design-system.*`
3. PRD file in root: `PRD.md`, `prd.md`, `README.md` with design section
4. Existing design tokens: `tailwind.config.*`, `theme.dart`, `tokens.json`

If no references found: run audit against UI/UX best practices only.

---

## Phase 2: Detect Framework

- `pubspec.yaml` → Flutter
- `next.config.*` or `app/` directory → Next.js / React
- Identify component library: shadcn/ui, MUI, Tailwind, custom

---

## Phase 3: Visual Audit

### 3.1 Layout & Spacing
- Consistent spacing scale used (8px grid or design system scale)
- No arbitrary pixel values that break the grid
- Proper padding/margin on all containers
- Content max-width respected on wide screens
- No content overflow or horizontal scroll on mobile

### 3.2 Typography
- Font families match design spec
- Font sizes follow a clear type scale (no random sizes)
- Line-height appropriate (1.4–1.6 for body, 1.1–1.3 for headings)
- Font weights consistent (not mixing 400/500/600/700 arbitrarily)
- Text contrast meets WCAG AA (4.5:1 body, 3:1 large text)

### 3.3 Color System
- Colors pulled from design tokens / theme, not hardcoded hex
- Primary, secondary, accent colors consistent across screens
- Surface/background colors consistent
- Dark mode: all colors have dark-mode variants if dark mode is supported
- No raw `Colors.red` or `#ff0000` in Flutter/React components

### 3.4 Component Consistency
- Buttons: same size, padding, border-radius across the app
- Inputs: consistent height, border, focus state
- Cards: consistent shadow, border-radius, padding
- Icons: single icon library used, consistent size (16/20/24px)
- Avatars: consistent size and shape

### 3.5 Responsive / Adaptive
- Mobile (360–480px): no overflow, touch targets ≥ 44px
- Tablet (768–1024px): layout adapts correctly
- Desktop (1280px+): no content stretches full-width awkwardly
- Flutter: `LayoutBuilder` / `MediaQuery` used for breakpoints

### 3.6 States & Feedback
- Loading states present (skeleton, spinner, shimmer)
- Empty states designed and implemented
- Error states present with user-friendly messages
- Hover/focus/active states on all interactive elements
- Disabled states styled distinctly

### 3.7 Animation & Motion
- Transitions are present on navigation and state changes
- Duration: 150–300ms for micro-interactions, 300–500ms for page transitions
- No jarring instant changes on visible UI elements
- Reduced motion respected if system preference set

### 3.8 Pixel-Perfect Check (if inspiration images provided)
Compare each screen against reference image:
- Header/hero matches layout
- Color palette matches
- Font treatment matches
- Spacing proportions match
- Component shapes (border-radius, shadows) match
- Image/illustration placement matches

---

## Phase 4: Score

Per screen:

```
Screen: Home Page
┌────────────────────────┬────────┬────────────────────────────────────┐
│ Category               │ Score  │ Issues                             │
├────────────────────────┼────────┼────────────────────────────────────┤
│ Layout & Spacing       │  85    │ Hero padding inconsistent on mobile │
│ Typography             │  90    │ -                                  │
│ Color System           │  70    │ 3 hardcoded hex colors found       │
│ Component Consistency  │  95    │ -                                  │
│ Responsive             │  80    │ Card overflows at 360px            │
│ States & Feedback      │  60    │ No loading state on data fetch     │
│ Animation              │  75    │ No page transition                 │
│ Pixel-Perfect Match    │  88    │ Button border-radius differs       │
└────────────────────────┴────────┴────────────────────────────────────┘
Screen Score: 80/100
```

---

## Phase 5: Fix (unless `audit` argument passed)

Apply fixes in priority order:

**P1 — Critical (fix always)**
- Hardcoded colors → replace with theme tokens
- Missing loading/error states → add skeleton or spinner
- Overflow/broken layout on mobile → fix responsive styles
- Touch targets < 44px → increase padding

**P2 — High (fix unless complex)**
- Inconsistent spacing → align to 8px grid
- Font size deviations → align to type scale
- Missing hover/focus states → add CSS/Flutter states
- Inconsistent border-radius → align to design system

**P3 — Medium (flag for manual review)**
- Animation missing or too fast/slow
- Pixel-perfect deviations > 8px
- Dark mode gaps

**Never change:**
- User-facing copy or labels
- Business logic
- API calls or data structures

---

## Phase 6: Report

```markdown
# UI Review Master Report — {date}

**Project**: {framework}
**Screens audited**: {N}
**Overall UI Score**: {X}/100

## Auto-Fixed
- [x] Replaced 7 hardcoded hex colors with theme tokens
- [x] Added loading skeleton to /dashboard data table
- [x] Fixed card overflow at 360px on /home
- [x] Added hover state to all CTA buttons

## Needs Manual Review
- [ ] /profile — avatar image crops differently than design reference
- [ ] /checkout — animation timing feels slow (500ms), consider 300ms
- [ ] Dark mode — /settings screen has 2 surfaces without dark variant

## Pixel-Perfect Deviations
| Screen    | Element          | Design  | Current | Delta |
|-----------|-----------------|---------|---------|-------|
| Home      | CTA button radius| 12px   | 8px     | -4px  |
| Product   | Card shadow      | 0 8 24 | 0 4 12  | diff  |
```

---

## Framework-Specific Checks

**Flutter**
- `ThemeData` used consistently, no `Theme.of(context).xxx` bypassed with hardcoded values
- `TextStyle` from theme, not inline
- `const` widgets used where possible for performance
- `Semantics` widget on interactive elements

**Next.js / React**
- Tailwind classes from design system palette only (`bg-primary` not `bg-[#3b82f6]`)
- CSS variables in `:root` for theme tokens
- `clsx` or `cn()` for conditional classes
- No inline `style={{}}` unless animation/dynamic value
