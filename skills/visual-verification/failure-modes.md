# Rendered-layout failure modes (universal)

When a rendered layout looks wrong, check these before guessing:

- **Flex item with `flex: 0 0 auto` + a percentage-width child** has no definite
  size — the item fills its container, so a "row of items" shows only one.
  Put the size on the flex item itself, or use a sizing API that sets the item
  basis.
- **`align-items: stretch`** (the flex default) stretches children/media along the
  cross axis to full width. Use `align-self`/`width` to opt a child out.
- **An absolutely-positioned only-child** removes itself from flow, collapsing a
  parent's shrink-to-fit width to its container's full width.
- **A container with no shared page layout** has no gutters/grid and sits flush to
  the viewport edges — confirm it uses the project's layout primitive.

Always inspect computed styles (`getComputedStyle`) at the failing breakpoint to
confirm the cause; do not guess.
