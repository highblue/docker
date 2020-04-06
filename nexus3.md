# how to setup your private repo using nexus3 with http.

## 0. prerequisite

1. suggest starting with clean docker env.
2. setup hostname, such as nexus.local, add that to your hosts file.

3. install the docker-ce
4. modify the /etc/docker/daemon.json

insert following
```
{
    "insecure-registries" : [ "nexus.local:8082","nexus.local:8083" ]
}
```

5. using the $docker info to check the insecure-registries effective or not.

```
$docker info

 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  nexus.local:8082
  nexus.local:8083
  127.0.0.0/8
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
//这里为什么是200的user group? 因为nexus3的运行用户id为200，所以需要把数据目录的owner改为200。

## 2. how to retrieve the password and using passwd to login

```
docker exec -u 0 -it nexus bash
cd nexus-data/
cat admin.password
```

## 3. configure nexus3
follow these links
<https://blog.sonatype.com/using-nexus-3-as-your-repository-part-3-docker-images>
and basic intro基础介绍
<https://yeasy.gitbooks.io/docker_practice/repository/nexus3_registry.html>

使用maven的方式 <https://www.jianshu.com/p/2f681fe265ce>

## 4. test with docker login

if met: error WARNING! Using --password via the CLI is insecure. Use --password-stdin.

```
echo 'admin'>pass.txt
cat pass.txt | docker login -u admin --password-stdin nexus.local:8082
```

## 5. docker login
Now we have to authenticate your machine to the repo with:
```
docker login -u admin -p admin123 your-repo:8082
docker login -u admin -p admin123 your-repo:8083
This will create an entry in ~/.docker/config.json:

{
	"auths": {
		"your-repo:8082": {
			"auth": "YWRtaW46YWRtaW4xMjM="
		},
		"your-repo:8083": {
			"auth": "YWRtaW46YWRtaW4xMjM="
		}
}
```
