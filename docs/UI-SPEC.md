# UI Specification & Design System

## Overview
This document defines the visual design system and component specifications for the Bakery Website, based on the Figma design. It includes color schemes, typography, spacing, components, and responsive design guidelines.

## Design Principles

1. **Clean & Modern**: Minimalist design with focus on product imagery
2. **Warm & Inviting**: Use of warm colors and friendly typography
3. **User-Friendly**: Intuitive navigation and clear call-to-actions
4. **Responsive**: Mobile-first approach with adaptive layouts
5. **Accessible**: WCAG 2.1 AA compliance

## Color Palette

### Primary Colors
```css
:root {
  --primary-brown: #933C24;      /* Main brand color */
  --primary-gold: #E9BD8C;       /* Accent color */
  --primary-cream: #F9F6F1;      /* Background color */
}
```

### Neutral Colors
```css
:root {
  --neutral-black: #111111;      /* Primary text */
  --neutral-dark: #5D5D5D;       /* Secondary text */
  --neutral-gray: #737373;       /* Tertiary text */
  --neutral-light: #B9B9B9;      /* Disabled text */
  --neutral-border: #D9D9D9;     /* Borders */
  --neutral-bg: #F5F5F5;         /* Alternative background */
  --white: #FFFFFF;              /* Pure white */
}
```

### Semantic Colors
```css
:root {
  --success: #10B981;            /* Success states */
  --warning: #F59E0B;            /* Warning states */
  --error: #EF4444;              /* Error states */
  --info: #3B82F6;               /* Information */
}
```

## Typography

### Font Families
```css
/* Headings */
@import url('https://fonts.googleapis.com/css2?family=Sansita+Swashed:wght@400;500;600;700&display=swap');

/* Body Text */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap');
```

### Type Scale
```css
/* Headings - Sansita Swashed */
.heading-xl {
  font-family: 'Sansita Swashed', cursive;
  font-size: 74px;
  line-height: 1.2;
  font-weight: 400;
}

.heading-lg {
  font-family: 'Sansita Swashed', cursive;
  font-size: 64px;
  line-height: 1.2;
  font-weight: 600;
}

.heading-md {
  font-family: 'Sansita Swashed', cursive;
  font-size: 48px;
  line-height: 1.2;
  font-weight: 500;
}

.heading-sm {
  font-family: 'Sansita Swashed', cursive;
  font-size: 36px;
  line-height: 1.2;
  font-weight: 600;
}

/* Body Text - Inter/Poppins */
.text-xl {
  font-family: 'Inter', sans-serif;
  font-size: 32px;
  line-height: 1.5;
  font-weight: 600;
}

.text-lg {
  font-family: 'Poppins', sans-serif;
  font-size: 24px;
  line-height: 1.5;
  font-weight: 600;
}

.text-md {
  font-family: 'Inter', sans-serif;
  font-size: 20px;
  line-height: 1.5;
  font-weight: 500;
}

.text-sm {
  font-family: 'Poppins', sans-serif;
  font-size: 18px;
  line-height: 1.5;
  font-weight: 400;
}

.text-xs {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  line-height: 1.5;
  font-weight: 400;
}
```

## Spacing System

Using an 8px base unit:
```css
:root {
  --space-xs: 4px;    /* 0.5x */
  --space-sm: 8px;    /* 1x */
  --space-md: 16px;   /* 2x */
  --space-lg: 24px;   /* 3x */
  --space-xl: 32px;   /* 4x */
  --space-2xl: 48px;  /* 6x */
  --space-3xl: 64px;  /* 8x */
  --space-4xl: 80px;  /* 10x */
}
```

## Component Specifications

### 1. Navigation Header
```css
.navbar {
  height: 80px;
  background: transparent;
  padding: 0 var(--space-4xl);
}

.nav-link {
  font-family: 'Poppins', sans-serif;
  font-size: 22px;
  font-weight: 600;
  color: var(--neutral-black);
  transition: color 0.2s ease;
}

.nav-link:hover {
  color: var(--primary-gold);
}

.nav-link.active {
  color: var(--primary-gold);
}
```

### 2. Hero Section
```css
.hero {
  height: 813px;
  background-image: url('/hero-bg.jpg');
  background-size: cover;
  background-position: center;
  position: relative;
}

.hero-content {
  padding: 318px 80px;
}

.hero-title {
  font-family: 'Sansita Swashed', cursive;
  font-size: 74px;
  color: var(--neutral-black);
  max-width: 424px;
}

.hero-subtitle {
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  color: var(--primary-gold);
  margin-bottom: var(--space-xl);
}
```

### 3. Product Card
```css
.product-card {
  width: 360px;
  height: 411px;
  background: var(--white);
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease;
}

.product-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.product-image {
  width: 100%;
  height: 240px;
  object-fit: cover;
}

.product-info {
  padding: var(--space-lg);
}

.product-name {
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  font-weight: 500;
  color: var(--neutral-black);
  margin-bottom: var(--space-sm);
}

.product-price {
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  font-weight: 600;
  color: var(--neutral-black);
}

.add-button {
  background: var(--primary-brown);
  color: var(--white);
  padding: 16px 12px;
  border-radius: 4px;
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: background 0.2s ease;
}

.add-button:hover {
  background: #7A3220;
}
```

