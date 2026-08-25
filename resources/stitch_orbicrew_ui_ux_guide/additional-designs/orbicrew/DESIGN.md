---
name: Orbicrew
colors:
  surface: '#fdf8ff'
  surface-dim: '#ddd5ff'
  surface-bright: '#fdf8ff'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f7f1ff'
  surface-container: '#f1ebff'
  surface-container-high: '#ece4ff'
  surface-container-highest: '#e6deff'
  on-surface: '#1c1639'
  on-surface-variant: '#474553'
  inverse-surface: '#312b4f'
  inverse-on-surface: '#f4eeff'
  outline: '#787584'
  outline-variant: '#c8c4d5'
  surface-tint: '#584fbc'
  primary: '#3b309e'
  on-primary: '#ffffff'
  primary-container: '#534ab7'
  on-primary-container: '#d1ccff'
  inverse-primary: '#c5c0ff'
  secondary: '#7f5700'
  on-secondary: '#ffffff'
  secondary-container: '#ffc566'
  on-secondary-container: '#775100'
  tertiary: '#683500'
  on-tertiary: '#ffffff'
  tertiary-container: '#8a4900'
  on-tertiary-container: '#ffc69a'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#e3dfff'
  primary-fixed-dim: '#c5c0ff'
  on-primary-fixed: '#140067'
  on-primary-fixed-variant: '#3f35a3'
  secondary-fixed: '#ffdeae'
  secondary-fixed-dim: '#f6bd5e'
  on-secondary-fixed: '#281900'
  on-secondary-fixed-variant: '#604100'
  tertiary-fixed: '#ffdcc3'
  tertiary-fixed-dim: '#ffb77d'
  on-tertiary-fixed: '#2f1500'
  on-tertiary-fixed-variant: '#6e3900'
  background: '#fdf8ff'
  on-background: '#1c1639'
  surface-variant: '#e6deff'
typography:
  display-lg:
    fontFamily: Bricolage Grotesque
    fontSize: 56px
    fontWeight: '600'
    lineHeight: '1.1'
    letterSpacing: -0.02em
  display-lg-mobile:
    fontFamily: Bricolage Grotesque
    fontSize: 40px
    fontWeight: '600'
    lineHeight: '1.2'
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Bricolage Grotesque
    fontSize: 32px
    fontWeight: '600'
    lineHeight: '1.3'
  headline-sm:
    fontFamily: Bricolage Grotesque
    fontSize: 24px
    fontWeight: '600'
    lineHeight: '1.4'
  body-lg:
    fontFamily: Hanken Grotesk
    fontSize: 18px
    fontWeight: '400'
    lineHeight: '1.6'
  body-md:
    fontFamily: Hanken Grotesk
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.5'
  label-md:
    fontFamily: Hanken Grotesk
    fontSize: 14px
    fontWeight: '600'
    lineHeight: '1'
    letterSpacing: 0.02em
  label-sm:
    fontFamily: Hanken Grotesk
    fontSize: 12px
    fontWeight: '500'
    lineHeight: '1'
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  base: 8px
  container-max: 1280px
  gutter: 24px
  margin-mobile: 20px
  margin-desktop: 40px
  stack-sm: 12px
  stack-md: 24px
  stack-lg: 48px
---

## Brand & Style

The design system for Orbicrew, by Leangine, is anchored in a **Minimalist Premium** aesthetic. It moves away from the typical "tech hype" visual language of AI, favoring a professional, restrained, and high-utility atmosphere that treats AI agents as colleagues rather than novelties.

The visual narrative is defined by:
- **Pragmatic Elegance:** High-end finishes achieved through precise alignment and generous whitespace rather than decorative effects.
- **Restraint:** No gradients or unnecessary textures. Color is used strictly for focus and semantic signaling.
- **Confidence:** Large, high-weight typography paired with a clean, low-contrast UI.
- **Voice:** Communication is direct, using plain verbs and transparent data. The UI should feel like a high-performance workspace, not a marketing landing page.

