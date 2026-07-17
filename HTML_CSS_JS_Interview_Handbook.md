# Front-End Fundamentals Interview Handbook
### HTML · CSS · JavaScript — Q&A prep + coding questions

---

## Table of Contents

**Part 1 — HTML**
1. Semantic HTML & Document Structure
2. Forms & Validation
3. Accessibility (a11y)
4. HTML5 APIs & Storage
5. Meta Tags & SEO Basics

**Part 2 — CSS**
6. Box Model, Selectors & Specificity
7. Flexbox
8. Grid
9. Positioning & Stacking Context
10. Responsive Design
11. Units & Custom Properties
12. Animations & Transitions
13. CSS Architecture (BEM, Preprocessors)

**Part 3 — JavaScript**
14. Fundamentals — Types, `var`/`let`/`const`, Hoisting
15. Functions, Closures & Scope
16. `this`, `call`/`apply`/`bind`
17. Prototypes, Inheritance & Classes
18. Event Loop & Asynchronous JS
19. DOM Manipulation & Events
20. ES6+ Features
21. Array & Object Methods
22. Error Handling
23. Most-Asked JavaScript Coding Questions

---

# PART 1: HTML

## 1. Semantic HTML & Document Structure

**Q1: What is semantic HTML, and why does it matter beyond just "best practice"?**

A: Semantic HTML uses elements that convey **meaning about their content**, not just visual appearance — `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>` instead of generic `<div>`s for everything. This matters concretely for: **accessibility** (screen readers use semantic landmarks to let users jump directly to navigation/main content), **SEO** (search engines weight content in semantic elements like `<article>`/`<h1>` differently than generic containers), and **maintainability** (a developer reading `<nav>` instantly understands its purpose vs. `<div class="nav-wrapper-2">`).

**Q2: What's the difference between `<section>`, `<article>`, and `<div>`?**

A: `<article>` — content that's **independently distributable/reusable** on its own (a blog post, a news story, a forum comment) — it should make sense extracted and placed elsewhere. `<section>` — a **thematic grouping** of content, typically with its own heading, that's part of a larger document but not necessarily self-contained (a chapter, a tabbed panel). `<div>` — a **purely structural/styling** container with **no semantic meaning** — use it only when no semantic element fits (e.g., a wrapper purely for CSS grid/flex layout purposes).

**Q3: What's the difference between `id` and `class` attributes, and how does that map to CSS specificity/JS selection?**

A: `id` must be **unique per page** (used for direct anchor links `#section`, `label for=`, and unique JS hooks like `getElementById`); `class` can be applied to **multiple elements** and is the standard way to share styling/behavior across many elements. In CSS specificity, an `id` selector (`#foo`) outweighs any number of class selectors, which is a common reason to **avoid styling via `id`** in larger codebases — it makes overriding styles later unnecessarily hard (see Section 6).

**Q4: What does `<!DOCTYPE html>` do, and what happens without it?**

A: It tells the browser to render the page in **standards mode**, following the modern HTML/CSS specifications consistently. Without it (or with a malformed doctype), older browsers fall back to **quirks mode**, emulating legacy, inconsistent rendering behaviors from 1990s browsers (different box-model calculations, different handling of certain CSS properties) — a common source of hard-to-debug cross-browser layout bugs if accidentally triggered.

**Q5: What's the difference between `<script>`, `<script defer>`, and `<script async>`?**

A: Plain `<script>` **blocks HTML parsing** — the browser stops parsing the page, fetches (if external) and executes the script immediately, then resumes parsing. `<script defer>` downloads the script **in parallel** with HTML parsing but delays **execution** until parsing is complete, and multiple deferred scripts execute **in document order** — the standard choice for most page scripts, since it doesn't block rendering and preserves predictable ordering. `<script async>` also downloads in parallel, but executes **as soon as it's downloaded**, potentially interrupting HTML parsing at an unpredictable point, and multiple async scripts can execute **out of order** relative to each other — best for independent scripts with no dependencies on DOM state or other scripts (e.g., analytics tags).

---

## 2. Forms & Validation

**Q1: What's the difference between built-in HTML5 form validation and JavaScript validation, and when do you need both?**

A: HTML5 provides declarative validation via attributes (`required`, `pattern`, `min`/`max`, `type="email"`) and the browser shows native error messages/blocks submission automatically — zero JS needed for basic cases. You still need **JavaScript validation** for: complex cross-field rules (password confirmation matching), async checks (username availability against a server), custom error message styling/UX beyond the browser's default tooltip, and — critically — you always need **server-side validation** too, since client-side validation (HTML or JS) can be bypassed entirely by a user submitting directly via a crafted request.

**Q2: What are the common HTML5 input types, and what UX/validation benefit does each give over `type="text"`?**

A: `email` (validates `@`/format, shows an email-optimized mobile keyboard), `tel` (numeric phone keypad on mobile, no built-in format validation since phone formats vary globally), `number` (numeric-only input, spinner controls, `min`/`max`/`step`), `date`/`time`/`datetime-local` (native date picker UI), `url`, `password` (masks input), `search` (may show a clear-button, semantic intent for search engines/autofill), `color` (native color picker). Using the correct semantic type improves mobile keyboard UX, enables better browser autofill, and reduces the JS validation you need to hand-write.

**Q3: How does the `<label>` element relate to accessibility, and what are the two ways to associate it with an input?**

A: A properly associated `<label>` lets screen readers announce the input's purpose, and (importantly) makes the **label text itself clickable** to focus/activate the input — a real usability win, especially for checkboxes/radio buttons on mobile.
```html
<!-- Explicit association (preferred, more flexible) -->
<label for="email">Email</label>
<input id="email" type="email">

<!-- Implicit association (wrapping) -->
<label>Email <input type="email"></label>
```

**Q4: What's the difference between `GET` and `POST` form submission methods?**

A: `GET` appends form data as **URL query parameters**, is cacheable/bookmarkable, has practical **length limits**, and should be used only for idempotent, non-sensitive data retrieval (search forms) — never for anything with side effects or sensitive data (it ends up in browser history, server logs, and the URL bar). `POST` sends data in the **request body**, isn't length-limited in practice, isn't cached/bookmarked by default, and is required for anything that mutates server state or includes sensitive data.

---

## 3. Accessibility (a11y)

**Q1: What is ARIA, and what's the "first rule of ARIA"?**

A: ARIA (Accessible Rich Internet Applications) is a set of attributes (`role`, `aria-label`, `aria-hidden`, `aria-expanded`, etc.) that add accessibility semantics to elements — primarily needed for **custom widgets** that don't have a native HTML equivalent (a custom dropdown, a tab panel, a modal). The **first rule of ARIA** is: **don't use ARIA if a native HTML element/attribute already provides the semantics you need** — e.g., don't build a `<div role="button">` with a click handler when a real `<button>` already gives you keyboard focus, `Enter`/`Space` activation, and correct semantics for free. ARIA misuse (wrong roles, missing required states) can actively make a page **less** accessible than using no ARIA at all.

**Q2: What's the difference between `aria-hidden`, `display: none`, and `visibility: hidden` regarding accessibility?**

