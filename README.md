# Prometheus-Grafana
Problem Statement=>
Integrate Prometheus, Grafana & K8s then perform in following tasks:
1. Deploy them as pods on top of Kubernetes by creating resources Deployment, ReplicaSet, Pods or Services
2. And make their data to be remain persistent.
3. And both of them should be exposed to outside world.
Solution=>
first see about tools needed or use of it
Prometheus=>
Prometheus is a free software application used for event monitoring and alerting.It records real-time metrics in a time series database built using a HTTP pull model, with flexible queries and real-time alerting. The project is written in Go and licensed under the Apache 2 License, with source code available on GitHub,[5] and is a graduated project of the Cloud Native Computing Foundation, along with Kubernetes and Envo
Grafana=>
Grafana is a multi-platform open source analytics and interactive visualization web application. It provides charts, graphs, and alerts for the web when connected to supported data sources. It is expandable through a plug-in system. End users can create complex monitoring dashboardsusing interactive query builders.
As a visualization tool, Grafana is a popular component in monitoring stacks,often used in combination with time series databases such as InfluxDB, Prometheus and Graphite; monitoring platforms such as Sensu,Icinga, Zabbix, Netdata, and PRTG; SIEMs such as Elasticsearch and Splunk; and other data sources.
Kubernetes=>
Kubernetes (commonly stylized as K8s) is an open-source container-orchestration system for automating computer application deployment, scaling, and management.
It was originally designed by Google and is now maintained by the Cloud Native Computing Foundation. It aims to provide a “platform for automating deployment, scaling, and operations of application containers across clusters of hosts”. It works with a range of container tools, including Docker.
Many cloud services offer a Kubernetes-based platform or infrastructure as a service (PaaS or IaaS) on which Kubernetes can be deployed as a platform-providing service. Many vendors also provide their own branded Kubernetes distributions.
now this we needed so for promotheus or grafan we have used docker image we have create own docker image
Docker=>
Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files; they can communicate with each other through well-defined channels. All containers are run by a single operating system kernel and therefore use fewer resources than virtual machines.
The service has both free and premium tiers. The software that hosts the containers is called Docker Engine. It was first started in 2013 and is developed by Docker, Inc.
Now First we create docker image of promotheus for this we create Dockerfile on redhat VM we used AWS for more faster but you can use any online offline
we create Dockerfile of prometheus
mkdir prometheus
vi Dockerfile
FROM centos:8
RUN yum install wget -y
RUN wget https://github.com/prometheus/prometheus/releases/download/v2.19.0/prometheus-2.19.0.linux-amd64.tar.gz
RUN tar -xvf prometheus-2.19.0.linux-amd64.tar.gz
WORKDIR prometheus-2.19.0.linux-amd64/
EXPOSE 9090
CMD ./prometheus
after create Dockerfile run command
docker build -t prometheus:v1 /root/prometheus
Now We create a Docker file of Grafana
mkdir grafana
vi Dockerfile
FROM centos:8
RUN yum install wget -y
RUN wget https://dl.grafana.com/oss/release/grafana-7.0.3.linux-amd64.tar.gz
RUN tar -zxvf grafana-7.0.3.linux-amd64.tar.gz
WORKDIR grafana-7.0.3/bin/
EXPOSE 3000
CMD ./grafana-serve
after create Dockerfile run command
docker build -t grafana:v1 /root/grafana
now we push both file in hub.docker.com👇 check
Docker Hub
Edit description
hub.docker.com
for upload first login docker in redhat or linux or create account then use
docker login
Username: give docker hub username
password: give your password
now after login we push
docker push promotheus:v1
docker push grafana:v1
You can direct use my image if you dont do this for this 👇
Docker Hub
Edit description
hub.docker.com
Docker Hub
Edit description
hub.docker.com
now create YML file for promotheus we give name promotheus.yml for this open CMD write
notepad promotheus.yml
now we create this file on it 👇
apiVersion: v1
kind: Service
metadata:
  name: promo-service
  labels:
    env: prometheus
spec:
  ports:
    - port: 9090
  selector:
    env: prometheus
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheuspvc
  labels:
    env: prometheus
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: promo-deployment
  labels:
    env: prometheus
spec:
  selector:
    matchLabels:
      env: prometheus
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        env: prometheus
    spec:
      containers:
      - image: vimal13/prometheus:latest
        name: prometheus
        ports:
        - containerPort: 9090
          name: prometheus
        volumeMounts:
        - name: prometheus-mount
          mountPath: /etc/prometheus/data
      volumes:
      - name: prometheus-mount
        persistentVolumeClaim:
          claimName: prometheuspvc
Now promotheus yml file created now we have run this in kubernetses for this first we start minikube
minikube start
Image for post
if you find error in minikube start so try
minikube start — driver=virtualbox
after minikube start run this file
kubernetes create -f promotheus.yml
now see its created or not for this we have to check with command
kubectl get pods
or to see all services use
kubectl get all
Image for post
now see all file running successfully if you se ready 1/1 or status running that means it work successful
minikube ip
check ip we have take port for all command
Image for post
now take IP and port and open in chrome
Image for post
now promotheus run Successfully 😎😎👆👆🤘🤘
now we create one more yml file for Grafana
notepad grafana.yml
Now create Grafan👇
apiVersion: v1
kind: Service
metadata:
  name: grafana-svc
  labels:
    env: grafana
spec:
  ports:
    - port: 3000
  selector:
    env: grafana
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
  labels:
    env: grafana
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: grafana-deployment
  labels:
    env: grafana
spec:
  selector:
    matchLabels:
      env: grafana
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        env: grafana
    spec:
      containers:
      - image: princeprashantsaini/grafana:v1
        name: grafana
        ports:
        - containerPort: 3000
          name: grafana
        volumeMounts:
        - name: grafana-volume
          mountPath: /var/lib/grafana
      volumes:
      - name: grafana-volume
        persistentVolumeClaim:
          claimName: grafana-pvc
now run this file also in kubernetes
kubectl create -f grafana.yml
after create check with kubectl command its run or not
kubectl get pods
kubectl get all
Image for post
now container is creating then we see its working or not wait some time depends on your internet speed check with same command after some time it show running
Image for post
now it running we keep on checking keep on checking
now take IP and port and open in chrome
minikube ip
take IP and port check ip with minikube ip command and port with kubectl get all
Image for post
But if you have open grafana first time they ask login or password so bydefult username and password is
Username: admin
Password: admin
use this username and password and login now you have see one set-password page some up now you can set new password
Image for post
give new password and confirm new password after it submit and you submit then you see this type of screen
Image for postImage for post
now you have successfully login
Image for post
now Grafan Work successfully👆👆😎😎🤘🤘
Now promotheus add in grafana so first check we have promotheus plugin install or not ( bydefault it installed )👇
Image for post
Now add promotheus in grafan for this we have to use
Data Source in Grafan
Image for post
Add resource to promotheus click on promotheus
Image for postImage for post
it show like this 👆 you can fill all details on it and at last save
Now see in daseboard promotheus come
Image for postImage for post
Now add successfully now you can give any query then used it
Now check its volume is permanent or not for this check our PVC
kubectl get pvc
Image for postImage for post
now all attached successfully with file where we they stored data
