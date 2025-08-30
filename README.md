# Personal Coding Standards (C# + Azure)

> Scope: All C#/.NET 8 repositories for Windows & Azure. This document enforces consistent engineering practice across apps, libraries, tests, scripts, and CI/CD.

---

## 1. Languages & Platforms
- **C# / .NET 8** for all development.
- Web apps in **Blazor** or **ASP.NET Core** (choose per requirements).
- **Relational data** hosted in **Microsoft SQL Server** or **Azure SQL**.
- **Unit tests**: a MSTest project **in every solution**; **only built in Debug**.
- **Logging** is mandatory: local sinks for Windows apps; **Azure Application Insights** for Azure-hosted systems.
- Prefer **Dependency Injection**; after startup/config read, register services into the DI container based on config.
- **Data access**: prefer **EF Core (Code-First)** for app-owned databases; use **Dapper** for third-party DBs and for calling stored procedures.


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
- **Resource Group**: application name, **lowercase**, dashes for spaces; for multi-env, append `-` plus first four letters of environment (lowercase).
- **Prefixes**:
  - Web Apps / App Services: `app-<rg>`
  - Functions: `fn-<rg>`
  - Application Insights: `in-<rg>`
  - SQL Databases: `sql-<rg>`
  - Storage Accounts: `st-<rg>`
  - Key Vaults: `kv-<rg>`

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
  <TargetFramework>net8.0</TargetFramework>
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

---

## 8. Deviation Policy
- If you must deviate, add a short **justification comment** and link the tracking issue.
- Suppress analyzer rules **locally** (not globally) with justification.

---