A: `display: none` and `visibility: hidden` remove the element **both visually and from the accessibility tree** (screen readers won't announce it) — the difference between them is purely visual/layout (`display: none` removes it from layout flow entirely; `visibility: hidden` keeps its layout space but makes it invisible). `aria-hidden="true"` hides an element **only from the accessibility tree** while it remains visually present — used for purely decorative content (an icon next to a text label that would be redundant if announced) that sighted users should see but screen reader users don't need announced.

**Q3: How do you make a custom interactive widget (e.g., a custom dropdown) keyboard-accessible?**

A: Ensure it's reachable via `Tab` (native interactive elements are focusable by default; custom `<div>`-based widgets need `tabindex="0"`), respond to the expected keyboard interactions for that widget pattern (Arrow keys to navigate options, `Enter`/`Space` to select, `Escape` to close), manage focus programmatically (move focus into an opened dropdown/modal, and return it to the triggering element on close), and add appropriate ARIA roles/states (`role="listbox"`, `aria-expanded`, `aria-activedescendant`) so assistive tech understands the widget's current state — matching the WAI-ARIA Authoring Practices patterns for that widget type is the standard reference.

**Q4: What's the purpose of `alt` text on images, and when should `alt=""` (empty) be used instead of omitting it?**

A: `alt` text is read aloud by screen readers in place of the image, and displayed if the image fails to load — it should describe the image's **content/purpose** in context (not just "image of..."). Use an **empty** `alt=""` (not omitting the attribute entirely) for **purely decorative** images that convey no information (a background flourish, a spacer) — this tells assistive tech to skip it silently, whereas omitting `alt` entirely can cause some screen readers to announce the filename instead, which is worse than saying nothing.

---

## 4. HTML5 APIs & Storage

**Q1: Compare `localStorage`, `sessionStorage`, and cookies.**

A:
| | `localStorage` | `sessionStorage` | Cookies |
|---|---|---|---|
| Persistence | Until explicitly cleared | Until tab/window closes | Configurable expiry |
| Capacity | ~5-10MB | ~5-10MB | ~4KB |
| Sent to server | No (JS-only, client-side) | No | Yes, on every matching HTTP request |
| Scope | Per-origin, shared across tabs | Per-origin, per-tab | Per-origin (+ path/domain rules) |

Cookies are the only one of the three automatically sent with HTTP requests, which is why they're used for session/auth tokens (though modern apps increasingly use `httpOnly` cookies specifically for security — see below) — `localStorage`/`sessionStorage` require explicit JS to read and attach to a request, and are more suited to storing client-side UI state/preferences.

**Q2: Why is storing a JWT/auth token in `localStorage` often discouraged, and what's a safer alternative?**

A: `localStorage` is fully accessible to **any JavaScript running on the page** — meaning a successful XSS attack (malicious script injection) can trivially read and exfiltrate anything stored there, including auth tokens. A safer common pattern: store the auth token in an **`httpOnly` cookie** (inaccessible to JavaScript entirely, only readable by the browser when sending requests to the server), combined with `Secure` (HTTPS-only) and `SameSite` attributes to also mitigate CSRF risk.

**Q3: What is the Canvas API used for, and how does it differ from SVG?**

A: `<canvas>` provides an **immediate-mode, pixel-based** drawing surface controlled entirely via JavaScript — you issue drawing commands (`ctx.fillRect()`, `ctx.drawImage()`) and the canvas has no memory of "shapes" as objects, just resulting pixels; great for games, complex animations, image manipulation, and performance-sensitive rendering with many elements. **SVG** is a **retained-mode, vector-based, DOM-integrated** format — each shape is an actual DOM element you can select/style with CSS and attach event listeners to individually, which is better for interactive diagrams, icons, and content that needs to stay crisp at any zoom level or be individually stylable/accessible.

**Q4: What is the Geolocation API, and what's a key requirement for using it in modern browsers?**

A: `navigator.geolocation.getCurrentPosition(success, error)` requests the user's location, requiring explicit **user permission** via a browser prompt. Modern browsers require the page to be served over **HTTPS** (secure context) to use Geolocation at all — a common gotcha for local development or misconfigured deployments where the API silently fails or is unavailable.

---

## 5. Meta Tags & SEO Basics

**Q1: What are the essential `<meta>` tags every page should have, and what does each do?**

A:
```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="description" content="Concise page summary for search results">
<title>Page Title</title>
```
`charset` declares text encoding (UTF-8 avoids garbled special characters). `viewport` is essential for **responsive design** — without it, mobile browsers render the page at a desktop-width virtual viewport and zoom out, ignoring your media queries entirely. `description` is commonly shown as the snippet text in search engine results (doesn't directly affect ranking much, but affects click-through rate).

**Q2: What are Open Graph tags, and why do they matter?**

A:
```html
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Description shown when shared">
<meta property="og:image" content="https://example.com/preview.jpg">
<meta property="og:url" content="https://example.com/page">
```
Open Graph tags control how a page's link preview appears when shared on social platforms (Facebook, LinkedIn, Slack, etc.) — without them, the platform falls back to guessing from page content, often producing a poor/inconsistent preview.

**Q3: What's the difference between `rel="nofollow"` and `rel="noopener"` on a link?**

A: `nofollow` tells search engines **not to pass SEO ranking credit** through that link (used for untrusted user-generated content, paid/sponsored links). `noopener` is a **security** attribute (unrelated to SEO) — required on links with `target="_blank"` to prevent the newly opened page from gaining a `window.opener` reference back to the originating page, which could otherwise be exploited (a malicious destination page manipulating `window.opener.location` to redirect the original tab in a phishing attack — "tabnabbing"). Modern browsers apply `noopener` behavior by default for `target="_blank"` now, but explicitly adding it remains a recommended defensive practice, especially for older-browser support.

---

# PART 2: CSS

## 6. Box Model, Selectors & Specificity

**Q1: Explain the CSS Box Model and the difference between `content-box` and `border-box`.**

A: Every element is a box composed of, from inside out: **content**, **padding**, **border**, **margin**. `box-sizing: content-box` (the default) means `width`/`height` apply **only to the content area** — padding and border are added **on top**, making the element's total rendered size larger than the specified width. `box-sizing: border-box` makes `width`/`height` include padding and border **within** that specified size — content shrinks to fit, but the box's total footprint matches exactly what you set, which is far more intuitive for layout and is why most CSS resets apply `* { box-sizing: border-box; }` globally.

**Q2: Explain CSS specificity — how is it calculated, and what wins when two rules conflict?**

A: Specificity is calculated as a tuple, roughly **(inline styles, IDs, classes/attributes/pseudo-classes, elements/pseudo-elements)**:
- Inline style (`style="..."`) — highest, beats any selector.
- ID selectors (`#header`) — worth more than any number of classes.
- Class, attribute, pseudo-class selectors (`.btn`, `[type=text]`, `:hover`) — next tier.
- Element and pseudo-element selectors (`div`, `::before`) — lowest tier.

When specificities are equal, the rule that appears **later in the source order** (or is loaded later) wins. `!important` overrides normal specificity entirely (highest priority short of browser/user `!important` styles) — but is generally discouraged as a crutch, since it makes future overrides require even more `!important` (a spiraling maintenance problem), rather than fixing the underlying specificity/architecture issue.

**Q3: What's the difference between `*`, descendant (` `), child (`>`), adjacent sibling (`+`), and general sibling (`~`) combinators?**

A:
```css
* { }           /* universal selector — matches everything */
div p { }       /* descendant — any <p> nested anywhere inside a <div> */
div > p { }     /* direct child — only <p> that is an IMMEDIATE child of <div> */
h2 + p { }      /* adjacent sibling — the <p> immediately following a sibling <h2> */
h2 ~ p { }      /* general sibling — any <p> that is a later sibling of an <h2> (not just the first) */
```

**Q4: What's the difference between pseudo-classes and pseudo-elements, syntactically and conceptually?**

A: **Pseudo-classes** (single colon, `:hover`, `:nth-child()`, `:focus`, `:not()`) select elements based on **state or position** that isn't expressible with a plain selector. **Pseudo-elements** (double colon, `::before`, `::after`, `::first-line`) target a **sub-part of an element** that doesn't otherwise exist as a real DOM node, letting you style/insert content at specific points (`content: "→"` via `::before`). Single-colon syntax for pseudo-elements (`:before`) still works for backward compatibility, but double-colon is the modern, technically-correct syntax and the one to use/recognize in interviews.

---

## 7. Flexbox

**Q1: What's the core mental model of Flexbox, and what are the two axes?**

A: Flexbox lays out children of a container along a **single dimension** at a time — a **main axis** (direction set by `flex-direction`: `row` default, or `column`) and a **cross axis** (perpendicular to it). Properties on the **container** (`justify-content`, `align-items`, `flex-direction`, `flex-wrap`) control alignment/distribution along these axes; properties on **children** (`flex-grow`, `flex-shrink`, `flex-basis`, `align-self`) control how individual items behave within that layout.

**Q2: Explain `justify-content` vs `align-items` — which axis does each control?**

A: `justify-content` aligns items along the **main axis** (horizontally, for the default `row` direction) — `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly`. `align-items` aligns items along the **cross axis** (vertically, for `row`) — `flex-start`, `flex-end`, `center`, `stretch` (default), `baseline`. If `flex-direction: column` is set, these axes **swap** — `justify-content` becomes vertical, `align-items` becomes horizontal.

**Q3: Explain the `flex` shorthand (`flex-grow`, `flex-shrink`, `flex-basis`) with a practical example.**

A: `flex: <grow> <shrink> <basis>` — `flex-grow` (how much an item expands to fill available extra space, relative to siblings' grow values), `flex-shrink` (how much it shrinks when space is insufficient), `flex-basis` (the item's initial size before growing/shrinking is applied, similar to `width` but axis-aware).
```css
.sidebar { flex: 0 0 250px; }  /* never grow, never shrink, fixed 250px base — a rigid sidebar */
.main    { flex: 1 1 auto; }   /* grow to fill remaining space, can shrink, starts from content size */
```
`flex: 1` is common shorthand for `flex: 1 1 0%` — "take an equal share of available space, ignoring content size as the starting point."

**Q4: How do you center an element both horizontally and vertically using Flexbox — the classic interview one-liner?**
```css
.container {
  display: flex;
  justify-content: center; /* horizontal, for row direction */
  align-items: center;     /* vertical, for row direction */
  height: 100vh;
}
```

---

## 8. Grid

**Q1: How does CSS Grid differ fundamentally from Flexbox, and when do you choose one over the other?**

A: Flexbox is **one-dimensional** — it lays out along a single axis at a time, and cross-axis alignment across multiple "rows" (when wrapping) isn't truly grid-aligned (each flex line sizes independently). **Grid** is **two-dimensional** — you define both rows and columns simultaneously, and items can span multiple rows/columns with true alignment across both axes at once. Rule of thumb: use **Flexbox** for laying out a one-dimensional sequence of items (a navbar, a button group, a single row/column of cards) where content size should often drive layout; use **Grid** for overall **page layout** or any genuinely two-dimensional arrangement (a dashboard with distinct header/sidebar/main/footer regions, a photo gallery needing precise row+column alignment).

**Q2: Explain `grid-template-columns`/`grid-template-rows` and the `fr` unit.**

A:
```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr 2fr; /* fixed sidebar, then two flexible columns sharing remaining space 1:2 */
  grid-template-rows: auto 1fr auto;    /* header/footer size to content, middle row fills remaining height */
  gap: 16px;
}
```
`fr` ("fraction") represents a share of the **remaining available space** after fixed-size tracks are accounted for — `1fr 2fr` splits leftover space in a 1:2 ratio, similar in spirit to `flex-grow` but native to Grid's two-dimensional track sizing.

**Q3: How do you place/span an item across multiple grid cells?**
```css
.item {
  grid-column: 1 / 3;  /* start at column line 1, end at column line 3 — spans 2 columns */
  grid-row: 2 / 4;      /* spans 2 rows */
  /* shorthand equivalents: grid-column: span 2; */
}
```

**Q4: What is `grid-template-areas`, and why is it popular for readable layout code?**
```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-areas:
    "sidebar header"
    "sidebar main"
    "sidebar footer";
}
.sidebar { grid-area: sidebar; }
.header  { grid-area: header; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```
It lets you visually "draw" the layout structure directly in CSS as an ASCII-art-like grid of named regions, making the overall page structure immediately readable at a glance — a commonly cited Grid feature that has no real Flexbox equivalent.

---

## 9. Positioning & Stacking Context

**Q1: Explain the difference between `static`, `relative`, `absolute`, `fixed`, and `sticky` positioning.**

A:
- **`static`** (default) — normal document flow; `top`/`left`/etc. have no effect.
- **`relative`** — stays in normal flow (still occupies its original space) but can be **visually offset** from its normal position via `top`/`left`/etc., and establishes a positioning context for absolutely-positioned descendants.
- **`absolute`** — removed from normal flow entirely (doesn't take up space where it "would" be); positioned relative to its **nearest positioned ancestor** (any ancestor with `position` other than `static`) or the viewport/initial containing block if none exists.
- **`fixed`** — removed from flow; positioned relative to the **viewport**, staying in place during scrolling (unless an ancestor has a `transform`/`filter`/`will-change`, which creates a new containing block that "traps" the fixed element — a common gotcha).
- **`sticky`** — behaves like `relative` until the element crosses a specified threshold (`top: 0`) during scroll, then "sticks" like `fixed` within the bounds of its containing block.

**Q2: What is a "stacking context," and how does `z-index` actually work?**

A: `z-index` only has meaning **within** a stacking context — it doesn't simply compare globally across the entire page. A new stacking context is created by, among other triggers: any positioned element (`relative`/`absolute`/`fixed`/`sticky`) with a `z-index` other than `auto`, an element with `opacity < 1`, `transform`, `filter`, or `will-change` set. A common bug: an element with `z-index: 9999` still appears **behind** another element with `z-index: 1`, because the high-z-index element is nested inside a **different (lower) stacking context** — its z-index is only compared against siblings within its own stacking context, not globally.

**Q3: Why do two adjacent vertical margins sometimes "collapse" into a single margin, and how do you prevent it?**

A: **Margin collapsing** happens between vertically-adjacent block-level elements in normal flow — if element A has `margin-bottom: 20px` and the next sibling B has `margin-top: 30px`, the actual gap between them is **30px (the larger), not 50px (the sum)**. This also happens between a parent and its first/last child if there's no padding/border/content separating them. Prevention: add padding or a border to the parent (breaks the adjacency), use `overflow: hidden`/`auto` on the parent (establishes a new block formatting context), or switch to Flexbox/Grid layout (margin collapsing doesn't apply to flex/grid items at all).

---

## 10. Responsive Design

**Q1: What's the difference between "responsive" and "adaptive" design?**

A: **Responsive** design uses fluid grids, flexible images, and media queries to **continuously adapt** the layout across a range of viewport sizes — the layout reflows smoothly as the browser is resized. **Adaptive** design serves a **fixed set of distinct layouts** targeted at specific breakpoints/device categories (e.g., a dedicated mobile layout vs a dedicated desktop layout), switching between them rather than fluidly scaling — historically more common before responsive techniques matured, though the terms are sometimes used loosely/interchangeably today.

**Q2: What's the difference between mobile-first and desktop-first media query strategies, and which is generally recommended?**

A: **Mobile-first** — base styles target the smallest screens, then `min-width` media queries progressively add complexity/layout changes for larger screens:
```css
.card { display: block; }              /* mobile default */
@media (min-width: 768px) { .card { display: flex; } }
```
**Desktop-first** — base styles target large screens, then `max-width` queries simplify for smaller ones. Mobile-first is generally recommended: it forces you to prioritize essential content/functionality first (a good UX discipline), typically produces leaner CSS for mobile users (who often have more constrained networks/devices), and aligns with the reality that mobile traffic dominates for most sites today.

**Q3: What's the difference between `rem`, `em`, `%`, `vw`/`vh` for responsive sizing (also covered in Section 11)?**

A: Briefly: `rem` is relative to the **root** font size (predictable, doesn't compound) — best for consistent spacing/typography scale. `em` is relative to the **parent's** font size (compounds through nesting, can cause surprising size drift). `%` is relative to the parent's corresponding dimension. `vw`/`vh` are relative to the **viewport** dimensions — useful for full-bleed, viewport-relative sizing (hero sections, full-height layouts) but risky for body text (can become illegibly small/large at extreme viewport sizes without `clamp()`).

**Q4: What is `clamp()`, and how does it enable fluid typography without media queries?**
```css
h1 {
  font-size: clamp(1.5rem, 4vw + 1rem, 3rem);
  /* minimum 1.5rem, preferred value scales with viewport, maximum 3rem */
}
```
`clamp(min, preferred, max)` lets a value scale fluidly between explicit min/max bounds based on a viewport-relative preferred value, achieving smooth fluid typography/spacing across all screen sizes **without** needing a series of discrete media-query breakpoints to jump between fixed sizes.

---

## 11. Units & Custom Properties

**Q1: When would you use `px` vs `rem` vs `%`, and what's the general best-practice guidance?**

A: `px` — absolute, predictable, but doesn't scale if a user changes their browser's default font size (an accessibility consideration) — generally best reserved for things that genuinely shouldn't scale with text (border widths, box-shadow offsets). `rem` — the generally recommended default for **font sizes and spacing**, since it respects user font-size preferences (accessibility) while staying predictable (unlike `em`'s compounding). `%` — useful for widths/dimensions relative to a parent container in fluid layouts.

**Q2: What are CSS Custom Properties (CSS variables), and how do they differ from Sass/Less variables?**
```css
:root {
  --primary-color: #3b82f6;
  --spacing-unit: 8px;
}
.button {
  background: var(--primary-color);
  padding: calc(var(--spacing-unit) * 2);
}
```
Key difference: CSS custom properties are **live, runtime values** that exist in the actual DOM/CSSOM — they can be read and changed dynamically via JavaScript (`element.style.setProperty('--primary-color', 'red')`), respect the **cascade** (can be overridden per-scope, e.g., a `.dark-theme` class redefining `--primary-color` for just that subtree), and update the page immediately without recompilation. Sass/Less variables are **compile-time only** — they're resolved once during the CSS build/preprocessing step into static values, with no runtime presence or ability to be read/changed by JavaScript afterward.

**Q3: How would you implement a dark mode toggle using CSS Custom Properties?**
```css
:root {
  --bg-color: white;
  --text-color: black;
}
[data-theme="dark"] {
  --bg-color: #1a1a1a;
  --text-color: white;
}
body {
  background: var(--bg-color);
  color: var(--text-color);
}
```
```js
document.documentElement.setAttribute('data-theme', 'dark');
```
Because the variables cascade and are re-evaluated live, toggling the `data-theme` attribute instantly re-themes every element referencing those variables, with no need to individually update each element's styles in JS.

---

## 12. Animations & Transitions

**Q1: What's the difference between CSS `transition` and `@keyframes` animation?**

A: `transition` animates a property change **between two states**, triggered by something else changing the property (a `:hover`, a class toggle via JS) — it's inherently a simple "from current value to new value" interpolation, with no concept of intermediate steps beyond the two endpoints.
```css
.button { transition: background-color 0.3s ease; }
.button:hover { background-color: darkblue; }
```
`@keyframes` + `animation` defines **multiple explicit steps/states** (0%, 50%, 100%, etc.) and can run **automatically on page load** (no trigger event needed), loop indefinitely, and control more advanced timing (delays, iteration counts, direction reversal) — appropriate for more complex, self-running, or multi-stage animations (a loading spinner, a multi-step entrance animation).

**Q2: Why are `transform` and `opacity` the recommended properties to animate for performance, vs animating `width`/`top`/`left`?**

A: Animating `width`, `height`, `top`, `left`, or `margin` triggers the browser's **layout (reflow)** step on every frame — recalculating the position/size of that element and potentially its siblings/ancestors, which is expensive. `transform` (`translate`, `scale`, `rotate`) and `opacity` can typically be handled entirely on the **GPU compositing layer**, skipping layout and even paint in many cases — resulting in smoother animations, especially on lower-powered devices. This is a very common performance-focused interview question: "how would you animate a moving element efficiently?" → use `transform: translateX()` instead of animating `left`.

**Q3: What does `will-change` do, and why shouldn't you apply it to everything preemptively?**

A: `will-change: transform;` hints to the browser that an element is **about to be animated/changed**, letting it proactively create an optimized rendering layer for that element ahead of time, avoiding a potential jank spike at the moment the animation actually starts. Overusing it (applying to many elements "just in case") **consumes extra GPU memory** per promoted layer and can actually **hurt** performance at scale — it should be applied narrowly, ideally just before the animation begins (e.g., on `:hover` or via JS right before triggering the animation) and removed afterward, not left on indefinitely as a blanket optimization.

---

## 13. CSS Architecture (BEM, Preprocessors)

**Q1: What is BEM (Block, Element, Modifier), and what naming problem does it solve?**

A: BEM is a naming convention: `.block__element--modifier`.
```html
<div class="card">
  <h2 class="card__title">Title</h2>
  <button class="card__button card__button--primary">Buy</button>
</div>
```
- **Block** — a standalone, reusable component (`.card`).
- **Element** — a part of that block, tied to it (`.card__title` — never used outside `.card`'s context).
- **Modifier** — a variant/state of a block or element (`.card__button--primary`).

It solves the problem of **CSS specificity wars and unpredictable cascading** in large codebases — since every BEM class is a single flat class selector with equal specificity, styles rarely need `!important` or fight for precedence, and the naming itself documents the component structure and relationships without needing to read the HTML nesting.

**Q2: What do CSS preprocessors (Sass/Less) offer that vanilla CSS historically didn't, and how much of that gap has modern CSS closed?**

A: Historically, preprocessors offered: **variables** (now native via custom properties, Section 11), **nesting** (now natively supported in modern CSS via **CSS Nesting**), **mixins/functions** for reusable style logic, and **partials/imports** for splitting styles into files that compile into one output (native `@import` exists but has performance drawbacks vs. a build-time bundle). Modern native CSS has closed much of this gap, but preprocessors still offer things CSS doesn't have natively: true **control-flow logic** (`@if`/`@each`/`@for` loops for programmatically generating styles), more powerful mixins with default parameters, and mature tooling/linting ecosystems — many teams still use Sass, but the "you need a preprocessor for X" list has shrunk significantly.

**Q3: What are CSS Modules or CSS-in-JS solving, in contrast to BEM's naming-convention approach?**

A: BEM solves naming collisions/specificity through **developer discipline** (a convention you must consistently follow) — nothing technically prevents a typo or forgetting the convention. **CSS Modules** (build-tool feature that auto-generates unique class names per file/component, e.g., `.card__title` becomes `.card__title_a3f8x` at build time) and **CSS-in-JS** (styles defined directly in JS/component files, scoped automatically to that component) solve the same underlying problem — accidental global collisions — but **enforce** scoping mechanically via tooling, rather than relying purely on naming discipline, which is generally considered more robust for large teams/codebases at the cost of added build complexity.

---

# PART 3: JAVASCRIPT

## 14. Fundamentals — Types, `var`/`let`/`const`, Hoisting

**Q1: What are JavaScript's primitive types, and how do primitives differ from objects?**

A: Primitives: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`. Primitives are **immutable** and compared **by value** (`"a" === "a"` is `true`). Everything else (objects, arrays, functions) is a **reference type** — compared **by reference**, not value (`{} === {}` is `false`, since they're two distinct object references even with identical contents), and mutable (you can add/change properties without creating a new reference).

**Q2: What's the difference between `var`, `let`, and `const`?**

A:
- **`var`** — function-scoped (or global if declared outside a function), **hoisted** and initialized to `undefined` at the top of its scope (accessible, but `undefined`, before the declaration line runs), and can be **redeclared** in the same scope without error.
- **`let`** — **block-scoped** (`{}`), hoisted but **not initialized** — accessing it before its declaration throws a `ReferenceError` (this gap is called the **"Temporal Dead Zone"**), and cannot be redeclared in the same scope.
- **`const`** — same block-scoping/TDZ behavior as `let`, but **cannot be reassigned** after initialization. Note: this only prevents **reassignment of the variable binding**, not mutation of an object/array it references — `const arr = []; arr.push(1);` is perfectly legal.

**Q3: What is hoisting, and how does it differ for `var` declarations, function declarations, and `let`/`const`?**

A: Hoisting is the JS engine's behavior of processing declarations during a **compile pass** before line-by-line execution — but what actually gets "hoisted" differs:
- **`var`** — declaration is hoisted **and initialized to `undefined`**; usable (as `undefined`) before its line.
- **Function declarations** (`function foo() {}`) — the **entire function** is hoisted, fully usable (callable) before its line in the source.
- **`let`/`const`** — the declaration is hoisted in the sense that the engine knows the binding exists (which is why shadowing behaves consistently), but it's **not initialized** until its declaration line executes — accessing it earlier throws due to the Temporal Dead Zone, rather than silently returning `undefined`.

**Q4: What's the difference between `==` and `===`, and what's the general guidance on which to use?**

A: `===` (strict equality) compares value **and type**, with no conversion. `==` (loose equality) performs **type coercion** before comparing, following a set of (somewhat notorious) coercion rules — e.g., `"5" == 5` is `true`, `null == undefined` is `true`, but `null == 0` is `false` (a common trick question, since `null`'s coercion rules are special-cased). General guidance: **always use `===`** unless you have a specific, well-understood reason to rely on coercion (the one commonly accepted exception is `x == null`, which conveniently checks for both `null` and `undefined` in one comparison) — relying on `==`'s full coercion table is a well-known source of subtle bugs.

**Q5: What's the difference between `null` and `undefined`?**

A: `undefined` means a variable has been **declared but not assigned a value** (or a function with no explicit `return` implicitly returns it, or accessing a non-existent object property/array index). `null` is an **explicit, intentional "no value"** assignment by the developer — signaling "this is deliberately empty," as opposed to `undefined`'s "this was never set." `typeof null` famously returns `"object"` — a long-standing, well-known bug preserved in the language for backward compatibility, and a common interview trivia question.

---

## 15. Functions, Closures & Scope

**Q1: What is a closure, and give a practical example of why it's useful?**

A: A closure is a function that **retains access to variables from its enclosing (outer) scope**, even after that outer function has finished executing — the inner function "closes over" those variables rather than losing them when the outer function's execution context is popped off the call stack.
```javascript
function createCounter() {
  let count = 0; // private to this closure
  return {
    increment: () => ++count,
    getCount: () => count,
  };
}
const counter = createCounter();
counter.increment(); counter.increment();
console.log(counter.getCount()); // 2 — `count` persisted across calls, and is inaccessible from outside
```
This is the standard mechanism for creating **private state/encapsulation** in JavaScript without classes — `count` can only be read/modified through the returned methods, not accessed directly from outside.

**Q2: What's the classic "closures in a loop" bug with `var`, and how does `let` fix it?**
```javascript
// BAD — logs 3, 3, 3 (all three closures share the SAME `i` variable, which ends at 3)
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}

// GOOD — logs 0, 1, 2 (let creates a NEW binding of `i` for each loop iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
```
Since `var` is function-scoped, all three `setTimeout` callbacks close over the **same single `i` variable**, which has already reached `3` by the time any callback actually runs. `let` is block-scoped **per iteration** — the loop creates a fresh binding each pass, so each closure captures its own distinct `i`. This is one of the most frequently asked "explain this output" JS interview questions.

**Q3: What's the difference between function scope and block scope?**

A: **Function scope** (the only scoping `var` respects) — a variable is visible throughout the entire function it's declared in, regardless of nested `{}` blocks (`if`, `for`, etc.) within that function. **Block scope** (`let`/`const`) — a variable is only visible within the nearest enclosing `{}` pair, including blocks that aren't function bodies (an `if` block, a bare `{}`, a `for` loop's body).

**Q4: What's the difference between an arrow function and a regular function regarding `this`?**

A: This is covered in depth in Section 16, but briefly: a regular `function` gets its own `this`, determined by **how it's called** (dynamic binding). An **arrow function has no `this` of its own** — it lexically inherits `this` from its enclosing scope at the point it's **defined**, not called. This makes arrow functions especially useful for callbacks nested inside a method, where you want `this` to still refer to the surrounding object rather than being reset by the callback's own invocation context.

---

## 16. `this`, `call`/`apply`/`bind`

**Q1: How is `this` determined in a regular function — what are the four/five binding rules?**

A: `this` is determined by **how a function is called**, not where it's defined (a critical distinction from lexical scoping):
1. **Default binding** — a plain function call (`foo()`) — `this` is `undefined` in strict mode, or the global object (`window`) in non-strict mode.
2. **Implicit binding** — called as a method on an object (`obj.foo()`) — `this` is `obj`.
3. **Explicit binding** — `call`/`apply`/`bind` explicitly set `this`.
4. **`new` binding** — calling a function with `new` — `this` is the newly created object.
5. **Arrow functions** — ignore all of the above; they inherit `this` lexically from their enclosing scope.

**Q2: What's the difference between `call`, `apply`, and `bind`?**

A: All three explicitly set what `this` refers to inside a function, but differ in invocation/return behavior:
```javascript
function greet(greeting) { console.log(`${greeting}, ${this.name}`); }
const person = { name: 'Alice' };

greet.call(person, 'Hello');   // invokes IMMEDIATELY, args passed individually: "Hello, Alice"
greet.apply(person, ['Hi']);   // invokes IMMEDIATELY, args passed as an ARRAY: "Hi, Alice"
const bound = greet.bind(person); // does NOT invoke — returns a NEW function with `this` permanently bound
bound('Hey'); // "Hey, Alice" — can be called later, `this` can't be re-overridden even by another .call()
```

**Q3: Give a classic real-world example of a `this` bug and how `bind`/arrow functions fix it.**
```javascript
class Button {
  constructor() { this.label = 'Submit'; }
  handleClick() { console.log(this.label); }
}
const btn = new Button();
element.addEventListener('click', btn.handleClick); // BUG: `this` inside handleClick is now the DOM element, not `btn`!

// Fix 1: bind in the constructor
this.handleClick = this.handleClick.bind(this);

// Fix 2: use an arrow function class field (inherits `this` lexically from the constructor's `this`)
handleClick = () => { console.log(this.label); };
```
When a method is passed as a **bare reference** (not called as `obj.method()`), it loses its implicit binding to the object — the caller (here, the event listener system) determines `this` at call time based on how it invokes the function, which for DOM event listeners is the element that fired the event.

---

## 17. Prototypes, Inheritance & Classes

**Q1: What is the prototype chain, and how does property lookup work?**

A: Every JS object has an internal link (`[[Prototype]]`, accessible via `Object.getPrototypeOf()` or the deprecated `__proto__`) to another object — when you access a property, the engine first checks the object itself, and if not found, walks **up the prototype chain** checking each linked prototype in turn, until it finds the property or reaches `null` (the end of the chain, `Object.prototype`'s prototype). This is how methods defined on `Array.prototype` (like `.map()`) are available on every array instance without being copied onto each one individually.

**Q2: How do ES6 `class`es relate to prototypes — are classes "real" classes like in Java, or syntactic sugar?**

A: ES6 `class` syntax is **syntactic sugar over the existing prototype-based inheritance system** — it doesn't introduce a fundamentally new object model, just a cleaner syntax for what was previously done manually with constructor functions and explicit `.prototype` manipulation:
```javascript
class Animal {
  constructor(name) { this.name = name; }
  speak() { console.log(`${this.name} makes a sound`); }
}
class Dog extends Animal {
  speak() { console.log(`${this.name} barks`); }
}
// under the hood, this is still prototype-chain-based:
// Dog.prototype.__proto__ === Animal.prototype
```
Unlike classical (e.g., Java) class-based inheritance where classes are a distinct compile-time construct, JS classes are still fundamentally objects and prototype links at runtime — you can still inspect/manipulate the prototype chain directly if needed, which isn't possible in truly class-based languages.

**Q3: What does `super` do inside a subclass, and why must `super()` be called before using `this` in a subclass constructor?**

A: `super(...)` calls the **parent class's constructor**, and `super.methodName()` calls a method from the parent class's prototype (useful for extending, not fully replacing, inherited behavior). In a subclass constructor, `this` is **not initialized** until `super()` has been called — attempting to use `this` before that throws a `ReferenceError`, because the parent constructor is responsible for actually setting up the base object that `this` will refer to.

**Q4: What's the difference between `Object.create()`, a constructor function with `new`, and `class` for creating objects with shared behavior?**

A: `Object.create(proto)` creates a new object with an **explicitly specified prototype**, without invoking any constructor function — the most direct, low-level way to set up prototypal inheritance. A **constructor function** (`function Animal(name) { this.name = name; }`, called with `new`) combines object creation with initialization logic, with shared methods manually attached to `Animal.prototype`. **`class`** is the same underlying mechanism as the constructor-function approach, just with cleaner, more familiar syntax (and some genuine behavioral differences, like classes not being hoisted the same way and always running in strict mode). All three ultimately produce objects linked via the same prototype chain mechanism.

---

## 18. Event Loop & Asynchronous JS

**Q1: Explain the JavaScript Event Loop — what are the call stack, task queue (macrotask), and microtask queue?**

A: JavaScript is **single-threaded** — one **call stack** executes one thing at a time. Async operations (timers, I/O, promise resolution) don't block that stack; instead, their callbacks are queued elsewhere and the **event loop** continuously checks: "is the call stack empty? If so, pull the next task from a queue and run it." Two relevant queues:
- **Microtask queue** — Promise `.then()`/`.catch()`/`.finally()` callbacks, `queueMicrotask()`. **Fully drained** (every microtask, including ones added by other microtasks) **before** the event loop proceeds to the next macrotask.
- **Macrotask (task) queue** — `setTimeout`, `setInterval`, I/O callbacks, UI rendering. One macrotask runs per event loop "tick," after which the entire microtask queue is drained again before the next macrotask.

**Q2: Predict the output — a classic interview "trace the execution order" question.**
```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1, 4, 3, 2
```
Explanation: synchronous code (`1`, `4`) always runs first, completing the initial call stack. Then, **before** any macrotask (`setTimeout`, even with a 0ms delay) runs, the **entire microtask queue is drained** — so the Promise's `.then()` (`3`) runs before the `setTimeout` callback (`2`), regardless of the nominal 0ms delay.

**Q3: How does `async`/`await` relate to Promises under the hood?**

A: `async`/`await` is **syntactic sugar over Promises** — an `async function` always returns a Promise (wrapping the return value, or a rejected Promise if it throws), and `await` pauses execution of that function (without blocking the main thread — other code can still run) until the awaited Promise settles, then resumes with either the resolved value or throws the rejection as a catchable exception. This makes asynchronous code read like synchronous code, avoiding the nested "callback pyramid" or long `.then()` chains of pure Promise-based code.

**Q4: How do you correctly handle errors in `async`/`await` code, and how does that differ from Promise `.catch()`?**
```javascript
// try/catch wraps await — the idiomatic async/await error handling pattern
async function fetchUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error('Failed to fetch');
    return await res.json();
  } catch (err) {
    console.error('Error fetching user:', err);
    throw err; // re-throw if the caller needs to know, or return a fallback
  }
}
```
A rejected `await`ed Promise throws a **catchable synchronous-style exception** within the `async` function, so ordinary `try/catch` works — a common bug is forgetting this and expecting unhandled rejections to be silently swallowed, when they actually propagate as a rejected Promise from the `async` function itself (and should be handled by the caller, or you'll get an "Unhandled Promise Rejection" warning).

**Q5: What's the difference between `Promise.all()`, `Promise.allSettled()`, `Promise.race()`, and `Promise.any()`?**

A:
- **`Promise.all()`** — waits for **all** to resolve; rejects **immediately** as soon as **any one** rejects (fail-fast), discarding the results of the others.
- **`Promise.allSettled()`** — waits for **all** to settle (resolve or reject), never short-circuits, and returns an array of `{status, value/reason}` objects for every promise — useful when you want results/errors from all operations regardless of individual failures.
- **`Promise.race()`** — settles (resolves or rejects) as soon as the **first** promise settles, whichever that is.
- **`Promise.any()`** — resolves as soon as the **first** promise **resolves** (ignoring rejections along the way); only rejects if **all** promises reject (with an `AggregateError` containing all the individual errors).

---

## 19. DOM Manipulation & Events

**Q1: What's the difference between event bubbling and event capturing, and what's the default?**

A: When an event fires on an element, it propagates through the DOM tree in two possible phases: **capturing** (top-down, from the root document down to the target element) and **bubbling** (bottom-up, from the target back up to the root). By default, `addEventListener(type, handler)` listens during the **bubbling** phase — pass a third argument `{ capture: true }` (or `true`) to listen during capturing instead. Most event handling relies on bubbling, which is also what makes **event delegation** possible.

**Q2: What is event delegation, and why is it a recommended pattern for lists of similar elements?**
```javascript
// Instead of attaching a listener to EVERY <li>, attach ONE to the parent
document.querySelector('ul').addEventListener('click', (e) => {
  if (e.target.matches('li')) {
    console.log('Clicked:', e.target.textContent);
  }
});
```
Because events bubble up, a single listener on a **common ancestor** can handle events from any current **or future** descendant matching a selector — this is more memory-efficient than attaching individual listeners to potentially hundreds of list items, and automatically works for elements **added dynamically later** (no need to re-attach listeners to new items), which is a real practical advantage, not just a performance micro-optimization.

**Q3: What do `e.preventDefault()` and `e.stopPropagation()` each do, and how do they differ?**

A: `preventDefault()` stops the browser's **default action** for that event (e.g., a link's navigation, a form's submission, a checkbox's toggling) but does **not** stop the event from continuing to bubble/propagate to ancestor listeners. `stopPropagation()` stops the event from **continuing to bubble (or capture)** further up (or down) the DOM tree — ancestor listeners for that event type won't fire at all — but does **not** prevent the browser's default action. They're independent and are frequently confused; you can call either, neither, or both depending on what you need.

**Q4: What's the difference between `element.innerHTML`, `element.textContent`, and `element.innerText`?**

A: `innerHTML` gets/sets content **parsed as HTML** — setting it with untrusted user input is a classic **XSS vulnerability** (injected `<script>`/event-handler attributes can execute). `textContent` gets/sets the **raw text content** of all descendant nodes (including hidden elements, script/style tag contents), always treating the input as plain text (safe from injection) — and is generally faster since it doesn't trigger HTML parsing/re-layout consideration for hidden elements. `innerText` is similar to `textContent` but is **aware of applied CSS/rendering** (won't return text from `display: none` elements, respects line breaks visually) — and its rendering-awareness makes it more expensive (triggers a reflow to compute) and less predictable across browsers than `textContent`.

**Q5: How would you safely insert user-provided text into the DOM to avoid XSS?**
```javascript
// SAFE — treated as plain text, no HTML parsing
element.textContent = userInput;

