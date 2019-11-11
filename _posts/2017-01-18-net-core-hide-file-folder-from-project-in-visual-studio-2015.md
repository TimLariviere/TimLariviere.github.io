---
layout: post
title: ".NET Core – Hide file / folder from project in Visual Studio 2015"
date: "2017-01-18"
---

A few warnings before we begin :

- This post is only about Visual Studio 2015. In Visual Studio 2017, you can find the option "Hide from Solution Explorer" in the context menu.
- Also this post is only about xproj-based projects. It applies to .NET Core libs and apps like ASP.NET Core. The next version of .NET Core (2.0) will revert back to csproj files.

Unlike csproj-based projects where files need to be referenced in the csproj, with xproj-based projects Visual Studio will list by default every files in the project folder in the Solution Explorer.

To avoid cluttering your Solution Explorer with unneeded files, you will need to edit the xproj file to add files or folders to exclude.

Here the available options :

| Tag | Description |
| --- | --- |
| DnxInvisibleContent | Hide a single file |
| DnxInvisibleFolder | Hide a single folder |

A limitation with these tags is that there no pattern available. It needs to be a specific file or folder name.

These tags needs to be written inside an _**ItemGroup**_ tag which in turn need to be put inside the _**Project**_ root of the xproj file. Like this :

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="14.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <ItemGroup>
        <DnxInvisibleContent Include="nuget.exe" />
        <DnxInvisibleFolder Include="node\_modules" />
    </ItemGroup>
</Project>
```