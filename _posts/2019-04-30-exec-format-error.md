---
layout: post
title: exec format error
tags: misl docker dotnet-core aspnet raspberry-pi
date: 2019-04-30 17:37:00
---

after [getting a working image]({% post_url 2019-04-30-moving-aspnet-code-app-to-docker %}), i pushed the image to docker hub. i then copied the GuidMaker.pfx to my raspberry pi:

```
scp ~/.aspnet/https/GuidMaker.pfx pi@pi-portable.local:.
```

then ssh'd to the pi and moved the pfx to a directory called https. now i tried to run my image:

```
docker run -it -p 5001:443 \
   -e ASPNETCORE_URLS="https://+443" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Password="crypticpassword" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/GuidMaker.pfx \
   -v ${HOME}/https:/https/ \
   --name guidmaker \
   legobuff/guidmaker
```

and was greeted with...

```
Unable to find image 'legobuff/guidmaker:latest' locally
latest: Pulling from legobuff/guidmaker
27833a3ba0a5: Pull complete 
e425c8f949e7: Pull complete 
2198a2fb121b: Pull complete 
5a76da1f4a07: Pull complete 
1460e406e879: Pull complete 
Digest: sha256:d6fd576ba36af6170453042d258ccda3b95ccacedb853a4012fcd355b1d83d36
Status: Downloaded newer image for legobuff/guidmaker:latest
standard_init_linux.go:207: exec user process caused "exec format error"
```

i had seen this error while researching raspberry-pi and docker and expected it. now, i need to create a arm architecture specific image. after a bit of googling i created a new [Dockerfile](https://github.com/legobuff/docker-tests/blob/master/GuidMaker/Dockerfile.debian-arm32-selfcontained) and proceeded to build and push to the docker hub and try to run it again.

```
docker build -f Dockerfile.debian-arm32-selfcontained -t legobuff/guidmaker:arm32 .

docker push legobuff/guidmaker:arm32

docker run -it -p 5001:443 \
   -e ASPNETCORE_URLS="https://+443" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Password="crypticpassword" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/GuidMaker.pfx \
   -v ${HOME}/https:/https/ \
   --name guidmaker \
   legobuff/guidmaker:arm32
```

and...

```
Hosting environment: Production
Content root path: /app
Now listening on: https://[::]:443
Application started. Press Ctrl+C to shut down.
```

success. now im my googling of the error i came across [.NET IL Linker](https://github.com/dotnet/core/blob/master/samples/linker-instructions.md) which can be used to reduce the size of .net core applications. seeing as i am deploying this on a raspberry pi, lets give it a go. i updated the Dockerfile and added the following before the `RUN dotnet publish...`

```
RUN dotnet add package ILLink.Tasks -v 0.1.5-preview-1841731 -s https://dotnet.myget.org/F/dotnet-core/api/v3/index.json
```

after a docker build i was greeted with...

```
Step 8/12 : RUN dotnet publish -c Release -r linux-arm -o out /p:ShowLinkerSizeComparison=true
 ---> Running in 415f68fe1163
Microsoft (R) Build Engine version 16.0.450+ga8dc7f1d34 for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restore completed in 7.68 sec for /app/GuidMaker.csproj.
  GuidMaker -> /app/bin/Release/netcoreapp2.2/linux-arm/GuidMaker.dll
ILLINK : warning : unresolved assembly System.Threading.AccessControl [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.IO.Packaging [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.CodeDom [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly Microsoft.Win32.SystemEvents [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.Configuration.ConfigurationManager [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.Diagnostics.PerformanceCounter [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.Diagnostics.EventLog [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.IO.Ports [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.ServiceProcess.ServiceController [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.ServiceModel.Syndication [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.Security.Cryptography.ProtectedData [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.Drawing.Common [/app/GuidMaker.csproj]
ILLINK : warning : unresolved assembly System.Data.Odbc [/app/GuidMaker.csproj]
  Restore completed in 1.84 sec for /app/GuidMaker.csproj.
/usr/share/dotnet/sdk/2.2.203/Sdks/Microsoft.NET.Sdk/targets/Microsoft.NET.CrossGen.targets(48,5): error NETSDK1016: Unable to find resolved path for 'coreclr'. [/app/GuidMaker.csproj]
The command '/bin/sh -c dotnet publish -c Release -r linux-arm -o out /p:ShowLinkerSizeComparison=true' returned a non-zero code: 1
```

~~and bummer... so a bit of googling and it looks like it is [not compatible with netcoreapp2.2](https://github.com/mono/linker/issues/405).~~

__updated @ 19:34:__ after googling around i stumbled upon this [post](https://www.hanselman.com/blog/ACompleteContainerizedNETCoreApplicationMicroserviceThatIsAsSmallAsPossible.aspx) from [Scott Hanselman](https://twitter.com/shanselman) where he had this nugget

> "I did end up hitting [this bug](https://github.com/mono/linker/issues/314?WT.mc_id=-blog-scottha) in the Linker (it's not Released) but [there's an easy workaround](https://github.com/mono/linker/issues/314?WT.mc_id=-blog-scottha#issuecomment-417030818). I just need to set the property _CrossGenDuringPublish_ to false in the project file."

after doing this, it works. now i need to do as Scott says...

> "...iterate. Build, trim, test, add an assembly by reading the error message, and repeat."
