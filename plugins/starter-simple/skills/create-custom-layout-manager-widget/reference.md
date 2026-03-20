# Reference: IWidget, ICustomComponentConfig, and registration

## IWidget (projects/sedapta-core/model/base/widget.model.ts)

Components used as layout manager widgets must implement this interface. All members are optional except the type must be compatible when the layout manager calls `configure` and reads `widgetUuid` / `fullscreen`.

```ts
export interface IWidget {
  widgetUuid: UUID;
  fullscreen: boolean;
  configure?: (config: IWidgetConfiguration) => void;
  activate?: (pendingEvent: IPendingEvent, direction?: WizardDirection) => void;
  deactivate?: (direction?: WizardDirection) => void;
  canActivate?: (direction?: WizardDirection) => Observable<boolean>;
  canDeactivate?: (direction?: WizardDirection) => Observable<boolean>;
  invalidate?: () => void;
  sizeChanged?: () => void;
  setFullScreenMode?: (fullscreen: boolean) => void;
  getMenuButtons?: () => IToolbarItem[];
  refresh?: (options?: IWidgetRefreshOptions) => void;
  setDialogApi?: (api: IDialogServiceApi) => void;
  setFilters?: (
    filters: IWidgetFilters,
    type: FilterType,
    source?: FilterSource
  ) => void;
  applyLumiFilter?: (widgetFilters: IWidgetFilters) => void;
  resetLumiFilter?: () => void;
  wizardCustomActions?: () => IWizardScreenCustomAction[];
}
```

## IWidgetConfiguration

Passed to `configure()`:

```ts
export interface IWidgetConfiguration {
  id: string;
  config: IGraphConfig;
  settings?: IGraphSettings;
  widgetUuid?: UUID;
  tabUuid?: UUID;
  filters?: IFilter[];
  data?: any[];
  active: boolean;
  item?: ILayoutManagerItem;
}
```

Use `config.item` for layout item (settings, widget id, etc.) when available.

---

## ICustomComponentConfig (GraphFactoryService.addCustomComponent)

Defined in `projects/sedapta-core/model/base/component-config.model.ts`. Used in `src/app/services/mix-component.service.ts`.

**Required:**

- `id: string` – Unique component id
- `label: string` – Palette label (or translation key)
- `typeId: CoreWidgetType | string` – Type id (e.g. same as `id` for custom types)
- `desktopType: Type<IWidget>` – Component class
- `icon: IconCode` – From `@sedapta/core/model` (IconCode.*) or `this.iconService.*`

**Optional:**

- `mobileType?: Type<IWidget>`
- `handheldType?: Type<IWidget>`
- `options?: { ... }`

**options (all optional):**

| Option | Type | Purpose |
|--------|------|--------|
| `group` | string | Palette group name |
| `hideMenu` | boolean | Hide widget menu |
| `cannotFilter` | boolean | Disable filter UI |
| `desktopCannotFullscreen` | boolean | Disable fullscreen on desktop |
| `mobileCannotFullscreen` | boolean | Disable fullscreen on mobile |
| `isMulti` | boolean | Allow multiple instances in layout |
| `configType` | Type<IGraphConfig> | Config class for widget (e.g. clock) |
| `outputEvents` | ILayoutManagerEvent[] | `{ EventId, Title }[]` for wiring outputs |
| `inputEvents` | ILayoutManagerInputEvent[] | `{ Func, Title, clientWiring? }[]` for wiring inputs |
| `configurable` | boolean | Show config UI (e.g. CustomWidget editor) |
| `dataSourceId` | string | Default data source id |
| `parameters` | IAvailableDataSourceParameter[] | Wizard/params |
| `licenseRoles` | string[] | Role restrictions |
| `injector` | Injector | Custom injector |
| `isContainer` | boolean | Container widget (splitter/tab) |
| `wizardInputParameters` | IWizardScreenParameter[] | Wizard input params |
| `wizardOutputParameters` | IWizardScreenParameter[] | Wizard output params |
| `wizardOutputPaths` | IWizardOutputPath[] | Wizard output paths |

---

## Where registration is used

- `MixComponentService.addComponents()` is invoked during app bootstrap (or layout/CT init). It calls `GraphFactoryService.addCustomComponent()` for each custom widget.
- `GraphFactoryService` is from `@sedapta/core/services`. The layout manager uses it to resolve the component type by `id`/`typeId` and to build config when creating the widget inside `GraphComponent` (see `dynamic-graph.directive.ts` and graph component).

## Icon sources

- **IconCode** – `import { IconCode } from "@sedapta/core/model"` (e.g. `IconCode.gear`, `IconCode.window`).
- **IconService** – Injected in `MixComponentService` as `this.iconService`; e.g. `this.iconService.clock`, `this.iconService.button`, `this.iconService.map`. Use for icons that are resolved from a central service.
