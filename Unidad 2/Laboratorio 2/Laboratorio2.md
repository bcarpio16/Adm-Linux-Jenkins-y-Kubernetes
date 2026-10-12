# Laboratorio 2 - Docker con imagen propia

## 1. Configuración inicial de Docker

### Verificación de las llaves de Docker

<p style="text-align: justify;">
Verificamos y descargamos la llave GPG oficial de Docker para permitir que APT valide los paquetes provenientes del repositorio oficial.
</p>

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

### Permisos de la llave

<p style="text-align: justify;">
Asignamos permisos de lectura a la llave para que pueda ser utilizada por el sistema de gestión de paquetes.
</p>

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
---

## 2. Configuración del repositorio de Docker

<p style="text-align: justify;">
Creamos o verificamos la configuración del repositorio oficial de Docker para Ubuntu.
</p>

```bash
sudo vim /etc/apt/sources.list.d/docker.sources
```

Agregamos la siguiente configuración:

```text
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: resolute
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
```

La configuración indica a APT desde dónde obtener los paquetes de Docker y qué llave utilizar para verificar su autenticidad.

---

## 3. Instalación y verificación de Docker

<p style="text-align: justify;">
Instalamos Docker Engine, su cliente, containerd y el complemento Buildx. Posteriormente habilitamos el servicio para que se inicie automáticamente y verificamos su estado.
</p>

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin
sudo systemctl enable --now docker
sudo systemctl status docker --no-pager
sudo systemctl status containerd --no-pager
docker version
```
### Descripción de los comandos

- **docker-ce:** instala Docker Engine.
- **docker-ce-cli:** instala el cliente de línea de comandos de Docker.
- **containerd.io:** proporciona el runtime utilizado para ejecutar contenedores.
- **docker-buildx-plugin:** permite utilizar funcionalidades avanzadas para la construcción de imágenes.
- **systemctl enable --now docker:** habilita e inicia Docker inmediatamente.
- **systemctl status:** permite verificar el estado de los servicios.

---

## 4. Creación de la aplicación

### Creación del directorio del proyecto

<p style="text-align: justify;">
Creamos el directorio donde se almacenará el proyecto de la API que posteriormente será convertida en una imagen Docker.
</p>


```bash
cd /home/adminepn/u2-docker
cd u2-docker
sudo mkdir api-institucional
```

### Creación de la aplicación Flask

<p style="text-align: justify;">
Creamos el archivo `app.py`, que contiene una API sencilla desarrollada con Flask.
</p>

```bash
sudo vim app.py
```

### Contenido de `app.py`


```python
from flask import Flask, jsonify
import os
import socket

app = Flask(__name__)

@app.get("/")
def index():
    return jsonify({
        "servicio": "api-institucional",
        "version": "1.0",
        "host": socket.gethostname(),
        "entorno": os.getenv("APP_ENV", "laboratorio")
    })