### 4. Category Tabs
```css
.category-tabs {
  display: flex;
  gap: var(--space-xl);
  padding: var(--space-lg) 0;
  border-bottom: 1px solid var(--neutral-border);
}

.category-tab {
  font-family: 'Poppins', sans-serif;
  font-size: 24px;
  font-weight: 600;
  color: var(--neutral-dark);
  padding-bottom: var(--space-sm);
  border-bottom: 3px solid transparent;
  cursor: pointer;
  transition: all 0.2s ease;
}

.category-tab:hover {
  color: var(--primary-brown);
}

.category-tab.active {
  color: var(--primary-brown);
  border-bottom-color: var(--primary-brown);
}
```

### 5. Buttons
```css
/* Primary Button */
.btn-primary {
  background: var(--primary-brown);
  color: var(--white);
  padding: 24px 38px;
  border-radius: 8px;
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  font-weight: 600;
  border: 1px solid var(--neutral-black);
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-primary:hover {
  background: #7A3220;
  transform: translateY(-2px);
}

/* Secondary Button */
.btn-secondary {
  background: transparent;
  color: var(--primary-gold);
  padding: 23px 35px;
  border-radius: 10px;
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  font-weight: 600;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: var(--space-sm);
  transition: all 0.2s ease;
}

.btn-secondary:hover {
  color: var(--primary-brown);
  transform: translateX(4px);
}
```

### 6. Form Elements
```css
.input {
  width: 100%;
  padding: 16px 20px;
  border: 1px solid var(--neutral-border);
  border-radius: 8px;
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  transition: border-color 0.2s ease;
}

.input:focus {
  outline: none;
  border-color: var(--primary-brown);
}

.input-error {
  border-color: var(--error);
}

.label {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  font-weight: 500;
  color: var(--neutral-black);
  margin-bottom: var(--space-sm);
  display: block;
}

.error-message {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  color: var(--error);
  margin-top: var(--space-xs);
}
```

## Layout Grid

### Desktop (1280px)
- Container: 1120px (80px padding each side)
- Columns: 12
- Gutter: 20px
- Product Grid: 3 columns

### Tablet (768px - 1279px)
- Container: 100% - 40px
- Columns: 8
- Gutter: 16px
- Product Grid: 2 columns

### Mobile (< 768px)
- Container: 100% - 20px
- Columns: 4
- Gutter: 12px
- Product Grid: 1 column

## Responsive Breakpoints

```css
/* Mobile First Approach */
@media (min-width: 640px) {
  /* Small tablets */
}

@media (min-width: 768px) {
  /* Tablets */
}

@media (min-width: 1024px) {
  /* Small desktops */
}

@media (min-width: 1280px) {
  /* Desktop */
}

@media (min-width: 1536px) {
  /* Large desktop */
}
```

## Animation Guidelines

### Transitions
```css
/* Default transition */
transition: all 0.2s ease;

/* Hover animations */
transition: transform 0.2s ease, box-shadow 0.2s ease;

/* Color transitions */
transition: color 0.2s ease, background-color 0.2s ease;
```

### Hover Effects
- Buttons: Slight translate up (-2px) or darken background
- Cards: Lift effect with shadow
- Links: Color change to primary-gold
- Images: Subtle zoom (scale: 1.05)

### Loading States
```css
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.skeleton {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
  background: var(--neutral-border);
}
```

## Accessibility

### Color Contrast
- Text on light backgrounds: Minimum 4.5:1 ratio
- Text on dark backgrounds: Minimum 4.5:1 ratio
- Large text (24px+): Minimum 3:1 ratio

### Focus States
```css
:focus-visible {
  outline: 2px solid var(--primary-brown);
  outline-offset: 2px;
}
```

### ARIA Labels
- All interactive elements must have descriptive labels
- Images must have alt text
- Form inputs must have associated labels

### Keyboard Navigation
- All interactive elements must be keyboard accessible
- Tab order must be logical
- Skip links for main content

## Icons

### Icon Library
Use Heroicons or similar for consistency:
- Size: 20px, 24px, or 32px
- Stroke width: 2px
- Color: Inherit from parent

### Common Icons
- Cart: Shopping bag icon
- User: User circle icon
- Menu: Hamburger menu (mobile)
- Close: X icon
- Arrow: Chevron right/down
- Info: Information circle

## Component States

### Interactive States
1. **Default**: Normal appearance
2. **Hover**: Visual feedback on mouse over
3. **Active**: During click/tap
4. **Focus**: Keyboard navigation
5. **Disabled**: Reduced opacity (0.5)
6. **Loading**: Skeleton or spinner

### Form States
1. **Empty**: Default state
2. **Filled**: User has entered data
3. **Focus**: Currently active
4. **Error**: Validation failed
5. **Success**: Validation passed
6. **Disabled**: Cannot be edited

## Mobile Considerations

### Touch Targets
- Minimum size: 44x44px
- Spacing between targets: 8px minimum

### Gestures
- Swipe for image galleries
- Pull to refresh (if applicable)
- Pinch to zoom on product images

### Performance
- Lazy load images
- Optimize image sizes
- Use WebP format with fallbacks
- Minimize JavaScript bundle

## Dark Mode (Future Enhancement)

```css
@media (prefers-color-scheme: dark) {
  :root {
    --neutral-black: #F5F5F5;
    --neutral-dark: #B9B9B9;
    --neutral-bg: #1A1A1A;
    /* Additional dark mode colors */
  }
}
```