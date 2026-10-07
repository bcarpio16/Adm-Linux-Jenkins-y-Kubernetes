# Laboratorio 1 - Instalación y Configuración de Docker Nginx

Se va a realizar sobre ambas máquinas virtuales

## Máquina 1



```bash
$ hostnamectl
```
Resultado 

```text
Static hostname: svr-devops01
Icon name: computer-vm
Chassis: vm 🖴
Machine ID: b569804ece59442896212539cece95f2
Boot ID: 37390afed7384fe98d5c1bbdbe590a0e
Virtualization: oracle
Operating System: Ubuntu 26.04.1 LTS
Kernel: Linux 7.0.0-34-generic
Architecture: x86-64
Hardware Vendor: innotek GmbH
Hardware Model: VirtualBox
Hardware Version: 1.2
Firmware Version: VirtualBox
Firmware Date: Fri 2006-12-01
Firmware Age: 19y 10month 5d
adminepn@svr-devops01:~$
```
<p style="text-align: justify;">
Verificamos la versión del sistema operativo
</p>

```bash
lsb_release -a 2>/dev/null || cat /etc/os-release
```
Resultado 

```text
Distributor ID: Ubuntu
Description:    Ubuntu 26.04.1 LTS
Release:        26.04
Codename:       resolute
```
```bash
ip -br addr
```
```text
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             172.25.4.81/24 fe80::a00:27ff:fe85:e384/64
```
```bash
uname -m
```

```text
x86_64
```
<p style="text-align: justify;">
Retiramos paquetes de dockers no oficiales 
</p>

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 docker-buildx podman-docker containerd runc; 
do sudo apt remove -y "$pkg" 2>/dev/null || true
 done
```

Resultado 
```text
El paquete «docker.io» no está instalado, no se eliminará
El paquete indicado a continuación se instaló de forma automática y ya no es necesario:
  networkd-dispatcher
Utilice «sudo apt autoremove» para eliminarlo.

Resumen:
  Actualizando: 0, Instalando 0, Eliminando: 0, no actualizando: 6
El paquete «docker-doc» no está instalado, no se eliminará
El paquete indicado a continuación se instaló de forma automática y ya no es necesario:
  networkd-dispatcher
