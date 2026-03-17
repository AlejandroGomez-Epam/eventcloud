# AGENTS.md — EventCloud Coding Agent Reference

EventCloud is a multi-tenant event-management SaaS demo built on **ASP.NET Boilerplate (ABP)**.
The repository contains two sub-projects: an **Angular 17** SPA (`angular/`) and an **ASP.NET Core**
backend (`aspnet-core/`).

---

## Project Layout

```
CodeCollab/
├── angular/                  # Angular 17 SPA (TypeScript)
│   ├── src/app/              # Feature modules: events, users, roles, tenants, home
│   ├── src/account/          # Auth pages: login, register, tenant selection
│   ├── src/shared/           # Base classes, service proxies (auto-generated), pipes, directives
│   └── e2e/                  # Protractor end-to-end tests
└── aspnet-core/
    ├── src/
    │   ├── EventCloud.Core/              # Domain entities, domain services
    │   ├── EventCloud.Application/       # App services, DTOs
    │   ├── EventCloud.EntityFrameworkCore/ # EF Core, migrations, repositories
    │   ├── EventCloud.Web.Core/          # JWT auth, shared web config
    │   └── EventCloud.Web.Host/          # ASP.NET Core host, Swagger, SignalR
    └── test/
        ├── EventCloud.Tests/             # xUnit unit/integration tests
        └── EventCloud.Web.Tests/         # Web integration tests
```

---

## Build, Lint, and Test Commands

### Angular (run from `angular/`)

```bash
# Install dependencies
npm install

# Start dev server (connects to backend at localhost:21021)
npm start
# or: ng serve --host 0.0.0.0 --port 4200

# Production build
ng build --configuration production

# Lint (TSLint)
npm run lint

# Run all unit tests (Karma + Jasmine + Chrome)
npm test

# Run a single spec file
ng test --include='**/app/events/events.component.spec.ts'

# Run end-to-end tests
npm run e2e
```

### ASP.NET Core (run from `aspnet-core/`)

```bash
# Build entire solution
dotnet build EventCloud.sln

# Run backend host
dotnet run --project src/EventCloud.Web.Host

# Apply database migrations
dotnet run --project src/EventCloud.Migrator

# Run ALL tests
dotnet test

# Run tests for a single project
dotnet test test/EventCloud.Tests/EventCloud.Tests.csproj

# Run a single test class
dotnet test --filter "FullyQualifiedName~EventAppService_Tests"

# Run a single test method
dotnet test --filter "FullyQualifiedName~Should_Create_Event"
```

### Docker (from `aspnet-core/build/`)

```bash
./build-with-ng.sh     # Builds both backend (abp/host) and Angular (abp/ng) images
docker-compose up      # aspnet-core/docker/ng/docker-compose.yml
```

---

## Code Style — Angular / TypeScript

### Formatting (TSLint + `.editorconfig`)

- **Indentation:** Spaces (no tabs)
- **Quotes:** Single quotes (`'`)
- **Semicolons:** Always required
- **Max line length:** 140 characters
- **Trailing whitespace:** Forbidden; file must end with a newline
- **Charset:** UTF-8

### Language Rules

- Use `const` by default; `let` only when reassignment is needed; `var` is forbidden
- Triple-equals (`===`) for all comparisons; `==` only for null-checks
- Curly braces required for all control-flow blocks
- `for...in` loops must include a `hasOwnProperty` guard
- `console.debug/info/time/timeEnd/trace` and `debugger` are forbidden
- `eval` is forbidden

### Types

- Do not annotate types the compiler can infer (`no-inferrable-types`): avoid `const x: string = ''`
- Prefer `interface Foo {}` over `type Foo = {}` for object shapes
- Return typed Observables: `Observable<EventListDto[]>`, not `Observable<any>`
- Minimize `any`; the auto-generated `service-proxies.ts` uses it — new code should not
- Strict template checking is on; some strict sub-options are relaxed (see `tsconfig.json`)

### Naming Conventions (Angular)

