I'm experiecing a weird behaviour with `dotnet build`. I have two projects, one main (`MyProject`) project and another one (`MyProject.Tests`) that references the main project as a project reference.

The `project.json` for `MyProject` looks like this

```
{
    "frameworks": {
        "net452": {
            "dependencies": {
                "Microsoft.CodeDom.Providers.DotNetCompilerPlatform": "1.0.1",
                "Microsoft.Net.Compilers": {
                    "version": "1.1.1",
                    "type": "build"
                }
            }
        }
    }
}
```

and for `MyProject.Tests` the `project.json` looks like

```
{
    "dependencies": {
        "MyProject": { "target": "project" }
    },

    "frameworks": {
        "net452": {

        }
    }
}
```

If I have a clean project (delete the `bin` folder`) and try to compile it, I end up with the following result

```
dotnet build
Project MyProject (.NETFramework,Version=v4.5.2) will be compiled because inputs
 were modified
Compiling MyProject for .NETFramework,Version=v4.5.2

Compilation succeeded.
    0 Warning(s)
    0 Error(s)

Time elapsed 00:00:01.4711453
Project MyProject.Tests (.NETFramework,Version=v4.5.2) will be compiled because
dependencies changed
Compiling MyProject.Tests for .NETFramework,Version=v4.5.2
c:\projects\buildbug\test\MyProject.Tests\project.json(3,23): error NU1001: The
dependency Microsoft.Net.Compilers >= 1.0.0 could not be resolved.

Compilation failed.
    0 Warning(s)
    1 Error(s)

Time elapsed 00:00:01.3664722
```
 
`MyProject.Tests\project.json(3,23)` is the line that references `MyProject` with `"MyProject": { "target": "project" }`. If I go into `MyProject\project.json` and remove _either_ of the two
dependencies, delete my bin folders, run `dotnet restore` again followed by `dotnet build` then it **WORKS**

So I need both those dependencies for it to fail. 

Also, if I run `dotnet build` again, after it failed but without deleting my `bin` folders then it _"compiles"_

```
c:\projects\buildbug\test\MyProject.Tests>dotnet build
Project MyProject (.NETFramework,Version=v4.5.2) was previously compiled. Skippi
ng compilation.
Project MyProject.Tests (.NETFramework,Version=v4.5.2) was previously compiled.
Skipping compilation.
```

It's almost like there are two bugs here

1. Why does it fail compiling when those dependencies are in the references project?
2. Why does it think that it does not need to compile it the second time?

I've tried on two CLI versions, each were built with a couple of days apart. This is the one I am currently on

```
.NET Command Line Tools (1.0.0-rc2-002498)

Product Information:
 Version:     1.0.0-rc2-002498
 Commit Sha:  55e561bfb3

Runtime Environment:
 OS Name:     Windows
 OS Version:  6.3.9600
 OS Platform: Windows
 RID:         win81-x64
```

