# C++/CLI을 위한 Fody 설정
**Note: [**PropertyChanged.Fody**](https://github.com/Fody/PropertyChanged), [**PropertyChanging.Fody**](https://github.com/Fody/PropertyChanging) 전용**

## C++/CLI 프로젝트에서 순수 MSIL Assembly 만들기 [(참고)](https://stackoverflow.com/questions/6695727/creating-a-pure-msil-assembly-from-a-c-cli-project)
### 1. Project 속성 설정
```
  [속성] - [일반] - [공용 언어 런타임 지원] = **/clr:pure**  
  [속성] - [C/C++] - [고급] - [기본 라이브러리 이름 생략] = **/Zl**  
  [속성] - [링커] - [일반] - [증분 링크 사용] = **/INCREMENTAL:NO**  
  [속성] - [링커] - [일반] - [라이브러리 종속성 링크] = **false**  
  [속성] - [링커] - [일반] - [대상 컴퓨터] = **Not Set**  
  [속성] - [링커] - [일반] - [CLR 이미지 형식] = **/CLRIMAGETYPE:PURE**
```
### 2. $(ProjectName).vcxproj 편집
  **Note: 파일 마지막 라인에 추가**
  ```xml
    <Target Name="AfterBuild">
        <Exec Command="corflags $(TargetPath) /32BIT-" />
    </Target>
  </Project>
  ```
### 3. 코드 추가
  ```cpp
  #pragma warning(disable:4483)
  void __clrcall __identifier(".cctor")() { }
  ```

## NuGet 설치
### 1. Fody 설치
```
Install-Package Fody
```
### 2. PropertyChanged.Fody, PropertyChanging.Fody 설치
**Note: PropertyChanged.Fody**와 **PropertyChanging.Fody**는 Native용 패키지가 없으므로, 참조 추가와 packages.config에 등록해주는 과정이 필요함.

## 프로젝트에 FodyWeavers.xml 생성 및 편집
```xml
<?xml version="1.0" encoding="utf-8"?>
<Weavers>
  <PropertyChanging />
  <PropertyChanged />
</Weavers>
```

## $(ProjectName).vcxproj 편집
**Note: `<Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />`** 아래에 추가
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
**필수**
```
출력 디렉터리와 중간 디렉터리의 경로가 같으면 안됩니다.
```
**권장**
```
  [속성] - [일반] - [출력 디렉터리] = $(ProjectDir)bin\$(Configuration)\
  [속성] - [일반] - [중간 디렉터리] = $(ProjectDir)$(BaseIntermediateOutputPath)$(Configuration)\
```