// UNSAFE if userInput isn't sanitized — HTML/script gets parsed and can execute
element.innerHTML = userInput;
```
If you genuinely need to insert user-provided **HTML** (rich text), it must first be run through a sanitization library (e.g., DOMPurify) to strip dangerous tags/attributes before ever being assigned to `innerHTML`.

---

## 20. ES6+ Features

**Q1: Explain destructuring with examples for both objects and arrays, including defaults and renaming.**
```javascript
// Object destructuring, with renaming and a default value
const { name: userName, age = 18 } = { name: 'Alice' };
// userName = 'Alice', age = 18 (default, since not present in the object)

// Array destructuring, with skipping and rest
const [first, , third, ...rest] = [1, 2, 3, 4, 5];
// first = 1, third = 3, rest = [4, 5]

// Common in function parameters, e.g. React props:
function UserCard({ name, age = 18 }) { /* ... */ }
```

**Q2: What's the difference between the spread operator (`...`) and rest parameters, given they use the same syntax?**
```javascript
// Spread — EXPANDS an iterable into individual elements
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]
const merged = { ...obj1, ...obj2 }; // shallow-merges objects, later properties win on conflict

// Rest — COLLECTS remaining elements/arguments into a single array
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }
sum(1, 2, 3); // nums = [1, 2, 3]
```
Same `...` syntax, opposite direction: spread **expands** a collection out; rest **gathers** individual items into a collection. Context (which side of an assignment, or in a function signature) determines which behavior applies.

**Q3: What are template literals, and what capabilities do they add beyond string concatenation?**
```javascript
const name = 'Alice';
const greeting = `Hello, ${name}!
This spans multiple lines natively.`; // no \n needed for multi-line strings

