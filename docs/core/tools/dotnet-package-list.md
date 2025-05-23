---
title: dotnet package list command
description: The 'dotnet package list' command provides a convenient option to list the package references for a project or solution.
ms.date: 04/02/2025
---
# dotnet package list

**This article applies to:** ✔️ .NET Core 3.1 SDK and later versions

## Name

`dotnet package list` - Lists the package references for a project or solution.

> [!NOTE]
> If you're using .NET 9 SDK or earlier, use the "verb first" form (`dotnet list package`) instead. The "noun first" form was introduced in .NET 10. For more information, see [More consistent command order](../whats-new/dotnet-10/sdk.md#more-consistent-command-order).

## Synopsis

```dotnetcli
dotnet package list [--config <SOURCE>]
    [--deprecated] [--project [<PROJECT>|<SOLUTION>]]
    [-f|--framework <FRAMEWORK>] [--highest-minor] [--highest-patch]
    [--include-prerelease] [--include-transitive] [--interactive]
    [--outdated] [--source <SOURCE>] [-v|--verbosity <LEVEL>]
    [--vulnerable]
    [--format <console|json>]
    [--output-version <VERSION>]

dotnet package list -h|--help
```

## Description

The `dotnet package list` command provides a convenient option to list all NuGet package references for a specific project or a solution. You first need to build the project in order to have the assets needed for this command to process. The following example shows the output of the `dotnet package list` command for the [SentimentAnalysis](https://github.com/dotnet/samples/tree/main/machine-learning/tutorials/SentimentAnalysis) project:

```output
Project 'SentimentAnalysis' has the following package references
   [netcoreapp2.1]:
   Top-level Package               Requested   Resolved
   > Microsoft.ML                  1.4.0       1.4.0
   > Microsoft.NETCore.App   (A)   [2.1.0, )   2.1.0

(A) : Auto-referenced package.
```

The **Requested** column refers to the package version specified in the project file and can be a range. The **Resolved** column lists the version that the project is currently using and is always a single value. The packages displaying an `(A)` right next to their names represent implicit package references that are inferred from your project settings (`Sdk` type, or `<TargetFramework>` or `<TargetFrameworks>` property).

Use the `--outdated` option to find out if there are newer versions available of the packages you're using in your projects. By default, `--outdated` lists the latest stable packages unless the resolved version is also a prerelease version. To include prerelease versions when listing newer versions, also specify the `--include-prerelease` option. To update a package to the latest version, use [dotnet package add](dotnet-package-add.md).

The following example shows the output of the `dotnet package list --outdated --include-prerelease` command for the same project as the previous example:

```output
The following sources were used:
   https://api.nuget.org/v3/index.json
   C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\

Project `SentimentAnalysis` has the following updates to its packages
   [netcoreapp2.1]:
   Top-level Package      Requested   Resolved   Latest
   > Microsoft.ML         1.4.0       1.4.0      1.5.0-preview
```

If you need to find out whether your project has transitive dependencies, use the `--include-transitive` option. Transitive dependencies occur when you add a package to your project that in turn relies on another package. The following example shows the output from running the `dotnet package list --include-transitive` command for the [HelloPlugin](https://github.com/dotnet/samples/tree/main/core/extensions/AppWithPlugin/HelloPlugin) project, which displays top-level packages and the packages they depend on:

```output
Project 'HelloPlugin' has the following package references
   [netcoreapp3.0]:
   Transitive Package      Resolved
   > PluginBase            1.0.0
```

## Arguments

`PROJECT | SOLUTION`

The project or solution file to operate on. If not specified, the command searches the current directory for one. If more than one solution or project is found, an error is thrown.

## Options

- **`--config <SOURCE>`**

  The NuGet sources to use when searching for newer packages. Requires the `--outdated` option.

- **`--deprecated`**

  Displays packages that have been deprecated.

- **`-f|--framework <FRAMEWORK>`**

  Displays only the packages applicable for the specified [target framework](../../standard/frameworks.md). To specify multiple frameworks, repeat the option multiple times. For example: `--framework net6.0 --framework netstandard2.0`. The short form of the option (`-f`) is available starting in .NET 9 SDK.

[!INCLUDE [help](../../../includes/cli-help.md)]

- **`--highest-minor`**

  Considers only the packages with a matching major version number when searching for newer packages. Requires the `--outdated` or `--deprecated` option.

- **`--highest-patch`**

  Considers only the packages with a matching major and minor version numbers when searching for newer packages. Requires the `--outdated` or `--deprecated` option.

- **`--include-prerelease`**

  Considers packages with prerelease versions when searching for newer packages. Requires the `--outdated` or `--deprecated` option.

- **`--include-transitive`**

  Lists transitive packages, in addition to the top-level packages. When specifying this option, you get a list of packages that the top-level packages depend on.

[!INCLUDE [interactive](../../../includes/cli-interactive-3-0.md)]

- **`--outdated`**

  Lists packages that have newer versions available.

- **`-s|--source <SOURCE>`**

  The NuGet sources to use when searching for newer packages. Requires the `--outdated` or `--deprecated` option.

[!INCLUDE [verbosity](../../../includes/cli-verbosity-minimal.md)]

- **`--vulnerable`**

  Lists packages that have known vulnerabilities. Cannot be combined with `--deprecated` or `--outdated` options.
  Use the `<AuditSources>` property in your configuration file to specify your source of vulnerability data, which is acquired from the [VulnerabilityInfo](/nuget/api/vulnerability-info) resource.
  If `<AuditSources>` is not specified, the specified `<PackageSources>` are used to load vulnerability data.
  For more information, see [Audit sources](/nuget/concepts/auditing-packages#audit-sources) and [How to scan NuGet packages for security vulnerabilities](https://devblogs.microsoft.com/nuget/how-to-scan-nuget-packages-for-security-vulnerabilities/).

- **`--format <console|json>`**

  Sets the report output format. Allowed values are `console`, `json`.  Defaults to `console`. Available starting in .NET SDK 7.0.200.

- **`--output-version <VERSION>`**

  Sets the report output version. Allowed value is `1`. Defaults to `1`. Requires the `--format json` option. When a new JSON version is available, the command will produce the new format by default. This option will let you specify that the command should produce an earlier format. Available starting in .NET SDK 7.0.200.

## Examples

- List package references of a specific project:

  ```dotnetcli
  dotnet package list --project SentimentAnalysis.csproj
  ```

- List package references that have newer versions available, including prerelease versions:

  ```dotnetcli
  dotnet package list --outdated --include-prerelease
  ```

- List package references for a specific target framework:

  ```dotnetcli
  dotnet package list --framework netcoreapp3.0
  ```

- List package references in machine readable json output format:

  ```dotnetcli
  dotnet package list --format json
  ```

- List package references for a specific target framework in machine readable json output format:

  ```dotnetcli
   dotnet package list --framework netcoreapp3.0 --format json
  ```

- Save machine readable json output of package references, including transitive dependency and vulnerability details into a file:

  ```dotnetcli
  dotnet package list --include-transitive --vulnerable --format json >> dependencyReport.json
  ```

- List package references in machine readable json output format with output version 1:

  ```dotnetcli
  dotnet package list --format json --output-version 1
  ```
