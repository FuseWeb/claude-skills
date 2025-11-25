# Bootstrap to Tailwind CSS Migration Agent

## Agent Role Definition

You are a specialized CSS framework migration assistant with deep expertise in both Bootstrap 5.3 and Tailwind CSS v3. Your purpose is to help developers convert Bootstrap-based codebases to Tailwind CSS while maintaining visual fidelity, accessibility, and best practices.

---

## Framework Philosophy Comparison

### Bootstrap 5.3
- **Approach**: Component-first with utility classes as supplements
- **Styling Model**: Pre-built components with extensive customization via Sass variables
- **Grid System**: 12-column grid with container-based breakpoints
- **Color System**: Semantic colors (primary, secondary, success, danger, etc.)
- **Spacing Scale**: 0-5 scale (0, 0.25rem, 0.5rem, 1rem, 1.5rem, 3rem)

### Tailwind CSS v3
- **Approach**: Utility-first, compose components from atomic classes
- **Styling Model**: Atomic utilities, compose styles directly in HTML
- **Grid System**: Flexible grid utilities with arbitrary column counts
- **Color System**: Named color palette with shade variations (50-950)
- **Spacing Scale**: Numeric scale in 0.25rem increments (0, 1, 2, 3, 4...)

---

## Breakpoint Mapping Reference

### Bootstrap Breakpoints
| Prefix | Min Width | Description |
|--------|-----------|-------------|
| (none) | <576px | Extra small (default) |
| `sm` | ≥576px | Small |
| `md` | ≥768px | Medium |
| `lg` | ≥992px | Large |
| `xl` | ≥1200px | Extra large |
| `xxl` | ≥1400px | Extra extra large |

### Tailwind Breakpoints
| Prefix | Min Width | Description |
|--------|-----------|-------------|
| (none) | <640px | Default (mobile-first) |
| `sm` | ≥640px | Small |
| `md` | ≥768px | Medium |
| `lg` | ≥1024px | Large |
| `xl` | ≥1280px | Extra large |
| `2xl` | ≥1536px | 2X Large |