// Tagged templates — a function processes the literal's parts before final assembly
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => `${acc}${str}${values[i] ? `<b>${values[i]}</b>` : ''}`, '');
}
highlight`Hello, ${name}!`; // "Hello, <b>Alice</b>!"
```
Beyond string interpolation and native multi-line support, **tagged templates** let a function intercept and transform a template literal's static and interpolated parts separately — used by libraries like styled-components (CSS-in-JS) and for safe HTML/SQL escaping utilities.

**Q4: What are ES modules (`import`/`export`), and how do they differ from CommonJS (`require`/`module.exports`)?**

A: ES modules are **statically analyzable** — imports/exports must be at the top level (not conditional/dynamic, aside from `import()` for dynamic imports), which lets bundlers perform **tree-shaking** (excluding unused exports from the final bundle) at build time. CommonJS (`require`) is resolved **dynamically at runtime**, can be called conditionally anywhere in code, and doesn't support tree-shaking as cleanly since the bundler can't statically know what's actually used without executing code. ES modules are also asynchronous-capable by design (`import()` returns a Promise) and are now natively supported in both modern browsers (`<script type="module">`) and Node.js, gradually displacing CommonJS as the standard.

**Q5: What's optional chaining (`?.`) and nullish coalescing (`??`), and how do they differ from `||`?**
```javascript
const city = user?.address?.city;        // safely returns undefined if user or address is null/undefined, instead of throwing
const displayName = user.name ?? 'Guest'; // uses 'Guest' ONLY if user.name is null or undefined
const count = 0 || 10;  // 10 — || treats ANY falsy value (0, '', NaN) as "use the fallback"
const count2 = 0 ?? 10; // 0  — ?? ONLY falls back for null/undefined, correctly preserving legitimate 0/''
```
`??` was introduced specifically to fix a common `||` bug: using `||` for defaults incorrectly overrides legitimately falsy-but-valid values like `0`, `''`, or `false` — `??` only triggers its fallback for `null`/`undefined` specifically.

---

## 21. Array & Object Methods

**Q1: `map()` vs `forEach()` vs `filter()` vs `reduce()` — what does each return and when do you use it?**

A:
- **`map()`** — transforms each element, returns a **new array of the same length**. Use when you need a transformed version of every element.
- **`forEach()`** — iterates for **side effects only** (logging, mutating external state); returns `undefined` — can't be chained, and shouldn't be used when you actually want a transformed array back (that's what `map` is for).
- **`filter()`** — returns a **new array containing only elements matching a predicate** (possibly shorter, never longer).
- **`reduce()`** — folds the array down into a **single accumulated value** (could be a number, an object, even another array) via a callback that receives an accumulator and the current element.

**Q2: Implement a word-frequency counter using `reduce()`.**
```javascript
const words = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple'];
const counts = words.reduce((acc, word) => {
  acc[word] = (acc[word] || 0) + 1;
  return acc;
}, {});
// { apple: 3, banana: 2, cherry: 1 }
```

**Q3: What's the difference between `Object.keys()`, `Object.values()`, and `Object.entries()`?**
```javascript
const obj = { a: 1, b: 2 };
Object.keys(obj);   // ['a', 'b']
Object.values(obj);  // [1, 2]
Object.entries(obj); // [['a', 1], ['b', 2]] — commonly used with destructuring in a for...of loop
for (const [key, value] of Object.entries(obj)) { console.log(key, value); }
```

**Q4: What's the difference between shallow copy and deep copy, and how do you perform each?**
```javascript
// Shallow copy — nested objects/arrays are still SHARED references
const shallow = { ...original };          // spread
const shallow2 = Object.assign({}, original);

