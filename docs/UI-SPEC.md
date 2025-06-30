# UI Specification & Design System

## Overview
This document defines the visual design system and component specifications for both the landing site and web application. Both applications use **shadcn/ui** as the foundation for all UI components, ensuring consistency, accessibility, and customizability. While maintaining visual consistency across both applications, each has specific design considerations optimized for its purpose.

## Design Principles

### Shared Principles
1. **Clean & Modern**: Minimalist design with focus on content
2. **Visual Hierarchy**: Clear structure and information flow
3. **Responsive**: Mobile-first approach with adaptive layouts
4. **Accessible**: WCAG 2.1 AA compliance
5. **Brand Consistency**: Unified visual language

### Landing Site Specific
1. **Conversion-Focused**: Clear CTAs and user journey
2. **Fast Loading**: Optimized assets and minimal JavaScript
3. **SEO-Friendly**: Semantic HTML and structured data
4. **Engaging**: Eye-catching visuals and animations

### Web Application Specific
1. **Functional**: Efficient task completion
2. **Information Dense**: Display complex data clearly
3. **Interactive**: Rich user interactions
4. **Consistent**: Predictable UI patterns

## Color Palette

### Primary Colors
```css
:root {
  --primary-main: #933C24;       /* Main brand color */
  --primary-accent: #E9BD8C;     /* Accent color */
  --primary-light: #F9F6F1;      /* Light background color */
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

### shadcn/ui Foundation

All components in the project are built on top of **shadcn/ui**, which provides:
- **Accessibility**: Built with Radix UI primitives for full keyboard navigation and screen reader support
- **Customization**: Tailwind CSS based styling that can be easily modified
- **TypeScript**: Full type safety out of the box
- **Copy & Paste**: Components are copied into the codebase for full control

### Component Library Setup

```bash
# Initialize shadcn/ui in each app
cd apps/landing
npx shadcn-ui@latest init

cd apps/web
npx shadcn-ui@latest init

# Add components as needed
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add form
npx shadcn-ui@latest add navigation-menu
```

### Shared Components (via @project/ui)

The shared UI package extends shadcn/ui components with project-specific customizations:

#### Extended Button Component
```typescript
// packages/ui/src/Button.tsx
import { Button as ShadcnButton } from '@/components/ui/button'
import { cn } from '@/lib/utils'

export const Button = ({ className, ...props }) => {
  return (
    <ShadcnButton
      className={cn(
        // Add project-specific default styles
        'font-poppins',
        className
      )}
      {...props}
    />
  )
}
```

#### Extended Card Component
```typescript
// packages/ui/src/Card.tsx
import { Card as ShadcnCard } from '@/components/ui/card'

export const Card = ShadcnCard
export const CardHeader = ShadcnCard.Header
export const CardContent = ShadcnCard.Content
export const CardFooter = ShadcnCard.Footer
```

### Landing Site Components

#### 1. Landing Navigation
```css
.landing-nav {
  height: 80px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  position: fixed;
  top: 0;
  width: 100%;
  z-index: 1000;
  transition: all 0.3s ease;
}

.landing-nav.scrolled {
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
}

.landing-nav-link {
  font-family: 'Poppins', sans-serif;
  font-size: 16px;
  font-weight: 500;
  color: var(--neutral-dark);
  transition: color 0.2s ease;
}

.landing-nav-link:hover {
  color: var(--primary-main);
}
```

### Web Application Components

#### 1. App Navigation
```css
.app-navbar {
  height: 64px;
  background: var(--white);
  border-bottom: 1px solid var(--neutral-border);
  padding: 0 var(--space-lg);
}

.app-nav-link {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--neutral-dark);
  transition: all 0.2s ease;
  padding: var(--space-sm) var(--space-md);
  border-radius: 6px;
}

.app-nav-link:hover {
  background: var(--neutral-bg);
  color: var(--neutral-black);
}

.app-nav-link.active {
  background: var(--primary-light);
  color: var(--primary-main);
}
```

### 2. Landing Hero Section
```css
.landing-hero {
  min-height: 100vh;
  background: linear-gradient(135deg, var(--primary-light) 0%, var(--white) 100%);
  position: relative;
  overflow: hidden;
}

