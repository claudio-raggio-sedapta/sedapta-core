# jQuery Clip Generation Reference

This reference provides a practical scaffold for generating a new clip pair modeled on `jqueryClip.html` and `jqueryClip.js`.

## Suggested Naming

- Keep file names in camelCase or kebab-case and match pairs:
  - `ordersClip.html`
  - `ordersClip.js`

## HTML Starter Scaffold

Use this as a base, then add only the controls needed for the target scenario.

```html
<style type="text/css">
  .clip-root {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    gap: 8px;
    overflow: auto;
    box-sizing: border-box;
    padding: 8px;
  }

  .clip-actions {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
  }

  .clip-btn {
    border: none;
    padding: 6px 10px;
    cursor: pointer;
    background: #269fd9;
    color: #fff;
  }

  .clip-panel {
    border: 1px solid #d9d9d9;
    min-height: 120px;
    padding: 8px;
  }
</style>

<div class="clip-root">
  <div class="clip-actions">
    <button id="clipRefreshBtn" class="clip-btn">Refresh</button>
    <button id="clipNotifyBtn" class="clip-btn">Notify</button>
  </div>
  <div id="clipItemsPanel" class="clip-panel"></div>
</div>
```

## JS Starter Scaffold

```js
function renderItems(items) {
  var $panel = $("#clipItemsPanel");
  $panel.empty();

  if (!items || !items.length) {
    $panel.text("No items");
    return;
  }

  items.forEach(function (item) {
    var text = item && item.Title ? item.Title : JSON.stringify(item);
    $panel.append($("<div>").text(text));
  });
}

$("#clipNotifyBtn").on("click", function () {
  interopInterface.notificationService.showSnack("jQuery Clip action");
});

$("#clipRefreshBtn").on("click", function () {
  interopInterface.widgetLoaderService.showLoader();
  setTimeout(function () {
    interopInterface.widgetLoaderService.hideLoader();
    renderItems(interopInterface.items || []);
  }, 300);
});

function setItems(items) {
  renderItems(items);
}

setItems(interopInterface.items || []);

return { setItems };
```

## Interop Fields Commonly Used

Use only what is needed:

- `interopInterface.items`
- `interopInterface.notificationService`
- `interopInterface.loaderService` / `interopInterface.widgetLoaderService`
- `interopInterface.layoutManagerService`
- `interopInterface.broadcastMessagesService`
- `interopInterface.translate(key)`
- `interopInterface.fireWiringEvent(payload)`

## Compatibility Checklist

- Script contains no `import`/`export`.
- Selectors in JS exist in HTML.
- Script ends with `return { setItems };` (or includes `setItems` in returned object).
- HTML and JS are plain text fragments suitable for widget settings fields.
- Files are saved in `SedApta.Core.Web.Angular/ClientApp/`.
