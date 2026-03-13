# EventCloud .NET Coaching Script

**Repository and UoW · Entity Configurations · Data Loading · LINQ to EF · Migrations**

Use this script while presenting. Each section has a narrative to read or adapt, then a **Slide-to-code map** so you can switch from your PPT to the right file and lines in the EventCloud `aspnet-core` solution.

**Reference:** [Developing Multi-Tenant SaaS – ASP.NET Core & Angular](https://aspnetboilerplate.com/Pages/Documents/Articles/Developing-MultiTenant-SaaS-ASP.NET-CORE-Angular/index.html)

---

## Intro (before Topic 1)

**Say something like:**

> "Today we’ll go through Repository and Unit of Work, Entity Configurations, Data Loading strategies, LINQ to EF, and Migrations. We’ll use a real solution — EventCloud — a multi-tenant SaaS built with ASP.NET Boilerplate and Entity Framework Core. The domain we’ll focus on is **Events** and **EventRegistrations**: tenants create events, users register, and we’ll see how the code uses repositories, UoW, and EF to support that."

**No code yet.** You can show the solution structure (Core, Application, EntityFrameworkCore) if helpful.

---

## 1. Repository and Unit of Work

### Narrative

**Definitions**

> "The **Repository** pattern gives us a single abstraction over persistence: we ask for entities or save them without caring whether the data lives in SQL Server, a test double, or something else. The **Unit of Work** pattern groups multiple operations into one logical transaction and commits once, so we don’t have scattered SaveChanges all over the place."

**How they support each other**

> "Repositories usually work inside a Unit of Work. We open one UoW per request, do several repository calls — inserts, updates, queries — and at the end we call SaveChanges once. That way, either everything commits or nothing does. In EventCloud, ABP provides both: we inject `IRepository<T>` and the framework manages the UoW per request. When we need to flush explicitly — for example right after inserting a registration so the next line can use its Id — we call `CurrentUnitOfWork.SaveChangesAsync()`."

### Slide-to-code map

| Moment | File | What to show |
|--------|------|--------------|
| Repository abstraction | `aspnet-core/src/EventCloud.EntityFrameworkCore/EntityFrameworkCore/Repositories/EventCloudRepositoryBase.cs` (full file) | Base for all repos; extends ABP’s `EfCoreRepositoryBase`; comment “Add your common methods for all repositories”. |
| Repository usage in domain | `aspnet-core/src/EventCloud.Core/Events/EventManager.cs` (lines 16–18, 33–34, 44–45, 54–56, 74–79) | Constructor injects `IRepository<Event, Guid>` and `IRepository<EventRegistration>`; `GetAsync` uses `FirstOrDefaultAsync`; `CreateAsync` uses `InsertAsync`; `GetRegisteredUsersAsync` uses `GetAll()` + query. |
| Repository in application layer | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` (lines 20–21, 32–39) | Injects `IRepository<Event, Guid>`; `GetListAsync` uses `_eventRepository.GetAll()` and chains LINQ. |
| Repository passed to entity | `aspnet-core/src/EventCloud.Core/Events/EventRegistration.cs` (lines 46–61) | `CancelAsync(IRepository<EventRegistration> repository)` — entity receives repo to perform delete within domain rule. |
| Unit of Work – explicit save | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` (lines 94–99) | `RegisterAndSaveAsync`: after `_eventManager.RegisterAsync`, `await CurrentUnitOfWork.SaveChangesAsync()` so the registration is committed in the same UoW. |

**Talking point to close:**  
> "Repository gives us a single place to talk to the database; Unit of Work ensures that when we insert a registration and then return it, we flush that change in one transaction so the next read sees it."

---

## 2. Entity Configurations

### Narrative

**Three ways to configure the model**

> "Entity Framework needs to know how our classes map to tables and columns. We have three options. **Data Annotations**: attributes on the entity, like `[Table("AppEvents")]` or `[Required]`. **Fluent API**: in `OnModelCreating` we call `modelBuilder.Entity<Event>().ToTable("AppEvents")` and so on — more flexible for complex mappings. **Conventions**: EF assumes things like ‘Id’ is the key and the table name is the type name unless we override."

**In EventCloud**

> "Here we use Data Annotations on Event and EventRegistration for table names, string lengths, and foreign keys. The DbContext only declares the DbSets; we don’t override `OnModelCreating`. The migration Designer files show the final model — that’s where you see the Fluent-style snapshot that EF generated from our annotations and conventions."

### Slide-to-code map

| Moment | File | What to show |
|--------|------|--------------|
| Data Annotations – table and constraints | `aspnet-core/src/EventCloud.Core/Events/Event.cs` (lines 13–39) | `[Table("AppEvents")]`, `[Required]`, `[StringLength]`, `[Range]`, `[ForeignKey("EventId")]` on `Registrations`. |
| Data Annotations – second entity | `aspnet-core/src/EventCloud.Core/Events/EventRegistration.cs` (lines 11–21) | `[Table("AppEventRegistrations")]`, `[ForeignKey("UserId")]` on `User`. |
| DbContext – no Fluent in project | `aspnet-core/src/EventCloud.EntityFrameworkCore/EntityFrameworkCore/EventCloudDbContext.cs` (full file) | Only `DbSet<Event>` and `DbSet<EventRegistration>`; no `OnModelCreating` override — “we could add Fluent here for complex mappings.” |
| Conventions in migrations | `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/20231227125039_Added_Event.Designer.cs` (lines 1–25, then search “AppEvents” / “Event”) | Designer shows the built model; table names and column types reflect annotations and conventions. |

**Talking point to close:**  
> "Here we rely on Data Annotations for table names and constraints; the DbContext stays thin. For more control, we’d override OnModelCreating and use Fluent API."

---

## 3. Data Loading Strategies

### Narrative

**Three strategies**

> "**Eager loading**: we load the main entity and its related data in one query using `Include` and `ThenInclude`. **Lazy loading**: we load related data the first time we touch the navigation property — EF uses proxies for that. **Explicit loading**: we already have an entity and later we call `Entry(e).Collection(e => e.Registrations).Load()` to load a collection on demand."

**In EventCloud**

> "We use **eager loading** only. For the event list we Include Registrations; for the detail we add ThenInclude for User. We don’t enable lazy loading — the DbContext configurer has no `UseLazyLoadingProxies()`. That keeps our SQL predictable and avoids N+1: we decide up front what we need and load it in one go."

### Slide-to-code map

| Moment | File | What to show |
|--------|------|--------------|
| Eager – list (one level) | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` (lines 32–39) | `GetListAsync`: `.Include(e => e.Registrations)` then `WhereIf`, `OrderByDescending`, `Take`, `ToListAsync`. |
| Eager – detail (two levels) | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` (lines 44–51) | `GetDetailAsync`: `.Include(e => e.Registrations).ThenInclude(r => r.User)` — Event → Registrations → User in one query. |
| Eager in domain service | `aspnet-core/src/EventCloud.Core/Events/EventManager.cs` (lines 73–80) | `GetRegisteredUsersAsync`: `.GetAll().Include(registration => registration.User)` then filter and project. |
| No lazy loading | `aspnet-core/src/EventCloud.EntityFrameworkCore/EntityFrameworkCore/EventCloudDbContextConfigurer.cs` (full file) | Only `UseSqlServer` — no `UseLazyLoadingProxies()`. “We don’t enable lazy loading; we load explicitly with Include.” |

**Talking point to close:**  
> "We always decide up front what we need: list view gets Events + Registrations; detail view adds User. That keeps SQL predictable and avoids N+1."

---

## 4. LINQ to EF (Queryables)

### Narrative

**IQueryable vs in-memory**

> "When we call `GetAll()` on the repository we get an `IQueryable`. That’s a description of a query — filters, sorts, includes — that hasn’t run yet. As long as we keep using methods EF can translate, everything stays in the database. The moment we call `ToListAsync()`, `FirstOrDefaultAsync()`, or `CountAsync()`, EF turns that into SQL and executes it. So we can chain `WhereIf`, `OrderByDescending`, `Take`, and only the final materialization hits the DB."

**In EventCloud**

> "In EventAppService we build one chain for the event list and one for detail. In UserAppService we have `CreateFilteredQuery` and `ApplySorting` that return and accept `IQueryable` — the base class composes them into a single paged query. In EventRegistrationPolicy we use `CountAsync` with a predicate so the count is done in the database."

### Slide-to-code map

| Moment | File | What to show |
|--------|------|--------------|
| Queryable chain – events | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` (lines 32–39) | `_eventRepository.GetAll()` → `Include` → `WhereIf` → `OrderByDescending` → `Take(64)` → `ToListAsync()`. One SQL query. |
| Queryable in paged services | `aspnet-core/src/EventCloud.Application/Users/UserAppService.cs` (lines 161–166, 181–184) | `CreateFilteredQuery`: returns `IQueryable<User>` with `WhereIf`. `ApplySorting`: takes `IQueryable`, returns `query.OrderBy(...)`. Base class builds paged query. |
| Queryable + policy (Count) | `aspnet-core/src/EventCloud.Core/Events/EventRegistrationPolicy.cs` (lines 42–54) | `CountAsync(r => r.UserId == user.Id && r.CreationTime >= oneMonthAgo)` — predicate stays in DB. |

**Talking point to close:**  
> "As long as we stay on IQueryable and use supported methods, EF translates to SQL. The moment we call ToListAsync or FirstOrDefaultAsync, the query runs."

---

## 5. Migrations (Transparency, Version Control, Code Review, Maintainability, Deployment)

### Narrative

**Why migrations matter**

> "Migrations make schema changes **transparent**: we see exactly what runs in `Up` and how to undo it in `Down`. They’re **versioned**: every change is a pair of files in source control. They’re **reviewable**: in a PR we can diff the migration and the Designer. They’re **maintainable**: we know the order of application and what each step does. And they’re **deployable**: we run the same migrations in dev, test, and production so the schema stays in sync with the code."

**In EventCloud**

> "We’ll look at the migration that added Event and EventRegistration: the Up method creates the tables, FKs, and indexes; the Down drops them in reverse order. The Designer holds the full model at that migration. In production we run `dotnet ef database update` or the same from a release pipeline so migrations run in order."

### Slide-to-code map

| Moment | File | What to show |
|--------|------|--------------|
| Migration class – Up/Down | `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/20231227125039_Added_Event.cs` (full file) | `Up`: CreateTable AppEvents, AppEventRegistrations, FKs, indexes. `Down`: DropTable in reverse order. Reproducibility and rollback. |
| Naming and versioning | Folder: `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/` | Timestamp + descriptive name (e.g. `Added_Event`, `Upgraded_To_Abp_*`). Good for version control and “what changed when.” |
| Designer / snapshot | `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/20231227125039_Added_Event.Designer.cs` (header + one entity block) | `[DbContext(typeof(EventCloudDbContext))]`, `[Migration("...")]`; BuildTargetModel shows full model at that version — code review can check schema. |
| Deployment | Mention only | “In production we run `dotnet ef database update` or equivalent in a release pipeline; migrations run in order, so schema stays in sync with code.” |

**Talking point to close:**  
> "Every change is a pair of files: the migration with Up/Down and the Designer. We commit them, review in PRs, and run the same migrations in every environment."

---

## Wrap-up

**Say something like:**

> "We covered Repository and Unit of Work and how they work together in EventCloud; Entity Configurations with Data Annotations, and where Fluent and conventions fit in; Data Loading with eager loading and why we avoid lazy loading here; LINQ to EF and how IQueryable defers execution until we materialize; and Migrations for transparency, version control, code review, maintainability, and deployment. You now have a map from each topic to the actual code in this solution — use the cheat sheet to jump from your slides to the right file and lines."

**Optional:** Point them to `docs/coaching-slide-to-code.md` for the one-page reference.
