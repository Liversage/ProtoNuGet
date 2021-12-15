# Introduction

This project demonstrates how it's possible to share `.proto` files for a gRPC API between two .NET projects using a NuGet package. This was inspired by a comment on GitHub issue [Enable referencing proto files in NuGet packages](https://github.com/grpc/grpc-dotnet/issues/183#issuecomment-513055273).

This version fixes the problem where several source files would appear as if they had syntax errors when viewed in both Visual Studio 2022 and Visual Studio Code (using OmniSharp). As the code compiled this must be a bug in the tooling but [the feedback](https://developercommunity.visualstudio.com/t/IntelliSense-reports-false-compile-error/1468739) I have provided haven't gotten any attention.

I based my project of the [Tutorial: Create a gRPC client and server in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/grpc/grpc-start). 
A local `NuGet.Config` file is used to be able to test the NuGet package.

# Build and run

First step is to build the NuGet package:

```
dotnet pack Proto
```

Then run the server:

```
dotnet run --project Server
```

In another console run the client:

```
dotnet run --project Client
```

All projects will compile and run and the gRPC API will be used in the test.

# Fixing IntelliSense problems

The original project used a `.targets` file to ensure that code generation ran for the `.proto` files in the NuGet package. Unfortunately, that broke IntelliSense and to work around this problem the `.targets` file has been removed and both client and server `.csproj` files now have to be edited to include the `.proto` files in the referenced NuGet package.

The NuGet package reference uses the `GeneratePathProperty="true"` attribute to create the variable `PkgProto` used later:

```xml
<PackageReference Include="Proto" Version="1.0.0" GeneratePathProperty="true" />
```

The following should be added to the `.csproj` to generate code for the `.proto` files:

```xml
<ItemGroup>
    <Protobuf Include="$(PkgProto)\content\**\*.proto" ProtoRoot="$(PkgProto)\content" />
</ItemGroup>
```

The `GrpcServices` and `Access` attributes can be used to modify how gRPC code generation is performed.

This approach is more complicated compared to the original solution where nothing more than a standard NuGet reference was required to use the shared `.proto` files. However, the IntelliSense problems that came along with that solution were unbearable so until they have been fixed (if ever?) this solution is fairly easy to use.