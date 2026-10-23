```bash
kubectl create namespace lab-bcarpio
```
namespace/lab-bcarpio created

```bash
kubectl get  namespace
```
Resultado

NAME                 STATUS   AGE
default              Active   28h
kube-node-lease      Active   28h
kube-public          Active   28h
kube-system          Active   28h
lab-bcarpio          Active   6s
lab-web              Active   6h28m
local-path-storage   Active   28h

Creamos el directorio del proyecto, cambiar el nombre

```bash
mkdir -p ~/laboratorios-k8s/lab-nginx_bcarpio

cd ~/laboratorios-k8s/lab-nginx_bcarpio/
```
Creamos la pag. web 

```bash
vim deber.html
```

Creamos el configmap, aqui se debe cambar el html-deber y tambien cambiar el nombre de la pag. en este caso deber.hmlt

```bash
kubectl -n lab-web create configmap html-deber --from-file=index.html=deber.html --dry-run=client -o yaml | kubectl apply -f -
```
Resultado

configmap/html-tramites created

Listamos los configmaps del namespace

```bash
kubectl -n lab-bcarpio get configmaps
```

NAME               DATA   AGE
html-deber         1      50s
kube-root-ca.crt   1      21m

Revisamos el configmap especifico

```bash
kubectl -n lab-bcarpio get configmaps html-deber
```

NAME         DATA   AGE
html-deber   1      87s

Ver el detalle del configmap 

```bash
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx_bcarpio$ kubectl -n lab-bcarpio describe configmaps html-deber
```
Resultado

Name:         html-deber
Namespace:    lab-bcarpio
Labels:       <none>
Annotations:  <none>

creamos el archivo *.yaml, la ruta debe ser donde se creo la carpeta del proyecto 

adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx_bcarpio$ ls -la

-rw-rw-r-- 1 adminepn adminepn 2016 Oct 22 21:05 deber.html
-rw-rw-r-- 1 adminepn adminepn 1170 Oct 22 23:47 portal-deber.yaml

```bash
vim portal-deber.yaml
```

Agregamos lo siguiente, aqui se debe cambiar el name y namespace por el creen en los pasos anteriores


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: portal-deber
namespace: lab-bcarpio

spec:
replicas: 1

selector:
matchLabels:
app: portal-deber

template:
metadata:
labels:
app: portal-deber

    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine

        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: "50m"
            memory: "32Mi"
          limits:
            cpu: "200m"
            memory: "128Mi"

        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 2
          periodSeconds: 5

        volumeMounts:
        - name: contenido-html
          mountPath: /usr/share/nginx/html
          readOnly: true

      volumes:
      - name: contenido-html
        configMap:
          name: html-deber

---
apiVersion: v1
kind: Service
metadata:
name: portal-deber
namespace: lab-bcarpio

spec:
type: ClusterIP

selector:
app: portal-deber

ports:
- name: http
  port: 80
  targetPort: 80
  protocol: TCP
```

luego ejecutamos lo siguiente 

```bash
kubectl apply --dry-run=client --validate=true -f portal-deber.yaml
```

Resultado

deployment.apps/portal-deber created (dry run)
service/portal-deber created (dry run)

```bash
kubectl apply -f portal-deber.yaml
```

Resultado

deployment.apps/portal-deber created
service/portal-deber created


```bash
kubectl -n lab-bcarpio get deployments
```
Resultado
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
portal-deber   1/1     1            1           36s

```bash
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx_bcarpio$ kubectl -n lab-bcarpio get pods
```

Resultado

NAME                           READY   STATUS    RESTARTS   AGE
portal-deber-bf68d5d97-z9lph   1/1     Running   0          59s

```bash
kubectl -n lab-bcarpio rollout status deployment/portal-deber --timeout=180s
```
Resultado

deployment "portal-deber" successfully rolled out

```bash
kubectl -n lab-bcarpio describe service portal-deber
```
Resultado

```text
Name:                     portal-deber
Namespace:                lab-bcarpio
Labels:                   <none>
Annotations:              <none>
Selector:                 app=portal-deber
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.82.135
IPs:                      10.96.82.135
Port:                     http  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.0.8:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

Levantamos el servicio, la IP va a cambiar acorde cada maquina virtual 
```bash
adminepn@svr-devops02:~/laboratorios-k8s/lab-nginx_bcarpio$ kubectl -n lab-bcarpio port-forward --address 172.25.4.82 services/portal-deber 8085:80
```
Forwarding from 172.25.4.82:8085 -> 80
Handling connection for 8085