.landing-hero-content {
  padding: 120px 80px 80px;
  max-width: 1200px;
  margin: 0 auto;
}

.landing-hero-title {
  font-family: 'Sansita Swashed', cursive;
  font-size: clamp(48px, 8vw, 74px);
  color: var(--neutral-black);
  line-height: 1.2;
  margin-bottom: var(--space-lg);
}

.landing-hero-subtitle {
  font-family: 'Inter', sans-serif;
  font-size: clamp(18px, 3vw, 24px);
  color: var(--neutral-dark);
  margin-bottom: var(--space-2xl);
  max-width: 600px;
}

.landing-hero-cta {
  display: flex;
  gap: var(--space-md);
  flex-wrap: wrap;
}

/* Animated background elements */
.landing-hero-bg-element {
  position: absolute;
  border-radius: 50%;
  background: var(--primary-accent);
  opacity: 0.1;
  animation: float 20s infinite ease-in-out;
}

@keyframes float {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-20px) rotate(180deg); }
}
```

### 3. Content Cards (Different Styles)

#### Landing Site - Feature Card
```css
.landing-feature-card {
  background: var(--white);
  border-radius: 16px;
  padding: var(--space-2xl);
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.06);
  transition: all 0.3s ease;
  height: 100%;
}

.landing-feature-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12);
}

.landing-feature-icon {
  width: 64px;
  height: 64px;
  background: var(--primary-light);
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: var(--space-lg);
}

.landing-feature-title {
  font-family: 'Poppins', sans-serif;
  font-size: 24px;
  font-weight: 600;
  color: var(--neutral-black);
  margin-bottom: var(--space-md);
}

.landing-feature-description {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  line-height: 1.6;
  color: var(--neutral-dark);
}
```

#### Web Application - Item Card
```css
.app-item-card {
  background: var(--white);
  border: 1px solid var(--neutral-border);
  border-radius: 8px;
  overflow: hidden;
  transition: all 0.2s ease;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.app-item-card:hover {
  border-color: var(--primary-main);
  box-shadow: 0 4px 12px rgba(147, 60, 36, 0.1);
}

.app-item-image {
  width: 100%;
  aspect-ratio: 4/3;
  object-fit: cover;
}

.app-item-content {
  padding: var(--space-md);
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}

.app-item-title {
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  font-weight: 600;
  color: var(--neutral-black);
  margin-bottom: var(--space-xs);
}

.app-item-meta {
  font-family: 'Inter', sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: var(--primary-main);
  margin-top: auto;
}

.app-item-actions {
  padding: var(--space-md);
  border-top: 1px solid var(--neutral-border);
  display: flex;
  gap: var(--space-sm);
}
```

### 4. Category Navigation

#### Landing Site - Category Showcase
```css
.landing-category-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-xl);
  margin: var(--space-3xl) 0;
}

.landing-category-card {
  background: var(--white);
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
}

.landing-category-card:hover {
  transform: scale(1.05);
}

.landing-category-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.landing-category-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(to top, rgba(0,0,0,0.8), transparent);
  padding: var(--space-xl) var(--space-lg) var(--space-lg);
}

.landing-category-name {
  font-family: 'Poppins', sans-serif;
  font-size: 24px;
  font-weight: 600;
  color: var(--white);
}
```

#### Web Application - Category Tabs
```css
.app-category-tabs {
  display: flex;
  gap: var(--space-sm);
  padding: var(--space-md) 0;
  border-bottom: 1px solid var(--neutral-border);
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

.app-category-tab {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--neutral-dark);
  padding: var(--space-sm) var(--space-md);
  border-radius: 20px;
  white-space: nowrap;
  cursor: pointer;
  transition: all 0.2s ease;
  background: var(--neutral-bg);
}

.app-category-tab:hover {
  background: var(--primary-light);
  color: var(--primary-main);
}

