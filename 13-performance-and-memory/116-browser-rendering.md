# Lesson 116 — Browser Rendering, Reflow and Repaint

Browser rendering performance is about how changes to HTML, CSS, and JavaScript become pixels on the screen.

A useful high-level pipeline is:

```text
HTML
↓
DOM

CSS
↓
CSSOM

DOM + CSSOM
↓
Render Tree
↓
Layout
↓
Paint
↓
Composite
```

Different changes can trigger different parts of this pipeline.

---

## 1. DOM Construction

The browser parses HTML into the DOM.

Example:

```html
<div class="card">
  <h2>Hello</h2>
</div>
```

becomes an object tree the browser can work with.

---

## 2. CSSOM

CSS is parsed into a structure representing style rules.

Conceptually:

```text
CSS text
↓
parsed rules
↓
CSSOM
```

The browser combines document structure and styles to determine what should appear.

---

## 3. Render Tree

The render tree represents visual elements that participate in rendering.

Not every DOM node necessarily appears directly in the render tree.

Example:

```css
.hidden {
  display: none;
}
```

An element with `display: none` is excluded from layout/rendering participation.

---

## 4. Layout

Layout determines geometry such as:

- width
- height
- position
- relationship to surrounding elements

Conceptually:

```text
What size is each box?
Where does each box go?
```

Layout is sometimes casually called **reflow**.

---

## 5. Paint

Paint determines how visual parts are drawn:

- text
- backgrounds
- borders
- shadows
- images

Conceptually:

```text
layout says where
↓
paint says what pixels
```

---

## 6. Composite

Modern browsers often split content into layers.

Compositing combines those layers into the final image shown on screen.

Changes to properties such as:

- `transform`
- `opacity`

can often be handled at the compositing stage without requiring full layout.

This is one reason they are common in smooth animations.

---

## 7. What Triggers Layout?

Layout may be needed when geometry changes.

Examples:

```js
element.style.width =
  "500px";

element.style.height =
  "200px";
```

Other examples include changes affecting:

- font size
- padding
- margin
- borders
- display
- content dimensions

The exact browser behavior depends on context.

---

## 8. What Triggers Paint?

Visual changes that do not necessarily alter geometry may still require repainting.

Examples:

```js
element.style.backgroundColor =
  "red";

element.style.color =
  "white";
```

Conceptually:

```text
geometry unchanged
↓
pixels changed
↓
paint
```

---

## 9. Layout Can Cause Paint

If layout changes where elements are placed, the browser may need to repaint affected areas too.

So:

```text
layout
often implies
paint
```

but a paint-only change does not necessarily require layout.

---

## 10. Forced Synchronous Layout

A common performance problem happens when code:

1. writes a layout-affecting change
2. immediately reads layout information

Example:

```js
element.style.width =
  "300px";

console.log(
  element.offsetWidth
);
```

The browser may need to calculate layout immediately so it can return an accurate value.

---

## 11. Layout Thrashing

Bad pattern:

```js
for (
  const item
  of items
) {
  item.style.width =
    "200px";

  console.log(
    item.offsetWidth
  );
}
```

Repeatedly mixing writes and reads can force repeated layout work.

This is called **layout thrashing**.

---

## 12. Better Read/Write Grouping

Prefer:

```text
read layout values
↓
calculate
↓
write styles
```

rather than:

```text
write
read
write
read
write
read
```

Batching work helps the browser optimize rendering.

---

## 13. Common Layout Reads

Properties/methods that may require up-to-date geometry include:

```js
element.offsetWidth
element.offsetHeight
element.clientWidth
element.clientHeight
element.getBoundingClientRect()
```

They are not "bad", but repeated reads after writes can become expensive.

---

## 14. requestAnimationFrame()

`requestAnimationFrame()` schedules visual work before the browser's next repaint opportunity.

```js
requestAnimationFrame(
  () => {
    element.style.transform =
      "translateX(100px)";
  }
);
```

