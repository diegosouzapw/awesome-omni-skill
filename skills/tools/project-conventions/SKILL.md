---
name: project-conventions
description:
  This skill should be used when creating or editing .csproj files, managing NuGet packages,
  configuring Directory.Build.props or Directory.Packages.props, organizing .NET solutions, or
  setting up global.json and .editorconfig.
version: 0.1.0
---

# .NET Project Conventions and Build Configuration

This skill defines comprehensive conventions for .NET project structure, Central Package Management,
build configuration, solution organization, and tooling setup.

## SDK-Style .csproj Structure

### Minimal .csproj for Source Projects

Source project files should be as minimal as possible. Properties shared across all projects belong
in `Directory.Build.props`, not in each `.csproj`.

```xml
<!-- CORRECT: Minimal .csproj relying on Directory.Build.props -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <RootNamespace>Catalog.Application</RootNamespace>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Catalog.Domain\Catalog.Domain.csproj" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="MediatR" />
  </ItemGroup>

</Project>
```

```xml
<!-- WRONG: Duplicating properties that belong in Directory.Build.props -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <LangVersion>latest</LangVersion>
    <RootNamespace>Catalog.Application</RootNamespace>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Catalog.Domain\Catalog.Domain.csproj" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="MediatR" Version="12.4.1" />
  </ItemGroup>

</Project>
```

### Web API Project Uses Web SDK

```xml
<!-- CORRECT: Web SDK for ASP.NET Core projects -->
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <RootNamespace>Catalog.Api</RootNamespace>
  </PropertyGroup>

</Project>
```

```xml
<!-- WRONG: Standard SDK with manual ASP.NET references -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <RootNamespace>Catalog.Api</RootNamespace>
    <OutputType>Exe</OutputType>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

</Project>
```

### Test Project Inherits from tests/Directory.Build.props

```xml
<!-- CORRECT: Test project with minimal configuration -->
<Project Sdk="Microsoft.NET.Sdk">

  <ItemGroup>
    <ProjectReference Include="..\..\src\Catalog.Application\Catalog.Application.csproj" />
  </ItemGroup>

</Project>
```

```xml
<!-- WRONG: Test project duplicating shared test dependencies -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\..\src\Catalog.Application\Catalog.Application.csproj" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.11.1" />
    <PackageReference Include="xunit" Version="2.9.2" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.8.2" />
    <PackageReference Include="FluentAssertions" Version="6.12.2" />
    <PackageReference Include="NSubstitute" Version="5.3.0" />
    <PackageReference Include="coverlet.collector" Version="6.0.2" />
  </ItemGroup>

</Project>
```

## Central Package Management (CPM)

### Always Use Directory.Packages.props

All package versions must be declared centrally. Individual `.csproj` files reference packages
without version numbers.

```xml
<!-- CORRECT: Directory.Packages.props declares all versions -->
<Project>

  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
    <CentralPackageTransitivePinningEnabled>true</CentralPackageTransitivePinningEnabled>
  </PropertyGroup>

  <ItemGroup>
    <PackageVersion Include="MediatR" Version="12.4.1" />
    <PackageVersion Include="FluentValidation" Version="11.11.0" />
    <PackageVersion Include="xunit" Version="2.9.2" />
  </ItemGroup>

</Project>
```

```xml
<!-- CORRECT: .csproj references without Version -->
<ItemGroup>
  <PackageReference Include="MediatR" />
  <PackageReference Include="FluentValidation" />
</ItemGroup>
```

```xml
<!-- WRONG: Version on PackageReference when CPM is enabled -->
<ItemGroup>
  <PackageReference Include="MediatR" Version="12.4.1" />
</ItemGroup>
```

### Group Packages by Category in CPM

```xml
<!-- CORRECT: Grouped and commented -->
<Project>

  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>

  <!-- ASP.NET Core -->
  <ItemGroup>
    <PackageVersion Include="Swashbuckle.AspNetCore" Version="6.9.0" />
    <PackageVersion Include="Serilog.AspNetCore" Version="8.0.3" />
  </ItemGroup>

  <!-- Entity Framework Core -->
  <ItemGroup>
    <PackageVersion Include="Microsoft.EntityFrameworkCore" Version="8.0.11" />
    <PackageVersion Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="8.0.11" />
  </ItemGroup>

  <!-- Testing -->
  <ItemGroup>
    <PackageVersion Include="xunit" Version="2.9.2" />
    <PackageVersion Include="FluentAssertions" Version="6.12.2" />
    <PackageVersion Include="NSubstitute" Version="5.3.0" />
  </ItemGroup>

</Project>
```

```xml
<!-- WRONG: Flat, unsorted, no grouping -->
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="xunit" Version="2.9.2" />
    <PackageVersion Include="Swashbuckle.AspNetCore" Version="6.9.0" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore" Version="8.0.11" />
    <PackageVersion Include="NSubstitute" Version="5.3.0" />
    <PackageVersion Include="Serilog.AspNetCore" Version="8.0.3" />
  </ItemGroup>
</Project>
```

