---
name: create-custom-layout-manager-widget
description: Guides creating a custom Layout Manager widget that implements IWidget, placed under src/app/layout-manager/components, and registered in GraphFactoryService via MixComponentService. Use when adding an app-level custom widget, implementing IWidget in the ClientApp, or registering a widget in mix-component.service.
---

# Create a Custom Layout Manager Widget (IWidget)

Use this skill when creating a **custom** widget for the Layout Manager that lives in the app (`src/app/layout-manager/components`) and is registered via `MixComponentService` and `GraphFactoryService`—not the sedapta-core library factory.

## Clarifications before starting

- **Core widget used internally:** If the user asks to use a **core widget** (sedapta-core) **inside** the custom component (e.g. embed or host a Grid, Chart, etc.), **ask which is the graph id** of that widget. The graph id identifies the widget configuration in Bricks/GraphFactoryService and is required to instantiate it (e.g. via `findComponentByConfig(id)` or layout-manager graph config callbacks).
- **Configurable widget:** If the user asks for a **configurable** widget (selectable in Bricks/Control Tower by type, with its own type id), use **`addComponentType(config: ILayoutManagerComponentConfig)`** for registration and follow the component pattern from `src/app/layout-manager/components/wizard/component-one/one.component.ts` (see section below).

## Workflow Checklist

- [ ] **1.** Create component under `src/app/layout-manager/components/<widget-name>/`
- [ ] **2.** Implement `IWidget` (minimum: `widgetUuid`, `fullscreen`; typically `configure`)
- [ ] **3.** Register in `MixComponentService.addComponents()` in `src/app/services/mix-component.service.ts`

---

## Step 1: Component location and structure

**Location:** `src/app/layout-manager/components/<widget-name>/`

Create a standalone Angular component with:

- `<widget-name>.component.ts`
- `<widget-name>.component.html`
- `<widget-name>.component.scss` (optional)

The component must **implement** `IWidget` from `@sedapta/core/model`. You can either implement it directly (minimal) or extend `CustomWidget` from `@sedapta/core/base` for full wiring, loaders, and filter support.

---

## Step 2: Implement IWidget

### Minimum implementation (simple widget)

Required fields: `widgetUuid: UUID` and `fullscreen: boolean`. The layout manager calls `configure(config: IWidgetConfiguration)` to pass id, config, item, etc.

```ts
import { Component, OnInit } from "@angular/core";
import { IWidget, IWidgetConfiguration } from "@sedapta/core/model";
import { UUID } from "angular2-uuid";

@Component({
  selector: "app-my-custom-widget",
  templateUrl: "./my-custom-widget.component.html",
  styleUrls: ["./my-custom-widget.component.scss"],
})
export class MyCustomWidgetComponent implements OnInit, IWidget {
  widgetUuid: UUID;
  fullscreen: boolean;

  configure = (config: IWidgetConfiguration) => {
    this.widgetUuid = config.widgetUuid;
    // use config.config, config.item, config.settings as needed
  };

  ngOnInit(): void {}
}
```

### Optional IWidget members

Implement only what the widget needs:

| Member | Purpose |
|--------|--------|
| `configure` | Receives layout config, item, widgetUuid; **recommended** |
| `refresh?(options?)` | Called when layout asks for a refresh |
| `setFilters?(filters, type, source?)` | Filter handling |
| `setFullScreenMode?(fullscreen)` | Fullscreen toggle |
| `getMenuButtons?()` | Custom toolbar items |
| `setDialogApi?(api)` | Dialog integration |

### Full-featured: extend CustomWidget

For wiring (input/output events), loader, and filter services, extend `CustomWidget` and use `CustomWidgetProviders`:

```ts
import { Component, OnInit } from "@angular/core";
import { CustomWidget, CustomWidgetProviders } from "@sedapta/core/base";
import { WiringInputMethod, IWiringMappingParameters } from "@sedapta/core/model";
import { UUID } from "angular2-uuid";

@Component({
  selector: "app-my-custom-widget",
  templateUrl: "./my-custom-widget.component.html",
  styleUrls: ["./my-custom-widget.component.scss"],
  providers: [CustomWidgetProviders],
})
export class MyCustomWidgetComponent extends CustomWidget implements OnInit {
  constructor() {
    super();
  }

  ngOnInit(): void {
    this.loadWidget({ loadData: false });
  }

  initWidget = (scenario?: string) => {
    // wire this.graphConfig.control, this.fireControlReady(), etc.
  };

  // Optional: wiring input handler
  myInputEvent: WiringInputMethod = (
    widgetFromUuid: UUID,
    data: object[],
    mappingParameters: IWiringMappingParameters[],
    mappingDataModels: IWiringMappingParameters[]
  ): void => {
    // handle wired data
  };
}
```