It is usually better suited to visual animation work than arbitrary timer loops.

---

## 15. Why setInterval Is Poor for Animation

```js
setInterval(
  updateAnimation,
  16
);
```

does not synchronize directly with the browser's paint cycle.

The tab may:

- be throttled
- be busy
- miss frames

`requestAnimationFrame()` lets the browser coordinate better.

---

## 16. Frame Budget

For a 60 Hz display, one frame is roughly:

```text
16.7ms
```

Within that time, the browser may need to do:

- JavaScript
- style calculation
- layout
- paint
- composite

If work takes too long, frames are missed and animation feels janky.

---

## 17. 120 Hz Displays

At 120 Hz, the frame window is closer to:

```text
8.3ms
```

So hardcoding "16ms is always enough" is not a universal performance rule.

The broader principle is:

> keep per-frame work small.

---

## 18. transform vs top/left

Animating:

```css
transform: translateX(...)
```

often avoids layout more effectively than repeatedly changing:

```css
left: ...
```

because transforms can frequently be handled by compositing.

But actual performance depends on browser/layout context.

---

## 19. opacity

Animating opacity is often efficient because it typically avoids layout.

Example:

```css
opacity: 0.5;
```

Again, do not treat this as a guarantee for every browser scenario.

---

## 20. Too Many Layers

Compositing layers also cost memory.

Forcing layers everywhere with hacks such as:

```css
will-change: transform;
```

can backfire.

Use `will-change` only where a real upcoming change justifies it.

---

## 21. DOM Size Matters

Very large DOM trees can increase work for:

- style calculation
- layout
- memory
- accessibility tree processing

Strategies include:

- pagination
- virtualization
- conditional rendering

---

## 22. Virtualization

Instead of rendering 10,000 rows:

```text
render only visible rows
+
small buffer
```

As the user scrolls, recycle/change which items are rendered.

This greatly reduces DOM work.

---

## 23. Hidden vs Removed

```css
display: none;
```

removes an element from layout.

```css
visibility: hidden;
```

preserves layout space but hides painting.

These have different rendering implications.

---

## 24. content-visibility

Modern CSS provides:

```css
content-visibility: auto;
```

which can let browsers skip rendering work for off-screen content in suitable scenarios.

Support and behavior should be checked for target browsers.

---

## 25. Images and Layout Shift

If image dimensions are unknown, content may move when images load.

Providing dimensions or aspect ratio helps avoid layout shifts.

Example:

```html
<img
  src="..."
  width="800"
  height="600"
/>
```

---

## 26. React Connection

React updates the DOM based on state/props.

But React optimization and browser rendering are different layers.

```text
React render/reconciliation
↓
DOM mutations
↓
browser style/layout/paint/composite
```

A fast React render can still cause expensive browser layout.

---

## 27. CSS Animation Connection

Prefer properties that avoid layout where possible.

Good candidates often include:

- transform
- opacity

Avoid animating layout-heavy properties unless necessary.

---

## 28. Measuring Rendering Performance

Browser DevTools Performance panel can show:

- scripting
- style recalculation
- layout
- paint
- composite
- long tasks
- frame timing

Measure instead of guessing.

---

## Interview Questions

### What is reflow/layout?

The browser calculating element geometry and position.

### What is repaint?

Redrawing visual pixels when appearance changes.

### What is layout thrashing?

Repeatedly alternating layout writes and layout reads, forcing frequent recalculation.

### Why use requestAnimationFrame?

It schedules visual updates in coordination with browser rendering.

---

## Key Takeaways

- Browser rendering involves DOM/CSSOM, layout, paint, and composite stages.
- Geometry changes may trigger layout.
- Visual changes may trigger paint.
- Layout can lead to additional paint work.
- Mixing layout writes and reads can cause layout thrashing.
- Group reads and writes when possible.
- `requestAnimationFrame()` is appropriate for visual updates.
- Large DOM trees and unnecessary layout work reduce performance.
- Measure actual rendering costs with DevTools.