| Element | Convention | Example |
|---|---|---|
| Files (component) | `kebab-case.component.ts` | `create-event.component.ts` |
| Files (service) | `kebab-case.service.ts` | `app-auth.service.ts` |
| Files (module) | `kebab-case.module.ts` | `events.module.ts` |
| Files (spec) | `kebab-case.component.spec.ts` | `events.component.spec.ts` |
| Utility/const files | `PascalCase.ts` | `AppConsts.ts`, `UrlHelper.ts` |
| Classes | `PascalCase` + role suffix | `EventsComponent`, `AppAuthService` |
| Abstract base classes | `PascalCase` + `Base` | `AppComponentBase` |
| Private injected fields | `_camelCase` (underscore prefix) | `_eventService`, `_router` |
| Public component fields | `camelCase` (no underscore) | `events`, `saving` |
| Methods | `camelCase` verbs | `loadEvents()`, `cancelEvent()` |
| Component selectors | `app-kebab-case` (element) | `<app-create-event>` |
| Directive selectors | `appCamelCase` (attribute) | `[appHasPermission]` |

### Imports (Angular)

Use the path aliases defined in `tsconfig.json`:

```typescript
import { AppComponentBase } from '@shared/app-component-base';
import { EventServiceProxy } from '@shared/service-proxies/service-proxies';
import { HomeComponent }     from '@app/home/home.component';
```

Import grouping order (blank line between groups):
1. `@angular/*` (framework)
2. Third-party libraries (`rxjs`, `ngx-bootstrap`, `moment`, `lodash`)
3. `@shared/*` (project-shared)
4. `@app/*` (feature-local)

Do **not** edit `src/shared/service-proxies/service-proxies.ts` — it is auto-generated by NSwag.
Regenerate it via the configuration in `angular/nswag/`.

### Angular-Specific Rules

- Always implement lifecycle interfaces explicitly (`implements OnInit`, `implements OnDestroy`)
- Use `@Input()` / `@Output()` decorators — never the `inputs`/`outputs`/`host` metadata properties
- Apply the `PipeTransform` interface to all pipes
- Component classes must have the `Component` suffix; directive classes the `Directive` suffix

---

## Code Style — C# / ASP.NET Core

### Formatting

- **Indentation:** 4 spaces; Allman brace style (opening `{` on its own line)
- **Naming:** `PascalCase` for public members, types, and methods; `_camelCase` for private fields
- **Async methods:** Always suffix with `Async` (e.g., `CreateAsync`, `GetListAsync`)

### Naming Conventions (C#)

| Element | Convention | Example |
|---|---|---|
| Namespace | `EventCloud.[Layer].[Feature]` | `EventCloud.Events.Dto` |
| Class / struct | `PascalCase` | `EventManager`, `EventCloudDbContext` |
| Interface | `I` + `PascalCase` | `IEventManager`, `IEventAppService` |
| Private field | `_camelCase` | `_eventRepository` |
| Public property | `PascalCase` | `Title`, `IsCancelled` |
| Constants | `PascalCase` | `MaxTitleLength` (defined on the entity) |
| DB tables | `AppPascalCase` via `[Table]` | `[Table("AppEvents")]` |
| Permissions | Dot-notation strings | `"Pages.Events"` |
| Test class | `ServiceName_Tests` | `EventAppService_Tests` |
| Test method | `Should_[Condition]` | `Should_Not_Create_Events_In_The_Past` |

### `using` Directive Order

1. `System.*`
2. `Microsoft.*`
3. `Abp.*`
4. `EventCloud.*`

### Domain / Entity Patterns

- Entity constructors are `protected`; expose a `public static T Create(...)` factory method.
  When creation requires async work (e.g., policy checks), use `public static async Task<T> CreateAsync(...)`.
- All entity properties use `virtual` for EF Core proxy support
- Property setters are `protected` to enforce invariants; mutations go through domain methods
- Private `Assert*` guard methods enforce invariants and throw `UserFriendlyException`
- The `@event` keyword escape is required whenever `event` is used as a variable name
- `IEventBus` on domain managers uses **property injection** with `NullEventBus.Instance` as the default:
  ```csharp
  public IEventBus EventBus { get; set; }
  // in constructor:
  EventBus = NullEventBus.Instance;
  ```

