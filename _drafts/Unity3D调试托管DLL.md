---
layout: post
---

>开发环境：

>Unity 5.3.4f1

>Visual Studio 2015


>新建postbuild.bat：

```shell
set SolutionDir=%1
set ProjectDir=%2
set TargetDir=%3
set TargetName=%4
set OutputDir=%5

set OUTPUT_DLL=%SolutionDir%%OutputDir%

if not exist %OUTPUT_DLL% (
    echo %OUTPUT_DLL% is not exist
    goto END
)

set MONO_HOME=D:\Unity\Editor\Data\MonoBleedingEdge
set MONO_EXE=%MONO_HOME%\bin\mono.exe
set PDB2MDB_EXE=%MONO_HOME%\lib\mono\4.5\pdb2mdb.exe

if not exist "%MONO_EXE%" (
    echo %MONO_EXE% is not exist
    goto END
)

if not exist "%PDB2MDB_EXE%" (
    echo %PDB2MDB_EXE% is not exist
    goto END
)

"%MONO_EXE%" "%PDB2MDB_EXE%" %TargetDir%%TargetName%.dll
copy %TargetDir%%TargetName%.dll %OUTPUT_DLL%
copy %TargetDir%%TargetName%.dll.mdb %OUTPUT_DLL%

:END
```

>VS生成事件

`$(ProjectDir)postbuild.bat $(SolutionDir) $(ProjectDir) $(TargetDir) $(TargetName) Assets\Plugins\`

`$(ProjectDir)postbuild.bat $(SolutionDir) $(ProjectDir) $(TargetDir) $(TargetName) Assets\Editor\Plugins\`

