# Orquestación de Servicios con Kubernetes

## Nodo 1


adminepn@svr-devops01:~$ sudo docker info --format '{{.DockerRootDir}}'
/var/lib/docker


adminepn@svr-devops01:~$ docker_root=$(sudo docker info --format '{{.DockerRootDir}}')
adminepn@svr-devops01:~$ echo $docker_root
/var/lib/docker



adminepn@svr-devops01:~$ df -h $docker_root
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   29G   12G   16G  43% /
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             172.25.4.80/24 fe80::a00:27ff:fea1:a2cb/64
br-cb6af62f3024  DOWN           192.168.2.1/24
br-e0d7021a87bb  DOWN           172.18.0.1/16
br-0675340bf040  DOWN           192.168.1.1/24
br-144bccab37cd  DOWN           192.168.0.1/24
docker0          UP             172.17.0.1/16 fe80::d495:95ff:feca:a459/64
veth0b55724@if2  UP             fe80::58d1:90ff:fe52:2d90/64
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$ ip route
```text 
default via 172.25.4.10 dev enp0s3 proto static
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
172.18.0.0/16 dev br-e0d7021a87bb proto kernel scope link src 172.18.0.1 linkdown
172.25.4.0/24 dev enp0s3 proto kernel scope link src 172.25.4.80
192.168.0.0/24 dev br-144bccab37cd proto kernel scope link src 192.168.0.1 linkdown
192.168.1.0/24 dev br-0675340bf040 proto kernel scope link src 192.168.1.1 linkdown
192.168.2.0/24 dev br-cb6af62f3024 proto kernel scope link src 192.168.2.1 linkdown
adminepn@svr-devops01:~$
```

adminepn@svr-devops01:~$ resolvectl status
Global
Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub

Link 2 (enp0s3)
Current Scopes: DNS
Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 8.8.8.8
DNS Servers: 8.8.8.8 1.1.1.1
Default Route: yes

Link 3 (br-cb6af62f3024)
Current Scopes: none
Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Default Route: no

Link 4 (br-e0d7021a87bb)
Current Scopes: none
Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Default Route: no

Link 5 (br-0675340bf040)
Current Scopes: none
Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Default Route: no

Link 6 (br-144bccab37cd)
Current Scopes: none
Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Default Route: no

Link 7 (docker0)
Current Scopes: none
Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Default Route: no

Link 8 (veth0b55724)
Current Scopes: none
Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Default Route: no
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$ curl -IL --connect-timeout 10 --max-time 20 https://registry.k8s.io/v2/
HTTP/2 401
content-type: text/plain; charset=utf-8
docker-distribution-api-version: registry/2.0
www-authenticate: Bearer realm="https://registry.k8s.io/token",service="registry.k8s.io"
x-content-type-options: nosniff
x-request-id: 110d3402e8418fec9eb329bfe9df0c45
x-cloud-trace-context: 110d3402e8418fec9eb329bfe9df0c45
content-length: 24
date: Tue, 20 Oct 2026 21:03:37 GMT
server: Google Frontend
via: 1.1 google
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000



Configuramos una nueva maquina virtual

ls -l /etc/netplan

adminepn@svr-devops01:~$ vim /etc/netplan/00-installer-config.yaml
adminepn@svr-devops01:~$ sudo vim /etc/netplan/00-installer-config.yaml
[sudo: authenticate] Password:
adminepn@svr-devops01:~$




adminepn@svr-devops01:~$ sudo more /etc/netplan/00-installer-config.yaml
# This is the network config written by 'subiquity'
network:
version: 2
ethernets:
enp0s3:
dhcp4: false
dhcp6: false
addresses:
- 172.25.4.82/24
routes:
- to: default
via: 172.25.4.10
nameservers:
addresses:
- 8.8.8.8
- 1.1.1.1
adminepn@svr-devops01:~$


$ sudo netplan generate
adminepn@svr-devops01:~$ sudo netplan try --timeout 120

adminepn@svr-devops01:~$ sudo netplan try --timeout 120
Do you want to keep these settings?