.app-category-tab.active {
  background: var(--primary-main);
  color: var(--white);
}
```

### 5. Button Styles

#### Landing Site Buttons
```css
/* Landing Primary CTA */
.landing-btn-primary {
  background: var(--primary-main);
  color: var(--white);
  padding: 18px 36px;
  border-radius: 50px;
  font-family: 'Poppins', sans-serif;
  font-size: 18px;
  font-weight: 600;
  border: none;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 14px rgba(147, 60, 36, 0.25);
}

.landing-btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(147, 60, 36, 0.35);
}

/* Landing Secondary CTA */
.landing-btn-secondary {
  background: transparent;
  color: var(--primary-main);
  padding: 18px 36px;
  border-radius: 50px;
  font-family: 'Poppins', sans-serif;
  font-size: 18px;
  font-weight: 600;
  border: 2px solid var(--primary-main);
  cursor: pointer;
  transition: all 0.3s ease;
}

.landing-btn-secondary:hover {
  background: var(--primary-main);
  color: var(--white);
}
```

#### Web Application Buttons
```css
/* App Primary Button */
.app-btn-primary {
  background: var(--primary-main);
  color: var(--white);
  padding: 10px 20px;
  border-radius: 6px;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;
}

.app-btn-primary:hover {
  background: #7A3220;
}

.app-btn-primary:active {
  transform: scale(0.98);
}

/* App Secondary Button */
.app-btn-secondary {
  background: var(--neutral-bg);
  color: var(--neutral-black);
  padding: 10px 20px;
  border-radius: 6px;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  border: 1px solid var(--neutral-border);
  cursor: pointer;
  transition: all 0.2s ease;
}

.app-btn-secondary:hover {
  background: var(--neutral-border);
}

/* App Icon Button */
.app-btn-icon {
  width: 36px;
  height: 36px;
  border-radius: 6px;
  border: 1px solid var(--neutral-border);
  background: var(--white);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
}

.app-btn-icon:hover {
  background: var(--neutral-bg);
  border-color: var(--neutral-dark);
}
```

### 6. Form Elements

#### Landing Site Forms
```css
/* Landing Contact Form */
.landing-form-group {
  margin-bottom: var(--space-lg);
}

.landing-label {
  font-family: 'Poppins', sans-serif;
  font-size: 16px;
  font-weight: 500;
  color: var(--neutral-black);
  margin-bottom: var(--space-sm);
  display: block;
}

.landing-input {
  width: 100%;
  padding: 16px 20px;
  border: 2px solid var(--neutral-border);
  border-radius: 12px;
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  transition: all 0.3s ease;
  background: var(--white);
}

.landing-input:focus {
  outline: none;
  border-color: var(--primary-main);
  box-shadow: 0 0 0 4px rgba(147, 60, 36, 0.1);
}

.landing-textarea {
  min-height: 120px;
  resize: vertical;
}
```

#### Web Application Forms
```css
/* App Form Elements */
.app-form-group {
  margin-bottom: var(--space-md);
}

.app-label {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--neutral-black);
  margin-bottom: var(--space-xs);
  display: block;
}

.app-label-required::after {
  content: '*';
  color: var(--error);
  margin-left: 4px;
}

.app-input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid var(--neutral-border);
  border-radius: 6px;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  transition: all 0.2s ease;
}

.app-input:focus {
  outline: none;
  border-color: var(--primary-main);
  box-shadow: 0 0 0 3px rgba(147, 60, 36, 0.1);
}

.app-input-error {
  border-color: var(--error);
}