// Deep copy — nested structures are fully independent
const deep = structuredClone(original);   // modern, built-in (Node 17+/modern browsers)
const deepOld = JSON.parse(JSON.stringify(original)); // older workaround — loses functions, undefined, Dates become strings, etc.
```
A shallow copy duplicates only the **top-level** properties — if a property's value is itself an object/array, both the original and the copy still point to the **same nested reference**, so mutating a nested object through either one affects both. `structuredClone()` (widely available now) is the modern, correct built-in way to deep-clone most data structures, superseding the old `JSON.parse(JSON.stringify())` hack, which silently breaks on functions, `undefined` values, `Date` objects (converted to strings), and circular references.

**Q5: How do `find()`, `findIndex()`, `some()`, and `every()` differ?**
```javascript
const nums = [1, 2, 3, 4, 5];
nums.find(n => n > 3);      // 4  — the first MATCHING ELEMENT (or undefined)
nums.findIndex(n => n > 3); // 3  — the INDEX of the first match (or -1)
nums.some(n => n > 3);      // true  — does AT LEAST ONE element match?
nums.every(n => n > 3);     // false — do ALL elements match?
```
All four **short-circuit** — stop iterating as soon as the answer is determined (unlike `map`/`filter`, which always process every element).

---

## 22. Error Handling

**Q1: What's the difference between a syntax error, a runtime error, and a logical error?**

A: **Syntax errors** — code that violates the language grammar, caught at **parse time**, before any code runs (`function( {` — unclosed). **Runtime errors** — code that's syntactically valid but fails during execution (`null.someProperty` — `TypeError`, calling an undefined function — `ReferenceError`). **Logical errors** — code that runs without throwing anything, but produces an **incorrect result** due to a mistake in the logic (e.g., using `<` instead of `<=` in a loop boundary) — the hardest category to catch automatically, since nothing "fails" from the engine's perspective.

**Q2: How does `try/catch/finally` work, and when does `finally` execute?**
```javascript
function process() {
  try {
    riskyOperation();
    return 'success';
  } catch (err) {
    console.error(err);
    return 'failed';
  } finally {
    console.log('cleanup runs regardless');
    // finally runs even if try/catch return, or even if an uncaught error is about to propagate
  }
}
```
`finally` runs **no matter what** — whether the `try` block completes normally, throws and is caught, or even if a `return` statement executed inside `try`/`catch` — making it the right place for guaranteed cleanup logic (closing a connection, hiding a loading spinner) regardless of success/failure.

**Q3: How do you create and use a custom Error type?**
```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

try {
  throw new ValidationError('Email is required', 'email');
} catch (err) {
  if (err instanceof ValidationError) {
    console.log(`Validation failed on ${err.field}: ${err.message}`);
  } else {
    throw err; // re-throw errors this catch block doesn't know how to handle
  }
}
```
Extending the built-in `Error` class preserves the standard `message`/`stack` properties while letting you attach custom, domain-specific data (`field`) and check the error's specific type via `instanceof` — a common pattern for distinguishing expected/handleable errors (validation failures) from truly unexpected ones that should propagate/crash loudly.

**Q4: What's an "unhandled promise rejection," and how do you catch it globally as a safety net?**
```javascript
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
  // report to an error-tracking service, e.g., Sentry
});
```
A rejected Promise that has **no `.catch()`** (or isn't inside a `try/catch` if awaited) anywhere in its chain triggers this warning — in Node.js, unhandled rejections can even crash the process in newer versions. This global listener is a defensive **last resort** for logging/monitoring, not a substitute for actually handling errors at the appropriate point in your code.

---

## 23. Most-Asked JavaScript Coding Questions

**Q1: Debounce a function.**
```javascript
function debounce(fn, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}
// usage: const debouncedSearch = debounce(searchAPI, 300);
```
Debounce delays execution until the function **stops being called for `delay` ms** — every new call resets the timer. Classic use case: search-as-you-type, waiting until the user pauses typing before firing an API call.

**Q2: Throttle a function.**
```javascript
function throttle(fn, limit) {
  let inThrottle = false;
  return function (...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => { inThrottle = false; }, limit);
    }
  };
}
```
Throttle ensures a function runs **at most once per `limit` ms**, regardless of how many times it's called in that window — used for scroll/resize handlers where you want regular, rate-limited execution rather than "wait until it stops" (debounce's behavior).

**Q3: Deep clone an object (without `structuredClone`, as a manual/recursive implementation — a common follow-up).**
```javascript
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (Array.isArray(obj)) return obj.map(deepClone);
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [key, deepClone(value)])
  );
}
```

**Q4: Flatten a nested array.**
```javascript
function flatten(arr) {
  return arr.reduce((flat, item) =>
    flat.concat(Array.isArray(item) ? flatten(item) : item), []);
}
// Or, natively: arr.flat(Infinity);
```

**Q5: Implement `Array.prototype.map` from scratch (a common "implement a built-in" question).**
```javascript
Array.prototype.myMap = function (callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this) result.push(callback(this[i], i, this));
  }
  return result;
};
```

**Q6: Check if a string has balanced parentheses.**
```javascript
function isBalanced(str) {
  const stack = [];
  const pairs = { ')': '(', ']': '[', '}': '{' };
  for (const char of str) {
    if (['(', '[', '{'].includes(char)) {
      stack.push(char);
    } else if (char in pairs) {
      if (stack.pop() !== pairs[char]) return false;
    }
  }
  return stack.length === 0;
}
```

**Q7: Implement a simple `curry` function.**
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn.apply(this, args);
    return (...moreArgs) => curried.apply(this, [...args, ...moreArgs]);
  };
}
const add3 = curry((a, b, c) => a + b + c);
add3(1)(2)(3);   // 6
add3(1, 2)(3);   // 6
add3(1, 2, 3);   // 6
```

