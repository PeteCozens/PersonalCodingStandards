# Personal Coding Standards (C# + Azure)

> Scope: All C#/.NET 10 repositories for Windows & Azure. This document enforces consistent engineering practice across apps, libraries, tests, scripts, and CI/CD.

---

## 1. Languages & Platforms
- **C# / .NET 10** for all development.
- Web apps in **Blazor** or **ASP.NET Core** (choose per requirements).
- **Relational data** hosted in **Microsoft SQL Server** or **Azure SQL**.
- **Bulk Data** hosted in **MongoDB** or **CosmosDB**
- **Unit tests**: a MSTest project **in every solution**; **only built in Debug**.
- **Logging** is mandatory: local sinks for Windows apps; **Azure Application Insights** for Azure-hosted systems.
- Prefer **Dependency Injection**; after startup/config read, register services into the DI container based on config.
- **Data access**: prefer **EF Core (Code-First)** for app-owned relational databases; use **Dapper** for third-party relational DBs and for calling stored procedures.
- Larger solutions should implement **Clean Architecture**, splitting the solution into the following projects (at a minimum):
  1. **Domain** - contains Data Models, Interfaces and common code that does NOT implement application logic or interact directly with external resources 
  2. **Infrastructure** - contains classes implementing interfaces in the Domain project for interacting with external resources
  3. **Application** - contains all of the business logic, without interacting directly with any external resources - instances of the appropriate infrastructure classes should be passed as parameters to the constructors or methods as objects of the appropriate Domain interface. In instances where a method with the Application needs to instantiate multiple instances of an infrastructure class object, a factory method should be passed as a Func<IMyInterface> parameter, rather than passing in an instance to the ServiceProvider.
  4. **Program** - contains the code for application startup and User Interface. Reads configuration, populates and hosts the ServiceProvider for Dependency Injection and contains code for the User Interface.
  5. **UnitTests** - contains unit tests for the other projects. Contains a child folder called TestData where files containing raw data for unit tests and expected outputs are stored as a part of the project.


---

## 2. NuGet Packages
**Selection**
- Avoid packages with **no maintenance in the last 3 years**.
- When a file uses a package, add a **comment at the top of the file** listing:
  - The package name(s).
  - Any **external dependencies** not included by that package.

**Governance**
- Prefer **1st-party Microsoft** packages when equivalent.
- Centralize versions via `Directory.Packages.props` where possible.

---

## 3. Configuration
- Store settings in **environment-specific `appsettings.json`** files.
- For **sensitive** values (e.g., connection strings), commit the key with value `"sensitive"`:
  - **Development** actual values in **User Secrets**.
  - **Test/Prod** in **environment variables** (or **Azure Key Vault** when appropriate).
- **Loading order** (earliest wins later):  
  1. `appsettings.json`  
  2. `appsettings.{Environment}.json`  
  3. **User Secrets** (Debug only)  
  4. **Environment variables**  
  5. **Azure Key Vault** (if used)
- Read configuration at **application startup** using a strongly-typed class and register the bound options into DI as a singleton.

---

## 4. Azure Standards
- **Region**: **North Europe** (unless explicitly specified otherwise).
- **Cost**: use **Free tiers** in Debug/Test (or the **lowest-cost** alternative if no free tier available).
- **Telemetry**: all logging flows to **Application Insights**.
- **Debug environment**: **do not create** App Services/Functions in Azure; run locally under debugger.

### 4.1 Azure Resource Naming
In the naming conventions below <rg> is the resource group name and <rnd> is a 6 character random alphanumeric string (lowercase) that is specific to the resource group
- **Resource Group**: application name, mixed case, dashes for spaces
- **App Service Plan**: `<rg>-asp`
- **App Service**: `<rg>-app-<rnd>`
- **Function**: `<rg>-fn-<rnd>`
- **Application Insights**: `<rg>-ai`
- **SQL Databases**: `<rg>-sql`
- **Storage Accounts**: `<rg>st<rnd>` lowercase, with any non-alphanumerics removed
- **Key Vaults**: `<rg>kv<rnd>` lowercase, with any non-alphanumerics removed
- Any other asset types should have their names start with the resource group name, sufficed with an abbreviation of the asset type. If the name needs to be globally unique, then it should be further suffixed with the random string.
- If assets are created for non-production environments, their names should be further suffixed with the environment name (prefixed with a dash, if allowed)
- Azure naming rules MUST be adhered to for every asset type
- Note that all resource names should be modified to comply with Azure naming conventions (all lowercase if required, no dashes if required, etc.)

---

## 5. Infrastructure as Code (Terraform)
- Use **Terraform** for any project that depends on Azure resources. The IaC must:
  - Deploy **all required resources** and **tear down** cleanly.
  - **Export connection strings/secrets** to **environment variables** or **Key Vault** (if specified).
- Environments: typically **Dev**, **Test**, **Prod**.  
  Provide a **variables file per environment** to define environment-specific values (e.g., environment name feeding the Azure naming).

---

## 6. C# Code Style & Project Defaults

### 6.1 Project SDK defaults
Add in each `.csproj` (or central `Directory.Build.props`):

```
<PropertyGroup>
  <TargetFramework>net10.0</TargetFramework>
  <LangVersion>latest</LangVersion>
  <Nullable>enable</Nullable>
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  <AnalysisLevel>latest</AnalysisLevel>
</PropertyGroup>
```

### 6.2 Naming (enforced via EditorConfig)
- **PascalCase**: namespaces, types, public members, events, properties, methods.
- **camelCase**: locals, parameters.
- **_camelCase**: private fields. Use `s_` for private static; `t_` for `[ThreadStatic]`.
- **I** prefix for interfaces; **Async** suffix for async methods; **Completed/Changed** for events.

### 6.3 Formatting & Layout
- **4-space** indentation; **LF** line endings; UTF-8; trim trailing whitespace; final newline.
- **File-scoped namespaces**; one public type per file; filename = type.
- `using` **outside** namespace; `System.*` first; remove unused.
- Prefer **braces**; allow expression-bodied members for trivial accessors.

### 6.4 Language Practices
- Prefer `var` when obvious; explicit type otherwise.
- Use modern features: pattern matching, switch expressions, `using` declarations, `await using`.
- Guard clauses: `ArgumentNullException.ThrowIfNull(x);`
- Avoid double enumeration with LINQ; materialize with `ToList()` when needed.
- **Exceptions**: use specific types; never swallow; don’t use for flow control.
- **Async**: avoid `.Result/.Wait()`; library code uses `ConfigureAwait(false)`.

### 6.5 Dependency Injection & Logging
- Prefer **constructor injection**. Avoid service locator/static state.
- **Structured logging** only (no string concat):  
  `logger.LogInformation("Processed {Count}", count);`  
  Never log secrets/PII.

### 6.6 Documentation & Comments
- XML docs for **public** APIs; comments capture **why**, not what.

### 6.7 Testing (MSTest)
- Deterministic, isolated tests; one logical assertion per test.
- Naming: `Method_Scenario_ExpectedResult`.
- For this repo family, the MSTest project **builds in Debug only** (as per §1).

---

## 7. Security & Secrets
- **Never commit secrets**. Use User Secrets (Dev) and **env vars/Key Vault** (Test/Prod) per §3.
- Do not log secrets/PII. Prefer **redaction** patterns.
- Validate all external input at boundaries; **parameterize** DB access (EF/Dapper).
- Ensure that no secrets are stored in terraform state files

---

## 8. Deviation Policy
- If you must deviate, add a short **justification comment** and link the tracking issue.
- Suppress analyzer rules **locally** (not globally) with justification.

---