@app.get("/health")
def health():
    return jsonify({"status": "ok"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
### Descripción

La aplicación proporciona dos endpoints:

- **`/`**: muestra información básica del servicio, versión, hostname y entorno.
- **`/health`**: permite comprobar el estado de salud de la aplicación.
- **`0.0.0.0:5000`**: permite que la aplicación escuche conexiones desde las interfaces disponibles del contenedor.

---

## 5. Archivo `.dockerignore`

<p style="text-align: justify;">
Definimos los archivos y directorios que no deben incluirse en el contexto utilizado para construir la imagen Docker.
</p>

```bash
sudo vim .dockerignore
```
Agregamos lo siguiente 

### Contenido

```text
.git
__pycache__
*.pyc
.env
*.log
```

### Descripción

El archivo `.dockerignore` permite evitar que archivos innecesarios o sensibles sean enviados al contexto de construcción de Docker.

- **`.git`**: evita copiar información del repositorio Git.
- **`__pycache__`**: excluye archivos temporales de Python.
- **`*.pyc`**: excluye archivos compilados de Python.
- **`.env`**: evita incluir archivos que pueden contener variables sensibles.
- **`*.log`**: evita copiar archivos de registro.

---

## 6. Creación del Dockerfile

<p style="text-align: justify;">
Creamos el archivo `Dockerfile`, que contiene las instrucciones necesarias para construir la imagen de nuestra aplicación.
</p>


```bash
sudo vim Dockerfile
```

### Contenido del `Dockerfile`

```dockerfile
FROM python:3.12-alpine

LABEL org.opencontainers.image.title="api-institucional-demo"
LABEL org.opencontainers.image.description="API de laboratorio para despliegues Docker"

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt \
    && addgroup -S appgroup \
    && adduser -S appuser -G appgroup

COPY app.py .

USER appuser
EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s --retries=3 CMD wget -qO- http://127.0.0.1:5000/health || exit 1

CMD ["python", "app.py"]
```
### Descripción

El `Dockerfile` define el proceso de construcción de la imagen:

- **`FROM`**: utiliza Python 3.12 sobre Alpine Linux.
- **`LABEL`**: agrega información descriptiva a la imagen.
- **`ENV`**: configura variables de entorno de Python.
- **`WORKDIR`**: establece `/app` como directorio de trabajo.
- **`COPY`**: copia archivos al interior de la imagen.
- **`RUN`**: instala las dependencias y crea un usuario sin privilegios.
- **`USER`**: ejecuta la aplicación utilizando `appuser`.
- **`EXPOSE`**: documenta el puerto utilizado por la aplicación.
- **`HEALTHCHECK`**: comprueba periódicamente el estado de la API.
- **`CMD`**: define el comando utilizado para iniciar la aplicación.

> **Nota:** el `Dockerfile` utiliza `requirements.txt`, por lo que este archivo debe existir dentro del directorio `api-institucional` antes de ejecutar la construcción de la imagen.

---

## 7. Construcción de la imagen Docker

<p style="text-align: justify;">
Construimos una imagen Docker utilizando el `Dockerfile` ubicado en el directorio actual.
</p>

```bash
sudo docker build -t api-institucional:1.0 .
```

### Descripción

- **`docker build`**: construye una imagen Docker.
- **`-t`**: asigna un nombre y una etiqueta a la imagen.
- **`api-institucional:1.0`**: nombre y versión de la imagen.
- **`.`**: indica que el contexto de construcción es el directorio actual.

### Resultado 

```text
[+] Building 2.4s (4/9)                                                                      docker:default
=> [internal] load build definition from Dockerfile                                                   0.0s
=> => transferring dockerfile: 601B                                                                   0.0s
=> [internal] load metadata for docker.io/library/python:3.12-alpine                                  1.6s
```
--- 

## 8. Verificación de la imagen

<p style="text-align: justify;">
Verificamos que la imagen "api-institucional" haya sido creada correctamente.
</p>

```bash
sudo docker image ls api-institucional
```

### Resultado

```text
                                                                                        i Info →   U  In Use
IMAGE                   ID             DISK USAGE   CONTENT SIZE   EXTRA
api-institucional:1.0   2a24d42d2f21       90.3MB         22.3MB
```
También podemos consultar el historial de capas de la imagen:

```bash
sudo docker history api-institucional:1.0 --no-trunc | head
```

### Resultado

```text 
IMAGE                                                                     CREATED              CREATED BY           SIZE      COMMENT
sha256:2a24d42d2f21fd9edc65d4e88c9dff1db4b42929a2e9e4c371e4d226264faeeb   About a minute ago   CMD ["python" 
```

Este comando permite revisar las diferentes capas utilizadas para construir la imagen.

---

## 9. Ejecución del contenedor

<p style="text-align: justify;">
Creamos y ejecutamos un contenedor utilizando la imagen construida anteriormente.
</p>


```bash
sudo docker run -d --name api-institucional-demo -p 5001:5000 -e APP_NEW=labunidad2 api-institucional:1.0
```
### Descripción

- **`-d`**: ejecuta el contenedor en segundo plano.
- **`--name`**: asigna un nombre al contenedor.
- **`-p 5001:5000`**: publica el puerto `5000` del contenedor en el puerto `5001` del servidor.
- **`-e`**: establece una variable de entorno.
- **`api-institucional:1.0`**: imagen utilizada para crear el contenedor.

### Resultado

```text
2e0c58ae740311ceae6b831e34011375bd55116826163ce64aad0687e2105bb0
```

---

## 10. Prueba de la API

### Endpoint principal

```bash
curl http://localhost:5001/
```

### Resultado

```text
{"entorno":"laboratorio","host":"2e0c58ae7403","servicio":"api-institucional","version":"1.0"}
```

### Endpoint de salud

```bash
curl http://localhost:5001/health
```

```resultado
{"status":"ok"}
```
El endpoint `/health` permite comprobar rápidamente si la aplicación está respondiendo correctamente.

---

## 11. Verificación del estado del contenedor

<p style="text-align: justify;">
Esperamos unos segundos para que se ejecute el `HEALTHCHECK` y posteriormente verificamos el estado de salud y el usuario utilizado por el contenedor.
</p>


```bash
sleep 15
sudo docker inspect --format '{{.State.Health.Status}}' api-institucional-demo
sudo docker inspect --format 'Usuario={{.Config.User}}' api-institucional-demo
```

```text
healthy
Usuario=appuser
```

El resultado confirma que el contenedor se encuentra saludable y que la aplicación se ejecuta con el usuario `appuser`, en lugar de utilizar `root`.

---

## 12. Verificación del contenedor

```bash
sudo docker ps -a --filter name=api-institucional-demo
```

### Resultado

```text
CONTAINER ID   IMAGE                   COMMAND           CREATED         STATUS                   PORTS                                         NAMES
2e0c58ae7403   api-institucional:1.0   "python app.py"   4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   api-institucional-demo
```

La salida permite verificar el estado del contenedor, la imagen utilizada, los puertos publicados y el estado del `HEALTHCHECK`.

---

## 13. Verificación del endpoint de salud

```bash
curl http://localhost:5001/health
```

### Resultado
```text
{"status":"ok"}
```
---

## 14. Verificación de la API y logs

<p style="text-align: justify;">
Realizamos nuevamente una prueba de la API y consultamos los logs del contenedor para comprobar las solicitudes recibidas por la aplicación.
</p>

```bash
curl http://localhost:5001
```

### Resultado

```text
{"entorno":"laboratorio","host":"2e0c58ae7403","servicio":"api-institucional","version":"1.0"}
```

Consultamos los registros del contenedor:

```bash
sudo docker logs api-institucional-demo
```

### Resultado

```text
* Serving Flask app 'app'
* Debug mode: off
  WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
* Running on all addresses (0.0.0.0)
* Running on http://127.0.0.1:5000
* Running on http://172.17.0.2:5000
  Press CTRL+C to quit
  172.17.0.1 - - [07/Oct/2026 21:20:59] "GET / HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:21:06] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:21:36] "GET /health HTTP/1.1" 200 -
  172.17.0.1 - - [07/Oct/2026 21:21:41] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:22:07] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:22:38] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:24:12] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:24:42] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:25:12] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:25:44] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:26:14] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:26:45] "GET /health HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:27:15] "GET /health HTTP/1.1" 200 -
  172.17.0.1 - - [07/Oct/2026 21:27:27] "GET /health HTTP/1.1" 200 -
  172.17.0.1 - - [07/Oct/2026 21:27:31] "GET / HTTP/1.1" 200 -
  127.0.0.1 - - [07/Oct/2026 21:27:46] "GET /health HTTP/1.1" 200 -
```
Los logs permiten comprobar que Flask inició correctamente y que las solicitudes a `/` y `/health` fueron procesadas con código HTTP `200`.

---

## 15. Verificación del contenedor

```bash
sudo docker ps -a --filter name=api-institucional-demo
```

### Resultado

```text
CONTAINER ID   IMAGE                   COMMAND           CREATED         STATUS                   PORTS                                         NAMES
2e0c58ae7403   api-institucional:1.0   "python app.py"   9 minutes ago   Up 9 minutes (healthy)   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   api-institucional-demo
```

---

## 16. Prueba de conectividad interna

<p style="text-align: justify;">
Intentamos acceder al endpoint de salud directamente desde el interior del contenedor utilizando `wget`.
</p>

```bash
sudo docker exec api-institucional-demo sh -c "wget -qO - http://localhost:5000/health"
```
### Resultado obtenido

```text
wget: can't connect to remote host: Connection refused
```

Este resultado corresponde a la prueba realizada durante el laboratorio. Posteriormente se debe revisar la causa de la conexión rechazada si se requiere validar el acceso interno mediante `wget`.

---

STACK multiconector de contenedores 


cd /home/adminepn/u2-docker

sudo mkdir -p ~/u2-docker/compose-institucional/api
sudo mkdir -p ~/u2-docker/compose-institucional/web


cd compose-institucional

vim web/index.html

<h1>Stack institucional DMQ con Docker Compose</h1>
<p>Servicio web ejecutado en Nginx.</p>
<p>API disponible en el puerto 5002 del host.</p>



sudo vim compose.yaml

```text
name: stack-institucional

services:
  api:
    build: ./api
    image: api-institucional:compose
    environment:
      APP_ENV: compose-lab
    ports:
      - "5002:5000"
    networks:
      - front
      - back

  web:
    image: nginx:alpine
    ports:
      - "8082:80"
    volumes:
      - ./web/index.html:/usr/share/nginx/html/index.html:ro
    networks:
      - front
    depends_on:
      - api

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app_dev_2026
      POSTGRES_DB: institucional
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - back

networks:
  front:
  back:

volumes:
  db_data:

```

validamos el .yaml 


docker compose config
name: stack-institucional
services:
api:

Copiamos el dockerfile 

cp -a ~/u2-docker/api-institucional/. ~/u2-docker/compose-institucional/api/


levantarmos el servicio 

sudo docker compose up -d --build
[+] up 0/1
⠋ Image postgres:16-alpine Pulling                       

sudo docker compose up -d --build
[+] Building 2.0s (12/12) FINISHED
=> [internal] load local bake definitions
=> => reading from stdin 566B


sudo docker compose exec -T db pg_isready -U app -d institucional

sudo docker network ls | grep stack-institucional

sudo docker volume ls

sudo docker compose logs --tail 2



sudo docker compose stop

sudo docker compose start

sudo docker compose down 

verificar puertos 

sudo docker ps --format "table {{.Names}}   {{.Ports}}"

sudo docker volume ls | grep db_data
local     stack-institucional_db_data


creamos el registry local 

ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             172.25.4.80/24 fe80::a00:27ff:fea1:a2cb/64
docker0          UP             172.17.0.1/16 fe80::8c86:6cff:feaa:3a6d/64
br-144bccab37cd  DOWN           192.168.0.1/24
veth9ab7cff@if2  UP             fe80::43a:16ff:fe4a:94b1/64
br-0675340bf040  UP             192.168.1.1/24 fe80::1c83:72ff:fe74:30ac/64
br-cb6af62f3024  UP             192.168.2.1/24 fe80::6815:a9ff:fe49:1b19/64
veth257aa32@if2  UP             fe80::40b2:70ff:fe8f:10ab/64
vethed838be@if2  UP             fe80::c47a:cfff:fec8:f705/64
vethbee2e22@if3  UP             fe80::2882:aaff:fed3:b280/64
vethac30287@if2  UP             fe80::1cf3:28ff:fe1a:a77f/64


export REGISTRY_HOST=172.25.4.80

echo $REGISTRY_HOST
172.25.4.80

docker rm -f registry-local 2>/dev/null || true
true

sudo mkdir -p /srv/docker-registry
sudo chown root:root /srv/docker-registry


sudo docker run -d --name registry-local --restart unless-stopped -p 5000:5000 -v /srv/docker-registry:/var/lib/registry registry:2

sudo docker ps --filter name=registry-local

CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                                         NAMES
33a5ec182848   registry:2   "/entrypoint.sh /etc…"   35 seconds ago   Up 34 seconds   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   registry-local


curl http://localhost:5000/v2/_catalog
{"repositories":[]}


sudo python3 - <<'PY'
import json, os, pathlib
p = pathlib.Path("/etc/docker/daemon.json")
cfg = json.loads(p.read_text()) if p.exists() and p.read_text().strip() else {}
cfg.setdefault("log-driver", "json-file")
cfg.setdefault("log-opts", {"max-size": "10m", "max-file": "3"})
cfg.setdefault("live-restore", True)
cfg["insecure-registries"] = [os.environ["REGISTRY_HOST"] + ":5000"]
p.write_text(json.dumps(cfg, indent=2) + "\n")
PY

Resultado

Traceback (most recent call last):
File "<stdin>", line 7, in <module>
File "<frozen os>", line 709, in __getitem__
KeyError: 'REGISTRY_HOST'

08-10-2026

Para solventar este error vamos a ejecutar lo siguiente 

sudo mkdir -p /srv/docker-registry

sudo chown root:root /srv/docker-registry

docker rm -f registry-local 2>/dev/null || true

sudo docker run -d --name registry-local --restart unless-stopped -p 5000:5000 -v /srv/docker-registry:/var/lib/registry registry:2
docker: Error response from daemon: Conflict. The container name "/registry-local" is already in use by container "33a5ec182848ba07bdaaf358a922cddc9d77cbefa72405397d51f663c7414941". You have to remove (or rename) that container to be able to reuse that name.

Run 'docker run --help' for more information
adminepn@svr-devops01:~$

export REGISTRY_HOST=172.25.4.80

sudo rm -rf /srv/docker-registry

$ sudo docker ps --filter name=registry-local
CONTAINER ID   IMAGE        COMMAND                  CREATED        STATUS        PORTS                                         NAMES
33a5ec182848   registry:2   "/entrypoint.sh /etc…"   21 hours ago   Up 21 hours   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   registry-local

$ sudo docker stop registry-local
registry-local

$ sudo docker ps --filter name=registry-local
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
adminepn@svr-devops01:~$


adminepn@svr-devops01:~$ curl http://127.0.0.1:5000/v2/
{}adminepn@svr-devops01:~$
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$ sudo docker ps --filter name=registry-local
CONTAINER ID   IMAGE        COMMAND                  CREATED        STATUS          PORTS                                         NAMES
33a5ec182848   registry:2   "/entrypoint.sh /etc…"   21 hours ago   Up 43 seconds   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   registry-local
adminepn@svr-devops01:~$



ejecutamos lo siguiente 

sudo env REGISTRY_HOST="$REGISTRY_HOST" python3 - <<'PY'
import json, os, pathlib

p = pathlib.Path("/etc/docker/daemon.json")

cfg = json.loads(p.read_text()) if p.exists() and p.read_text().strip() else {}

cfg.setdefault("log-driver", "json-file")
cfg.setdefault("log-opts", {"max-size": "10m", "max-file": "3"})
cfg.setdefault("live-restore", True)

cfg["insecure-registries"] = [
os.environ["REGISTRY_HOST"] + ":5000"
]

p.write_text(json.dumps(cfg, indent=2) + "\n")
PY


luego 

adminepn@svr-devops01:~$ sudo mkdir -p /srv/docker-registry/

adminepn@svr-devops01:~$ sudo chown root:root /srv/docker-registry/


adminepn@svr-devops01:~$ cd /srv/docker-registry/
adminepn@svr-devops01:/srv/docker-registry$
adminepn@svr-devops01:/srv/docker-registry$
adminepn@svr-devops01:/srv/docker-registry$ sudo python3 -m json.tool /etc/docker/daemon.json
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "live-restore": true,
    "default-address-pools": [
        {
            "base": "192.168.0.0/16",
            "size": 24
        }
    ],
    "insecure-registries": [
        "172.25.4.80:5000"
    ]
}


sudo systemctl restart docker


sudo docker tag api-institucional:1.0 ${REGISTRY_HOST}:5000/api-institucional:1.0




adminepn@svr-devops01:/srv/docker-registry$ sudo docker ps -a --filter name=api-institucional-demo
CONTAINER ID   IMAGE                   COMMAND           CREATED        STATUS                  PORTS                                         NAMES
2e0c58ae7403   api-institucional:1.0   "python app.py"   23 hours ago   Up 23 hours (healthy)   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   api-institucional-demo
adminepn@svr-devops01:/srv/docker-registry$
adminepn@svr-devops01:/srv/docker-registry$ sudo docker ps -a
CONTAINER ID   IMAGE                       COMMAND                  CREATED        STATUS                    PORTS                                         NAMES
33a5ec182848   registry:2                  "/entrypoint.sh /etc…"   21 hours ago   Up 7 minutes              0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   registry-local
0a594da3d3ce   nginx:alpine                "/docker-entrypoint.…"   22 hours ago   Up 22 hours               0.0.0.0:8082->80/tcp, [::]:8082->80/tcp       stack-institucional-web-1
af4a2eedc7af   postgres:16-alpine          "docker-entrypoint.s…"   22 hours ago   Up 22 hours               5432/tcp                                      stack-institucional-db-1
cdd275b46320   api-institucional:compose   "python app.py"          22 hours ago   Up 22 hours (healthy)     0.0.0.0:5002->5000/tcp, [::]:5002->5000/tcp   stack-institucional-api-1
2e0c58ae7403   api-institucional:1.0       "python app.py"          23 hours ago   Up 23 hours (healthy)     0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   api-institucional-demo
e75b158d945f   nginx:alpine                "/docker-entrypoint.…"   25 hours ago   Exited (0) 25 hours ago                                                 portal-demo
adminepn@svr-devops01:/srv/docker-registry$


adminepn@svr-devops01:/srv/docker-registry$ sudo docker info | grep -A3 "Insecure Registries"
Insecure Registries:
172.25.4.80:5000
127.0.0.0/8
::1/128


$ sudo docker image ls
i Info →   U  In Use
IMAGE                                    ID             DISK USAGE   CONTENT SIZE   EXTRA
172.25.4.80:5000/api-institucional:1.0   2a24d42d2f21       90.3MB         22.3MB    U
alpine:3.20                              d9e853e87e55       12.2MB         3.71MB
api-institucional:1.0                    2a24d42d2f21       90.3MB         22.3MB    U
api-institucional:compose                7d0cc0813ecc       90.3MB         22.3MB    U
hello-world:latest                       5e2309035332       25.9kB         9.49kB
nginx:alpine                             df221db836e1       94.4MB         27.2MB    U
postgres:16-alpine                       721873c34ceb        420MB          117MB    U
registry:2                               a3d8aaa63ed8       37.4MB         10.3MB    U
adminepn@svr-devops01:/srv/docker-registry$


adminepn@svr-devops01:/srv/docker-registry$ sudo docker rm -f api-service 2 > /dev/null || true

$ sudo docker run -d --name api-secure \                      -p 5004:5000 \
--read-only \
--tmpfs /tmp \
--cap-drop ALL \
--security-opt no-new-privileges=true \
--memory 256m \
--cpus 0.50 \
-e APP_ENV=secure-lab \
api-institucional:1.
Unable to find image 'api-institucional:1.0


63e023087e0cec135e943135c0c609441ce03af4934d55dbfa76aa423121a14b

$curl http://localhost:5004/health
{"status":"ok"}


$ sudo docker image ls
i Info →   U  In Use
IMAGE                                    ID             DISK USAGE   CONTENT SIZE   EXTRA
172.25.4.80:5000/api-institucional:1.0   2a24d42d2f21       90.3MB         22.3MB    U
alpine:3.20                              d9e853e87e55       12.2MB         3.71MB
api-institucional:1.0                    2a24d42d2f21       90.3MB         22.3MB    U
api-institucional:compose                7d0cc0813ecc       90.3MB         22.3MB    U
hello-world:latest                       5e2309035332       25.9kB         9.49kB
nginx:alpine                             df221db836e1       94.4MB         27.2MB    U
postgres:16-alpine                       721873c34ceb        420MB          117MB    U
registry:2                               a3d8aaa63ed8       37.4MB         10.3MB    U
adminepn@svr-devops01:/srv/docker-registry$



$ sudo docker inspect api-secure --format 'SecurityOpt={{json .HostConfig.SecurityOpt}} CapDrop={{json .HostConfig.CapDrop}}'
SecurityOpt=["no-new-privileges=true"] CapDrop=["ALL"]


y$ sudo docker stats --no-stream api-secure
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT   MEM %     NET I/O         BLOCK I/O   PIDS
63e023087e0c   api-secure   0.03%     22.18MiB / 256MiB   8.66%     1.57kB / 730B   0B / 0B     1
adminepn@svr-devops01:/srv/docker-registry$


sudo docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] s

