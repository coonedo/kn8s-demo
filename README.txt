kn8s-demo README
================


Created by: Donal Cooney

Note: 	This demo is a very basic introduction to the topic named above created 
	purely as a tutorial for myself. Absolutely no gaurantees whatsoever 
	wrt these instructions and/or software.


kn8s-demo Summary:
------------------

1. 	Environment (Assumed installed/created already): kn8s-demo runs on 
	Windows PC using Oracle VirtualBox managed by Vagrant. Essentially 
	Unbuntu running virtualised on Windows PC.
	NOTE: Typically you need at least 3 machines, 1 Master Controller 
	and at least 2 nodes.
2. 	








Instructions:
-------------

-	START DEMO
- 	Windoze: Create a directory 'kubernetes_project'

- 	Windoze: From dir 'kubernetes_project', create a file 'Vagrantfile' with 
	the following contents (ref: https://www.vagrantup.com/docs/)

Vagrant.configure(2) do |config|
  config.vm.define "kn8s" do |kn8s|
    kn8s.vm.box = "ubuntu/trusty64"
    kn8s.vm.network "private_network", ip: "192.168.0.250"
    kn8s.vm.hostname = "kn8s.demo.com"
    kn8s.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end
  end
end


OR

-- Clone the following repo from my github

	https://github.com/coonedo/kn8s-demo


- 	Windoze: From dir 'kubernetes_project', run the following command: 
	'vagrant up'
	C:\Users\coonedo\kubernetes_project>vagrant up


-	Windoze: ssh into the box
C:\Users\coonedo\kubernetes_project>vagrant ssh
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-83-generic x86_64)



-	UPDATE TO THE LATEST DOCKER (the following steps are from linux )

	Follow the instructions at: 
	https://docs.docker.com/engine/installation/linux/ubuntulinux/


	In summary:

        - Update package information, ensure that APT works with the https
        method, and that CA certificates are installed.

        $ sudo apt-get update

        $ sudo apt-get install apt-transport-https ca-certificates



        - Add the new GPG key.

        $ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80
        --recv-keys 58118E89F3A912897C070ADBF76221572C52609D




        - Open the /etc/apt/sources.list.d/docker.list file or create
        Add: (For Ubuntu Trusty 14.04 (LTS) )
        deb https://apt.dockerproject.org/repo ubuntu-trusty main



        - Update your APT package index.
        $ sudo apt-get update

        - Install Docker.
        $ sudo apt-get install docker-engine


        - Check version:
        vagrant@docker:~$ sudo docker --versio
        Docker version 1.12.1, build 23cf638




-	CREATE A DOCKER GROUP, ADD YOUR USERID (AVOID HAVING TO SUDO)

	vagrant@docker:~$ sudo groupadd docker
	vagrant@docker:~$ sudo usermod -aG docker vagrant

	logout / login

	vagrant@docker:~$ docker ps
	Cannot connect to the Docker daemon. Is the docker daemon running on 
	this host?



	- stop / restart the docker daemon (when docker daemon starts, it
	will make the unix socker used by docker r/w by docker group)

	vagrant@docker:~$ sudo service docker stop
	docker stop/waiting

	vagrant@docker:~$ sudo service docker start
	docker start/running, process 21539


-	Enable cgroups and also swapaccounting
	edit: vagrant@kn8s:/etc/default$ sudo vi grub

	Edit the following:
	GRUB_CMDLINE_LINUX="cgroup_enable-memory swapaccount=1"



-	NOW, START KUBERNETES. NOTE: THIS IS JUST A DEMO/TEST SETUP - 
	NORMALLY YOU NEED MULTIPLE MACHINES 


-	RUN etcd
	etcd is a highly-available key value store which Kubernetes uses for
	persistent storage of all of its REST API objects

	used here to store the config settings

	vagrant@kn8s:~$docker run --volume=/var/etcd:/var/etcd --net=host -d
gcr.io/google_containers/etcd:2.0.12 /usr/local/bin/etcd --addr=127.0.0.1:4001
--bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data




-	Run the master

sudo docker run \
--volume=/:/rootfs:ro \
--volume=/sys:/sys:ro \
--volume=/dev:/dev \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
--volume=/var/run:/var/run:rw \
--net=host \
--pid=host \
--privileged=true \
-d gcr.io/google_containers/hyperkube:v1.0.1 \
/hyperkube kubelet --containerized --hostname-override="127.0.0.1"
--address="0.0.0.0" --api-servers=http://localhost:8080
--config=/etc/kubernetes/manifests


-	Run the service proxy

Run the service proxy
docker run -d --net=host --privileged
gcr.io/google_containers/hyperkube:v1.0.1 /hyperkube proxy
--master=http://127.0.0.1:8080 --v=2



-	Download kubectl

	kubectl is the cmdline utility that we use to communicate with the
	cluster

wget https://storage.googleapis.com/kubernetes-release/release/v1.0.1/bin/linux/amd64/kubectl

chmod +x kubectl
mkdir bin
mv kubectl bin
. .profile




-	CREATING A POD


	Let's run the nodejs app (hello world) from Decker-demo on kubernetes

	Need: (1) Pod definition in yaml (2) A docker image in a repository


	(2) Docker image in a repo

  	Lets use the docker images from Docker-demo:

vagrant@docker:~$
vagrant@docker:~$ git clone https://github.com/coonedo/docker-demo
Cloning into 'docker-demo'...
remote: Counting objects: 35, done.
remote: Total 35 (delta 0), reused 0 (delta 0), pack-reused 35
Unpacking objects: 100% (35/35), done.
Checking connectivity... done.
vagrant@docker:~$ cd docker-demo
vagrant@docker:~/docker-demo$ ls
Docker_Compose_README.txt   Docker_networks_README.txt  README.txt
docker-compose.yml          index-db.js                 Vagrantfile
Docker_concepts_README.txt  index.js
Dockerfile                  package.json
vagrant@docker:~/docker-demo$
vagrant@docker:~/docker-demo$
vagrant@docker:~/docker-demo$
vagrant@docker:~/docker-demo$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you
don't
have a Docker ID, head over to https://hub.docker.com to create one.
Username: coonedo
Password:
Login Succeeded
vagrant@docker:~/docker-demo$
vagrant@docker:~/docker-demo$
vagrant@docker:~/docker-demo$
vagrant@docker:~/docker-demo$
vagrant@docker:~/docker-demo$ docker build -t coonedo/kn8s-demo .

.
.
.
vagrant@docker:~/docker-demo$ docker push coonedo/kn8s-demo
The push refers to a repository [docker.io/coonedo/kn8s-demo]
bc29b290d404: Pushed
3c66d0dbe246: Pushed
3c54413f5113: Pushed
e98ba489f57c: Mounted from library/node
60e6c2d9d442: Mounted from library/node
2101d3c01933: Mounted from library/node
751f5d9ad6db: Mounted from library/node
17587239b3df: Mounted from library/node
9e63c5bce458: Mounted from library/node
latest: digest:
sha256:5165731ef705766d53ee6f674a5840ee5489cba6f684c0ac4735200cd
5f74459 size: 2213
vagrant@docker:~/docker-demo$





	(1) POD definition in yaml:w


	See pod-kn8s-demo.yml


	Copy files PC (cloned from git)
	
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
To be continued 
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!







	

-	END DEMO



Additional Notes
----------------