Press ENTER before the timeout to accept the new configuration


Changes will revert in 108 seconds
Configuration accepted.

adminepn@svr-devops01:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             172.25.4.82/24 fe80::a00:27ff:fe26:629f/64




Instalamos paquetes 

adminepn@svr-devops01:~$ sudo apt install -y ca-certificates curl gnupg git jq wget unzip nano vim openssh-client iputils-ping dnsutils net-tools netcat-openbsd 

adminepn@svr-devops02:~$ command -v docker
adminepn@svr-devops02:~$ dpkg -l | grep -E 'docker|containerd|runc' || true

adminepn@svr-devops02:~$ sudo install -m 0755 -d /etc/apt/keyrings

adminepn@svr-devops02:~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
adminepn@svr-devops02:~$
adminepn@svr-devops02:~$ ls -la /etc/apt/keyrings/
total 12
drwxr-xr-x 2 root root 4096 Oct 21 09:22 .
drwxr-xr-x 9 root root 4096 Sep 27 03:47 ..
-rw-r--r-- 1 root root 3817 Oct 21 09:22 docker.asc
adminepn@svr-devops02:~$

adminepn@svr-devops02:~$ sudo chmod a+r /etc/apt/keyrings/docker.asc

adminepn@svr-devops02:~$ sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF


adminepn@svr-devops02:~$ more /etc/apt/sources.list.d/docker.sources
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: resolute
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc


adminepn@svr-devops02:~$ sudo apt update



adminepn@svr-devops02:~$ apt-cache policy docker-ce
docker-ce:
Instalados: (ninguno)
Candidato:  5:29.9.2-1~ubuntu.26.04~resolute
Tabla de versión:
5:29.9.2-1~ubuntu.26.04~resolute 500
500 https://download.docker.com/linux/ubuntu resolute/stable amd64 Packages

adminepn@svr-devops02:~$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

adminepn@svr-devops02:~$ sudo systemctl status docker --no-pager
● docker.service - Docker Application Container Engine
Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
Active: active (running) since Wed 2026-10-21 15:32:32 UTC; 2min 46s ago
Invocation: 8324eaf522384ff28c51e9a5e5a39413
TriggeredBy: ● docker.socket
Docs: https://docs.docker.com
Main PID: 1279 (dockerd)
Tasks: 10
Memory: 143.9M (peak: 144.2M)
CPU: 529ms
CGroup: /system.slice/docker.service
└─1279 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

oct 21 15:32:31 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:31.764491835Z" level=info msg="Restoring containers: start."
oct 21 15:32:31 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:31.814394669Z" level=info msg="Deleting nftables IPv4 rules" error="running nft: /dev/stdin:1:17-30: Error: Could not proc…n exit status 1"
oct 21 15:32:31 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:31.826025554Z" level=info msg="Deleting nftables IPv6 rules" error="running nft: /dev/stdin:1:18-31: Error: Could not proc…n exit status 1"
oct 21 15:32:32 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:32.284659411Z" level=info msg="Loading containers: done."
oct 21 15:32:32 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:32.293632948Z" level=info msg="Docker daemon" commit=0367cbe containerd-snapshotter=true storage-driver=overlayfs version=29.9.2
oct 21 15:32:32 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:32.293905395Z" level=info msg="Initializing buildkit"
oct 21 15:32:32 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:32.346908158Z" level=info msg="Completed buildkit initialization"
oct 21 15:32:32 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:32.352793415Z" level=info msg="Daemon has completed initialization"
oct 21 15:32:32 svr-devops02 dockerd[1279]: time="2026-10-21T15:32:32.352844512Z" level=info msg="API listen on /run/docker.sock"
oct 21 15:32:32 svr-devops02 systemd[1]: Started docker.service - Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.


adminepn@svr-devops02:~$ sudo systemctl enable --now docker


adminepn@svr-devops02:~$ sudo systemctl is-active docker
active
adminepn@svr-devops02:~$
adminepn@svr-devops02:~$ docker --version
Docker version 29.9.2, build a82e528

