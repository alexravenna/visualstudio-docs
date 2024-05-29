---
title: Customize Docker containers in Visual Studio
author: ghogen
description: Explore Visual Studio fast mode, and modify the Dockerfile to customize your container images for both debug and production builds.
ms.author: ghogen
ms.date: 09/8/2023
ms.subservice: container-tools
ms.topic: how-to
---

# Customize Docker containers in Visual Studio

You can customize your container images by editing the Dockerfile that Visual Studio generates when you add Docker support to your project. Whether you're building a customized container from the Visual Studio IDE, or setting up a command-line build, you need to know how Visual Studio uses the Dockerfile to build your projects. You need to know such details because, for performance reasons, Visual Studio follows a special process for building and running containerized apps that isn't obvious from the Dockerfile.

Suppose you want to make a change in the Dockerfile and see the results in both debugging and in production containers. In that case, you can add commands in the Dockerfile to modify the first stage (usually `base`). See [Modify the container image for debugging and production](#modify-container-image-for-debugging-and-production). But, if you want to make a change only when debugging, but not production, then you should create another stage, and use the `DockerfileFastModeStage` build setting to tell Visual Studio to use that stage for debug builds. See [Modify the container image only for debugging](#modify-container-image-only-for-debugging).

This article explains the Visual Studio build process for containerized apps in some detail, then it contains information on how to modify the Dockerfile to affect both debugging and production builds, or just for debugging.

## Dockerfile builds in Visual Studio

::: moniker range=">=vs-2022"
> [!NOTE]
> This section describes the container build process that Visual Studio uses when you choose the Dockerfile container build type. If you are using the .NET SDK build type, the customization options are different, and the information in this section isn't applicable. Instead, see [Containerize a .NET app with dotnet publish](/dotnet/core/docker/publish-as-container?pivots=dotnet-8-0) and use the properties described at [Customize your container](https://github.com/dotnet/sdk-container-builds/blob/main/docs/ContainerCustomization.md) to configure the container build process.
::: moniker-end

### Multistage build

When Visual Studio builds a project that doesn't use Docker containers, it invokes MSBuild on the local machine and generates the output files in a folder (typically `bin`) under your local solution folder. For a containerized project, however, the build process takes account of the Dockerfile's instructions for building the containerized app. The Dockerfile that Visual Studio uses is divided into multiple stages. This process relies on Docker's *multistage build* feature.

The multistage build feature helps make the process of building containers more efficient, and makes containers smaller by allowing them to contain only the bits that your app needs at run time. Multistage build is used for .NET Core projects, not .NET Framework projects.

The multistage build allows container images to be created in stages that produce intermediate images. As an example, consider a typical Dockerfile. The first stage is called `base` in the Dockerfile that Visual Studio generates, although the tools don't require that name.

```
FROM mcr.microsoft.com/dotnet/aspnet:3.1-buster-slim AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

The lines in the Dockerfile begin with the ASP.NET image from Microsoft Container Registry (mcr.microsoft.com) and create an intermediate image `base` that exposes ports 80 and 443, and sets the working directory to `/app`.

The next stage is `build`, which appears as follows:

```
FROM mcr.microsoft.com/dotnet/sdk:3.1-buster-slim AS build
WORKDIR /src
COPY ["WebApplication43/WebApplication43.csproj", "WebApplication43/"]
RUN dotnet restore "WebApplication43/WebApplication43.csproj"
COPY . .
WORKDIR "/src/WebApplication43"
RUN dotnet build "WebApplication43.csproj" -c Release -o /app/build
```

You can see that the `build` stage starts from a different original image from the registry (`sdk` rather than `aspnet`), rather than continuing from base. The `sdk` image has all the build tools, and for that reason it's a lot bigger than the aspnet image, which only contains runtime components. The reason for using a separate image becomes clear when you look at the rest of the Dockerfile:

```
FROM build AS publish
RUN dotnet publish "WebApplication43.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication43.dll"]
```

The final stage starts again from `base`, and includes the `COPY --from=publish` to copy the published output to the final image. This process makes it possible for the final image to be a lot smaller, since it doesn't need to include all of the build tools that were in the `sdk` image.

### MSBuild

::: moniker range=">=vs-2022"
> [!NOTE]
> This section describes how you can customize your Docker containers when you choose the Dockerfile container build type. If you are using the .NET SDK build type, the customization options are different, and the information in this article isn't applicable. Instead, see [Containerize a .NET app with dotnet publish](/dotnet/core/docker/publish-as-container?pivots=dotnet-8-0).
::: moniker-end

Dockerfiles created by Visual Studio for .NET Framework projects (and for .NET Core projects created with versions of Visual Studio prior to Visual Studio 2017 Update 4) aren't multistage Dockerfiles. The steps in these Dockerfiles don't compile your code. Instead, when Visual Studio builds a .NET Framework Dockerfile, it first compiles your project using MSBuild. When that succeeds, Visual Studio then builds the Dockerfile, which simply copies the build output from MSBuild into the resulting Docker image. Because the steps to compile your code aren't included in the Dockerfile, you can't build .NET Framework Dockerfiles using `docker build` from the command line. You should use MSBuild to build these projects.

To build an image for a single Docker container project, you can use MSBuild with the `/t:ContainerBuild` command option. This option tells MSBuild to build the target `ContainerBuild` rather than the default target `Build`. For example:

```cmd
MSBuild MyProject.csproj /t:ContainerBuild /p:Configuration=Release
```

You see output similar to what you see in the **Output** window when you build your solution from the Visual Studio IDE. Always use `/p:Configuration=Release`, since in cases where Visual Studio uses the multistage build optimization, results when building the **Debug** configuration might not be as expected. See [Debugging](#debugging).

If you're using a Docker Compose project, use this command to build images:

```cmd
msbuild /p:SolutionPath=<solution-name>.sln /p:Configuration=Release docker-compose.dcproj
```

## Debugging

::: moniker range=">=vs-2022"
> [!NOTE]
> This section describes how you can customize your Docker containers when you choose the Dockerfile container build type. If you are using the .NET SDK build type, the customization options are different, and most of the information in this section isn't applicable. Instead, see [Containerize a .NET app with dotnet publish](/dotnet/core/docker/publish-as-container?pivots=dotnet-8-0).
::: moniker-end

When you build in the **Debug** configuration, there are several optimizations that Visual Studio does that help with the performance of the build process for containerized projects. The build process for containerized apps isn't as straightforward as simply following the steps outlined in the Dockerfile. Building in a container is slower than building on the local machine. So, when you build in the **Debug** configuration, Visual Studio actually builds your projects on the local machine, and then shares the output folder to the container using volume mounting. A build with this optimization enabled is called a *Fast* mode build.

In **Fast** mode, Visual Studio calls `docker build` with an argument that tells Docker to build only the first stage in the Dockerfile (normally the `base` stage). You can change that by setting the MSBuild property, `DockerfileFastModeStage`, described at [Container Tools MSBuild properties](container-msbuild-properties.md). Visual Studio handles the rest of the process without regard to the contents of the Dockerfile. So, when you modify your Dockerfile, such as to customize the container environment or install additional dependencies, you should put your modifications in the first stage. Any custom steps placed in the Dockerfile's `build`, `publish`, or `final` stages aren't executed.

This performance optimization only occurs when you build in the **Debug** configuration. In the **Release** configuration, the build occurs in the container as specified in the Dockerfile.

If you want to disable the performance optimization and build as the Dockerfile specifies, then set the **ContainerDevelopmentMode** property to **Regular** in the project file as follows:

```xml
<PropertyGroup>
   <ContainerDevelopmentMode>Regular</ContainerDevelopmentMode>
