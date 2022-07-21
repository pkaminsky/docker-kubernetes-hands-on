## Make a new app from a template

We will use C#

With the Visual Studio Wizard
1. open visual studio
2. create a new project
3. select ASP.NET Core Web App
4. Call it HelloDocker or some nonsense
5. `Check ENABLE DOCKER`
6. Docker OS Is Linux
7. Click "Create"

Visual Studio has some weird thing that tries to actually launch it in Docker. I don't use that. If I want to run it, I just run it locally.

You can run the app with F5.

## Look at the Dockerfile

The reason we made a C# template app was because they create a great Dockerfile with a two-stage build, and because Microsoft is one of the best maintainers of their base images.

```
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["HelloDocker/HelloDocker.csproj", "HelloDocker/"]
RUN dotnet restore "HelloDocker/HelloDocker.csproj"
COPY . .
WORKDIR "/src/HelloDocker"
RUN dotnet build "HelloDocker.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "HelloDocker.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "HelloDocker.dll"]

```

https://hub.docker.com/_/microsoft-windows-base-os-images
https://docs.microsoft.com/en-us/dotnet/architecture/microservices/net-core-net-framework-containers/official-net-docker-images

## There's also a .dockerignore file

When you build with Dockerbuild, (and this may be changing) it takes all the directories under your current directory, and loads them into some temporary space where it's building the image. If there's a lot of files or a lot of size, it takes a while. So it's important to be aware of this and ignore any files that don't need to be in the build.

You also want to make sure to ignore and files you don't want to be in the resulting image, such as anything that contains secrets. `.dockerignore` often resembles `.gitignore`. Since the two-stage Dockerfile will be building the binaries, we also don't want to bother copying the binaries in, since they are large, so for example `**/bin` is ignored.

## Build the dockerfile

go to your command line and type this.

> docker build -f Hello/DockerDockerfile .

It might not work depending on what directory you're in, and how the directories of your project are set up!

`-f HelloDocker/Dockerfile` is the relative path to the Dockerfile and `.` is the `build context.` Any file you want to be available to the image build process must be within the build context. If you made the project with the Visual Studio Template, the Dockerfile is awkwardly in the project folder, not the solution folder, because C# likes to make all kinds of extra directories all over the place. So, we have to invoke Docker from the solution directory, and tell it the path to the Docker file.

`docker build` builds an *image*. When an image is running, it's called a *container.* Do beware that most people get sloppy and use these terms interchangeably, so listen for context clues!

## Now run the image

Ok so we built the image and the message was something like 

> Successfully built 5ac885e1379d

The hash is the ID of the image. That sucks as a name, so let's give it a name. You can give things a name (called a tag) by passing `-t name`to `docker build`. 

We can also tag the image we already built

> docker tag 5ac885e1379d hello-docker:latest

> docker image list | grep hello-docker

So now it's easier to run the image.

> docker run hello-docker:latest

It should spew some stuff. However there's no way to connect to it in a browser...

> ctrl + c

Let's prune it
> docker container list --all

You can now remove those. Relevant commands, `docker container prune`, `docker kill`, `docker container rm -f`

This time run docker with some more flags

> docker run -p 8888:80 hello-docker:latest

This maps port 8888 on the host machine to 80 inside the container. Now you should be able to go to your site at http://localhost:8888/

## Some last important flags for docker run

- `-it` this combination of flags allows you to attach a terminal/CLI in the container when it runs
- `--entrypoint` containers have a default entrypoint (usually) but you can pass an entrypoint. passing `/bin/bash` with the aforementioned `-it` will give you a bash terminal, useful for debugging your image build to see what's in there. If the image just uses `args` instead of `entrypoint`, then you just pass the starting command as the last arguments to docker run.
- `-d` detached, important if you don't want the container to hog your command line you launched it from.

>  docker run -p 8888:80 -it --entrypoint "/bin/bash" hello-docker:latest