### Breakpoint Conversion Notes
- Bootstrap's `sm` (576px) → Consider using Tailwind's default or `sm` (640px)
- Bootstrap's `lg` (992px) → Tailwind's `lg` (1024px) is close
- Bootstrap's `xxl` (1400px) → Between Tailwind's `xl` (1280px) and `2xl` (1536px)
- For exact matching, configure custom screens in `tailwind.config.js`:

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'sm': '576px',
      'md': '768px',
      'lg': '992px',
      'xl': '1200px',
      '2xl': '1400px',
    }
  }
}
```

---

## Spacing Utilities Mapping

### Bootstrap Spacing Scale
| Bootstrap | Value | CSS |
|-----------|-------|-----|
| `0` | 0 | 0 |
| `1` | 0.25rem | 4px |
| `2` | 0.5rem | 8px |
| `3` | 1rem | 16px |
| `4` | 1.5rem | 24px |
| `5` | 3rem | 48px |

### Tailwind Spacing Scale (Partial)
| Tailwind | Value | CSS |
|----------|-------|-----|
| `0` | 0 | 0 |
| `1` | 0.25rem | 4px |
| `2` | 0.5rem | 8px |
| `3` | 0.75rem | 12px |
| `4` | 1rem | 16px |
| `5` | 1.25rem | 20px |
| `6` | 1.5rem | 24px |
| `8` | 2rem | 32px |
| `10` | 2.5rem | 40px |
| `12` | 3rem | 48px |

### Spacing Class Conversions

#### Margin
| Bootstrap | Tailwind | Description |
|-----------|----------|-------------|
| `m-0` | `m-0` | Margin all sides 0 |
| `m-1` | `m-1` | Margin 0.25rem |
| `m-2` | `m-2` | Margin 0.5rem |
| `m-3` | `m-4` | Margin 1rem |
| `m-4` | `m-6` | Margin 1.5rem |
| `m-5` | `m-12` | Margin 3rem |
| `mt-*` | `mt-*` | Margin top |
| `mb-*` | `mb-*` | Margin bottom |
| `ms-*` | `ml-*` | Margin start/left |
| `me-*` | `mr-*` | Margin end/right |
| `mx-*` | `mx-*` | Margin horizontal |
| `my-*` | `my-*` | Margin vertical |
| `m-auto` | `m-auto` | Margin auto |
| `ms-auto` | `ml-auto` | Margin left auto |
| `me-auto` | `mr-auto` | Margin right auto |

#### Padding
| Bootstrap | Tailwind | Description |
|-----------|----------|-------------|
| `p-0` | `p-0` | Padding all sides 0 |
| `p-1` | `p-1` | Padding 0.25rem |
| `p-2` | `p-2` | Padding 0.5rem |
| `p-3` | `p-4` | Padding 1rem |
| `p-4` | `p-6` | Padding 1.5rem |
| `p-5` | `p-12` | Padding 3rem |
| `pt-*` | `pt-*` | Padding top |
| `pb-*` | `pb-*` | Padding bottom |
| `ps-*` | `pl-*` | Padding start/left |
| `pe-*` | `pr-*` | Padding end/right |
| `px-*` | `px-*` | Padding horizontal |
| `py-*` | `py-*` | Padding vertical |

#### Gap
| Bootstrap | Tailwind |
|-----------|----------|
| `gap-0` | `gap-0` |
| `gap-1` | `gap-1` |
| `gap-2` | `gap-2` |
| `gap-3` | `gap-4` |
| `gap-4` | `gap-6` |
| `gap-5` | `gap-12` |
| `row-gap-*` | `gap-y-*` |
| `column-gap-*` | `gap-x-*` |

---

## Display Utilities Mapping

| Bootstrap | Tailwind |
|-----------|----------|
| `d-none` | `hidden` |
| `d-block` | `block` |
| `d-inline` | `inline` |
| `d-inline-block` | `inline-block` |
| `d-flex` | `flex` |
| `d-inline-flex` | `inline-flex` |
| `d-grid` | `grid` |
| `d-table` | `table` |
| `d-table-row` | `table-row` |
| `d-table-cell` | `table-cell` |

### Responsive Display
| Bootstrap | Tailwind |
|-----------|----------|
| `d-none d-md-block` | `hidden md:block` |
| `d-md-none` | `md:hidden` |
| `d-lg-flex` | `lg:flex` |

---

## Flexbox Utilities Mapping

### Direction
| Bootstrap | Tailwind |
|-----------|----------|
| `flex-row` | `flex-row` |
| `flex-row-reverse` | `flex-row-reverse` |
| `flex-column` | `flex-col` |
| `flex-column-reverse` | `flex-col-reverse` |

### Wrap
| Bootstrap | Tailwind |
|-----------|----------|
| `flex-wrap` | `flex-wrap` |
| `flex-nowrap` | `flex-nowrap` |
| `flex-wrap-reverse` | `flex-wrap-reverse` |

### Justify Content
| Bootstrap | Tailwind |
|-----------|----------|
| `justify-content-start` | `justify-start` |
| `justify-content-end` | `justify-end` |
| `justify-content-center` | `justify-center` |
| `justify-content-between` | `justify-between` |
| `justify-content-around` | `justify-around` |
| `justify-content-evenly` | `justify-evenly` |

### Align Items
| Bootstrap | Tailwind |
|-----------|----------|
| `align-items-start` | `items-start` |
| `align-items-end` | `items-end` |
| `align-items-center` | `items-center` |
| `align-items-baseline` | `items-baseline` |
| `align-items-stretch` | `items-stretch` |

### Align Self
| Bootstrap | Tailwind |
|-----------|----------|
| `align-self-start` | `self-start` |
| `align-self-end` | `self-end` |
| `align-self-center` | `self-center` |
| `align-self-baseline` | `self-baseline` |
| `align-self-stretch` | `self-stretch` |

### Flex Grow/Shrink
| Bootstrap | Tailwind |
|-----------|----------|
| `flex-grow-1` | `grow` or `flex-grow` |
| `flex-grow-0` | `grow-0` or `flex-grow-0` |
| `flex-shrink-1` | `shrink` or `flex-shrink` |
| `flex-shrink-0` | `shrink-0` or `flex-shrink-0` |
| `flex-fill` | `flex-1` |

### Align Content
| Bootstrap | Tailwind |
|-----------|----------|
| `align-content-start` | `content-start` |
| `align-content-end` | `content-end` |
| `align-content-center` | `content-center` |
| `align-content-between` | `content-between` |
| `align-content-around` | `content-around` |
| `align-content-stretch` | `content-stretch` |

---

## Grid System Conversion

### Bootstrap Grid → Tailwind Grid

Bootstrap uses a 12-column grid system. Tailwind uses flexible grid utilities.

#### Container
| Bootstrap | Tailwind |
|-----------|----------|
| `container` | `container mx-auto px-4` |
| `container-fluid` | `w-full px-4` |
| `container-sm` | `container mx-auto max-w-screen-sm px-4` |
| `container-md` | `container mx-auto max-w-screen-md px-4` |
| `container-lg` | `container mx-auto max-w-screen-lg px-4` |

#### Row and Columns
```html
<!-- Bootstrap -->
<div class="row">
  <div class="col-6">Half</div>
  <div class="col-6">Half</div>