</PropertyGroup>
```

To restore the performance optimization, remove the property from the project file.

When you start debugging (**F5**), a previously started container is reused, if possible. If you don't want to reuse the previous container, you can use **Rebuild** or **Clean** commands in Visual Studio to force Visual Studio to use a fresh container.

The process of running the debugger depends on the type of project and container operating system:

|Scenario|Debugger process|
|-|-|
| **.NET Core apps (Linux containers)**| Visual Studio downloads `vsdbg` and maps it to the container, then it gets called with your program and arguments (that is, `dotnet webapp.dll`), and then debugger attaches to the process. |
| **.NET Core apps (Windows containers)**| Visual Studio uses `onecoremsvsmon` and maps it to the container, runs it as the entry point and then Visual Studio connects to it and attaches to the program. This is similar to how you would normally set up remote debugging on another computer or virtual machine.|
| **.NET Framework apps** | Visual Studio uses `msvsmon` and maps it to the container, runs it as part of the entry point where Visual Studio can connect to it, and attaches to the program.|

For information on `vsdbg.exe`, see [Offroad debugging of .NET Core on Linux and OS X from Visual Studio](https://github.com/Microsoft/MIEngine/wiki/Offroad-Debugging-of-.NET-Core-on-Linux---OSX-from-Visual-Studio).

### Modify container image for debugging and production

To modify the container image for both debugging and production, modify the `base` stage. Add your customizations to the Dockerfile in the base stage section, usually the first section in the Dockerfile. Refer to the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) in the Docker documentation for information about Dockerfile commands.

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
# <add your commands here>

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["WebApplication3/WebApplication3.csproj", "WebApplication3/"]
RUN dotnet restore "WebApplication3/WebApplication3.csproj"
COPY . .
WORKDIR "/src/WebApplication3"
RUN dotnet build "WebApplication3.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "WebApplication3.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication3.dll"]
```

