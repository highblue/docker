# how to setup your private repo using nexus3 with http.

## 0. prerequisite

1. suggest starting with clean docker env.
2. setup hostname, such as nexus.local, add that to your hosts file.
3. modify the /etc/docker/daemon.json

//insert following
```
{
    "insecure-registries" : [ "nexus.local:8082","nexus.local:8083" ]
}
```

## 1. start the docker image

```
docker run -d -p 8081:8081 -p 8082:8082 -p 8083:8083 --name nexus sonatype/nexus3
```

or you can use persistent-volume way
https://hub.docker.com/r/sonatype/nexus3/

```
$ mkdir /some/dir/nexus-data && chown -R 200 /some/dir/nexus-data
$ docker run -d -p 8081:8081 -p 8082:8082 -p 8083:8083 --name nexus -v /home/user/nexus-data:/nexus-data sonatype/nexus3
```
//这里为什么是200的user group?
//nexus3的运行用户id为200，所以需要把数据目录的owner改为200。

## 2. how to retrieve the password and using passwd to login

```
docker exec -u 0 -it nexus bash
cd nexus-data/
cat admin.password
```

// if met: error WARNING! Using --password via the CLI is insecure. Use --password-stdin.

```
echo 'admin'>pass.txt
cat pass.txt | docker login -u admin --password-stdin nexus.local:8082
```

## 3. configure nexus3
follow these two links
https://blog.sonatype.com/using-nexus-3-as-your-repository-part-3-docker-images

and

https://yeasy.gitbooks.io/docker_practice/repository/nexus3_registry.html

