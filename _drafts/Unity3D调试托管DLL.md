>开发环境：

>Unity 5.3.4f1

>Visual Studio 2015


>新建postbuild.bat：
```bat
set TargetDir=%1
set TargetName=%2
set SolutionDir=%3

set OUTPUT=%SolutionDir%\Plugins

if not exist %OUTPUT% (
    echo %OUTPUT% is not exist
    goto END
)

set MONO_HOME=C:\Program Files\Unity\Editor\Data\MonoBleedingEdge
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
copy %TargetDir%%TargetName%.dll %OUTPUT%\
copy %TargetDir%%TargetName%.dll.mdb %OUTPUT%\

:END
```

>VS生成事件

`$(ProjectDir)postbuild.bat $(TargetDir) $(TargetName) $(SolutionDir)Assets`

`$(ProjectDir)postbuild.bat $(TargetDir) $(TargetName) $(SolutionDir)Assets\Editor`