# Frontend Mentor - Product preview card component solution

This is a solution to the [Product preview card component challenge on Frontend Mentor](https://www.frontendmentor.io/challenges/product-preview-card-component-GO7UmttRfa). Frontend Mentor challenges help you improve your coding skills by building realistic projects.

## Table of contents

- [Overview](#overview)
  - [The challenge](#the-challenge)
  - [Screenshot](#screenshot)
  - [Links](#links)
- [My process](#my-process)
  - [Built with](#built-with)
  - [What I learned](#what-i-learned)
  - [Continued development](#continued-development)
  - [Useful resources](#useful-resources)
  - [AI Collaboration](#ai-collaboration)
- [Author](#author)

## Overview

### The challenge

Users should be able to:

- View the optimal layout depending on their device's screen size
- See hover and focus states for interactive elements

### Screenshot

<img width="1571" height="721" alt="image" src="https://github.com/user-attachments/assets/4befdd91-c2db-41af-95dc-9dd306405a27" />
<img width="153" height="362" alt="image" src="https://github.com/user-attachments/assets/bd0cd868-fb67-4238-87eb-9f9a67facd94" />


### Links

- Solution URL: https://github.com/gmansoain/frontend-mentor-product-preview-card.git
- Live Site URL: https://gon-frontend-mentor-product-preview.netlify.app/

## My process

### Built with

- Semantic HTML5 markup
- CSS custom properties (design tokens)
- CSS Flexbox
- Media queries (desktop-first workflow)
- Self-hosted `@font-face` for Montserrat and Fraunces
- The `<picture>` element for viewport-conditional images

### What I learned

This was one of my first real projects, and even though the component looks simple, it packed in a surprising number of concepts. Here are the ones I want to remember.

#### Headings represent structure, not visual size

My first instinct was to use `<h2>` for the small "PERFUME" label because it looked smaller than the product name. But headings define the **document outline** for screen readers, not the visual hierarchy. The main subject of the card is the product, so it needs to be the `<h1>`. The "PERFUME" label is an eyebrow / category tag — a `<p>` with a class fits its role better.

```html
<p class="category">Perfume</p>
<h1>Gabrielle Essence Eau De Parfum</h1>
```

#### Meaningful vs. decorative `alt` text

Not every image should have descriptive alt text. The product photo adds meaning that isn't in the page → describe it. The cart icon sits next to the visible text "Add to Cart" → announcing "Cart icon, Add to Cart" is redundant noise, so `alt=""` tells screen readers to skip it.

```html
<img src="images/image-product-mobile.jpg" alt="Bottle of Gabrielle Essence Eau De Parfum">
<img src="images/icon-cart.svg" alt="">
```

#### `<s>` is the semantic element for an old price

Instead of using `text-decoration: line-through` on a generic span, I used `<s>` — HTML's element for content that is *no longer accurate or relevant*. It gets a strikethrough by default and carries the correct meaning for assistive tech.

```html
<div class="price">$149.99<s>$169.99</s></div>
```

#### `text-decoration` is drawn by the parent, not inherited by children

I tried adding `text-decoration: line-through` on the whole `.price` container and then removing it on the child span with `text-decoration: none`. It didn't work — no matter the specificity. The reason: the parent literally draws the line across all its inline descendants; children can't erase a line they aren't drawing. The fix is to put `line-through` only on the element that should have it.

#### `<picture>` for viewport-conditional images

The desktop image is a tall portrait; the mobile image is a wide landscape. Using the wrong aspect ratio for a viewport causes ugly cropping. The `<picture>` element lets the browser pick the right file based on media queries.

```html
<picture>
    <source media="(max-width: 40rem)" srcset="images/image-product-mobile.jpg">
    <img src="images/image-product-desktop.jpg" alt="Bottle of Gabrielle Essence Eau De Parfum">
</picture>
```

#### Border-radius on the parent + `overflow: hidden`

`border-radius` on a parent only rounds the parent's own background/border. Children keep their sharp corners and cover up the rounded shape. `overflow: hidden` clips the children to the parent's border-radius shape.

The reward: when flex direction flips from row to column on mobile, the same rule keeps working. No per-child radius juggling needed.

```css
.card {
    border-radius: var(--radius-sm);
    overflow: hidden;
}
```

#### Making an image fill a flex-stretched parent

Flex siblings stretch to the tallest one's height by default. But an `<img>` with `height: auto` keeps its natural aspect-ratio height and doesn't fill its parent — leaving a gap. The fix combines `height: 100%` (fill the parent) with `object-fit: cover` (fill without stretching or distorting).

```css
.card-image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}
```

#### The responsive width pattern: `width: 100%` + `max-width`

Fixed widths (`width: 70rem`) can't shrink on narrow screens. Combining `width: 100%` with `max-width: 70rem` says: *fill the available space, but never exceed 70rem*. Then adding outer breathing room via `padding` on the parent (`main`), not on the card itself, gives the card space to breathe at any viewport size.

```css
.card {
    max-width: 70rem;
    width: 100%;
}

main {
    padding: 1.6rem;
}
```

#### `:focus-visible` is a better default than `:focus`

`:focus` triggers on mouse clicks too, leaving buttons stuck in a "focused" state that feels visually noisy. `:focus-visible` only triggers when the browser thinks the user actually needs a focus indicator — mainly keyboard navigation. Best of both worlds.

```css
.card-text .btn:hover,
.card-text .btn:focus-visible {
    background-color: var(--primary-green-700);
}
```

I also learned to *not* remove the default browser `outline` — it gives keyboard users an additional cue on top of the color change.

#### `rem` in media queries is 16px, not your `html` font-size

This one really caught me out. Inside my CSS, I use `1rem = 10px` (thanks to `html { font-size: 62.5% }`). But inside a `@media (...)` condition, `1rem = 16px` — the browser's default. Media queries are evaluated *before* CSS is applied, so they can't depend on CSS declarations. The takeaway: `40rem` in a media query = 640px, not 400px.

```css
/* This breakpoint is 640px, not 400px */
@media (max-width: 40rem) {
    .card { flex-direction: column; }
    /* etc. */
}
```

#### Consistency matters more than following a "best practice" in isolation

I originally wrote my `<picture>` as mobile-first (with `min-width` and mobile as the default `<img>`). But my CSS was desktop-first (default rules for desktop, `@media (max-width: ...)` overrides for mobile). I switched the `<picture>` to match the desktop-first pattern. The bigger lesson: pick one responsive approach and apply it consistently across the codebase, so anyone reading the code (including future-me) uses a single mental model.

#### `padding` is inside, not outside

When I tried to give the card space on narrow screens by adding `padding` to `.card`, it *seemed* to remove the border-radius. It didn't — the padding pushed the children inward, and since `.card` has no background, the still-rounded corners became invisible against the cream page. Padding is space *inside* an element. What I actually wanted was space *outside*, which comes from the parent's padding (or the element's margin).

### Continued development

- Explore CSS Grid for layouts where two-dimensional control matters (this project only needed one axis, so Flexbox was enough).
- Try building the same layout mobile-first for practice, since desktop-first is what I ended up using here.
- Get more comfortable with fluid typography (`clamp()`) so the h1 scales smoothly between breakpoints instead of snapping at 40rem.
- Learn about the `<picture>` element's other superpowers: high-DPI (retina) images with `srcset` density descriptors, and modern format fallbacks (AVIF → WebP → JPG).

### Useful resources

- [MDN: `text-decoration`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration) — Explained why children can't remove a parent's strikethrough.
- [MDN: `object-fit`](https://developer.mozilla.org/en-US/docs/Web/CSS/object-fit) — Same idea as `background-size: cover`, but for `<img>` and `<video>`.
- [MDN: The `<picture>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture) — The definitive reference for responsive images.
- [MDN: `:focus-visible`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible) — Why this pseudo-class is the accessible default over `:focus`.
- [MDN: Using media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries) — Covers the `rem`-in-media-queries gotcha.
- [MDN: The `<s>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/s) — Semantic strikethrough for "no longer accurate" content, distinct from `<del>` ("deleted").

### AI Collaboration

I used Claude (via Claude Code) as a coding mentor throughout this project. The project directory included an `AGENTS.md` file that instructed the AI to teach through hints and questions rather than writing complete solutions, which matched the "learn by doing" spirit of Frontend Mentor.

**What worked well:**
- Asking about *why* something behaved a certain way (e.g. "why doesn't `text-decoration: none` on the child override the parent's `line-through`?"). The AI explained the underlying concept (`text-decoration` is drawn by the parent, not inherited) rather than just handing me a fix. I learned the concept for good.
- The AI pushing back gently when I did something suboptimal (e.g. using `<b>` for the "PERFUME" label). Being asked "is this really a heading, or is it more like a category label?" made me think about intent rather than just what looked right.
- Getting a diagnostic path when things looked wrong ("open DevTools and hover over the parent — do the child bounds match?") rather than a copy-paste fix.

**What worked less well / what I had to push back on:**
- The AI initially suggested mobile-first for the `<picture>` element even though my CSS was desktop-first. I asked whether internal consistency should take priority, and it agreed. Good reminder that AI defaults to conventions, but the developer still has to make the architectural call.

**What I'd do differently next time:**
- Ask for design comparisons more actively. On the mobile design, the image and text container have a subtle gap that I only noticed near the end. Reviewing the design side-by-side with my browser at each stage would catch these earlier.

## Author

- Frontend Mentor - [@Gon](https://www.frontendmentor.io/profile/Gon)