### Enable Transitive Pinning

Always enable `CentralPackageTransitivePinningEnabled` to control transitive dependency versions.

```xml
<!-- CORRECT: Transitive pinning enabled -->
<PropertyGroup>
  <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  <CentralPackageTransitivePinningEnabled>true</CentralPackageTransitivePinningEnabled>
</PropertyGroup>
```

```xml
<!-- WRONG: Missing transitive pinning -->
<PropertyGroup>
  <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
</PropertyGroup>
```

## Directory.Build.props

### Root Props for Shared Settings

Place at the solution root. Applies to all projects in the directory tree.

```xml
<!-- CORRECT: Shared properties at solution root -->
<Project>

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <LangVersion>latest</LangVersion>
    <AnalysisLevel>latest-recommended</AnalysisLevel>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <Deterministic>true</Deterministic>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Meziantou.Analyzer" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" PrivateAssets="all" />
  </ItemGroup>

</Project>
```

```xml
<!-- WRONG: Missing critical settings -->
<Project>

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>

</Project>
```

### Test Props Import Parent Then Add Test-Specific Settings

```xml
<!-- CORRECT: tests/Directory.Build.props -->
<Project>

  <Import Project="$([MSBuild]::GetPathOfFileAbove('Directory.Build.props', '$(MSBuildThisFileDirectory)../'))" />

  <PropertyGroup>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
    <NoWarn>$(NoWarn);CA1707</NoWarn>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="coverlet.collector" />
    <PackageReference Include="FluentAssertions" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" />
    <PackageReference Include="NSubstitute" />
    <PackageReference Include="xunit" />
    <PackageReference Include="xunit.runner.visualstudio" />
  </ItemGroup>

</Project>
```

```xml
<!-- WRONG: Not importing parent props -->
<Project>

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>

  <!-- Lost all parent settings: Nullable, TreatWarningsAsErrors, analyzers -->

</Project>
```

## Solution Layout

### Layered Architecture with src/ and tests/

```text
CORRECT:
project-root/
├── src/
│   ├── MyApp.Api/           (Web layer)
│   ├── MyApp.Application/   (Business logic)
│   ├── MyApp.Domain/        (Domain models)
│   └── MyApp.Infrastructure/ (Data access, external)
├── tests/
│   ├── Directory.Build.props (test-specific)
│   ├── MyApp.Application.Tests/
│   ├── MyApp.Domain.Tests/
│   └── MyApp.Api.Tests/
├── Directory.Build.props     (shared)
├── Directory.Packages.props  (CPM)
├── global.json
├── .editorconfig
└── MyApp.sln
```

```text
WRONG:
project-root/
├── MyApp.Api/
├── MyApp.Application/
├── MyApp.Domain/
├── MyApp.Infrastructure/
├── MyApp.Tests/             (single test project for everything)
└── MyApp.sln
```

### Test Project Mirrors Source Project

```text
CORRECT:
src/Catalog.Application/Services/ProductService.cs
tests/Catalog.Application.Tests/Services/ProductServiceTests.cs

src/Catalog.Domain/Models/Order.cs
tests/Catalog.Domain.Tests/Models/OrderTests.cs
```

```text
WRONG:
src/Catalog.Application/Services/ProductService.cs
tests/Tests/ProductServiceTest.cs

src/Catalog.Domain/Models/Order.cs
tests/Tests/OrderTests.cs
```

## global.json

### Always Pin SDK Version

```json
{
  "sdk": {
    "version": "8.0.404",
    "rollForward": "latestPatch",
    "allowPrerelease": false
  }
}
```

```json
// WRONG: No global.json at all
// Different developers may use different SDK versions,
// causing inconsistent builds
```

### Use latestPatch Roll-Forward

```json
{
  "sdk": {
    "version": "8.0.404",
    "rollForward": "latestPatch"
  }
}
```

```json
// WRONG: latestMajor allows any SDK
{
  "sdk": {
    "version": "8.0.404",
    "rollForward": "latestMajor"
  }
}
```

## .editorconfig

### Must Exist at Solution Root

Every .NET solution must have an `.editorconfig` at the root. At minimum, it must set:

```text
# CORRECT: Minimum .editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.{csproj,props,targets}]
indent_size = 2

[*.cs]
csharp_style_namespace_declarations = file_scoped:warning
dotnet_style_qualification_for_field = false:warning
```

### Enforce File-Scoped Namespaces

```text
# CORRECT: Enforce file-scoped namespaces
[*.cs]
csharp_style_namespace_declarations = file_scoped:warning
```

```text
# WRONG: No namespace style enforcement
[*.cs]
csharp_style_namespace_declarations = file_scoped:suggestion
```

