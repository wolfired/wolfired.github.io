开发环境：
Unity 5.3.4f1
Visual Studio 2012


新建postbuild.bat：
```
set MONO_HOME=C:\Program Files\Unity\Editor\Data\MonoBleedingEdge
set MONO_EXE="%MONO_HOME%\bin\mono.exe"
set PDB2MDB_EXE="%MONO_HOME%\lib\mono\4.5\pdb2mdb.exe"

%MONO_EXE% %PDB2MDB_EXE% %1
```