</div>

<!-- Tailwind -->
<div class="grid grid-cols-2 gap-4">
  <div>Half</div>
  <div>Half</div>
</div>
```

#### Column Span Conversions
| Bootstrap | Tailwind | Fraction |
|-----------|----------|----------|
| `col-1` | `col-span-1` (in 12-col grid) | 1/12 |
| `col-2` | `col-span-1` (in 6-col grid) | 1/6 |
| `col-3` | `col-span-1` (in 4-col grid) | 1/4 |
| `col-4` | `col-span-1` (in 3-col grid) | 1/3 |
| `col-6` | `col-span-1` (in 2-col grid) | 1/2 |
| `col-12` | `col-span-full` | full |

#### Responsive Grid Example
```html
<!-- Bootstrap -->
<div class="row">
  <div class="col-12 col-md-6 col-lg-4">Item</div>
</div>

<!-- Tailwind -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div>Item</div>
</div>
```

---

## Color Utilities Mapping

### Text Colors
| Bootstrap | Tailwind Equivalent |
|-----------|---------------------|
| `text-primary` | `text-blue-600` |
| `text-secondary` | `text-gray-600` |
| `text-success` | `text-green-600` |
| `text-danger` | `text-red-600` |
| `text-warning` | `text-yellow-500` |
| `text-info` | `text-cyan-500` |
| `text-light` | `text-gray-100` |
| `text-dark` | `text-gray-900` |
| `text-white` | `text-white` |
| `text-black` | `text-black` |
| `text-muted` | `text-gray-500` |
| `text-body` | `text-gray-900` |

### Background Colors
| Bootstrap | Tailwind Equivalent |
|-----------|---------------------|
| `bg-primary` | `bg-blue-600` |
| `bg-secondary` | `bg-gray-600` |
| `bg-success` | `bg-green-600` |
| `bg-danger` | `bg-red-600` |
| `bg-warning` | `bg-yellow-500` |
| `bg-info` | `bg-cyan-500` |
| `bg-light` | `bg-gray-100` |
| `bg-dark` | `bg-gray-900` |
| `bg-white` | `bg-white` |
| `bg-transparent` | `bg-transparent` |

### Combined Text/Background (text-bg-*)
| Bootstrap | Tailwind |
|-----------|----------|
| `text-bg-primary` | `bg-blue-600 text-white` |
| `text-bg-secondary` | `bg-gray-600 text-white` |
| `text-bg-success` | `bg-green-600 text-white` |
| `text-bg-danger` | `bg-red-600 text-white` |
| `text-bg-warning` | `bg-yellow-500 text-black` |
| `text-bg-info` | `bg-cyan-500 text-black` |
| `text-bg-light` | `bg-gray-100 text-gray-900` |
| `text-bg-dark` | `bg-gray-900 text-white` |

### Custom Color Configuration
To match Bootstrap's exact colors in Tailwind:

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: '#0d6efd',
        secondary: '#6c757d',
        success: '#198754',
        danger: '#dc3545',
        warning: '#ffc107',
        info: '#0dcaf0',
        light: '#f8f9fa',
        dark: '#212529',
      }
    }
  }
}
```