.app-error-message {
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  color: var(--error);
  margin-top: var(--space-xs);
  display: flex;
  align-items: center;
  gap: 4px;
}
```

## Layout Systems

### Landing Site Layout

#### Desktop (1280px+)
- Container: 1200px max-width
- Padding: 80px (sides)
- Hero: Full viewport height
- Features Grid: 3 columns
- Testimonials: 2 columns

#### Tablet (768px - 1279px)
- Container: 100% - 48px
- Padding: 24px (sides)
- Hero: Min 600px height
- Features Grid: 2 columns
- Testimonials: 1 column

#### Mobile (< 768px)
- Container: 100% - 32px
- Padding: 16px (sides)
- Hero: Min 500px height
- All grids: 1 column
- Stacked layout

### Web Application Layout

#### Desktop (1280px+)
- Sidebar: 240px (collapsible)
- Main Content: Flexible
- Container: 1120px max-width
- Item Grid: 4 columns
- Data Tables: Full width

#### Tablet (768px - 1279px)
- Sidebar: Hidden (hamburger menu)
- Main Content: Full width - 40px
- Item Grid: 3 columns
- Data Tables: Horizontal scroll

#### Mobile (< 768px)
- Sidebar: Full screen overlay
- Main Content: Full width - 20px
- Item Grid: 2 columns
- Data Tables: Card view

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

### Landing Site Animations

#### Scroll Animations
```css
/* Fade in on scroll */
.landing-animate-fade-in {
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.6s ease-out;
}

.landing-animate-fade-in.in-view {
  opacity: 1;
  transform: translateY(0);
}

