---
name: create-jquery-clip-files
description: Creates a new jQuery Clip pair (markup and script) for the ClientApp root, following the JqueryClipComponent contract in projects/sedapta-core/widgets/jquery-clip/jquery-clip.component.ts. Use when the user asks to generate a jquery clip html/js file pair, clone jqueryClip.html/jqueryClip.js, or scaffold a new jQuery Clip implementation.
---

# Create jQuery Clip Files

Use this skill to generate two files in the ClientApp root:

- `<clip-name>.html` (markup + styles)
- `<clip-name>.js` (jQuery behavior + interop logic)

The output must follow the `JqueryClipComponent` runtime contract in `projects/sedapta-core/widgets/jquery-clip/jquery-clip.component.ts`.

## Required Output Location

Create both files under:

- `SedApta.Core.Web.Angular/ClientApp/`

Do not place the files inside `projects/` or widget folders unless explicitly requested.

## Contract from JqueryClipComponent

The generated clip must work with this flow:

1. The HTML text is injected into container `innerHTML` from `jqueryClipSettings.HtmlContent`.
2. The JS text is executed via:
   - `new Function("interopInterface", script)(interopInterface)`
3. The JS should return an API object. Prefer returning:
   - `return { setItems };`
4. If the script exposes `setItems`, the widget calls it when data changes.

## Generation Workflow

- [ ] **1. Define the clip name** (example: `ordersClip`)
- [ ] **2. Create HTML file** (`ordersClip.html`) with IDs/classes used by JS
- [ ] **3. Create JS file** (`ordersClip.js`) using `interopInterface`
- [ ] **4. Ensure event handlers are bound once** and selectors match HTML IDs
- [ ] **5. Return a public script API** with `setItems` (and optionally other methods)
- [ ] **6. Verify no module syntax** (`import`/`export`) is used in the JS file

## HTML File Rules

- Keep styles scoped with clip-specific classes/IDs to reduce collisions.
- Include container elements for all interactive controls.
- Keep structure simple and robust for dynamic widget container sizing (`width: 100%; height: 100%` where appropriate).

## JS File Rules

- Use jQuery selectors/events (`$("#id").on("click", ...)`) consistently.
- Use `interopInterface` services for platform interactions (notifications, loaders, wiring events, etc.).
- Avoid assumptions that Kendo is always loaded; only use Kendo widgets if required by settings.
- Include `setItems(items)` to refresh UI from data bindings.

## Minimal Script Template

```js
// Called as: new Function("interopInterface", scriptText)(interopInterface)
(function () {
  function renderItems(items) {
    // Render/update DOM here
  }

  function setItems(items) {
    renderItems(items || []);
  }

  // Initial render from widget data
  setItems(interopInterface.items || []);

  $("#myButton").on("click", function () {
    interopInterface.notificationService.showSnack("Action executed");
  });

  return { setItems };
})();
```

Note: when storing in widget settings, keep only script body compatible with `new Function("interopInterface", ...)` and ending with a returned API object.

## Delivery Format

When asked to generate a new clip:

1. Create both files in `ClientApp` with the requested names.
2. Report created file paths.
3. Briefly list expected configuration mapping:
   - `HtmlContent` <- contents of `<clip-name>.html`
   - `Script` <- contents of `<clip-name>.js`

## Additional Guidance

- For complete examples and a richer starter scaffold, see [reference.md](reference.md).
