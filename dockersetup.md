## Set up in Docker in RHEL with private nexus repository
As Root do the following

yum install docker

systemctl start docker
docker info

docker --version

Do the following only if Nexus serving in http. For https enabled nexus this is not required

vi /etc/docker/daemon.json

Add the following block

{
  "insecure-registries": [
    "host:8082",
    "host:8083"
  ],
  "disable-legacy-registry": true
}
systemctl restart docker

docker login -u "nexus-user" -p pwd nexus.example.com:8444
docker login -u "nexus-user" -p pwd nexus.example.com:8445

docker run nexus.example.com:8444/hello-world



## Useful Docker commands
Build Docker image
docker build --tag=nexus.example.com:8445/deepag/example/example-base:latest --rm=true .

Push Docker image to docker registry
docker push nexus.example.com:8445/deepag/example/example-base:latest

Run Docker image in Foreground
docker run -it --rm --name=example-base --publish=8080:8080 52.24.195.43:8083/deepag/example/example-base:latest

Create Network bridge
docker network create bigd-network
Run Image in detatched mode

docker run -d --name=approval --network=example-network --publish=9030:9030 -v "/opt/example/deployment/releases/conf:/opt/example/deployment/releases/conf" -v "/opt/example/deployment/keys:/opt/example/deployment/keys" host:8083/deepag/example/approval:latest

Export image as tar
docker save -o "notification.tar" 52.24.195.43:8083/deepag/example/notification
Import image from a tar
docker load -i notification.tar

Enter the container shell

docker exec -it <container name> /bin/sh
kill all running containers - docker kill $(docker ps -q)
delete all stopped containers - docker rm $(docker ps -a -q)

delete all images - docker rmi $(docker images -q)
