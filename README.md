# Sign-Up Form — Bugs Found & Fixes

## HTML

**1. Inline style overrides CSS**
```html
<img src="logo.png" style="height: 100%; width: 100%;">
```
Inline styles beat stylesheet rules regardless of specificity. Forced `.back-logo img` to use `!important` to override it. Fix: delete the inline `style` attribute, size the logo in CSS.

**2. Invalid input type / id / name (early version)**
```html
<input type="confirm password" id="confirm password" name="confirm password">
```
- `"confirm password"` is not a real `type` — browser silently fell back to `type="text"`, so the confirm-password field rendered as plain visible text, not masked.
- Spaces in `id`/`name` are invalid.
Fix: `type="password"`, `id="confirmpassword"`, `name="confirmpassword"` (no spaces).

**3. Broken div/label nesting (early version)**
```html
<div><label for="fname">FIRST NAME
<input ...>
</div></label>
```
Closing `</div>` before `</label>` — invalid. Browser force-closes the label to let the div close, so it rendered by accident, not correctly. Also caused inconsistent structure: 3 of 6 rows had a `<div>` wrapper, 3 didn't (missing opening `<div>`).
Fix: close tags in the order they opened —
```html
<div><label for="fname">FIRST NAME
<input ...>
</label></div>
```

**4. Missing space between attributes**
```html
<input type="password" id="confirmpassword"required>
```
`id="confirmpassword"required` — no space before `required`. Some parsers tolerate it, don't rely on that.

**5. `.button`/`.form` structural churn**
`.button` moved locations across revisions (sibling of `.background`/`.info` at `<body>` level → nested inside `.form` → nested inside `<form>`). Each move required updating the CSS grid placement, since `grid-row`/`grid-column` only apply to direct children of the grid container. Current version (button inside `<form>`, `<form>` inside `.form`) matches the CSS as given.

**6. `<input type="button">` doesn't trigger form submission**
```html
<input type="button" value="Create Account">
```
`type="button"` has no default behavior — it doesn't fire a `submit` event, so a `submit` listener on the `<form>` would never run no matter what JS you wrote.
Fix: `type="submit"`.

**7. Button lived outside the `<form>` element**
`.button` (containing the Create Account input) was a sibling of `<form>`, not inside it. Even with `type="submit"`, a submit button only submits the form it belongs to — outside the form, clicking it does nothing (no `submit` event fires, nothing to `preventDefault()` on).
Fix: moved the `.button` div inside `<form>...</form>`, alongside the field rows.

**8. Default form submission (page reload)**
```html
<form action="#" method="post">
```
`<input type="submit">` inside this form does what forms do: navigates to `action="#"` — page appears to reload/error. Not a bug, just unhandled default behavior.
Fix (in `script.js`):
```js
document.querySelector("form").addEventListener("submit", function (e) {
  e.preventDefault();
  // validation / next steps here
});
```
Attach the listener to the `<form>`, not the submit button — `submit` fires on the form.

## CSS

**9. Layout model followed HTML structure, not the other way round**
Every time `.button`'s nesting level changed, the grid placement had to change with it (top-level grid item → nested block using negative-margin breakout). No fix needed now, just noting: if you restructure the HTML again, the CSS grid placement will likely need to change too.

## Current status
As of the last HTML/CSS pair provided: valid nesting, correct input types, consistent structure, matches CSS. Remaining action item is wiring up `script.js` for `preventDefault()` and whatever validation logic you're writing yourself.
