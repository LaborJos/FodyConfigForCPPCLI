# Fody Configuration for C ++ / CLI
**Note: Only [**PropertyChanged.Fody**](https://github.com/Fody/PropertyChanged), [**PropertyChanging.Fody**](https://github.com/Fody/PropertyChanging)**

## Creating a pure MSIL assembly from a C++/CLI project
[**Reference**](https://stackoverflow.com/questions/6695727/creating-a-pure-msil-assembly-from-a-c-cli-project)
### Configuration Properties
[Properties] - [General] - [Common Language Runtime Support] = **/clr:pure**  
[Properties] - [C/C++] - [Advanced] - [Omit Default Library Name] = **/Zl**  
[Properties] - [Linker] - [General] - [Incremental Linking] = **/INCREMENTAL:NO**  
[Properties] - [Linker] - [General] - [Link Library Dependencies] = **false**  
[Properties] - [Linker] - [General] - [Target Machine] = **Not Set**  
[Properties] - [Linker] - [General] - [CLR Image Type] = **/CLRIMAGETYPE:PURE**
### Edit $(ProjectName).vcxproj
**Note: Add bottom of the file**
```xml
    <Target Name="AfterBuild">
        <Exec Command="corflags $(TargetPath) /32BIT-" />
    </Target>
</Project>
```
### Add Code
```cpp
#pragma warning(disable:4483)
void __clrcall __identifier(".cctor")() { }
```

## NuGet installation
### Install Fody
```
Install-Package Fody
```
### Install PropertyChanged.Fody, PropertyChanging.Fody
```
Install-Package PropertyChanged.Fody
Install-Package PropertyChanging.Fody
```
### Notes
* **PropertyChanged.Fody** and **PropertyChanging.Fody** is no package for Native, you need to manually add references and register them in packages.config.

## Creating and editing FodyWeavers.xml in your project
```xml
<?xml version="1.0" encoding="utf-8"?>
<Weavers>
  <PropertyChanging />
  <PropertyChanged />
</Weavers>
```

## Edit $(ProjectName).vcxproj
> Add  under **`<Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />`**
```xml
<Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />
...
<Target Name="AfterCompile" AfterTargets="BuildLink">
  <ItemGroup>
    <SourceFile Include="$(OutDir)$(ProjectName)$(TargetExt)" />
  </ItemGroup>
  <Move SourceFiles="@(SourceFile)" DestinationFolder="$(IntDir)" />
</Target>
<Target Name="MoveTarget" AfterTargets="FodyTarget">
  <ItemGroup>
    <TargetFile Include="$(IntDir)$(ProjectName)$(TargetExt)" />
  </ItemGroup>
  <Move SourceFiles="@(TargetFile)" DestinationFolder="$(OutDir)" />
</Target>
...
```