## Colors

The palette utilizes a professional violet foundation to convey intelligence and stability, accented by gold to signify premium value and success. This light mode configuration focuses on clarity and a clean, airy workspace environment.

- **Primary Violet (#534ab7):** Used for primary actions and the brand's core presence. In light mode, it provides a strong, authoritative anchor against bright surfaces.
- **Accent Gold (#e0a94c):** Reserved for highlighting specialized "AI Employee" features, premium tiers, or milestones. It serves as a sophisticated signal of quality.
- **System Colors:** Success, Warning, and Danger are calibrated for high legibility on light backgrounds, maintaining a professional tone without becoming overly saturated.
- **Contrast:** High legibility is the priority. All text and interactive elements are designed to meet WCAG AA standards against light neutral surfaces.

## Typography

This design system utilizes a distinctive typographic pairing to balance character with utility.

- **Bricolage Grotesque:** Used for all headings and brand moments. Its quirky yet structured nature provides the "human" element of the AI employees.
- **Hanken Grotesk:** A modern, clean sans-serif used for all body text, data points, and interface labels. It is highly legible at small sizes.
- **Weights:** Use 600 for headings to create strong visual hierarchy. Use 400 for body text to ensure a light, airy feel within the "generous whitespace" layout.

## Layout & Spacing

The layout philosophy is a **Fixed-Fluid Hybrid Grid**. Content is centered within a 1280px container on desktop, but margins and gutters are generous to ensure a premium feel.

- **Grid Model:** 12-column grid for desktop (40px margins), 8-column for tablet, and 4-column for mobile (20px margins).
- **Rhythm:** An 8px base unit drives all spacing. For "generous whitespace," prioritize `stack-lg` (48px) between major sections and `stack-md` (24px) between grouped elements.
- **Vertical Spacing:** Emphasize top-padding on pages to give headings room to breathe. Components should never feel cramped; when in doubt, increase the padding.

## Elevation & Depth

In light mode, the system uses **Tonal Layers** and **Soft Shadows** to create a sense of organized hierarchy and physical presence.

- **Surfaces:** Depth is created by placing white or very light grey cards on subtly tinted off-white backgrounds (Surface Container).
- **Outlines:** All containers and cards use a 1px solid border in a soft neutral-variant (`#c8c4d5`) to define structure without visual noise.
- **Shadows:** Use shadows to indicate elevation levels. Shadows should be very soft, using a slight primary-tinted neutral color rather than pure black to maintain a clean, professional look.
- **Interactive States:** On hover, elements may show a subtle increase in shadow depth or a slight color shift in the border to `Primary Violet`.

## Shapes

The shape language is **Rounded**, reflecting a sophisticated but approachable tool.

- **Standard Radius:** 0.5rem (8px) for buttons, input fields, and small components.
- **Large Radius:** 1rem (16px) for cards and main content containers.
- **Icons:** Use a consistent 2px stroke width for icons to match the weight of the Hanken Grotesk labels. The "O" orb icon core and satellite dots should follow a perfect circle geometry.

## Components

- **Buttons:** Primary buttons use `Primary Violet` with white text. No gradients. Secondary buttons use a transparent background with a 1px border. "Hire" actions or premium features can use a subtle `Accent Gold` outline.
- **Input Fields:** Minimalist styling with a 1px border. On focus, the border transitions to `Primary Violet` with a subtle focus ring to signal activity.
- **Cards:** White or light-tinted surface background, 1px soft border, 16px corner radius. Card headers should use `Headline-sm` (Bricolage Grotesque).
- **Chips/Badges:** Use for AI "Skills" or "Status." These should have a light tinted background (e.g., 15% opacity of the semantic color) and bold 12px text to ensure they stand out.
- **AI Employee List:** Use a clean list format with generous vertical padding (24px per row). Use the "O" orb icon variant as a placeholder for employee avatars.
- **Checkboxes & Radios:** Sharp, custom-styled components using the `Primary Violet` color for the checked state.