$sudo docker version --format '{{.Server.Version}}'
29.8.2


### Instalar 


docker --version
Docker version 29.8.2, build 7fc2dff

adminepn@svr-devops01:/srv/docker-registry$ docker info
Client: Docker Engine - Community
Version:    29.8.2
Context:    default
Debug Mode: false
Plugins:
buildx: Docker Buildx (Docker Inc.)
Version:  v0.37.1



sudo mkdir Labotarorio_Docker_MiniSOC
adminepn@svr-devops01:~/u2-docker$ cd Labotarorio_Docker_MiniSOC/

sudo docker run --name web-minisoc -d -p 8081:60 nginx:alpine


sudo docker run --name web-minisoc -d -p 8081:80 nginx:alpine
b9df3df1eb6a6a218e6aa46c9baaa1d43c3e1ff22ea2a16cd8924eb41436cfb9


sudo find . -type f -exec grep -e "5000" {} \;
- "127.0.0.1:8080:5000"
- "import urllib.request; urllib.request.urlopen('http://127.0.0.1:5000/salud', timeout=2)"
- `127.0.0.1:8080:5000` significa puerto 8080 en el host (solo loopback) conectado al 5000 dentro del contenedor.
- `port is already allocated` para 8080: liberar ese puerto o cambiar `127.0.0.1:8080:5000` por otro, por ejemplo 8082.


sudo docker build -t mini-soc-web:lab .

sudo docker history mini-soc-web:labio@q

sudo docker compose up -d --build

sudo docker compose exec redis redis-cli PING

sudo docker compose exec redis redis-cli GET evento:acceso

