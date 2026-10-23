### Laboratorio

## PC ext --> Kubernets 
##  nginx 1 puerto 8081

## Nodeport, permite conectase a uno o varios puertos 
## Kind, gestiona el acceso

adminepn@svr-devops02:~$ docker --version
Docker version 29.9.2, build a82e528

adminepn@svr-devops02:~$ sudo docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS          PORTS                       NAMES
e4272c583a9e   kindest/node:v1.37.0   "/usr/local/bin/entr…"   3 hours ago   Up 33 minutes   127.0.0.1:44587->6443/tcp   curso-k8s-control-plane

adminepn@svr-devops02:~$ kubectl version --client
Client Version: v1.37.1
Kustomize Version: v5.8.1

Instalar certificados

adminepn@svr-devops02:~$ sudo apt install -y curl ca-certificates

adminepn@svr-devops02:~$ sudo docker info

Client: Docker Engine - Community
Version:    29.9.2
Context:    default
Debug Mode: false

Keyrings, descargar llaves de docker 

sudo install -m 0755 -d /etc/apt/keyrings

sudo chmod a+r /etc/apt/keyrings/docker.asc
adminepn@svr-devops01:~$ sudo vim /etc/apt/sources.list.d/docker.sources
adminepn@svr-devops01:~$ sudo more /etc/apt/sources.list.d/docker.sources
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: resolute
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
adminepn@svr-devops01:~$

adminepn@svr-devops02:~$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


adminepn@svr-devops02:~$ sudo systemctl enable --now docker
adminepn@svr-devops02:~$ sudo systemctl is-active docker

Lab


adminepn@svr-devops02:~$ kubectl create namespace lab-web
namespace/lab-web created
adminepn@svr-devops02:~$
adminepn@svr-devops02:~$
adminepn@svr-devops02:~$
adminepn@svr-devops02:~$ kubectl get  namespace
NAME                 STATUS   AGE
default              Active   21h
kube-node-lease      Active   21h
kube-public          Active   21h
kube-system          Active   21h
lab-web              Active   44s
local-path-storage   Active   21h
adminepn@svr-devops02:~$


adminepn@svr-devops02:~$ mkdir -p ~/laboratorios-k8s/lab-nginx

adminepn@svr-devops02:~$
adminepn@svr-devops02:~$ cd ~/laboratorios-k8s/lab-nginx/
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ ll
total 8
drwxrwxr-x 2 adminepn adminepn 4096 Oct 22 14:35 ./
drwxrwxr-x 4 adminepn adminepn 4096 Oct 22 14:35 ../



adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ ll
total 16
drwxrwxr-x 2 adminepn adminepn 4096 Oct 22 14:43 ./
drwxrwxr-x 4 adminepn adminepn 4096 Oct 22 14:35 ../
-rw-rw-r-- 1 adminepn adminepn 1633 Oct 22 14:43 soporte.html
-rw-rw-r-- 1 adminepn adminepn 1569 Oct 22 14:42 tramites.html

Mapa de como va a comportarse el kuernetes

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web create configmap html-tramites --from-file=index.html=tramites.html --dry-run=client -o yaml | kubectl apply -f -
configmap/html-tramites created


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get configmaps
NAME               DATA   AGE
html-tramites      1      113s
kube-root-ca.crt   1      17m


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get configmap html-tramites\
NAME            DATA   AGE
html-tramites   1      2m15s


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web describe configmap html-tramites
Name:         html-tramites
Namespace:    lab-web
Labels:       <none>
Annotations:  <none>

Data
====
index.html:
----


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web create configmap html-soporte --from-file=index.html=soporte.html --dry-run=client -o yaml | kubectl apply -f -
configmap/html-soporte created

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get configmaps
NAME               DATA   AGE
html-soporte       1      9s
html-tramites      1      8m47s
kube-root-ca.crt   1      24m
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$



adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web describe configmap html-soporte
Name:         html-soporte
Namespace:    lab-web
Labels:       <none>
Annotations:  <none>

Data
====
index.html:
----


deployment: describe lo q se va a ejecutar 
pod: unidad de ejecucion 
service: punto de acceso hacia los pods



adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ vim portal-tramites.yaml

Verifica 

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl apply --dry-run=client --validate=true -f portal-tramites.yaml
deployment.apps/portal-tramites created (dry run)
service/portal-tramites created (dry run)

Aplica 

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl apply -f portal-tramites.yaml
deployment.apps/portal-tramites created
service/portal-tramites created

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
portal-tramites   1/1     1            1           50s


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get pods
NAME                               READY   STATUS    RESTARTS   AGE
portal-tramites-6b747f4bbc-8bvbb   1/1     Running   0          72s
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$