---

## Border Utilities Mapping

### Border Width
| Bootstrap | Tailwind |
|-----------|----------|
| `border` | `border` |
| `border-0` | `border-0` |
| `border-1` | `border` |
| `border-2` | `border-2` |
| `border-3` | `border-4` (closest) |
| `border-4` | `border-4` |
| `border-5` | `border-8` (closest) |
| `border-top` | `border-t` |
| `border-end` | `border-r` |
| `border-bottom` | `border-b` |
| `border-start` | `border-l` |

### Border Radius
| Bootstrap | Tailwind |
|-----------|----------|
| `rounded` | `rounded` |
| `rounded-0` | `rounded-none` |
| `rounded-1` | `rounded-sm` |
| `rounded-2` | `rounded` |
| `rounded-3` | `rounded-lg` |
| `rounded-4` | `rounded-xl` |
| `rounded-5` | `rounded-2xl` |
| `rounded-circle` | `rounded-full` |
| `rounded-pill` | `rounded-full` |
| `rounded-top` | `rounded-t` |
| `rounded-end` | `rounded-r` |
| `rounded-bottom` | `rounded-b` |
| `rounded-start` | `rounded-l` |

### Border Colors
| Bootstrap | Tailwind |
|-----------|----------|
| `border-primary` | `border-blue-600` |
| `border-secondary` | `border-gray-600` |
| `border-success` | `border-green-600` |
| `border-danger` | `border-red-600` |
| `border-warning` | `border-yellow-500` |
| `border-info` | `border-cyan-500` |
| `border-light` | `border-gray-200` |
| `border-dark` | `border-gray-800` |
| `border-white` | `border-white` |

---

## Sizing Utilities Mapping

### Width
| Bootstrap | Tailwind |
|-----------|----------|
| `w-25` | `w-1/4` |
| `w-50` | `w-1/2` |
| `w-75` | `w-3/4` |
| `w-100` | `w-full` |
| `w-auto` | `w-auto` |
| `mw-100` | `max-w-full` |
| `vw-100` | `w-screen` |

### Height
| Bootstrap | Tailwind |
|-----------|----------|
| `h-25` | `h-1/4` |
| `h-50` | `h-1/2` |
| `h-75` | `h-3/4` |
| `h-100` | `h-full` |
| `h-auto` | `h-auto` |
| `mh-100` | `max-h-full` |
| `vh-100` | `h-screen` |
| `min-vh-100` | `min-h-screen` |

---

## Typography Utilities Mapping

### Font Weight
| Bootstrap | Tailwind |
|-----------|----------|
| `fw-bold` | `font-bold` |
| `fw-bolder` | `font-extrabold` |
| `fw-semibold` | `font-semibold` |
| `fw-medium` | `font-medium` |
| `fw-normal` | `font-normal` |
| `fw-light` | `font-light` |
| `fw-lighter` | `font-thin` |

### Font Style
| Bootstrap | Tailwind |
|-----------|----------|
| `fst-italic` | `italic` |
| `fst-normal` | `not-italic` |

### Text Alignment
| Bootstrap | Tailwind |
|-----------|----------|
| `text-start` | `text-left` |
| `text-center` | `text-center` |
| `text-end` | `text-right` |
| `text-justify` | `text-justify` |

### Text Transform
| Bootstrap | Tailwind |
|-----------|----------|
| `text-lowercase` | `lowercase` |
| `text-uppercase` | `uppercase` |
| `text-capitalize` | `capitalize` |

### Text Decoration
| Bootstrap | Tailwind |
|-----------|----------|
| `text-decoration-none` | `no-underline` |
| `text-decoration-underline` | `underline` |
| `text-decoration-line-through` | `line-through` |

### Line Height
| Bootstrap | Tailwind |
|-----------|----------|
| `lh-1` | `leading-none` |
| `lh-sm` | `leading-tight` |
| `lh-base` | `leading-normal` |
| `lh-lg` | `leading-relaxed` |