/* Stagger children */
.landing-animate-stagger > * {
  transition-delay: calc(var(--index) * 0.1s);
}
```

#### Hero Animations
```css
@keyframes heroSlideIn {
  from {
    opacity: 0;
    transform: translateX(-30px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.landing-hero-title {
  animation: heroSlideIn 0.8s ease-out;
}

.landing-hero-subtitle {
  animation: heroSlideIn 0.8s ease-out 0.2s both;
}

.landing-hero-cta {
  animation: heroSlideIn 0.8s ease-out 0.4s both;
}
```

### Web Application Animations

#### Micro-interactions
```css
/* Quick transitions for responsiveness */
.app-transition-all {
  transition: all 0.15s ease;
}

/* Loading states */
.app-spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Skeleton loading */
.app-skeleton {
  background: linear-gradient(
    90deg,
    var(--neutral-border) 25%,
    var(--neutral-bg) 50%,
    var(--neutral-border) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

### Shared Hover Effects
- Landing buttons: Scale + shadow enhancement
- App buttons: Subtle background change
- Cards: Lift effect appropriate to context
- Links: Color transitions
- Images: Zoom on hover (landing only)

## shadcn/ui Component Usage

### Core Components Used

#### Navigation
- **NavigationMenu** - Main navigation (customized for landing vs app)
- **Sheet** - Mobile navigation drawer
- **Tabs** - Category navigation in web app

#### Forms
- **Form** - React Hook Form integration
- **Input** - Text inputs with validation states
- **Select** - Dropdown selections
- **Textarea** - Multi-line text input
- **Checkbox** - Boolean selections
- **RadioGroup** - Single choice selections

#### Feedback
- **Alert** - Information messages
- **Toast** - Temporary notifications
- **AlertDialog** - Confirmation dialogs
- **Dialog** - Modal windows

#### Data Display
- **Table** - Data tables in admin
- **Card** - Content containers
- **Badge** - Status indicators
- **Avatar** - User profile images

#### Layout
- **Separator** - Visual dividers
- **ScrollArea** - Custom scrollbars
- **AspectRatio** - Responsive media containers

### Customization Approach

```typescript
// Example: Customizing shadcn/ui theme
// apps/web/src/styles/globals.css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 6.7%;
    
    --primary: 15 59% 36%; /* #933C24 */
    --primary-foreground: 0 0% 98%;
    
    --secondary: 34 47% 72%; /* #E9BD8C */
    --secondary-foreground: 0 0% 9%;
    
    --muted: 30 20% 97%; /* #F9F6F1 */
    --muted-foreground: 0 0% 45.1%;
    
    --accent: 34 47% 72%;
    --accent-foreground: 0 0% 9%;
    
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 98%;
    
    --border: 0 0% 85.1%; /* #D9D9D9 */
    --input: 0 0% 85.1%;
    --ring: 15 59% 36%;
    
    --radius: 0.5rem;
  }
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

### Landing Site States
Focused on conversion and engagement:

#### CTA Button States
1. **Default**: High contrast, shadow
2. **Hover**: Enhanced shadow, slight scale
3. **Active**: Pressed appearance
4. **Loading**: Pulse animation

#### Form Field States
1. **Empty**: Inviting appearance
2. **Focus**: Strong visual indicator
3. **Filled**: Subtle success state
4. **Error**: Clear but not alarming
5. **Success**: Green check indicator

### Web Application States
Focused on efficiency and clarity:

#### Interactive Element States
1. **Default**: Clean, minimal
2. **Hover**: Subtle highlight
3. **Active**: Clear selection
4. **Focus**: Accessibility outline
5. **Disabled**: 50% opacity
6. **Loading**: Inline spinner

#### Data States
1. **Empty**: Helpful empty state
2. **Loading**: Skeleton screens
3. **Error**: Inline error messages
4. **Success**: Toast notifications
5. **Partial**: Progress indicators

## Platform-Specific Considerations

### Landing Site Mobile

#### Performance First
- Critical CSS inline
- Lazy load below-fold content
- Preload hero images
- Minimal JavaScript (< 50KB)
- Service worker for offline

#### Touch Optimizations
- Large CTA buttons (min 48px)
- Swipeable testimonials
- Smooth scrolling
- Tap to expand FAQ items

### Web Application Mobile

#### Functionality Focus
- Bottom navigation bar
- Swipe gestures for actions
- Pull to refresh lists
- Optimistic UI updates
- Offline capability

#### Responsive Tables
```css
/* Mobile table transformation */
@media (max-width: 768px) {
  .app-table-responsive {
    display: block;
  }
  
  .app-table-responsive thead {
    display: none;
  }
  
  .app-table-responsive tr {
    display: block;
    margin-bottom: var(--space-md);
    border: 1px solid var(--neutral-border);
    border-radius: 8px;
    padding: var(--space-md);
  }
  
  .app-table-responsive td {
    display: flex;
    justify-content: space-between;
    padding: var(--space-xs) 0;
  }
  
  .app-table-responsive td::before {
    content: attr(data-label);
    font-weight: 600;
    color: var(--neutral-dark);
  }
}
```

## Theme Support

### Landing Site Theme
The landing site uses a fixed light theme optimized for marketing:

```css
/* Landing always light mode for consistency */
.landing-root {
  color-scheme: light only;
}
```

### Web Application Theme Support

#### Light Mode (Default)
```css
.app-theme-light {
  --app-bg-primary: #FFFFFF;
  --app-bg-secondary: #F5F5F5;
  --app-text-primary: #111111;
  --app-text-secondary: #5D5D5D;
  --app-border: #D9D9D9;
}
```

#### Dark Mode
```css
.app-theme-dark {
  --app-bg-primary: #1A1A1A;
  --app-bg-secondary: #252525;
  --app-text-primary: #F5F5F5;
  --app-text-secondary: #B9B9B9;
  --app-border: #404040;
}

/* Automatic dark mode */
@media (prefers-color-scheme: dark) {
  .app-theme-auto {
    --app-bg-primary: #1A1A1A;
    /* ... rest of dark theme */
  }
}
```

#### Theme Toggle
```typescript
// Store theme preference
const theme = localStorage.getItem('app-theme') || 'auto';
document.documentElement.classList.add(`app-theme-${theme}`);
```

## Design Token Export

For maintaining consistency across applications:

```typescript
// packages/ui/src/tokens.ts
export const tokens = {
  colors: {
    primary: {
      main: '#933C24',
      accent: '#E9BD8C',
      light: '#F9F6F1',
    },
    neutral: {
      black: '#111111',
      dark: '#5D5D5D',
      gray: '#737373',
      light: '#B9B9B9',
      border: '#D9D9D9',
      bg: '#F5F5F5',
      white: '#FFFFFF',
    },
    semantic: {
      success: '#10B981',
      warning: '#F59E0B',
      error: '#EF4444',
      info: '#3B82F6',
    },
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px',
    '2xl': '48px',
    '3xl': '64px',
    '4xl': '80px',
  },
  typography: {
    fontFamily: {
      heading: "'Sansita Swashed', cursive",
      body: "'Inter', sans-serif",
      accent: "'Poppins', sans-serif",
    },
  },
};
```