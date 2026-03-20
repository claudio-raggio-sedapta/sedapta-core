---
name: unit-testing
description: Write and run Angular unit tests using Jasmine and Karma in this workspace. Use when adding or editing .spec.ts files, fixing failing tests, running unit tests, or when the user asks for unit test help, test coverage, or spec files.
---

# Unit Testing (SedApta Angular)

This project uses **Jasmine** and **Karma** for unit tests. Spec files are colocated with source (`*.spec.ts` next to the unit under test).

## Running tests

From the workspace root (`ClientApp`):

- **sedapta-core**: `ng test sedapta-core`
- **sedapta-test**: `ng test sedapta-test`
- **sedapta-elisa**: `ng test sedapta-elisa`
- **sedapta-cube**: `ng test sedapta-cube`

Scripts (if present): `npm run core_test_dev`, `core_test_dev_coverage`, etc.

## Test setup

1. **Use the shared testing module** for components and services in `sedapta-core` and `sedapta-elisa`:

   ```ts
   import { SedaptaCoreTestingModule } from '@sedapta/test';
   ```

2. **Component tests**: Prefer `waitForAsync` in `beforeEach` when configuring the testing module and compiling:

   ```ts
   beforeEach(waitForAsync(() => {
     TestBed.configureTestingModule({
       imports: [SedaptaCoreTestingModule],
       declarations: [MyComponent],
       providers: [/* override only what you need */]
     }).compileComponents();
   }));
   ```

3. **Override dependencies** with Jasmine spies when testing behavior:

   ```ts
   const serviceSpy = jasmine.createSpyObj('MyService', ['method1', 'method2']);
   TestBed.configureTestingModule({
     imports: [SedaptaCoreTestingModule],
     providers: [{ provide: MyService, useValue: serviceSpy }]
   });
   ```

4. **DOM queries**: Use `By` from `@angular/platform-browser` and `fixture.debugElement.query(By.css('.class'))`.

5. **Mutable mock data**: Use `cloneDeep` from `lodash-es` when assigning mock objects to component inputs to avoid shared state between tests.

## File and structure conventions

- One spec file per source file: `kpi-value.component.ts` → `kpi-value.component.spec.ts`.
- Group tests with `describe('ComponentName', () => { ... })` and nested `describe` blocks for areas (e.g. "Input Properties", "Template Rendering").
- Prefer small, focused `it` cases; avoid one large test that does many things.

## Mocks and test data

- **Shared mocks** (Auth, HTTP, Loader, etc.): Provided by `SedaptaCoreTestingModule` from `@sedapta/test` (see `projects/sedapta-test/src/lib/modules/core-testing.module.ts` and `projects/sedapta-test/src/lib/mocks/`).
- **Domain mocks**: `projects/sedapta-core/test/mocks-data/` (e.g. `grid.mock.ts`, `gridster.mock.ts`, `lm-items.mock.ts`). Reuse or extend these for consistency.

When adding new tests, prefer existing mocks and the testing module; add project-specific mocks under `projects/sedapta-core/test/mocks-data/` or next to the spec only when necessary.

## Quick reference

- [reference.md](reference.md) – Imports, providers, and patterns used in this project.