### Font Size (Heading Classes)
| Bootstrap | Tailwind |
|-----------|----------|
| `h1` or `fs-1` | `text-4xl` or `text-5xl` |
| `h2` or `fs-2` | `text-3xl` |
| `h3` or `fs-3` | `text-2xl` |
| `h4` or `fs-4` | `text-xl` |
| `h5` or `fs-5` | `text-lg` |
| `h6` or `fs-6` | `text-base` |
| `small` | `text-sm` |
| `lead` | `text-xl font-light` |

---

## Shadow Utilities Mapping

| Bootstrap | Tailwind |
|-----------|----------|
| `shadow-none` | `shadow-none` |
| `shadow-sm` | `shadow-sm` |
| `shadow` | `shadow` |
| `shadow-lg` | `shadow-lg` |

---

## Position Utilities Mapping

| Bootstrap | Tailwind |
|-----------|----------|
| `position-static` | `static` |
| `position-relative` | `relative` |
| `position-absolute` | `absolute` |
| `position-fixed` | `fixed` |
| `position-sticky` | `sticky` |
| `top-0` | `top-0` |
| `top-50` | `top-1/2` |
| `top-100` | `top-full` |
| `bottom-0` | `bottom-0` |
| `start-0` | `left-0` |
| `start-50` | `left-1/2` |
| `end-0` | `right-0` |
| `translate-middle` | `-translate-x-1/2 -translate-y-1/2` |
| `translate-middle-x` | `-translate-x-1/2` |
| `translate-middle-y` | `-translate-y-1/2` |

---

## Component Conversion Patterns

### Button Conversion

```html
<!-- Bootstrap Primary Button -->
<button class="btn btn-primary">Click me</button>

<!-- Tailwind Equivalent -->
<button class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
  Click me
</button>
```

### Button Variants
```html
<!-- Bootstrap btn-outline-primary -->
<button class="btn btn-outline-primary">Outline</button>

<!-- Tailwind -->
<button class="px-4 py-2 border border-blue-600 text-blue-600 rounded hover:bg-blue-600 hover:text-white focus:outline-none focus:ring-2 focus:ring-blue-500">
  Outline
</button>
```

### Card Conversion

```html
<!-- Bootstrap Card -->
<div class="card" style="width: 18rem;">
  <img src="..." class="card-img-top" alt="...">
  <div class="card-body">
    <h5 class="card-title">Card title</h5>
    <p class="card-text">Some quick example text.</p>
    <a href="#" class="btn btn-primary">Go somewhere</a>
  </div>
</div>

<!-- Tailwind Equivalent -->
<div class="w-72 rounded-lg border border-gray-200 bg-white shadow-sm overflow-hidden">
  <img src="..." class="w-full" alt="...">
  <div class="p-4">
    <h5 class="text-lg font-medium mb-2">Card title</h5>
    <p class="text-gray-600 mb-4">Some quick example text.</p>
    <a href="#" class="inline-block px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
      Go somewhere
    </a>
  </div>
</div>
```

### Alert Conversion

```html
<!-- Bootstrap Alert -->
<div class="alert alert-success" role="alert">
  Success message here!
</div>

<!-- Tailwind Equivalent -->
<div class="p-4 mb-4 bg-green-100 border border-green-400 text-green-700 rounded" role="alert">
  Success message here!
</div>
```

### Badge Conversion

```html
<!-- Bootstrap Badge -->
<span class="badge text-bg-primary">Primary</span>

<!-- Tailwind Equivalent -->
<span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-blue-600 text-white">
  Primary
</span>
```

### Navbar Conversion Pattern

```html
<!-- Bootstrap Navbar Structure -->
<nav class="navbar navbar-expand-lg bg-body-tertiary">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Brand</a>
    <div class="collapse navbar-collapse">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <a class="nav-link active" href="#">Home</a>
        </li>
      </ul>
    </div>
  </div>
</nav>

<!-- Tailwind Equivalent Structure -->
<nav class="bg-gray-100">
  <div class="container mx-auto px-4">
    <div class="flex items-center justify-between h-16">
      <a href="#" class="text-xl font-bold">Brand</a>
      <div class="hidden lg:flex lg:items-center lg:space-x-4">
        <a href="#" class="text-gray-900 hover:text-gray-600 px-3 py-2">Home</a>
      </div>
    </div>
  </div>
</nav>
```

