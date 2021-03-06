# Generating AssemblyInfo files

**Note:  This documentation is for FAKE.exe after version 5. The documentation for previous version (<=4) can be found [here](legacy-assemblyinfo.html)! **

In this article the AssemblyInfo task is used in order to set specific version information to .NET assemblies.

If you succeeded with the [Getting Started tutorial](gettingstarted.html), then you just have to modify your *BuildApp* target to the following:

```fsharp
#r "paket:
nuget Fake.DotNet.AssemblyInfo
nuget Fake.DotNet.MSBuild
nuget Fake.Core.Target //"
open Fake.DotNet

Target.Create "BuildApp" (fun _ ->
    AssemblyInfoFile.CreateCSharp "./src/app/Calculator/Properties/AssemblyInfo.cs"
        [AssemblyInfo.Title "Calculator Command line tool"
            AssemblyInfo.Description "Sample project for FAKE - F# MAKE"
            AssemblyInfo.Guid "A539B42C-CB9F-4a23-8E57-AF4E7CEE5BAA"
            AssemblyInfo.Product "Calculator"
            AssemblyInfo.Version version
            AssemblyInfo.FileVersion version]

    AssemblyInfoFile.CreateFSharp "./src/app/CalculatorLib/Properties/AssemblyInfo.fs"
        [AssemblyInfo.Title "Calculator library"
            AssemblyInfo.Description "Sample project for FAKE - F# MAKE"
            AssemblyInfo.Guid "EE5621DB-B86B-44eb-987F-9C94BCC98441"
            AssemblyInfo.Product "Calculator"
            AssemblyInfo.Version version
            AssemblyInfo.FileVersion version]

    MSBuild.RunRelease buildDir "Build" appReferences
        |> Log "AppBuild-Output: "
)
```

As you can see generating an AssemblyInfo.cs file is pretty easy with FAKE. You can read more about the C# and F# AssemblyInfo tasks in the [API docs](apidocs/fake-assemblyinfofile.html).

## Setting the version no.

The version parameter can be declared as a property or fetched from a build server like TeamCity:

```fsharp
let version =
  match buildServer with
  | TeamCity -> buildVersion
  | _ -> "0.2"
```

![alt text](pics/assemblyinfo/result.png "The file version is set by FAKE")

## Storing the githash in the AssemblyInfo

Storing the githash with the assembly can make it easier to identify exactly what code is running. There isn't an attribute that
directly fits with doing this, but one way is by storing it as Metadata (warning: this attribute is only available in .NET 4.5 and above)

If your solution is inside a git repository you can get the git hash like this (remember to `open Fake.Git`):

```fsharp
let commitHash = Information.getCurrentHash()
```

And set like this:

```fsharp
AssemblyInfo.Metadata("githash", commitHash)
```

One of the easiest ways to retrieve this hash is to load use a reflector program, like [ILSpy](https://github.com/icsharpcode/ILSpy):

![alt text](pics/assemblyinfo/assemblymetadata.png "Checking the git hash of an assembly")

## Using the SolutionInfo approach

Some companies split their AssemblyInfo into a SolutionInfo.cs which is shared by all projects and a specific AssemblyInfo per project which contains the product data.
All versioning data is generated by the AssemblyInfo task into the SolutionInfo.cs and the AssemblyInfo files are edited manually. This could look like this:

![alt text](pics/assemblyinfo/solutioninfo.png "SolutionInfo.cs is shared between projects")

The generated SolutionInfo.cs looks like this:

![alt text](pics/assemblyinfo/generated.png "Generated SolutionInfo.cs")
