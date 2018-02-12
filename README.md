
# Fody Configuration for C++/CLI
**Note: Only [**PropertyChanged.Fody**](https://github.com/Fody/PropertyChanged), [**PropertyChanging.Fody**](https://github.com/Fody/PropertyChanging)**

## Creating a pure MSIL assembly from a C++/CLI project [(Reference)](https://stackoverflow.com/questions/6695727/creating-a-pure-msil-assembly-from-a-c-cli-project)
### 1. Configuration Properties
[Properties] - [General] - [Common Language Runtime Support] = **/clr:pure**  
[Properties] - [C/C++] - [Advanced] - [Omit Default Library Name] = **/Zl**  
[Properties] - [Linker] - [General] - [Incremental Linking] = **/INCREMENTAL:NO**  
[Properties] - [Linker] - [General] - [Link Library Dependencies] = **false**  
[Properties] - [Linker] - [General] - [Target Machine] = **Not Set**  
[Properties] - [Linker] - [General] - [CLR Image Type] = **/CLRIMAGETYPE:PURE**
### 2. Edit $(ProjectName).vcxproj
**Note: Add bottom of the file**
```xml
    <Target Name="AfterBuild">
        <Exec Command="corflags $(TargetPath) /32BIT-" />
    </Target>
</Project>
```
### 3. Add Code
```cpp
#pragma warning(disable:4483)
void __clrcall __identifier(".cctor")() { }
```

## NuGet installation
### 1. Install Fody
```
Install-Package Fody
```
### 2. Install PropertyChanged.Fody, PropertyChanging.Fody
**Note: PropertyChanged.Fody** and **PropertyChanging.Fody** is no package for Native, you need to manually add references and register them in packages.config.

## Creating and editing FodyWeavers.xml in your project
```xml
<?xml version="1.0" encoding="utf-8"?>
<Weavers>
  <PropertyChanging />
  <PropertyChanged />
</Weavers>
```

## Edit $(ProjectName).vcxproj
**Note: Add  under **`<Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />`**
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
## Project 속성 설정
**Required**
```
Output directory and the intermediate directory must not have the same path.
```
**Recommend**
```
  [Properties] - [General] - [Output Directory] = $(ProjectDir)bin\$(Configuration)\
  [Properties] - [General] - [Intermediate Directory] = $(ProjectDir)$(BaseIntermediateOutputPath)$(Configuration)\
```