adminepn@svr-devops02:~$ sudo usermod -aG docker "$USER"
adminepn@svr-devops02:~$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


adminepn@svr-devops02:~$ docker compose version
Docker Compose version v5.6.1
adminepn@svr-devops02:~$
adminepn@svr-devops02:~$ docker buildx version
github.com/docker/buildx v0.38.1 0dbe87f36ed472fe36b6796c3f70b4210a731fc5
adminepn@svr-devops02:~$

adminepn@svr-devops02:~$ mkdir -p ~/laboratorios-k8s/preparacion

adminepn@svr-devops02:~$ cd ~/laboratorios-k8s/preparacion/

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ curl -fsSL https://dl.k8s.io/release/stable-1.37.txt
v1.37.1adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ K8S_VERSION=$(curl -fsSL https://dl.k8s.io/release/stable-1.37.txt)
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ echo $K8S_VERSION
v1.37.1

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ curl -fL --retry 3 -o kubectl "https://dl.k8s.io/release/${K8S_VERSION}/bin/linux/amd64/kubectl"
% Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
Dload  Upload  Total   Spent   Left   Speed
100 59.01M 100 59.01M   0      0 32.42M      0   00:01   00:01         31.34M
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ curl -fL --retry 3 -o kubectl.sha256 "https://dl.k8s.io/release/${K8S_VERSION}/bin/linux/amd64/kubectl.sha256"
% Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
Dload  Upload  Total   Spent   Left   Speed
100     64 100     64   0      0    195      0                              0

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ ls
kubectl  kubectl.sha256


adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ cat kubectl.sha256
65691ff77eb6fa44c908b77a1082c9f092c3b9733b5cefabec0d1104890e21a8adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sha256sum kubectl
65691ff77eb6fa44c908b77a1082c9f092c3b9733b5cefabec0d1104890e21a8  kubectl

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ echo "$(cat kubectl.sha256) kubectl" | sha256sum --check
kubectl: OK

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ command -v kubectl
/usr/local/bin/kubectl

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl version --client
Client Version: v1.37.1
Kustomize Version: v5.8.1

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ pwd
/home/adminepn/laboratorios-k8s/preparacion
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ curl -fL --retry 3 -o kind https://kind.sigs.k8s.io/dl/v0.33.0/kind-linux-amd64
% Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
Dload  Upload  Total   Spent   Left   Speed
100     97 100     97   0      0    116      0                              0
0      0   0      0   0      0      0      0           00:01              0
100 10.04M 100 10.04M   0      0  6.06M      0   00:01   00:01              0
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ ls -lh kind
-rw-rw-r-- 1 adminepn adminepn 11M Oct 21 16:44 kind
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo install -o root -g root -m 0755 kind /usr/local/bin/kind
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ command -v kind
/usr/local/bin/kind

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo docker info --format '{{.ServerVersion}}'
29.9.2

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kind get clusters
No kind clusters found.


adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo journalctl -u docker -n 10
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo journalctl -u kind -n 10



adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ apt-cache policy docker-ce

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kubectl get nodes
E1021 16:50:22.405364   20312 memcache.go:381] "Couldn't get current server API group list" err="Get \"http://localhost:8080/api?timeout=32s\": dial tcp 127.0.0.1:8080: connect: connection refused"
E1021 16:50:22.405686   20312 memcache.go:381] "Couldn't get current server API group list" err="Get \"http://localhost:8080/api?timeout=32s\": dial tcp 127.0.0.1:8080: connect: connection refused"
E1021 16:50:22.407553   20312 memcache.go:381] "Couldn't get current server API group list" err="Get \"http://localhost:8080/api?timeout=32s\": dial tcp 127.0.0.1:8080: connect: connection refused"
E1021 16:50:22.407834   20312 memcache.go:381] "Couldn't get current server API group list" err="Get \"http://localhost:8080/api?timeout=32s\": dial tcp 127.0.0.1:8080: connect: connection refused"
E1021 16:50:22.409916   20312 memcache.go:381] "Couldn't get current server API group list" err="Get \"http://localhost:8080/api?timeout=32s\": dial tcp 127.0.0.1:8080: connect: connection refused"
The connection to the server localhost:8080 was refused - did you specify the right host or port?
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$


