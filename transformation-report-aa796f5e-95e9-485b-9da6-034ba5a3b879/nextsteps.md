# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a NuGet package restore to ensure all dependencies are resolved correctly before building:

```bash
dotnet restore
```

Review the output for any warnings about deprecated or incompatible packages and update them if necessary using:

```bash
dotnet list package --outdated
dotnet add package <PackageName>
```

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Ensure the build completes with zero errors and review any warnings that may indicate compatibility concerns.

---

## 3. Run Unit Tests

Execute the test project to verify that existing business logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any failing tests that may indicate behavioral regressions
- Any skipped tests that may need to be re-enabled
- Code coverage gaps in critical domain logic

---

## 4. Validate the Data Layer

Since `Bookstore.Data` handles data access, verify the following:

- **Database provider compatibility**: Confirm that the Entity Framework Core (or whichever ORM is in use) provider targets the correct cross-platform version.
- **Connection strings**: Ensure connection strings in `appsettings.json` or environment variables are correctly configured for the target environment.
- **Migrations**: If using Entity Framework Core, verify existing migrations are intact and apply them against a test database:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

---

## 5. Run the Web Application Locally

Start the web application to confirm it runs correctly on the new .NET runtime:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Manually verify the following:
- Application starts without runtime exceptions
- Key pages and routes load correctly
- Data reads and writes function as expected against the database
- Any authentication or authorization flows behave correctly

Check the application logs for runtime warnings or errors that did not surface at build time.

---

## 6. Validate the CDK Project

If `Bookstore.Cdk` defines infrastructure, review the generated output to confirm it reflects the intended target environment:

```bash
dotnet build app/Bookstore.Cdk/Bookstore.Cdk.csproj --configuration Release
```

If this project uses the AWS CDK, run a synthesis step to validate the infrastructure definition:

```bash
cdk synth
```

Review the synthesized output for any configuration that may need to be updated to reflect the new runtime or deployment target.

---

## 7. Review Target Framework Versions

Confirm that all projects are targeting a consistent and supported .NET version. Open each `.csproj` file and verify the `<TargetFramework>` element is uniform across the solution, for example:

```xml
<TargetFramework>net8.0</TargetFramework>
```

Mixing target frameworks across projects can cause runtime issues even when the build succeeds.

---

## 8. Deploy to Target Environment

Once all local validation steps pass:

1. Publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

2. Verify the contents of the `./publish` directory are complete.
3. Deploy the published output to the target hosting environment according to your infrastructure setup.
4. Run a smoke test against the deployed environment to confirm the application is functioning correctly end to end.