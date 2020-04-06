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
## 6. troubleshooting
if met following issue:
Docker repository server gave HTTP response to HTTPS client
then
should consider checking docker info to validate your insecure conf working or not.
still not, then following this [link](https://stackoverflow.com/questions/42211380/add-insecure-registry-to-docker)
go to the last section

```
# /etc/default/docker    
DOCKER_OPTS="--insecure-registry=a.example.com --insecure-registry=b.example.com"
```
## 7. pull image 
follow this 

```
To pull images from your repo, use (notice port 8082 being used):

docker pull your-repo:8082/httpd:2.4-alpine
To push your own images to your repo, you have to tag the image with a tag that points to the repo. This is strange to me, since I was trying to think about Docker tags the same way I do about Git tags, but they seem be somewhat different (notice port 8083 being used):

docker tag your-own-image:1 your-repo:8083/your-own-image:1
docker push your-repo:8083/your-own-image:1
To pull your own images from the repo, you can use:

docker tag your-own-image:1 your-repo:8082/your-own-image:1
# or
docker tag your-own-image:1 your-repo:8083/your-own-image:1
Both ports will work. I suspect that is because using port 8083 will connect directly to the hosted repo, whilst using port 8082 will connect to the group repo, which contains the hosted repo. I suggest you to stick to port 8083 to avoid duplicate images in your machines. If you chose to stick with port 8083 to pull your own images, you probably could skip creating the group repo, if you prefer.
```
