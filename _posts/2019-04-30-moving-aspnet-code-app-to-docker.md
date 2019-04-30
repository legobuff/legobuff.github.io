---
layout: post
title: moving a dotnet core aspnet app to docker (part 1)
tags: misl docker dotnet-core aspnet
---
as i learn docker, and with my professional life working primarily in the microsoft ecosystem, i have read many [blogs](https://blog.alexellis.io), [articles](https://www.hanselman.com/blog/GettingStartedWithNETCoreAndDockerAndTheMicrosoftContainerRegistry.aspx), and [tutorials](https://docs.docker.com/docker-for-mac/). these have been educational, yet I continue to run into issues and nuances as i work to move a dotnet core aspnet app into a container. i am documenting these here as i run into them.

[GuidMaker](https://github.com/legobuff/docker-tests) is a simple dotnet core aspnet app that generates a [GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) and shows which hostname generated it. it was inspired by [Alex Ellis'](https://www.alexellis.io) [guid generator in node.js](https://blog.alexellis.io/microservice-swarm-mode/). this is what I am moving into a container.

### initial move, needlessly exposing ports, https woes

when creating a Dockerfile i was adding `EXPOSE` lines like [this](https://thomasbandt.com/running-aspnetcore-with-https-in-a-docker-container) indicated. doing this did not cause any errors, however, after reviewing the [dotnet-docker](https://github.com/dotnet/dotnet-docker) repo and the [ASP.NET Core Docker Sample](https://github.com/dotnet/dotnet-docker/tree/master/samples/aspnetapp), i noticed the [sample](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/Dockerfile) does not use `EXPOSE`, and so i removed these from the GuidMaker Dockerfile.

after running docker build, i am now able to run GuidMaker with a simple port map by:

```
docker run --name guidmaker --rm -it -p 5001:443 legobuff/guidmaker
```

yet it is not responding when navigating to `https://localhost:5001/api/guid`, looking at the container output i see...

```
Hosting environment: Production
Content root path: /
Now listening on: http://[::]:80
Application started. Press Ctrl+C to shut down.
```

which shows it listening to http, and seeing as i do not have a port mapping for that, makes sense. so trying again with with...

```
docker run --name guidmaker --rm -it -p 5000:80 legobuff/guidmaker
```

now it works with http, but the container output shows...

```
warn: Microsoft.AspNetCore.HttpsPolicy.HttpsRedirectionMiddleware[3]
      Failed to determine the https port for redirect.
```

again, this time mapping an additional port...

```
docker run --name guidmaker --rm -it -p 5000:80 -p 5001:443 legobuff/guidmaker
```

no dice. if i was thinking, i would have known that would not work as that it telling docker to map a local port to 443 and not configuring aspnet. i also did not pay close attention to the aspnet page above. after re-reading it, i noticed the "See [Hosting ASP.NET Core Images with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https.md) to use HTTPS with this image" line. that link explains how to run the container configured for https. after following the steps outlined, i am able to run via...

```
docker run -it -p 5001:443 \
   -e ASPNETCORE_URLS="https://+443" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Password="crypticpassword" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/GuidMaker.pfx \
   -v ${HOME}/.aspnet/https:/https/ \
   --name guidmaker \
   legobuff/guidmaker
```

and now `https://localhost:5001/api/guid` works. at this point i have a working base image. next i want to follow along with this [tutorial](https://blog.alexellis.io/microservice-swarm-mode/), but that will come in part 2. 
