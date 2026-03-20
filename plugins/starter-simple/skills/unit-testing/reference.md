# Unit Testing Reference

## Imports (typical)

```ts
import { waitForAsync, ComponentFixture, TestBed } from '@angular/core/testing';
import { By } from '@angular/platform-browser';
import { SedaptaCoreTestingModule } from '@sedapta/test';
import { cloneDeep } from 'lodash-es';
```

## Karma config

- **sedapta-core**: `projects/sedapta-core/karma.conf.js` (and `karma-dev.conf.js` for dev).
- Framework: Jasmine. Browser: ChromeHeadlessNoSandbox.
- Coverage: `projects/sedapta-core/coverage/test/`.

## Testing module contents

`SedaptaCoreTestingModule` (`@sedapta/test`) provides:

- `AuthTestingModule`, `RouterTestingModule`, `DialogTestingModule`, `TranslateTestingModule`, `ConfigTestingModule`
- Mocks for: `LoaderService`, `InteropService`, `IdeService`, `NotificationService`, `KeyboardService`, `MetroDumpService`, `KendoLocalesService`, `AcmContextStoreService`, `AcmAttributesStoreService`, `DexieCacheService`, `RequestInfoService`, `AuthService`
- `provideHttpClientTesting()`, `provideAnimations()`, `PathLocationStrategy`
- Real: `DataService`, `GraphTitleService`, `WidgetLoaderService`, `WidgetFilterService`, `LayoutHubService`, `WidgetsConversationService`, `TranslateService`

When a component or service under test needs a different implementation, override in `TestBed.configureTestingModule({ providers: [{ provide: X, useValue: spyOrMock }] })`.

## Component test pattern

```ts
describe('MyComponent', () => {
  let component: MyComponent;
  let fixture: ComponentFixture<MyComponent>;

  beforeEach(waitForAsync(() => {
    TestBed.configureTestingModule({
      imports: [SedaptaCoreTestingModule],
      declarations: [MyComponent],
    }).compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
    // set inputs
    fixture.detectChanges();
  });

  it('should create', () => expect(component).toBeTruthy());
});
```

## Service test pattern

```ts
describe('MyService', () => {
  beforeEach(() =>
    TestBed.configureTestingModule({ imports: [SedaptaCoreTestingModule] })
  );

  it('should be created', () => {
    const service = TestBed.inject(MyService);
    expect(service).toBeTruthy();
  });
});
```

## Feature-specific providers

Some features export provider arrays (e.g. `SIMUMATRIX_PROVIDERS`). Include them in `TestBed` and override only the services you need to spy on:

```ts
TestBed.configureTestingModule({
  imports: [SedaptaCoreTestingModule],
  providers: [
    ...SIMUMATRIX_PROVIDERS,
    { provide: SimumatrixKpiService, useValue: kpiServiceSpy },
  ],
});
```
