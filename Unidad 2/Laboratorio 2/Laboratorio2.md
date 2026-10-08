# Laboratorio 2 - Docker con imagen propia

<p style="text-align: justify;">
Verificamos que tenga acceso a las llaves de ubuntu 
</p>

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```
```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Creamos el repositorio de docker o verificamos 

```bash
sudo vim /etc/apt/sources.list.d/docker.sources
```
Agregamos el siguiente texto

```text
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: resolute
Components: stable
Architectures: amd64
Signed-By: /etc/apt/keyrings/docker.asc
```

Verificamos los servicios 
```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin
sudo systemctl enable --now docker
sudo systemctl status docker --no-pager
sudo systemctl status containerd --no-pager
docker version
```

Creamos el directorio donde va a estar el proyecto

```bash
cd /home/adminepn/u2-docker
cd u2-docker
sudo mkdir api-institucional
```
Editamos el archivo 

```bash
sudo vim app.py
```
Agregamos el siguiente contenido

```text
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

En contenedores debemos controlar los archivos que se van a ignorar 

```bash
sudo vim .dockerignore
```
Agregamos lo siguiente 

```text
.git
__pycache__
*.pyc
.env
*.log
```

Tambien agregamos el archivo Dockerfile

```bash
sudo vim Dockerfile
```

Agregamos los siguiente 

```text
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

Construimos una imagen Docker a partir del Dockerfile que se encuentre en el directorio actual.

```bash
sudo docker build -t api-institucional:1.0 .
```
Resultado 

```text
[+] Building 2.4s (4/9)                                                                      docker:default
=> [internal] load build definition from Dockerfile                                                   0.0s
=> => transferring dockerfile: 601B                                                                   0.0s
=> [internal] load metadata for docker.io/library/python:3.12-alpine                                  1.6s
```


sudo docker image ls api-institucional
                                                                                        i Info →   U  In Use
IMAGE                   ID             DISK USAGE   CONTENT SIZE   EXTRA
api-institucional:1.0   2a24d42d2f21       90.3MB         22.3MB

docker history api-institucional:1.0 --no-trunc | head

sudo docker run -d --name api-institucional-demo -p 5001:5000 -e APP_NEW=labunidad2 api-institucional:1.0
2e0c58ae740311ceae6b831e34011375bd55116826163ce64aad0687e2105bb0


curl http://localhost:5001/
{"entorno":"laboratorio","host":"2e0c58ae7403","servicio":"api-institucional","version":"1.0"}

curl http://localhost:5001/health
{"status":"ok"}

```bash
sleep 15
sudo docker inspect --format '{{.State.Health.Status}}' api-institucional-demo
sudo docker inspect --format 'Usuario={{.Config.User}}' api-institucional-demo
```

```text
healthy
Usuario=appuser
```

```bash
sudo docker ps -a --filter name=api-institucional-demo
```
Resultado 

```text
CONTAINER ID   IMAGE                   COMMAND           CREATED         STATUS                   PORTS                                         NAMES
2e0c58ae7403   api-institucional:1.0   "python app.py"   4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   api-institucional-demo
```

Verificamos 

```bash
curl http://localhost:5001/health
```
```text
{"status":"ok"}
```




adminepn@svr-devops01:~/u2-docker/api-institucional$
adminepn@svr-devops01:~/u2-docker/api-institucional$ curl http://localhost:5001
{"entorno":"laboratorio","host":"2e0c58ae7403","servicio":"api-institucional","version":"1.0"}
adminepn@svr-devops01:~/u2-docker/api-institucional$
adminepn@svr-devops01:~/u2-docker/api-institucional$ sudo docker logs api-institucional-demo
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
  adminepn@svr-devops01:~/u2-docker/api-institucional$


sudo docker ps -a --filter name=api-institucional-demo
CONTAINER ID   IMAGE                   COMMAND           CREATED         STATUS                   PORTS                                         NAMES
2e0c58ae7403   api-institucional:1.0   "python app.py"   9 minutes ago   Up 9 minutes (healthy)   0.0.0.0:5001->5000/tcp, [::]:5001->5000/tcp   api-institucional-demo




sudo docker exec api-institucional-demo sh -c "wget -qO - http://localhost:5000/health"
wget: can't connect to remote host: Connection refused

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