**Q8: Find duplicates in an array.**
```javascript
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();
  for (const item of arr) {
    if (seen.has(item)) duplicates.add(item);
    seen.add(item);
  }
  return [...duplicates];
}
```

**Q9: Implement `Promise.all` from scratch (a very common senior-level question).**
```javascript
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;
    if (promises.length === 0) resolve([]);
    promises.forEach((p, index) => {
      Promise.resolve(p)
        .then(value => {
          results[index] = value; // preserve original order, not completion order
          completed++;
          if (completed === promises.length) resolve(results);
        })
        .catch(reject); // fail-fast, matching native Promise.all behavior
    });
  });
}
```

**Q10: Write a simple event emitter (pub/sub) class.**
```javascript
class EventEmitter {
  constructor() { this.events = {}; }
  on(event, listener) {
    (this.events[event] ??= []).push(listener);
    return () => this.off(event, listener); // return an unsubscribe function
  }
  off(event, listener) {
    this.events[event] = (this.events[event] || []).filter(l => l !== listener);
  }
  emit(event, ...args) {
    (this.events[event] || []).forEach(listener => listener(...args));
  }
}
```

---

## Quick-Reference Cheat Sheet

| Topic | One-line takeaway |
|---|---|
| Semantic HTML | Use `<nav>`/`<article>`/`<section>` for meaning; `<div>` only for pure structure |
| Accessibility | Prefer native elements over ARIA; empty `alt=""` for decorative images |
| Storage | Cookies auto-send to server; `localStorage`/`sessionStorage` don't; avoid tokens in `localStorage` (XSS risk) |
| Box model | `border-box` includes padding/border in width — set it globally |
| Specificity | inline > ID > class/attribute/pseudo-class > element; later source order breaks ties |
| Flexbox | One-dimensional; `justify-content` = main axis, `align-items` = cross axis |
| Grid | Two-dimensional; `fr` unit splits remaining space; `grid-template-areas` for readable layout |
| Positioning | `absolute` positions relative to nearest non-static ancestor; new stacking context breaks `z-index` expectations |
| Responsive | Mobile-first with `min-width` queries; `clamp()` for fluid sizing without breakpoints |
| Animations | Animate `transform`/`opacity` (GPU-friendly), not `top`/`width` (triggers layout) |
| `var`/`let`/`const` | `var` = function-scoped, hoisted+undefined; `let`/`const` = block-scoped, Temporal Dead Zone |
| Closures | Inner function retains access to outer scope variables — the mechanism behind private state |
| `this` | Determined by call-site, not definition-site (except arrow functions, which are lexical) |
| Prototypes | `class` is sugar over prototype chains; property lookup walks up `[[Prototype]]` |
| Event loop | Microtasks (Promises) fully drain before each macrotask (`setTimeout`) — even a 0ms timeout loses to `.then()` |
| Event delegation | Attach one listener to a parent, check `e.target` — works for dynamically added children too |
| `??` vs `||` | `??` only falls back on `null`/`undefined`; `||` incorrectly falls back on any falsy value (`0`, `''`) |
| `map`/`filter`/`reduce` | `map` transforms (same length), `filter` selects (shorter/equal), `reduce` folds to one value |
| Shallow vs deep copy | Spread/`Object.assign` = shallow (nested refs shared); `structuredClone` = deep |

---

*Tip: front-end interviews love "explain this output" trick questions (closures in loops, `==` coercion, event loop ordering, `this` binding). When you get one, narrate your reasoning step by step out loud rather than just stating the final answer — that's usually what's actually being evaluated.*