---

## Step 3: Register in MixComponentService

**File:** `src/app/services/mix-component.service.ts`

1. Import the new component at the top.
2. In `addComponents()`, call `this.factoryService.addCustomComponent({ ... })` with an `ICustomComponentConfig` (see [reference.md](reference.md) for full options).

**Required fields:**

| Field | Description |
|-------|-------------|
| `id` | Unique string id (e.g. `"MyWidget"`) |
| `label` | Display name in palette (e.g. `"My Widget"` or translation key) |
| `typeId` | Type identifier (string; can match `id`) |
| `desktopType` | The component class (e.g. `MyCustomWidgetComponent`) |
| `icon` | `IconCode.*` or `this.iconService.*` from `IconService` |

**Example (minimal):**

```ts
import { MyCustomWidgetComponent } from "../layout-manager/components/my-custom-widget/my-custom-widget.component";

// In addComponents():
this.factoryService.addCustomComponent({
  id: "MyWidget",
  label: "My Widget",
  typeId: "MyWidget",
  desktopType: MyCustomWidgetComponent,
  icon: IconCode.gear,
  options: {
    isMulti: true,
    desktopCannotFullscreen: true,
    cannotFilter: true,
  },
});
```

**Example (with wiring events):**

```ts
this.factoryService.addCustomComponent({
  id: "MyWidget",
  label: "My Widget",
  typeId: "MyWidget",
  desktopType: MyCustomWidgetComponent,
  icon: IconCode.gear,
  options: {
    group: "Custom",
    isMulti: true,
    outputEvents: [
      { EventId: "myOutputEvent", Title: "Output Event" },
    ],
    inputEvents: [
      { Func: "myInputEvent", Title: "Input Function" },
    ],
  },
});
```

---

## Configurable widget (addComponentType)

When the user wants a **configurable** widget (visible in Bricks/Control Tower as a widget type with its own type id, configurable per instance), register it with **`GraphFactoryService.addComponentType`** instead of `addCustomComponent`. The component can follow the same structure as `src/app/layout-manager/components/wizard/component-one/one.component.ts` (extends `CustomWidget`, uses `CustomWidgetProviders`, optional `WizardService`, etc.).

**Registration** in `src/app/services/mix-component.service.ts` inside `addComponents()`:

```ts
this.factoryService.addComponentType({
  type: "MyConfigurableType",           // type id used in Bricks CustomComponent
  desktopType: MyConfigurableComponent,
  icon: IconCode.gear,
  config: MyDefaultConfig,              // optional: Type<IGraphConfig> or factory (id) => IGraphConfig
  outputEvents: [
    { EventId: "out", Title: "Output Event" },
  ],
  inputEvents: [
    { Func: "inputEvent", Title: "Input Event" },
  ],
  // optional: isCustom, wizardInputParameters, wizardOutputParameters, wizardOutputPaths, editor, etc.
});
```

**Component example** (structure like `one.component.ts`): standalone component extending `CustomWidget`, `providers: [CustomWidgetProviders]`, `loadWidget({ loadData: false })` in `ngOnInit`, and implement any wiring inputs/outputs or wizard callbacks as needed. In Bricks, create a widget configuration of type **Custom** and set **CustomComponent** to the `type` string used in `addComponentType`.

---

## Key files

| Purpose | Path |
|--------|------|
| Custom component folder | `src/app/layout-manager/components/<widget-name>/` |
| Registration | `src/app/services/mix-component.service.ts` |
| IWidget / IWidgetConfiguration | `projects/sedapta-core/model/base/widget.model.ts` |
| ICustomComponentConfig | `projects/sedapta-core/model/base/component-config.model.ts` |
| CustomWidget (optional base) | `projects/sedapta-core/base/base/custom/custom-widget.ts` |

Existing examples in this app: `CustomWidgetOneComponent` (extends CustomWidget), `ButtonOneComponent` (implements IWidget only), `LmFormComponent`, `LmProgressBarComponent`—all under `src/app/layout-manager/components` and registered in `mix-component.service.ts`. For a **configurable** widget (registered with `addComponentType`), use **`OneComponent`** in `src/app/layout-manager/components/wizard/component-one/one.component.ts` as the component structure reference (CustomWidget, CustomWidgetProviders, optional wizard/wiring).

For full `ICustomComponentConfig` and `IWidget` reference, see [reference.md](reference.md).