Verifica si ya se descargó la imagen del nginx 

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web rollout status deployment/portal-tramites --timeout=180s
deployment "portal-tramites" successfully rolled out
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$


Creamos el archivo de soporte


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ vim portal-soporte.yaml
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl apply --dry-run=client --validate=true -f portal-soporte.yaml
deployment.apps/portal-soporte created (dry run)
service/portal-soporte created (dry run)
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl apply -f portal-soporte.yaml
deployment.apps/portal-soporte created
service/portal-soporte created


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web rollout status deployment/portal-soporte --timeout=180s
deployment "portal-soporte" successfully rolled out

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
portal-soporte    1/1     1            1           103s
portal-tramites   1/1     1            1           5h12m



adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP           NODE                      NOMINATED NODE   READINESS GATES
portal-soporte-5b46877449-mmhx9    1/1     Running   0          2m24s   10.244.0.7   curso-k8s-control-plane   <none>           <none>
portal-tramites-6b747f4bbc-8bvbb   1/1     Running   0          5h13m   10.244.0.6   curso-k8s-control-plane   <none>   


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get services
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
portal-soporte    ClusterIP   10.96.180.96   <none>        80/TCP    3m36s
portal-tramites   ClusterIP   10.96.58.27    <none>        80/TCP    5h14m
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web exec deployments/portal-tramites -- cat /usr/share/nginx/html/index.html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Portal de Trámites</title>



adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get endpointslices -l kubernetes.io/service-name=portal-tramites
NAME                    ADDRESSTYPE   PORTS   ENDPOINTS    AGE
portal-tramites-nx72p   IPv4          80      10.244.0.6   5h22m

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web get endpointslices -l kubernetes.io/service-name=portal-soporte
NAME                   ADDRESSTYPE   PORTS   ENDPOINTS    AGE
portal-soporte-f9c7q   IPv4          80      10.244.0.7   11m


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web describe service portal-tramites
Name:                     portal-tramites
Namespace:                lab-web
Labels:                   <none>
Annotations:              <none>
Selector:                 app=portal-tramites
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.58.27
IPs:                      10.96.58.27
Port:                     http  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.0.6:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web describe service portal-soporte
Name:                     portal-soporte
Namespace:                lab-web
Labels:                   <none>
Annotations:              <none>
Selector:                 app=portal-soporte
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.180.96
IPs:                      10.96.180.96
Port:                     http  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.0.7:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>


adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web logs deployments/portal-soporte --tail=20
10.244.0.1 - - [22/Oct/2026:20:41:42 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:41:47 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:41:52 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:41:57 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:02 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:07 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:12 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:17 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:22 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:27 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:32 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:37 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:42 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:47 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:52 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:57 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:02 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:07 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:12 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:17 +0000] "GET / HTTP/1.1" 200 1633 "-" "kube-probe/1.37" "-"
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web logs deployments/portal-tramites --tail=20
10.244.0.1 - - [22/Oct/2026:20:41:46 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:41:51 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:41:56 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:01 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:06 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:11 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:16 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:21 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:26 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:31 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:36 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:41 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:46 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:51 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:42:56 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:01 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:06 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:11 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:16 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
10.244.0.1 - - [22/Oct/2026:20:43:21 +0000] "GET / HTTP/1.1" 200 1569 "-" "kube-probe/1.37" "-"
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ sudo ss -lntp | grep -E ':(8081|8082)\b' || true
[sudo: authenticate] Password:
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$




Publicamos

Terminal 1

dminepn@svr-devops02:~/laboratorios-k8s/lab-nginx$ kubectl -n lab-web port-forward --address 172.25.4.82 services/portal-tramites 8081:80
Forwarding from 172.25.4.82:8081 -> 80

Otra terminal 

adminepn@svr-devops02:~$ kubectl -n lab-web port-forward --address 172.25.4.82 services/portal-tramites 8082:80
Forwarding from 172.25.4.82:8082 -> 80


adminepn@svr-devops02:~$ kubectl -n lab-web get pods,service
NAME                                   READY   STATUS    RESTARTS   AGE
pod/portal-soporte-5b46877449-mmhx9    1/1     Running   0          29m
pod/portal-tramites-6b747f4bbc-8bvbb   1/1     Running   0          5h40m

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/portal-soporte    ClusterIP   10.96.180.96   <none>        80/TCP    29m
service/portal-tramites   ClusterIP   10.96.58.27    <none>        80/TCP    5h40m
adminepn@svr-devops02:~$