Creamos la primera aplicacion

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl config use-context kind-curso-k8s
error: no context exists with the name: "kind-curso-k8s"

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl config get-contexts
CURRENT   NAME   CLUSTER   AUTHINFO   NAMESPACE




adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kind create cluster --name curso-k8s
Creating cluster "curso-k8s" ...
✓ Ensuring node image (kindest/node:v1.37.0) 🖼️
✓ Preparing nodes 📦
✓ Writing configuration 📜
✓ Starting control-plane 🕹️
✓ Installing CNI 🔌
✓ Installing StorageClass 💾
Set kubectl context to "kind-curso-k8s"
You can now use your cluster with:

kubectl cluster-info --context kind-curso-k8s

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/




adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kubectl cluster-info --context kind-curso-k8s
Kubernetes control plane is running at https://127.0.0.1:44587
CoreDNS is running at https://127.0.0.1:44587/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kubectl get nodes
NAME                      STATUS   ROLES           AGE   VERSION
curso-k8s-control-plane   Ready    control-plane   14m   v1.37.0
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$



adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kubectl config current-context
kind-curso-k8s
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$

Automaticamente ya se crea el contenedor 


adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                       NAMES
e4272c583a9e   kindest/node:v1.37.0   "/usr/local/bin/entr…"   15 minutes ago   Up 15 minutes   127.0.0.1:44587->6443/tcp   curso-k8s-control-plane
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$



adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kind export kubeconfig \
--name curso-k8s \
--kubeconfig /home/adminepn/.kube/config
Set kubectl context to "kind-curso-k8s"

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo chown adminepn:adminepn /home/adminepn/.kube/config
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ chmod 600 /home/adminepn/.kube/config
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         kind-curso-k8s   kind-curso-k8s   kind-curso-k8s
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl config use-context kind-curso-k8s
Switched to context "kind-curso-k8s".

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl get nodes
NAME                      STATUS   ROLES           AGE   VERSION
curso-k8s-control-plane   Ready    control-plane   21m   v1.37.0


adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl get pods -n kube-system
NAME                                              READY   STATUS    RESTARTS   AGE
coredns-559f6c778d-cr7vt                          1/1     Running   0          22m
coredns-559f6c778d-pnznn                          1/1     Running   0          22m
etcd-curso-k8s-control-plane                      1/1     Running   0          22m
kindnet-7cdfh                                     1/1     Running   0          22m
kube-apiserver-curso-k8s-control-plane            1/1     Running   0          22m
kube-controller-manager-curso-k8s-control-plane   1/1     Running   0          22m
kube-proxy-fpjbd                                  1/1     Running   0          22m
kube-scheduler-curso-k8s-control-plane            1/1     Running   0          22m
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$


adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ sudo kind get clusters
curso-k8s
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         kind-curso-k8s   kind-curso-k8s   kind-curso-k8s

Creamos el servicio 

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl create deployment portal-web --image=nginx:stable
deployment.apps/portal-web created

adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
portal-web   1/1     1            1           24s


Para ver el status de descarga
adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl rollout status deployment/portal-web --timeout=120s
deployment "portal-web" successfully rolled out


adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                      NOMINATED NODE   READINESS GATES
portal-web-7b7d54b4-d6z5n   1/1     Running   0          96s   10.244.0.5   curso-k8s-control-plane   <none>           <none>


adminepn@svr-devops02:~/laboratorios-k8s/preparacion$ kubectl port-forward deployments/portal-web 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80


si se deja abierto puedo abrir otra terminal 

adminepn@svr-devops02:~$ curl -I http://127.0.0.1:8080
HTTP/1.1 200 OK
Server: nginx/1.30.6
Date: Wed, 21 Oct 2026 17:25:59 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Fri, 02 Oct 2026 14:33:28 GMT
Connection: keep-alive
ETag: "6abfc0b8-380"
Accept-Ranges: bytes


