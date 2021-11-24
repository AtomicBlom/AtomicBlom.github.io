## Title: Porting Drawboard Projects to Windows App SDK 1.0 - Migrating the Projects Lead: Starting the migration Tags: [ WindowsAppSDK, WinUI, DrawboardProjects ] Published: 2021-11-22

Porting Drawboard Projects to Windows App SDK 1.0

Part 2 – Migrating the Project files

In case you haven’t been following the buzz, there is a massive difference
between UWP and .net 6 project files. They’ve been dramatically simplified, and
you shouldn’t feel the pain of horrible merge conflicts just because two people
added new files to the project.

They’re generally so different that it’s not worth keeping the old one and
recreating them from scratch.

# Before we begin

Make sure you’re using a supported version of Visual Studio. At least 2019, but
preferably 2022

[Install the tools for developing apps for Windows 10 and Windows
11](https://docs.microsoft.com/en-us/windows/apps/windows-app-sdk/set-up-your-development-environment)
with the Windows App SDK. I recommend having both the SDK and the VSIX
installed.

Make sure that you’ve created a new branch for the port in your source control
of choice.

If you’re still using the old packages.config method of managing NuGet packages,
I’d highly recommend that you [Migrate from packages.config to
PackageReferences](https://docs.microsoft.com/en-us/nuget/consume-packages/migrate-packages-config-to-package-reference)
before you start, we won’t be covering it here.

Look through all your project files and keep an eye out for any specific
customizations and hacks that you or your team may have added to them. You may
or may not need to add them again later.

# Replacing the project file for your main application

Start by copying any NuGet \<PackageReference\> to a new file, we’ll be
referring to them later.

Next, copy any \<ProjectReference\> elements to a new file, we’ll be adding them
back in exactly as they are.

Now, we can just replace the whole .csproj with a trimmed down one. I generated
our new .csproj by creating a “Blank App, Packaged (WinUI 3 in Desktop)”
project, provided by the VSIX.

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <!-- Modify the windows version to set your target version of windows -->
        <TargetFramework>net6.0-windows10.0.19041.0</TargetFramework>
        <!-- Modify this to the minimum supported version of windows -->
        <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
        <!-- Set the root namespace if you need to -->
        <!--RootNamespace>DrawboardBullclip</RootNamespace-->        
        <ApplicationManifest>app.manifest</ApplicationManifest>
        <!-- Modify these to reflect the platforms you wish to support -->
        <Platforms>x86;x64;arm64</Platforms>
        <RuntimeIdentifiers>win10-x86;win10-x64;win10-arm64</RuntimeIdentifiers>
        <!-- This enables support for WinUI, you probably won't need it for utility libraries-->
        <UseWinUI>true</UseWinUI>
        <!-- Unless you've changed your C# version manually, you probably aren't using Nullable Reference Types -->
        <Nullable>disable</Nullable>
    </PropertyGroup>
    <!-- 
        Include this property group if you DO NOT already have a packaging project.
        Feel free to merge it into the property group above    
    -->
    <PropertyGroup>        
        <EnablePreviewMsixTooling>true</EnablePreviewMsixTooling>
        <PublishProfile>win10-$(Platform).pubxml</PublishProfile>
    </PropertyGroup>

    <!-- Nuget References will be added here -->
    <ItemGroup>
        <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.0.0" />
        <PackageReference Include="Microsoft.Windows.SDK.BuildTools" Version="10.0.22000.196" />
    </ItemGroup>
    
    <!-- Additional Project References -->
    <ItemGroup>
        <ProjectReference Include="..\Framework\Drawboard.Framework.Win10\Drawboard.Framework.Win10.csproj">
            <Project>{0abd599e-a3b2-41e5-8630-ab2466087570}</Project>
            <Name>Drawboard.Framework.Win10</Name>
        </ProjectReference>
        <ProjectReference Include="..\Framework\Drawboard.Framework\Drawboard.Framework.csproj">
            <Project>{646D701A-F345-4227-850E-7D6917BF8945}</Project>
            <Name>Drawboard.Framework</Name>
        </ProjectReference>
        <ProjectReference Include="..\Bullclip.DataStorage\Bullclip.DataStorage.csproj">
            <Project>{B05E4BA9-0674-4BB1-B5FA-8FF71B6B15C3}</Project>
            <Name>Bullclip.DataStorage</Name>
        </ProjectReference>
        <ProjectReference Include="..\Model\Bullclip.Shared.csproj">
            <Project>{89EDE92F-ABF3-4A69-ACF3-C2E98953C5F0}</Project>
            <Name>Bullclip.Shared</Name>
        </ProjectReference>
    </ItemGroup>

    <ItemGroup>
        <Manifest Include="$(ApplicationManifest)" />
    </ItemGroup>

    <!-- Do not include this itemgroup if you are using a dedicated packaging project -->
    <!-- Defining the "Msix" ProjectCapability here allows the Single-project MSIX Packaging
        Tools extension to be activated for this project even if the Windows App SDK Nuget
        package has not yet been restored -->
    <ItemGroup Condition="'$(DisableMsixProjectCapabilityAddedByProject)'!='true' and '$(EnablePreviewMsixTooling)'=='true'">
        <ProjectCapability Include="Msix" />
    </ItemGroup>
</Project>
```

Feel free to trim out any comments you don’t need, and to merge together any
\<PropertyGroup\> elements you think necessary, cleaning it up as you see fit.

Once cleaned up, give or take NuGet packages and some minor other tweaks, the
main UWP project file for Drawboard Projects has been reduced from 1,850 lines
of XML down to only 49.

Looking back at our original project files, we also had

|  \<PropertyGroup Condition="'\$(Configuration)\|\$(Platform)' == 'Debug\|ARM'"\>  \<DebugSymbols\>true\</DebugSymbols\>  \<OutputPath\>bin\\ARM\\Debug\\\</OutputPath\>  \<DefineConstants\>DEBUG;TRACE;NETFX_CORE;WINDOWS_UWP;CODE_ANALYSIS;DISABLE_ANALYTICS\</DefineConstants\>  \<NoWarn\>  \</NoWarn\>  \<DebugType\>full\</DebugType\>  \<PlatformTarget\>ARM\</PlatformTarget\>  \<UseVSHostingProcess\>false\</UseVSHostingProcess\>  \<ErrorReport\>prompt\</ErrorReport\>  \<Prefer32Bit\>true\</Prefer32Bit\>  \</PropertyGroup\>  |  \<PropertyGroup Condition="'\$(Configuration)' != 'Release'"\>  \<DefineConstants\>\$(DefineConstants);DISABLE_ANALYTICS\</DefineConstants\>  \</PropertyGroup\> |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
