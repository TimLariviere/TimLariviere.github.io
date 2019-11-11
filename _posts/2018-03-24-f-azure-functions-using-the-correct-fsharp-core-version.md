---
layout: post
title: "F# - Azure Functions - Using the correct FSharp.Core version"
date: "2018-03-24"
---

After deploying your beloved Azure Function written in F#, you might end up with the following exception when executing the function: `Could not load file or assembly 'FSharp.Core, Version=4.4.3.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies`

But why such an exception? When running the function locally, all was fine.

That's because when Azure runs your function, it runs it inside a process that is already using an older version of FSharp.Core. And when it tries to load the FSharp.Core.dll from the /bin folder, it fails. The F# Core Engineering team warns us about it: [Some systems supporting F# (e.g. Azure Functions, Azure Notebooks, F# Interactive) may assume a fixed FSharp.Core](https://fsharp.github.io/2015/04/18/fsharp-core-notes.html#some-systems-supporting-f-eg-azure-functions-azure-notebooks-f-interactive-may-assume-a-fixed-fsharpcore)

So what should you do? As of now, there's no way to use a version of FSharp.Core higher than 4.4.1.0 (nuget version 4.2.3). The only way is to force NuGet to restore the FSharp.Core package 4.2.3.

If you're using .NET Standard 2.0 and the new fsproj format, you should have the following project file: 
```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>
</Project>
```

FSharp.Core is included by default when using `Microsoft.NET.Sdk` If you're deploying from Mac, your function will run fine, but not if you're deploying from Windows. That's because the default FSharp.Core included is not the same.

_Microsoft.FSharp.NetSdk.targets on Mac_
```xml
<PropertyGroup Condition=" '$(FSharpCoreImplicitPackageVersion)' == '' ">
    <FSharpCoreImplicitPackageVersion>4.2.*</FSharpCoreImplicitPackageVersion>
</PropertyGroup>
```

_Microsoft.FSharp.NetSdk.targets on Windows_
```xml
<PropertyGroup Condition=" '$(FSharpCoreImplicitPackageVersion)' == '' ">
    <FSharpCoreImplicitPackageVersion>4.3.*</FSharpCoreImplicitPackageVersion>
</PropertyGroup>
```

To ensure that your function will run whatever OS you're deploying from, just set the value of `FSharpCoreImplicitPackageVersion`.

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
        <FSharpCoreImplicitPackageVersion>4.2.3</FSharpCoreImplicitPackageVersion>
    </PropertyGroup>
</Project>
```