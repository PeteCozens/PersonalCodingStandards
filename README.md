# Personal Coding Standards
This document contains my personal standards for developing C# projects for Windows and Azure. It is intended solely to provide consistency and maintainability of my code.

# Development Languages
* All development is to be done in C#, using .Net 8
* Web Applications are to be written in either Blazor or Asp.Net Core, depending on requirements
* Transactional / Relational data should be stored in either a Microsoft SQL Server or Azure SQL database
* Each solution must contain a Unit Test project, that contains unit tests for all functionality using the Microsoft Test SDK. The unit test project will only be built in the Debug configuration
* All projects will perform diagnostic logging (locally in the case of windows software, or Application Insights for Azure-hosted systems)
* Applications should always use dependency injection unless they are considered too basic to require it. Once the application has started, and the configuration has been read, the appropriate services can be added to the ServiceProvider based on the configuration settings.
* Applications should generally use Entity Framework (Code First) for interaction with their own databases. Dapper should be used to query third-party databases and may also be used to call stored procedures.

# NuGet packages
## Package selection
* Any NuGet packages that have not been maintained within the last three years are generally considered defunct and should not be used.
* Whenever functionality from a NuGet package is used within a C# class, the name of the package it is from should always be included as a comment at the top of the file, generally after any "using" statements that reference that package. This comment should also contain details of any external dependencies that are not included in the package.

# Configuration
* All configuration values should be stored in an environment-specific appsettings.json files.
* In the case of sensitive data, such as connection strings, the values in the .json files should read "sensitive". The actual values should be stored in a local secrets file in the development environment, and in environment variables in test or production.
* Configuration settings should be read in the following order:
  1. appsettings.json
  2. appsettings.{environment}.json
  3. local secrets (Debug mode only)
  4. environment variables 
  5. Azure key vault (if appropriate)
* Configuration settings should be read in application startup. If the application uses dependency injection, the configuration should be added to the application's ServiceProvider

# Azure
* All resources should be hosted in the North Europe region unless otherwise specified
* All resources should use Free Tiers in debug and test environments, or the lowest cost alternatives if no free tier is available
* All logging will use Azure Application Insights
* App Services such as Web Apps, Functions and so on are not to be created for Debug environments as it is expected that they will be run in a local debugger
  
## Naming conventions
Resources in Azure will follow a strict naming convention, as follows:

* Resource groups will be named after the application, all in lower case with dashes instead of any spaces. In the case of a project with multiple environments, this will be followed by a dash and the first four letters of the environment name in lower case.
* Azure Web Apps and App Services will be named "app-" followed by the resource group name
* Azure Functions will be named "fn-" followed by the resource group name
* Azure Insights will be named "in-" followed by the resource group name
* SQL Databases will be named "sql-" followed by the resource group name
* Storage Accounts will be named "st-" followed by the resource group name
* Azure Key Vaults will be named "kv-" followed by the resource group name

# Terraform
Any project or solution that depends upon Azure resources should use an Infrastructure As Code approach, and generate the appropriate Terraform files to deploy and tear-down all required Azure resources, including storing connection strings for all generated resources as environment variables or in Azure Key Vault (if a vault has been specified as part of the project requirements). Generally, projects will have Development, Test and Production environments, and in such cases a terraform variables file will be required for each to define any environment-specific variables and configuration (such as the environment name, which will feed into Azure resource names)
