This repo shows who to build a netcore console app, including all files needed to deploy to a system with netcore already installed.

This sample targets one dependency (NodaTime) for illustrative purposes

## Moving pieces

### CopyLocalLockFileAssemblies

The project needs to have `CopyLocalLockFileAssemblies` set to `true`.

```xml
<CopyLocalLockFileAssemblies>true</CopyLocalLockFileAssemblies>
```

This will force all the package dependencies to be copied to the output.

### Primary build

the primary build will produce the following files

```
MyConsole.dll
MyConsole.pdb
NodaTime.dll
MyConsole.deps.json
MyConsole.runtimeconfig.dev.json
MyConsole.runtimeconfig.json
```
Note that is it missing the bootstrap exe `MyConsole.exe` and the host polciy and host resolver files.


### Secondary build

A secondary nested build is used to produce the missing bootstrap and hosting files. 

It effectively executes the following command:

```
 dotnet build MyConsole.csproj -r win-x64 /p:CopyLocalLockFileAssemblies=true;IsNestedBuild=true
```

The `IsNestedBuild` property is used to prevent infinite recursion. 

```xml
<PropertyGroup>
  <Temp>$(SolutionDir)\packaging\</Temp>
</PropertyGroup>

<ItemGroup>
  <BootStrapFiles Include="$(Temp)hostpolicy.dll;$(Temp)$(ProjectName).exe;$(Temp)hostfxr.dll;" />
</ItemGroup>

<Target Name="GenerateNetcoreExe" AfterTargets="Build" Condition="'$(IsNestedBuild)' != 'true'">
  <RemoveDir Directories="$(Temp)" />
  <Exec ConsoleToMSBuild="true" Command="dotnet build $(ProjectPath) -r win-x64 /p:CopyLocalLockFileAssemblies=false;IsNestedBuild=true --output $(Temp)">
    <Output TaskParameter="ConsoleOutput" PropertyName="OutputOfExec" />
  </Exec>
  <Copy SourceFiles="@(BootStrapFiles)" DestinationFolder="$(OutputPath)" />

</Target>
```

## Final output

The full resultant file list

```
hostfxr.dll
hostpolicy.dll
MyConsole.deps.json
MyConsole.dll
MyConsole.exe
MyConsole.pdb
MyConsole.runtimeconfig.dev.json
MyConsole.runtimeconfig.json
NodaTime.dll
```