### Enforce Naming Conventions

```text
# CORRECT: Naming rules enforced as errors
dotnet_naming_rule.interfaces_must_begin_with_i.severity = error
dotnet_naming_rule.types_must_be_pascal_case.severity = error
dotnet_naming_rule.private_fields_must_be_camel_case.severity = warning
```

```text
# WRONG: Naming rules as suggestions (not enforced)
dotnet_naming_rule.interfaces_must_begin_with_i.severity = suggestion
dotnet_naming_rule.types_must_be_pascal_case.severity = suggestion
```

## NuGet Publishing

### Library Projects Include Package Metadata

```xml
<!-- CORRECT: Package metadata in .csproj for publishable library -->
<PropertyGroup>
    <PackageId>Acme.Shared.Contracts</PackageId>
    <Version>1.0.0</Version>
    <Description>Shared contracts and DTOs for Acme services</Description>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>
    <PublishRepositoryUrl>true</PublishRepositoryUrl>
    <EmbedUntrackedSources>true</EmbedUntrackedSources>
</PropertyGroup>
```

```xml
<!-- WRONG: Missing symbol packages and source link -->
<PropertyGroup>
    <PackageId>Acme.Shared.Contracts</PackageId>
    <Version>1.0.0</Version>
</PropertyGroup>
```

### Non-Publishable Projects Are Marked IsPackable false

```xml
<!-- CORRECT: Test and API projects are not packaged -->
<PropertyGroup>
    <IsPackable>false</IsPackable>
</PropertyGroup>
```

```xml
<!-- WRONG: Not setting IsPackable on non-library projects -->
<!-- dotnet pack would create a .nupkg for the API project -->
```

## Target Framework

### Use Latest LTS Framework

```xml
<!-- CORRECT: Latest LTS -->
<PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
</PropertyGroup>
```

```xml
<!-- WRONG: End-of-life or preview framework -->
<PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
</PropertyGroup>
```

### Libraries May Multi-Target for Compatibility

```xml
<!-- CORRECT: Multi-target for broad compatibility -->
<PropertyGroup>
    <TargetFrameworks>net8.0;net9.0</TargetFrameworks>
</PropertyGroup>
```

```xml
<!-- WRONG: Multi-target including unsupported frameworks -->
<PropertyGroup>
    <TargetFrameworks>net6.0;net7.0;net8.0</TargetFrameworks>
</PropertyGroup>
```

## Analyzer Configuration

### Analyzers Are Configured in Directory.Build.props

```xml
<!-- CORRECT: Analyzers in Directory.Build.props, applied to all projects -->
<ItemGroup>
    <PackageReference Include="Meziantou.Analyzer" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" PrivateAssets="all" />
</ItemGroup>
```

```xml
<!-- WRONG: Analyzers added per project -->
<!-- In Catalog.Api.csproj -->
<ItemGroup>
    <PackageReference Include="Meziantou.Analyzer" PrivateAssets="all" />
</ItemGroup>

<!-- In Catalog.Application.csproj -->
<ItemGroup>
    <PackageReference Include="Meziantou.Analyzer" PrivateAssets="all" />
</ItemGroup>
```

### Analyzer Severity in .editorconfig, Not .csproj

```text
# CORRECT: Severity in .editorconfig
[*.cs]
dotnet_diagnostic.CA1062.severity = none
dotnet_diagnostic.CA2007.severity = none
dotnet_diagnostic.IDE0005.severity = warning
```

```xml
<!-- WRONG: Severity in .csproj -->
<PropertyGroup>
    <NoWarn>$(NoWarn);CA1062;CA2007</NoWarn>
</PropertyGroup>
```

## Build Determinism

### Enable Deterministic Builds

```xml
<!-- CORRECT: Deterministic build in Directory.Build.props -->
<PropertyGroup>
    <Deterministic>true</Deterministic>
    <ContinuousIntegrationBuild Condition="'$(CI)' == 'true'">true</ContinuousIntegrationBuild>
</PropertyGroup>
```

```xml
<!-- WRONG: No determinism settings -->
<!-- Builds may produce different binaries for the same source -->
```

## dotnet format Integration

### Format Exclusions Are Applied Consistently

Always exclude generated code from formatting checks:

```bash
# CORRECT: Exclude generated code
dotnet format --verify-no-changes --exclude obj/ --exclude Migrations/
```

```bash
# WRONG: No exclusions (fails on EF migrations)
dotnet format --verify-no-changes
```

## .gitignore Essentials

### Must Ignore Build Artifacts and IDE Files

```text
# CORRECT: Standard .NET .gitignore
[Bb]in/
[Oo]bj/
TestResults/
.vs/
.idea/
*.user
*.suo
launchSettings.json
```

```text
# WRONG: Missing critical entries
bin/
obj/
# Missing .vs/, TestResults/, *.user
```