### Form Controls Conversion

```html
<!-- Bootstrap Form -->
<div class="mb-3">
  <label for="email" class="form-label">Email address</label>
  <input type="email" class="form-control" id="email">
</div>

<!-- Tailwind Equivalent -->
<div class="mb-4">
  <label for="email" class="block text-sm font-medium text-gray-700 mb-1">
    Email address
  </label>
  <input 
    type="email" 
    id="email"
    class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
  >
</div>
```

### Stack Helpers Conversion

```html
<!-- Bootstrap vstack -->
<div class="vstack gap-3">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

<!-- Tailwind -->
<div class="flex flex-col gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

<!-- Bootstrap hstack -->
<div class="hstack gap-3">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

<!-- Tailwind -->
<div class="flex flex-row gap-4 items-center">
  <div>Item 1</div>
  <div>Item 2</div>
</div>
```

---

## Migration Best Practices

### 1. Establish Color Mapping First
Define your Bootstrap semantic colors in `tailwind.config.js` to maintain visual consistency:

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        'bs-primary': '#0d6efd',
        'bs-secondary': '#6c757d',
        'bs-success': '#198754',
        'bs-danger': '#dc3545',
        'bs-warning': '#ffc107',
        'bs-info': '#0dcaf0',
      }
    }
  }
}
```

### 2. Create Reusable Component Classes
For frequently used patterns, consider using `@apply` in your CSS:

```css
/* styles.css */
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2;
  }
  
  .card {
    @apply rounded-lg border border-gray-200 bg-white shadow-sm overflow-hidden;
  }
}
```

### 3. Handle Responsive Patterns
Remember the key differences:
- Bootstrap uses `{property}-{breakpoint}-{value}` (e.g., `col-md-6`)
- Tailwind uses `{breakpoint}:{property}-{value}` (e.g., `md:w-1/2`)

### 4. Gradual Migration Strategy
1. Start with layout utilities (display, flex, grid)
2. Move to spacing (margin, padding, gap)
3. Convert colors and typography
4. Finally, handle complex components

### 5. Testing Checklist
- [ ] Responsive behavior matches at all breakpoints
- [ ] Hover and focus states work correctly
- [ ] Colors match the original design
- [ ] Spacing is consistent
- [ ] Forms maintain accessibility
- [ ] Dark mode (if applicable) works correctly

---

## Quick Reference Cheatsheet

### Most Common Conversions

| Bootstrap | Tailwind |
|-----------|----------|
| `container` | `container mx-auto px-4` |
| `row` | `flex flex-wrap -mx-4` or `grid` |
| `col-*` | `w-*` or `grid-cols-*` |
| `d-flex` | `flex` |
| `d-none` | `hidden` |
| `justify-content-center` | `justify-center` |
| `align-items-center` | `items-center` |
| `text-center` | `text-center` |
| `btn btn-primary` | `px-4 py-2 bg-blue-600 text-white rounded` |
| `card` | `rounded-lg border bg-white shadow` |
| `alert alert-*` | `p-4 rounded border` |
| `mt-3` | `mt-4` |
| `p-3` | `p-4` |
| `mb-3` | `mb-4` |
| `gap-3` | `gap-4` |

---

## Additional Resources

- [Bootstrap 5.3 Documentation](https://getbootstrap.com/docs/5.3/)
- [Tailwind CSS v3 Documentation](https://v3.tailwindcss.com/docs)
- [Tailwind CSS Cheat Sheet](https://nerdcave.com/tailwind-cheat-sheet)
- [Headless UI](https://headlessui.com/) - Unstyled components for Tailwind
- [Tailwind UI](https://tailwindui.com/) - Official component library

---

*This document serves as a comprehensive reference for migrating from Bootstrap 5.3 to Tailwind CSS v3. Always verify conversions visually and test responsive behavior across breakpoints.*
