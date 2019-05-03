---
layout: post
title: running a container in swarm
tags: misl docker dotnet-core raspberry-pi
---

now that i have a slimmed down from the previous [post]({% post_url 2019-04-30-exec-format-error %}) lets get it up in running as a service. now not reading before trying, i took the working `docker run` example from earlier post and jumped straight in with...

```
docker service create -p 5001:443 \
   -e ASPNETCORE_URLS="https://+443" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Password="crypticpassword" \
   -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/GuidMaker.pfx \
   -v ${HOME}/https:/https/ \
   --name guidmaker \
   legobuff/guidmaker:arm32
```

and quickly recieved:

```
unknown shorthand flag: 'v' in -v
See 'docker service create --help'.
```

okay, so to the [documentation](https://docs.docker.com/engine/reference/commandline/service_create/)... where I find that `-v ${HOME}/https:/https` needs to be swapped out with `--mount type=bind,source=${HOME}/https,destination=/https`, trying again...

```
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
```

so that works, though after reading the documentation looks like docker service "prefers" to use the full arguments so lets try again with...

```
docker service create --name guidmaker \
   --publish published=5001,target=443 \
   --env ASPNETCORE_URLS="https://+443" \
   --env ASPNETCORE_Kestrel__Certificates__Default__Password="crypticpassword" \
   --env ASPNETCORE_Kestrel__Certificates__Default__Path=/https/GuidMaker.pfx \
   --mount type=bind,source=/Users/jamiewallingford/https,destination=/https \
   legobuff/guidmaker:arm32

Error response from daemon: rpc error: code = InvalidArgument desc = port '5001' is already in use by service 'guidmaker' (d0oudeiu51kuj26cemtcesflk) as an ingress port
```

oops, i forgot to remove the service before creating it again...  `docker service rm guidmaker` takes care of that and trying again, success. now a `docker service ls` gives...

```
ID                  NAME                MODE                REPLICAS            IMAGE                       PORTS
3y16p9y4oppe        guidmaker           replicated          1/1                 legobuff/guidmaker:arm32   *:5001->443/tcp
```

lets see if it scales...

```
docker service scale guidmaker=6
guidmaker scaled to 6
overall progress: 6 out of 6 tasks 
1/6: running   [==================================================>] 
2/6: running   [==================================================>] 
3/6: running   [==================================================>] 
4/6: running   [==================================================>] 
5/6: running   [==================================================>] 
6/6: running   [==================================================>] 
verify: Service converged 
```

and...

```
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                       PORTS
3y16p9y4oppe        guidmaker           replicated          6/6                 legobuff/guidmaker:arm32   *:5001->443/tcp

...

docker service ps guidmaker 
ID                  NAME                IMAGE                       NODE                    DESIRED STATE       CURRENT STATE                ERROR               PORTS
er26ed2gxufy        guidmaker.1         legobuff/guidmaker:arm32    linuxkit-025000000001   Running             Running 2 minutes ago                            
8bnd1v1j52r5        guidmaker.2         legobuff/guidmaker:arm32    linuxkit-025000000001   Running             Running about a minute ago                       
ppot66z4u5i7        guidmaker.3         legobuff/guidmaker:arm32    linuxkit-025000000001   Running             Running about a minute ago                       
7h47h0z553ah        guidmaker.4         legobuff/guidmaker:arm32    linuxkit-025000000001   Running             Running about a minute ago                       
f6j7mcvsc4kw        guidmaker.5         legobuff/guidmaker:arm32    linuxkit-025000000001   Running             Running about a minute ago                       
9av1kjqlfu51        guidmaker.6         legobuff/guidmaker:arm32    linuxkit-025000000001   Running             Running about a minute ago                       
```

double checking by hitting the service at `https://localhost:5001/api/guid`, and 👍
