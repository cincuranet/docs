---
title: .NET project SDK overview
titleSuffix: ""
description: Learn about the .NET project SDKs.
ms.date: 10/15/2024
ms.topic: concept-article
no-loc: ["EmbeddedResource", "Compile", "None", "Blazor"]
---
# .NET project SDKs

Modern .NET projects are associated with a project software development kit (SDK). Each *project SDK* is a set of MSBuild [targets](/visualstudio/msbuild/msbuild-targets) and associated [tasks](/visualstudio/msbuild/msbuild-tasks) that are responsible for compiling, packing, and publishing code. A project that references a project SDK is sometimes referred to as an *SDK-style project*.

## Available SDKs

The available SDKs include:

| ID                         | Description                                                                          | Repo                                   |
|----------------------------|--------------------------------------------------------------------------------------|----------------------------------------|
| `Microsoft.NET.Sdk`        | The .NET SDK                                                                         | <https://github.com/dotnet/sdk>        |
| `Microsoft.NET.Sdk.Web`    | The .NET [Web SDK](/aspnet/core/razor-pages/web-sdk)                                 | <https://github.com/dotnet/sdk>        |
| `Microsoft.NET.Sdk.Razor`  | The .NET [Razor SDK](/aspnet/core/razor-pages/sdk)                                   | <https://github.com/dotnet/aspnetcore> |
| `Microsoft.NET.Sdk.BlazorWebAssembly` | The .NET [Blazor WebAssembly SDK](/aspnet/core/blazor#blazor-webassembly) | <https://github.com/dotnet/aspnetcore> |
| `Microsoft.NET.Sdk.Worker` | The .NET [Worker Service SDK](../extensions/workers.md)                              | <https://github.com/dotnet/aspnetcore> |
| `Aspire.AppHost.Sdk`       | The .NET [Aspire SDK](/dotnet/aspire/fundamentals/dotnet-aspire-sdk)                 | <https://github.com/dotnet/aspire>     |
| `MSTest.Sdk`               | The [MSTest SDK](../testing/unit-testing-mstest-sdk.md)                              | <https://github.com/microsoft/testfx>  |

The .NET SDK is the base SDK for .NET. The other SDKs reference the .NET SDK, and projects that are associated with the other SDKs have all the .NET SDK properties available to them. The Web SDK, for example, depends on both the .NET SDK and the Razor SDK.

For Windows Forms and Windows Presentation Foundation (WPF) projects, you specify the .NET SDK (`Microsoft.NET.Sdk`) and set some additional properties in the project file. For more information, see [Enable .NET Desktop SDK](msbuild-props-desktop.md#enable-net-desktop-sdk).

MSBuild SDKs, which you can use to configure and extend your build, are listed at [MSBuild SDKs](https://github.com/microsoft/MSBuildSdks/blob/main/README.md).

You can also author your own SDK that can be distributed via NuGet.

## Project files

.NET projects are based on the [MSBuild](/visualstudio/msbuild/msbuild) format. Project files, which have extensions like *.csproj* for C# projects and *.fsproj* for F# projects, are in XML format. The root element of an MSBuild project file is the [Project](/visualstudio/msbuild/project-element-msbuild) element. The `Project` element has an optional `Sdk` attribute that specifies which SDK (and version) to use. To use the .NET tools and build your code, set the `Sdk` attribute to one of the IDs in the [Available SDKs](#available-sdks) table.

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <!-- Omitted for brevity... -->
</Project>
```

The `Project/Sdk` attribute and `Sdk` element enable additive SDKs. Consider the following example, where the .NET Aspire SDK (`Aspire.AppHost.Sdk`) is added to the project atop the `Microsoft.NET.Sdk`:

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <Sdk Name="Aspire.AppHost.Sdk" Version="9.0.0" />
    <!-- Omitted for brevity... -->

</Project>
```

In the preceding project file, both SDKs are used to resolve dependencies in an additive nature. For more information, see [.NET Aspire SDK](/dotnet/aspire/fundamentals/dotnet-aspire-sdk).

To specify an SDK that comes from NuGet, include the version at the end of the name, or specify the name and version in the *global.json* file.

```xml
<Project Sdk="MSBuild.Sdk.Extras/2.0.54">
  ...
</Project>
```

Another way to specify the SDK is with the top-level [`Sdk`](/visualstudio/msbuild/sdk-element-msbuild) element:

```xml
<Project>
  <Sdk Name="Microsoft.NET.Sdk" />
  ...
</Project>
```

Referencing an SDK in one of these ways greatly simplifies project files for .NET. While evaluating the project, MSBuild adds implicit imports for `Sdk.props` at the top of the project file and `Sdk.targets` at the bottom.

```xml
<Project>
  <!-- Implicit top import -->
  <Import Project="Sdk.props" Sdk="Microsoft.NET.Sdk" />
  ...
  <!-- Implicit bottom import -->
  <Import Project="Sdk.targets" Sdk="Microsoft.NET.Sdk" />
</Project>
```

> [!TIP]
> On a Windows machine, the *Sdk.props* and *Sdk.targets* files can be found in the *%ProgramFiles%\dotnet\sdk\\[version]\Sdks\Microsoft.NET.Sdk\Sdk* folder.

### Preprocess the project file

You can see the fully expanded project as MSBuild sees it after the SDK and its targets are included by using the `dotnet msbuild -preprocess` command. The [preprocess](/visualstudio/msbuild/msbuild-command-line-reference#preprocess) switch of the [`dotnet msbuild`](../tools/dotnet-msbuild.md) command shows which files are imported, their sources, and their contributions to the build without actually building the project.

If the project has multiple target frameworks, focus the results of the command on only one framework by specifying it as an MSBuild property. For example:

`dotnet msbuild -property:TargetFramework=net8.0 -preprocess:output.xml`

## Default includes and excludes

The default includes and excludes for [`Compile` items](/visualstudio/msbuild/common-msbuild-project-items#compile), [embedded resources](/visualstudio/msbuild/common-msbuild-project-items#embeddedresource), and [`None` items](/visualstudio/msbuild/common-msbuild-project-items#none) are defined in the SDK. Unlike non-SDK .NET Framework projects, you don't need to specify these items in your project file, because the defaults cover most common use cases. This behavior makes the project file smaller and easier to understand and edit by hand, if needed.

The following table shows which elements and which [globs](https://en.wikipedia.org/wiki/Glob_(programming)) are included and excluded in the .NET SDK:

| Element | Include glob | Exclude glob | Remove glob |
|---------|--------------|--------------|-------------|
| Compile | \*\*/\*.cs (or other language extensions) | \*\*/\*.user;  \*\*/\*.\*proj;  \*\*/\*.sln(x);  \*\*/\*.vssscc  | N/A |
| EmbeddedResource  | \*\*/\*.resx | \*\*/\*.user; \*\*/\*.\*proj; \*\*/\*.sln(x); \*\*/\*.vssscc | N/A |
| None | \*\*/\* | \*\*/\*.user; \*\*/\*.\*proj; \*\*/\*.sln(x); \*\*/\*.vssscc     | \*\*/\*.cs; \*\*/\*.resx |

> [!NOTE]
> The `./bin` and `./obj` folders, which are represented by the `$(BaseOutputPath)` and `$(BaseIntermediateOutputPath)` MSBuild properties, are excluded from the globs by default. Excludes are represented by the [DefaultItemExcludes property](msbuild-props.md#defaultitemexcludes).

The .NET Desktop SDK has additional includes and excludes for WPF. For more information, see [WPF default includes and excludes](msbuild-props-desktop.md#wpf-default-includes-and-excludes).

If you explicitly define any of these items in your project file, you're likely to get a [NETSDK1022](../tools/sdk-errors/netsdk1022.md) build error. For information about how to resolve the error, see [NETSDK1022: Duplicate items were included](../tools/sdk-errors/netsdk1022.md).

## Implicit using directives

Starting in .NET 6, implicit [`global using` directives](../../csharp/language-reference/keywords/using-directive.md#the-global-modifier) are added to new C# projects. This means that you can use types defined in these namespaces without having to specify their fully qualified name or manually add a `using` directive. The *implicit* aspect refers to the fact that the `global using` directives are added to a generated file in the project's *obj* directory.

Implicit `global using` directives are added for projects that use one of the following SDKs:

- `Microsoft.NET.Sdk`
- `Microsoft.NET.Sdk.Web`
- `Microsoft.NET.Sdk.Worker`
- `Microsoft.NET.Sdk.WindowsDesktop`

A `global using` directive is added for each namespace in a set of default namespaces that are based on the project's SDK. These default namespaces are shown in the following table.

| SDK | Default namespaces |
|-----|--------------------|
| Microsoft.NET.Sdk | <xref:System><br/><xref:System.Collections.Generic?displayProperty=fullName><br/><xref:System.IO?displayProperty=fullName><br/><xref:System.Linq?displayProperty=fullName><br/><xref:System.Net.Http?displayProperty=fullName><br/><xref:System.Threading?displayProperty=fullName><br/><xref:System.Threading.Tasks?displayProperty=fullName> |
| Microsoft.NET.Sdk.Web | Microsoft.NET.Sdk namespaces<br/><xref:System.Net.Http.Json?displayProperty=fullName><br/><xref:Microsoft.AspNetCore.Builder?displayProperty=fullName><br/><xref:Microsoft.AspNetCore.Hosting?displayProperty=fullName><br/><xref:Microsoft.AspNetCore.Http?displayProperty=fullName><br/><xref:Microsoft.AspNetCore.Routing?displayProperty=fullName><br/><xref:Microsoft.Extensions.Configuration?displayProperty=fullName><br/><xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName><br/><xref:Microsoft.Extensions.Hosting?displayProperty=fullName><br/><xref:Microsoft.Extensions.Logging?displayProperty=fullName> |
| Microsoft.NET.Sdk.Worker | Microsoft.NET.Sdk namespaces<br/><xref:Microsoft.Extensions.Configuration?displayProperty=fullName><br/><xref:Microsoft.Extensions.DependencyInjection?displayProperty=fullName><br/><xref:Microsoft.Extensions.Hosting?displayProperty=fullName><br/><xref:Microsoft.Extensions.Logging?displayProperty=fullName> |
| Microsoft.NET.Sdk.WindowsDesktop (Windows Forms) | Microsoft.NET.Sdk namespaces<br/><xref:System.Drawing?displayProperty=fullName><br/><xref:System.Windows.Forms?displayProperty=fullName> |
| Microsoft.NET.Sdk.WindowsDesktop (WPF) | Microsoft.NET.Sdk namespaces<br/>Removed <xref:System.IO?displayProperty=fullName><br/>Removed <xref:System.Net.Http?displayProperty=fullName> |

If you want to disable this feature, or if you want to enable implicit `global using` directives in an existing C# project, you can do so via the [`ImplicitUsings` MSBuild property](msbuild-props.md#implicitusings).

You can specify additional implicit `global using` directives by adding `Using` items (or `Import` items for Visual Basic projects) to your project file, for example:

  ```xml
  <ItemGroup>
    <Using Include="System.IO.Pipes" />
  </ItemGroup>
  ```

> [!NOTE]
> Starting with the .NET 8 SDK, <xref:System.Net.Http> is [no longer included](../compatibility/sdk/8.0/implicit-global-using-netfx.md) in `Microsoft.NET.Sdk` when targeting .NET Framework.

## Implicit package references

When your project targets .NET Standard 1.0-2.0, the .NET SDK adds implicit references to certain *metapackages*. A metapackage is a framework-based package that consists only of dependencies on other packages. Metapackages are implicitly referenced based on the target frameworks specified in the [TargetFramework](msbuild-props.md#targetframework) or [TargetFrameworks (plural)](msbuild-props.md#targetframeworks) property of your project file.

```xml
<PropertyGroup>
  <TargetFramework>netstandard2.0</TargetFramework>
</PropertyGroup>
```

```xml
<PropertyGroup>
  <TargetFrameworks>netstandard2.0;net462</TargetFrameworks>
</PropertyGroup>
```

If needed, you can disable implicit package references using the [DisableImplicitFrameworkReferences](msbuild-props.md#disableimplicitframeworkreferences) property, and add explicit references to just the frameworks or packages you need.

Recommendations:

- When targeting .NET Framework or .NET Standard 1.0-2.0, don't add an explicit reference to the `NETStandard.Library` metapackages via a `<PackageReference>` item in your project file. For .NET Standard 1.0-2.0 projects, these metapackages are implicitly referenced. For .NET Framework projects, if any version of `NETStandard.Library` is needed when using a .NET Standard-based NuGet package, NuGet automatically installs that version.
- If you need a specific version of the `NETStandard.Library` metapackage when targeting .NET Standard 1.0-2.0, you can use the `<NetStandardImplicitPackageVersion>` property and set the version you need.

## Build events

In SDK-style projects, use an MSBuild target named `PreBuild` or `PostBuild` and set the `BeforeTargets` property for `PreBuild` or the `AfterTargets` property for `PostBuild`.

```xml
<Target Name="PreBuild" BeforeTargets="PreBuildEvent">
    <Exec Command="&quot;$(ProjectDir)PreBuildEvent.bat&quot; &quot;$(ProjectDir)..\&quot; &quot;$(ProjectDir)&quot; &quot;$(TargetDir)&quot;" />
</Target>

<Target Name="PostBuild" AfterTargets="PostBuildEvent">
   <Exec Command="echo Output written to $(TargetDir)" />
</Target>
```

> [!NOTE]
>
> - You can use any name for the MSBuild targets. However, the Visual Studio IDE recognizes `PreBuild` and `PostBuild` targets, so by using those names, you can edit the commands in the IDE.
> - The properties `PreBuildEvent` and `PostBuildEvent` are not recommended in SDK-style projects, because macros such as `$(ProjectDir)` aren't resolved. For example, the following code is not supported:
>
> ```xml
> <PropertyGroup>
>   <PreBuildEvent>"$(ProjectDir)PreBuildEvent.bat" "$(ProjectDir)..\" "$(ProjectDir)" "$(TargetDir)"</PreBuildEvent>
> </PropertyGroup>
> ```

## Customize the build

There are various ways to [customize a build](/visualstudio/msbuild/customize-your-build). You might want to override a property by passing it as an argument to the [msbuild](/visualstudio/msbuild/msbuild-command-line-reference) or [dotnet](../tools/index.md) command. You can also add the property to the project file or to a [*Directory.Build.props* file](/visualstudio/msbuild/customize-by-directory#directorybuildprops-and-directorybuildtargets). For a list of useful properties for .NET projects, see [MSBuild reference for .NET SDK projects](msbuild-props.md).

> [!TIP]
> An easy way to create a new *Directory.Build.props* file from the command line is by using the command `dotnet new buildprops` at the root of your repository.

### Custom targets

.NET projects can package custom MSBuild targets and properties for use by projects that consume the package. Use this type of extensibility when you want to:

- Extend the build process.
- Access artifacts of the build process, such as generated files.
- Inspect the configuration under which the build is invoked.

You add custom build targets or properties by placing files in the form `<package_id>.targets` or `<package_id>.props` (for example, `Contoso.Utility.UsefulStuff.targets`) in the *build* folder of the project.

The following XML is a snippet from a *.csproj* file that instructs the [`dotnet pack`](../tools/dotnet-pack.md) command what to package. The `<ItemGroup Label="dotnet pack instructions">` element places the targets files into the *build* folder inside the package. The `<Target Name="CollectRuntimeOutputs" BeforeTargets="_GetPackageFiles">` element places the assemblies and *.json* files into the *build* folder.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  ...
  <ItemGroup Label="dotnet pack instructions">
    <Content Include="build\*.targets">
      <Pack>true</Pack>
      <PackagePath>build\</PackagePath>
    </Content>
  </ItemGroup>
  <Target Name="CollectRuntimeOutputs" BeforeTargets="_GetPackageFiles">
    <!-- Collect these items inside a target that runs after build but before packaging. -->
    <ItemGroup>
      <Content Include="$(OutputPath)\*.dll;$(OutputPath)\*.json">
        <Pack>true</Pack>
        <PackagePath>build\</PackagePath>
      </Content>
    </ItemGroup>
  </Target>
  ...

</Project>
```

To consume a custom target in your project, add a `PackageReference` element that points to the package and its version. Unlike the tools, the custom targets package is included in the consuming project's dependency closure.

You can configure how to use the custom target. Since it's an MSBuild target, it can depend on a given target, run after another target, or be manually invoked by using the `dotnet msbuild -t:<target-name>` command. However, to provide a better user experience, you can combine per-project tools and custom targets. In this scenario, the per-project tool accepts whatever parameters are needed and translates that into the required [`dotnet msbuild`](../tools/dotnet-msbuild.md) invocation that executes the target. You can see a sample of this kind of synergy on the [MVP Summit 2016 Hackathon samples](https://github.com/dotnet/MVPSummitHackathon2016) repo in the [`dotnet-packer`](https://github.com/dotnet/MVPSummitHackathon2016/tree/master/dotnet-packer) project.

## See also

- [Customize your build (MSBuild)](/visualstudio/msbuild/customize-your-build)
- [How to use MSBuild project SDKs](/visualstudio/msbuild/how-to-use-project-sdk)
- [Package custom MSBuild targets and props with NuGet](/nuget/create-packages/creating-a-package#include-msbuild-props-and-targets-in-a-package)
