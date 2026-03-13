# EventCloud coaching – Slide to code (one-page cheat sheet)

Use this to jump from your PPT slide/section to the right file and lines in the **aspnet-core** solution. Paths are relative to the repo root.

| Topic | Slide / section | File | Lines / scope |
|-------|-----------------|------|----------------|
| **Repository & UoW** | Repository abstraction | `aspnet-core/src/EventCloud.EntityFrameworkCore/EntityFrameworkCore/Repositories/EventCloudRepositoryBase.cs` | Full file |
| | Repository in domain | `aspnet-core/src/EventCloud.Core/Events/EventManager.cs` | 16–18, 33–34, 44–45, 54–56, 74–79 |
| | Repository in app layer | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` | 20–21, 32–39 |
| | Repository passed to entity | `aspnet-core/src/EventCloud.Core/Events/EventRegistration.cs` | 46–61 |
| | Unit of Work – explicit save | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` | 94–99 |
| **Entity configurations** | Data Annotations (Event) | `aspnet-core/src/EventCloud.Core/Events/Event.cs` | 13–39 |
| | Data Annotations (EventRegistration) | `aspnet-core/src/EventCloud.Core/Events/EventRegistration.cs` | 11–21 |
| | DbContext (no Fluent) | `aspnet-core/src/EventCloud.EntityFrameworkCore/EntityFrameworkCore/EventCloudDbContext.cs` | Full file |
| | Conventions in migrations | `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/20231227125039_Added_Event.Designer.cs` | 1–25, then search "AppEvents" |
| **Data loading** | Eager – list | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` | 32–39 |
| | Eager – detail (ThenInclude) | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` | 44–51 |
| | Eager in domain | `aspnet-core/src/EventCloud.Core/Events/EventManager.cs` | 73–80 |
| | No lazy loading | `aspnet-core/src/EventCloud.EntityFrameworkCore/EntityFrameworkCore/EventCloudDbContextConfigurer.cs` | Full file |
| **LINQ to EF / Queryables** | Queryable chain (events) | `aspnet-core/src/EventCloud.Application/Events/EventAppService.cs` | 32–39 |
| | Queryable (paged / sorting) | `aspnet-core/src/EventCloud.Application/Users/UserAppService.cs` | 161–166, 181–184 |
| | Queryable + CountAsync | `aspnet-core/src/EventCloud.Core/Events/EventRegistrationPolicy.cs` | 42–54 |
| **Migrations** | Up/Down | `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/20231227125039_Added_Event.cs` | Full file |
| | Naming / versioning | `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/` | Folder listing |
| | Designer / snapshot | `aspnet-core/src/EventCloud.EntityFrameworkCore/Migrations/20231227125039_Added_Event.Designer.cs` | Header + one entity block |

Full narrative and talking points: **docs/coaching-script-eventcloud.md**