### Application Service Patterns

- App services inherit from `EventCloudAppServiceBase` and are decorated with `[AbpAuthorize]`
- Use `ObjectMapper.MapTo<TDestination>(source)` (ABP's AutoMapper wrapper) — not `_mapper.Map<T>()`
- Use `AbpSession.GetTenantId()` when a tenant ID is required (throws if not in a tenant context);
  use `AbpSession.TenantId` (nullable) when host-level code must handle both host and tenant
- Call `await CurrentUnitOfWork.SaveChangesAsync()` explicitly when you need generated IDs back
  within the same request before the unit of work completes automatically
- Never implement domain logic inside app services — delegate to the domain manager

### Angular Route & Permission Patterns

- Protected routes declare their required permission via `data: { permission: 'Pages.Feature' }`
  and use `canActivate: [AppRouteGuard]`
- The `MenuItem` constructor signature is `(label, route, icon, permissionName?)`; permission is
  optional — omit it for routes visible to all authenticated users
- Use `abp.session.userId` / `abp.session.tenantId` in templates for current-user checks

### Error Handling (C#)

- **User-visible errors:** Throw `UserFriendlyException` — ABP serializes these to the client
- **Null parameter guards:** Throw `ArgumentNullException`
- **Entity not found:** Throw `EntityNotFoundException`
- **Identity failures:** Call `identityResult.CheckErrors(LocalizationManager)` — produces a localized `UserFriendlyException`
- Do not add try/catch in domain or application layers; let ABP's host-level middleware handle formatting

### Error Handling (Angular)

- HTTP errors are caught globally by ABP's `AbpHttpInterceptor` and shown as toast notifications automatically
- In `.subscribe()` callbacks, the second (error) argument should at minimum reset loading state: `() => { this.saving = false; }`
- User notifications: `this.notify.success(...)`, `this.notify.error(...)`, `this.notify.info(...)`
- Destructive-action confirmations: `abp.message.confirm(...)`

---

## Testing — C# (xUnit + Shouldly + NSubstitute)

- Inherit from `EventCloudTestBase` for full ABP IoC container and in-memory DB
- Resolve services with `Resolve<IServiceInterface>()`
- Assertions use Shouldly: `result.ShouldBe(...)`, `result.ShouldNotBeNull()`
- Mock dependencies with NSubstitute: `Substitute.For<IEmailSender>()`
- Direct DB access in tests: `UsingDbContext(ctx => { ... })`
- Test async exceptions: `await Assert.ThrowsAsync<UserFriendlyException>(() => ...)`
- Use `[MultiTenantFact]` (custom attribute) for multi-tenant scenarios

## Testing — Angular (Karma + Jasmine)

- Spec files sit next to the source file they test (`*.component.spec.ts`)
- Use `TestBed.configureTestingModule` with `declarations`, `imports`, and `providers`
- Mock services inline with `useValue: { methodName: () => of(...) }`
- Use `fit()` / `fdescribe()` to focus a single test during development (remove before committing)
- Run a single file: `ng test --include='**/path/to/file.spec.ts'`

---

## Key Conventions to Follow

1. **Never edit `service-proxies.ts` by hand** — regenerate it with NSwag.
2. **Preserve the domain invariant pattern:** always use entity factory methods, never call `new Event()` directly.
3. **Multi-tenancy is always on** (`MultiTenancyEnabled = true`); every entity implements `IMustHaveTenant`.
4. **ABP handles cross-cutting concerns** (auditing, localization, permissions, unit-of-work, background jobs) — do not implement these manually.
5. **DTOs carry `[AutoMapFrom]` attributes**; add mapping configuration there, not in `Startup`.
6. **Domain events** (e.g., `EventDateChangedEvent`) are triggered via `DomainEvents.EventBus.Trigger(...)` inside domain entities or managers — not inside app services.