Utilice «sudo apt autoremove» para eliminarlo.
...
```

Luego realizamos una actualización del sistema operativo

```bash
sudo apt update
```

Instalamos certificados 

```bash
sudo apt install -y ca-certificates curl
```
Permisos de lectura y ejecucion 

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

Repositorios

vim /etc/apt/sources.list.d/docker.sources

agragamos lo siguiente 

Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: resolute
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc

sudo apt update

Instalamos 

sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker

sudo systemctl status docker


● docker.service - Docker Application Container Engine
Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
Active: active (running) since Wed 2026-10-07 11:17:54 -05; 6min ago
Invocation: 6f1dc58b37f84edcb32d3b348f0d967c
TriggeredBy: ● docker.socket
Docs: https://docs.docker.com
Main PID: 566481 (dockerd)
Tasks: 10
Memory: 26M (peak: 27.3M)
CPU: 1.632s
CGroup: /system.slice/docker.service
└─566481 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock


status containerd.service

● containerd.service - containerd container runtime
Loaded: loaded (/usr/lib/systemd/system/containerd.service; enabled; preset: enabled)
Active: active (running) since Wed 2026-10-07 11:17:51 -05; 7min ago
Invocation: 62c79570bc3f408090ca45752a6b92e4
Docs: https://containerd.io
Main PID: 566320 (containerd)
Tasks: 9
Memory: 13.4M (peak: 13.4M)
CPU: 1.887s
CGroup: /system.slice/containerd.service
└─566320 /usr/bin/containerd


sudo docker version

Client: Docker Engine - Community
Version:           29.8.2
API version:       1.56
Go version:        go1.26.8
Git commit:        7fc2dff
Built:             Wed Sep 30 19:32:32 2026
OS/Arch:           linux/amd64
Context:           default

Server: Docker Engine - Community
Engine:
Version:          29.8.2
API version:      1.56 (minimum version 1.40)
Go version:       go1.26.8
Git commit:       8af9fe3
Built:            Wed Sep 30 19:32:32 2026
OS/Arch:          linux/amd64
Experimental:     false
containerd:
Version:          v2.3.6
GitCommit:        ee2735368117d2eb259779949d5e75cdafec9761
runc:
Version:          1.5.1
GitCommit:        v1.5.1-0-g8f2685a4
docker-init:
Version:          0.19.0
GitCommit:        de40ad0


sudo docker info
Client: Docker Engine - Community
Version:    29.8.2
Context:    default
Debug Mode: false
Plugins:
buildx: Docker Buildx (Docker Inc.)
Version:  v0.37.1
Path:     /usr/libexec/docker/cli-plugins/docker-buildx
compose: Docker Compose (Docker Inc.)
Version:  v5.6.0
...

~$ sudo docker run --rm hello-world
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: pull access denied for hello-world, repository does not exist or may require 'docker login'

Run 'docker run --help' for more information
adminepn@svr-devops01:~$


if [ -f /etc/docker/daemon.json ]; then
sudo cp /etc/docker/daemon.json /etc/docker/daemon.json.bak.$(date +%Y%m%d%H%M%S)
fi


$ echo $(date +%Y%m%d%H%M%S)
20261007113555

sudo vim /etc/docker/daemon.json

{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "live-restore": true
}



sudo systemctl restart docker



docker info --format '{{.LoggingDriver}}'


$ getent group docker
docker:x:981:
adminepn@svr-devops01:~$ ls -l /var/run/docker.sock
srw-rw---- 1 root docker 0 Oct  7 11:56 /var/run/docker.sock

sudo newgrp docker


docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


docker run --rm hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete
d5e71e642bf5: Download complete
...


mkdir -p ./u2-docker
cd u2-docker/



docker pull nginx:alpine
alpine: Pulling from library/nginx
e72112c14215: Pull complete
e2de96513ba9: Pull complete
d9aae54b5831: Pull complete
9a9a644fdd6a: Pull complete
64c8194480fe: Pull complete
e76228b47809: Pull complete
6c53d0b2a666: Pull complete
745dfb2690dd: Pull complete
0deea27e58d9: Download complete
d54d3939625e: Download complete
Digest: sha256:df221db836e1754089190208cee7eeda94f233197056426eda74a43ab1abeac2
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine


docker image ls nginx
i Info →   U  In Use
IMAGE          ID             DISK USAGE   CONTENT SIZE   EXTRA
nginx:alpine   df221db836e1       94.4MB         27.2MB

sudo docker run -d --name portal-demo -p 8080:80 nginx:alpine
5b19b6e580aaa876c2a3ceced77bbe3a10f6f357ae729f391a8b20b5ebb796d3


sudo docker ps --filter name=portal-demo
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                     NAMES
5b19b6e580aa   nginx:alpine   "/docker-entrypoint.…"   10 minutes ago   Up 10 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   portal-demo



adminepn@svr-devops01:~$ ss -ltnp
State     Recv-Q    Send-Q        Local Address:Port         Peer Address:Port    Process
LISTEN    0         4096                0.0.0.0:22                0.0.0.0:*
LISTEN    0         128               127.0.0.1:6010              0.0.0.0:*
LISTEN    0         4096             127.0.0.54:53                0.0.0.0:*
LISTEN    0         4096                0.0.0.0:8080              0.0.0.0:*
LISTEN    0         5                   0.0.0.0:8089              0.0.0.0:*
LISTEN    0         4096          127.0.0.53%lo:53                0.0.0.0:*
LISTEN    0         4096                   [::]:22                   [::]:*
LISTEN    0         128                   [::1]:6010                 [::]:*
LISTEN    0         4096                   [::]:8080                 [::]:*


adminepn@svr-devops01:~$ curl -I http://localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.31.6
Date: Wed, 07 Oct 2026 17:48:51 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Tue, 15 Sep 2026 14:18:52 GMT
Connection: keep-alive
ETag: "6aa953cc-380"
Accept-Ranges: bytes

Docker asigna una ip 

dminepn@svr-devops01:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             172.25.4.80/24 fe80::a00:27ff:fea1:a2cb/64
docker0          UP             172.17.0.1/16 fe80::8c86:6cff:feaa:3a6d/64
vethca6f893@if2  UP             fe80::30b1:8cff:fe1c:1193/64


LOGS 

sadminepn@svr-devops01:~$ sudo docker logs --tail 5 portal-demo
172.17.0.1 - - [07/Oct/2026:17:48:51 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.18.0" "-"
172.25.4.58 - - [07/Oct/2026:17:51:21 +0000] "\x16\x03\x01\x07>\x01\x00\x07:\x03\x03O\x22\x851\xCF\x05];\xE87\xA86\x18mGou\xA7k\x94\x97\xC76\x81\xBA\x17\xF79\x9B\x14c\x03 \xA9\xCBP\x81PU\xB2\xED\xD5\x0C\xB4\xEA\x02o\xB6%\x9AA\xAC@\xD7\x9E\xD4U\x0F&\x1DV\xF1\x05hk\x00\x1E\x13\x01\x13\x03\x13\x02\xC0+\xC0/\xCC\xA9\xCC\xA8\xC0,\xC00\xC0\x13\xC0\x14\x00\x9C\x00\x9D\x00/\x005\x01\x00\x06\xD3\x00\x17\x00\x00\xFF\x01\x00\x01\x00\x00" 400 157 "-" "-" "-"
172.25.4.58 - - [07/Oct/2026:17:51:32 +0000] "GET / HTTP/1.1" 200 896 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:157.0) Gecko/20100101 Firefox/157.0" "-"
2026/10/07 17:51:32 [error] 32#32: *3 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.25.4.58, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "172.25.4.80:8080", referrer: "http://172.25.4.80:8080/"
172.25.4.58 - - [07/Oct/2026:17:51:32 +0000] "GET /favicon.ico HTTP/1.1" 404 153 "http://172.25.4.80:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:157.0) Gecko/20100101 Firefox/157.0" "-"


sudo docker inspect portal-demo --format 'Nombre={{.Name}} IP={{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}} Estado={{.State.Status}}'

Nombre=/portal-demo IP=172.17.0.2 Estado=running



docker exec -it portal-demo sh

/ #
/ # ls -l /usr/share/nginx/
total 4
drwxr-xr-x    2 root     root          4096 Sep 22 21:19 html
/ #
/ #




sudo docker stop portal-demo
portal-demo


sudo docker ps -a --filter name=portal-demo
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                          PORTS     NAMES
5b19b6e580aa   nginx:alpine   "/docker-entrypoint.…"   31 minutes ago   Exited (0) About a minute ago             portal-demo


Eliminamos el contenedor 

sudo docker rm -f portal-demo
portal-demo


sudo docker image ls nginx
i Info →   U  In Use
IMAGE          ID             DISK USAGE   CONTENT SIZE   EXTRA
nginx:alpine   df221db836e1       94.4MB         27.2MB
adminepn@svr-devops01:~$


Nunca detuvimos el contenedor sin embargo se elimino las conexiones al puerto 8080

ss -ltpn
State     Recv-Q    Send-Q         Local Address:Port         Peer Address:Port    Process
LISTEN    0         4096                 0.0.0.0:22                0.0.0.0:*
LISTEN    0         128                127.0.0.1:6010              0.0.0.0:*
LISTEN    0         4096              127.0.0.54:53                0.0.0.0:*
LISTEN    0         5                    0.0.0.0:8089              0.0.0.0:*
LISTEN    0         4096           127.0.0.53%lo:53                0.0.0.0:*
LISTEN    0         4096                    [::]:22                   [::]:*
LISTEN    0         128                    [::1]:6010                 [::]:*
adminepn@svr-devops01:~$

### Comunicacion y Persistencia

Creamos la red

sudo docker network create --driver bridge net-institucional
50ee45223f9fccf70890bd68f678268a5b57e513526b39b3de29a1843656ff5f

sudo docker network ls
NETWORK ID     NAME                DRIVER    SCOPE
f9b575de6881   bridge              bridge    local
5725cce1800b   host                host      local
50ee45223f9f   net-institucional   bridge    local
1b8514fbaf18   none                null      l

sudo docker network inspect net-institucional

```json
[
    {
        "Name": "net-institucional",
        "Id": "50ee45223f9fccf70890bd68f678268a5b57e513526b39b3de29a1843656ff5f",
        "Created": "2026-10-07T13:53:19.011003375-05:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Options": {},
        "Labels": {},
        "Containers": {},
        "Status": {
            "IPAM": {
                "Subnets": {
                    "172.18.0.0/16": {
                        "IPsInUse": 3,
                        "DynamicIPsAvailable": 65533
                    }
                }
            }
        }
    }
]
```


sudo docker run -d --name portal-demo --network net-institucional nginx:alpine
e75b158d945f774f9a3252af3d2324a5c62d1a3a0ee17543c04397a93b4c0fd5


sudo docker ps -a --filter name=portal-demo

sudo docker ps -a --filter name=portal-demo
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
e75b158d945f   nginx:alpine   "/docker-entrypoint.…"   14 seconds ago   Up 13 seconds   80/tcp    portal-demo

sudo docker run --rm --network net-institucional alpine:3.20 sh -c "wget -qO- http://portal.net"


Unable to find image 'alpine:3.20' locally
3.20: Pulling from library/alpine
25f1d6b1951a: Pull complete
3a030ca3f633: Download complete
d4050d56ebf2: Download complete
Digest: sha256:d9e853e87e55526f6b2917df91a2115c36dd7c696a35be12163d44e6e2a4b6bc
Status: Downloaded newer image for alpine:3.20
wget: bad address '..'
adminepn@svr-devops01:~$

a pesar de borrar el contenedor sigue ejecutandose 


sudo docker ps -a --filter name=portal-demo
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
e75b158d945f   nginx:alpine   "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes   80/tcp    portal-demo


para esto se debe parar el servicio 


sudo docker stop  portal-demo
portal-demo

El volumen sigue funcionando aunq se borre la red, el contenedor, es persistente
sudo docker volume create vol-portal
vol-portal

Verificamos

sudo docker volume inspect vol-portal
```jason
 sudo docker volume inspect vol-portal
[
    {
        "CreatedAt": "2026-10-07T14:15:26-05:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/vol-portal/_data",
        "Name": "vol-portal",
        "Options": null,
        "Scope": "local"
    }
]
```


Docker almacena los volumenes en 

sudo ls /var/lib/docker/volumes
backingFsBlockDev  metadata.db  vol-portal

Se recomienda crear un nuevo disco separado para los volumenes y se monta en e
/var/lib/docker/volumes



adminepn@svr-devops01:~$ df
Filesystem                        1K-blocks    Used Available Use% Mounted on
tmpfs                               1520452    1120   1519332   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  29751268 8780240  19434416  32% /
tmpfs                               3801124       0   3801124   0% /dev/shm
none                                   1024       0      1024   0% /run/credentials/systemd-journald.service
tmpfs                               3801124       0   3801124   0% /tmp
/dev/sda2                           1992552  187316   1683996  11% /boot
none                                   1024       0      1024   0% /run/credentials/getty@tty1.service
tmpfs                                760224       8    760216   1% /run/user/1000
none                                   1024       0      1024   0% /run/credentials/systemd-networkd.service
none                                   1024       0      1024   0% /run/credentials/systemd-resolved.service