### Modify container image only for debugging

This scenario applies when you want to do something with your containers to help you in the debugging process, such as installing something for diagnostic purposes, but you don't want that installed in production builds.

To modify the container only for debugging, create a stage and then use the MSBuild property `DockerfileFastModeStage` to tell Visual Studio to use your customized stage when debugging. Refer to the [Dockerfile reference](https://docs.docker.com/engine/reference/builder/) in the Docker documentation for information about Dockerfile commands.

In the following example, we install the package `procps-ng`, but only in debug mode. This package supplies the command `pidof`, which Visual Studio requires but isn't in the Mariner image used here. The stage we use for fast mode debugging is `debug`, a custom stage defined here. The fast mode stage doesn't need to inherit from the `build` or `publish` stage, it can inherit directly from the `base` stage, because Visual Studio mounts a volume that contains everything needed to run the app, as described earlier in this article.

```dockerfile
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:6.0-cbl-mariner2.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM base AS debug
RUN tdnf install procps-ng -y

FROM mcr.microsoft.com/dotnet/sdk:6.0-cbl-mariner2.0 AS build
WORKDIR /src
COPY ["WebApplication1/WebApplication1.csproj", "WebApplication1/"]
RUN dotnet restore "WebApplication1/WebApplication1.csproj"
COPY . .
WORKDIR "/src/WebApplication1"
RUN dotnet build "WebApplication1.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "WebApplication1.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebApplication1.dll"]
```

In the project file, add this setting to tell Visual Studio to use your custom stage `debug` when debugging.

```xml
  <PropertyGroup>
     <!-- other property settings -->
     <DockerfileFastModeStage>debug</DockerfileFastModeStage>
  </PropertyGroup>
```

The next sections contain information that might be useful in certain cases, such as when you want to specify a different entry point, or if your app is SSL-enabled and you're changing something that might affect how the SSL certificates are handled.

## Building from the command line

If you want to build a container project with a Dockerfile outside of Visual Studio, you can use `docker build`, `MSBuild`, `dotnet build`, or `dotnet publish` to build from the command line.

:::moniker range=">=vs-2022"
If you're using the .NET SDK build type, you don't have a Dockerfile, so you can't use `docker build`; instead, use `MSBuild`, `dotnet build` or `dotnet publish` to build on the command line.
:::moniker-end

### Use Docker build

To build a containerized solution from the command line, you can usually use the command `docker build <context>` for each project in the solution. You provide the *build context* argument. The *build context* for a Dockerfile is the folder on the local machine that's used as the working folder to generate the image. For example, it's the folder that you copy files from when you copy to the container. In .NET Core projects, use the folder that contains the solution file (.sln). Expressed as a relative path, this argument is typically ".." for a Dockerfile in a project folder, and the solution file in its parent folder. For .NET Framework projects, the build context is the project folder, not the solution folder.

```cmd
docker build -f Dockerfile ..
```

## Project warmup

*Project warmup* refers to a series of steps that happen when the Docker profile is selected for a project (that is, when a project is loaded or Docker support is added) in order to improve the performance of subsequent runs (**F5** or **Ctrl**+**F5**). This behavior is configurable under **Tools** > **Options** > **Container Tools**. Here are the tasks that run in the background:

- Check that Docker Desktop is installed and running.
- Ensure that Docker Desktop is set to the same operating system as the project.
- Pull the images in the first stage of the Dockerfile (the `base` stage in most Dockerfiles).
- Build the Dockerfile and start the container.

Warmup only happens in **Fast** mode, so the running container has the *app* folder volume-mounted. That means that any changes to the app don't invalidate the container. This behavior improves the debugging performance significantly and decreases the wait time for long running tasks such as pulling large images.

## Volume mapping

For debugging to work in containers, Visual Studio uses volume mapping to map the debugger and NuGet folders from the host machine. Volume mapping is described in the Docker documentation [here](https://docs.docker.com/storage/volumes/). You can view the volume mappings for a container by using the [Containers window in Visual Studio](view-and-diagnose-containers.md).

:::moniker range="<=vs-2019"
Here are the volumes that are mounted in your container:

|Volume|Description|
|-|-|
| **App folder** | Contains the project folder where the Dockerfile is located.|
| **NuGet packages folders** | Contains the NuGet packages and fallback folders that are read from the *obj\{project}.csproj.nuget.g.props* file in the project. |
| **Remote debugger** | Contains the bits required to run the debugger in the container depending on the project type. See the [Debugging](#debugging) section.|
| **Source folder** | Contains the build context that is passed to Docker commands.|

:::moniker-end

::: moniker range=">=vs-2022"
Here are the volumes that are mounted in your container. What you see in your containers might differ depending on the minor version of Visual Studio 2022 you are using.

|Volume|Description|
|-|-|
| **App folder** | Contains the project folder where the Dockerfile is located.|
| **HotReloadAgent** | Contains the files for the hot reload agent. |
| **HotReloadProxy** | Contains the files required to run a service that enables the host reload agent to communicate with Visual Studio on the host. |
| **NuGet packages folders** | Contains the NuGet packages and fallback folders that are read from the *obj\{project}.csproj.nuget.g.props* file in the project. |
| **Remote debugger** | Contains the bits required to run the debugger in the container depending on the project type. This is explained in more detail in the [Debugging](#debugging) section.|
| **Source folder** | Contains the build context that is passed to Docker commands.|
| **TokenService.Proxy** | Contains the files required to run a service the enables VisualStudioCredential to communicate with Visual Studio on the host. |

For .NET 8, additional mount points at root and for the app user that contain user secrets and the HTTPS certificate might also be present. And in Visual Studio 17.10 preview, the Hot Reload and Token Service volume, along with another component, the Distroless Helper, are combined under a single mount point, `/VSTools`.

> [!NOTE]
> **Visual Studio 17.10 preview** If you're using Docker Engine in Windows Subsystem for Linux (WSL) without Docker Desktop, set the environment variable `VSCT_WslDaemon=1` to have Visual Studio use WSL paths when creating volume mounts. The NuGet package [Microsoft.VisualStudio.Azure.Containers.Tools.Targets 1.20.0-Preview 1](https://www.nuget.org/packages/Microsoft.VisualStudio.Azure.Containers.Tools.Targets/1.20.0-Preview.1) is also required.

:::moniker-end

For ASP.NET core web apps, there might be two additional folders for the SSL certificate and the user secrets, which is explained in more detail in the next section.

## Enable detailed container tools logs

For diagnostic purposes, you can enable certain Container Tools logs. You can enable these logs by setting certain environment variables. For single container projects, the environment variable is `MS_VS_CONTAINERS_TOOLS_LOGGING_ENABLED`, which then logs in `%tmp%\Microsoft.VisualStudio.Containers.Tools`. For Docker Compose projects, it's `MS_VS_DOCKER_TOOLS_LOGGING_ENABLED`, which then logs in `%tmp%\Microsoft.VisualStudio.DockerCompose.Tools`.

:::moniker range=">=vs-2022"
> [!CAUTION]
> When logging is enabled and you're using a token proxy for Azure authentication, authentication credentials could be logged as plain text. See [Configure Azure authentication](container-tools-configure.md#configure-azure-authentication).
:::moniker-end

## Container entry point

Visual Studio uses a custom container entry point depending on the project type and the container operating system, here are the different combinations:

|Container type|Entry point|
|-|-|
| **Linux containers** | The entry point is `tail -f /dev/null`, which is an infinite wait to keep the container running. When the app is launched through the debugger, it's the debugger that is responsible to run the app (that is, `dotnet webapp.dll`). If launched without debugging, the tooling runs a `docker exec -i {containerId} dotnet webapp.dll` to run the app.|
| **Windows containers**| The entry point is something like `C:\remote_debugger\x64\msvsmon.exe /noauth /anyuser /silent /nostatus` which runs the debugger, so it's listening for connections. This method applies when the debugger runs the app. When launched without debugging, a `docker exec` command is used. For .NET Framework web apps, the entry point is slightly different where `ServiceMonitor` is added to the command.|

The container entry point can only be modified in Docker Compose projects, not in single-container projects.

## SSL-enabled ASP.NET Core apps

Container tools in Visual Studio support debugging an SSL-enabled ASP.NET core app with a dev certificate, the same way you'd expect it to work without containers. To make that happen, Visual Studio adds a couple of more steps to export the certificate and make it available to the container. Here is the flow that Visual Studio handles for you when debugging in the container:

1. Ensures the local development certificate is present and trusted on the host machine through the `dev-certs` tool.
2. Exports the certificate to `%APPDATA%\ASP.NET\Https` with a secure password that is stored in the user secrets store for this particular app.
3. Volume-mounts the following directories:

   - `*%APPDATA%\Microsoft\UserSecrets`
   - `*%APPDATA%\ASP.NET\Https`

ASP.NET Core looks for a certificate that matches the assembly name under the *Https* folder, which is why it's mapped to the container in that path. The certificate path and password can alternatively be defined using environment variables (that is, `ASPNETCORE_Kestrel__Certificates__Default__Path` and `ASPNETCORE_Kestrel__Certificates__Default__Password`) or in the user secrets json file, for example:

```json
{
  "Kestrel": {
    "Certificates": {
      "Default": {
        "Path": "c:\\app\\mycert.pfx",
        "Password": "strongpassword"
      }
    }
  }
}
```

If your configuration supports both containerized and non-containerized builds, you should use the environment variables, because the paths are specific to the container environment.

For more information about using SSL with ASP.NET Core apps in containers, see [Hosting ASP.NET Core images with Docker over HTTPS](/aspnet/core/security/docker-https).

For a code sample that demonstrates creating custom certificates for a multi-service app that are trusted on the host and in the containers for HTTPS service-to-service communication, see [CertExample](https://github.com/NCarlsonMSFT/CertExample).

## Related content

- [MSBuild properties for container projects](container-msbuild-properties.md).
- [MSBuild](../msbuild/msbuild.md)
- [Dockerfile on Windows](/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile)
- [Linux containers on Windows](/virtualization/windowscontainers/deploy-containers/linux-containers)
