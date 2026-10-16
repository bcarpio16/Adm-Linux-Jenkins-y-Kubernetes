## Integración Jenkins con GIT 

Creamos el repositorio en el servidor 2 - srv-node02


adminepn@srv-node02:~$ mkdir -p "$HOME/unidad3"

ambos servidores deben tener esa carpeta

adminepn@svr-devops01:~/unidad3$ sudo vim "$HOME/unidad3/env.sh"

adminepn@svr-devops01:~/unidad3$ cat "$HOME/unidad3/env.sh"
export NODE_USER="devuser02"
export NODE_HOST="srv-node02"
export LAB_REPO="$HOME/unidad3/portal"


luego hay q hacer las exportaciones 

adminepn@svr-devops01:~/unidad3$ source "$HOME/unidad3/env.sh"


Para ver la cuenta remota 

adminepn@svr-devops01:~/unidad3$ ssh -t "$NODE_USER@$NODE_HOST" "command -v git || (sudo apt update && apt install -y git)"
adminepn@srv-node02's password:
/usr/bin/git
Connection to srv-node02 closed.

verificamos en el otro servidor 


adminepn@srv-node02:~$ command git -v git
git version 2.53.0

adminepn@srv-node02:~$ git remote remove origin
fatal: not a git repository (or any of the parent directories): .git


para verificar 

cat /etc/hosts
cat /etc/nsswitch.conf
cat /etc/resolv.conf


adminepn@svr-devops01:~/unidad3$ test "$NODE_USER" != adminepn && ssh "$NODE_USER@$NODE_HOST" hostname


no retorna errores

## CRear el repositorio

adminepn@svr-devops01:~/unidad3$ mkdir -p "$HOME/unidad3/portal/web" "$HOME/unidad3/portal/tests"

adminepn@svr-devops01:~/unidad3$ cd portal/

adminepn@svr-devops01:~/unidad3/portal$ sudo vim web/index.html

contenido 

<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8"> <title> Portal Ejemplo EPN </title>
  </head>
  <body>
    <h1>Portal de Servicios Munipales</h1> <p>Version 1.0</p>
  </body>
</html>

luego creamos una funcion para validar 

adminepn@svr-devops01:~/unidad3/portal$ sudo vim tests/test_content.sh

Contenido 

#!/usr/bin/env bash
set -euo pipefail
test -s web/index.html
grep -Fq "Portal Ejemplo" web/index.html
grep -Fq "Version" web/index.html
echo "Pruebas validadas"


damos permisos 

adminepn@svr-devops01:~/unidad3/portal$ sudo chmod +x tests/test_content.sh

verificamos

adminepn@svr-devops01:~/unidad3/portal$ bash tests/test_content.sh
Pruebas validadas


Imiciamos el git, desde la carpeta en la q nos encontremos


adminepn@svr-devops01:~/unidad3/portal$ ll
total 16
drwxrwxr-x 4 adminepn adminepn 4096 Oct 15 16:48 ./
drwxrwxr-x 3 adminepn adminepn 4096 Oct 15 16:48 ../
drwxrwxr-x 2 adminepn adminepn 4096 Oct 15 16:53 tests/
drwxrwxr-x 2 adminepn adminepn 4096 Oct 15 16:50 web/
adminepn@svr-devops01:~/unidad3/portal$
adminepn@svr-devops01:~/unidad3/portal$ git init -b main
Initialized empty Git repository in /home/adminepn/unidad3/portal/.git/
adminepn@svr-devops01:~/unidad3/portal$

adminepn@svr-devops01:~/unidad3/portal$ ls -la
total 20
drwxrwxr-x 5 adminepn adminepn 4096 Oct 15 16:57 .
drwxrwxr-x 3 adminepn adminepn 4096 Oct 15 16:48 ..
drwxrwxr-x 6 adminepn adminepn 4096 Oct 15 16:57 .git
drwxrwxr-x 2 adminepn adminepn 4096 Oct 15 16:53 tests
drwxrwxr-x 2 adminepn adminepn 4096 Oct 15 16:50 web


aqui creamos usuarios de identificacion de quien ejecuto cosas

adminepn@svr-devops01:~/unidad3/portal$ git config user.name "Byron Carpio"
adminepn@svr-devops01:~/unidad3/portal$ git config user.email "byron.carpio@quito.gob.ec"


ahora agregamos 

adminepn@svr-devops01:~/unidad3/portal$ git add web/index.html tests/test_content.sh

para publicar se usa 


adminepn@svr-devops01:~/unidad3/portal$ git commit -m "Archivos iniciales de Portal Ejemplo EPN"
[main (root-commit) d06c306] Archivos iniciales de Portal Ejemplo EPN
2 files changed, 15 insertions(+)
create mode 100755 tests/test_content.sh
create mode 100644 web/index.html

ver historial


adminepn@svr-devops01:~/unidad3/portal$ git log -1
commit d06c306d3ab8c4ae7e9d9e65752db04013b0d015 (HEAD -> main)
Author: Byron Carpio <byron.carpio@quito.gob.ec>
Date:   Thu Oct 15 17:05:43 2026 -0500

    Archivos iniciales de Portal Ejemplo EPN
adminepn@svr-devops01:~/unidad3/portal$


adminepn@svr-devops01:~/unidad3/portal$ git status --short && git branch --show-current
main


En el nodo 2 vamos a descargar 


# Creamos llaves entre servidores

adminepn@svr-devops01:~/unidad3/portal$ mkdir -p "$HOME/.ssh"
adminepn@svr-devops01:~/unidad3/portal$ chmod 700 "$HOME/.ssh"

if [ ! -f "$HOME/.ssh/jenkins_git_lab" ]; then ssh-keygen -t ed25519 -f "$HOME/.ssh/jenkins_git_lab" -N "" -C "jenkins-unidad3-lab" fi

adminepn@svr-devops01:~/unidad3/portal$ if [ ! -f "$HOME/.ssh/jenkins_git_lab" ]; then                          ssh-keygen -t ed25519 -f "$HOME/.ssh/jenkins_git_lab" -N "" -C "jenkins-unidad3-lab"
fi
Generating public/private ed25519 key pair.
Your identification has been saved in /home/adminepn/.ssh/jenkins_git_lab
Your public key has been saved in /home/adminepn/.ssh/jenkins_git_lab.pub
The key fingerprint is:
SHA256:nw3p8DgzEns3mTFMYASZ2ILGLMN/kx4Y557SG3uBtWw jenkins-unidad3-lab
The key's randomart image is:
+--[ED25519 256]--+
|.o . o.=+        |
|o.=.o.+. .       |
| +. =..   .      |
|   o * . o .     |
|    = O S *      |
|   . B E * O     |
|    . B B X .    |
|     o + = .     |
|      .          |
+----[SHA256]-----+

adminepn@svr-devops01:~/unidad3/portal$ ssh-copy-id -i "$HOME/.ssh/jenkins_git_lab.pub" "$NODE_USER@$NODE_HOST"
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/adminepn/.ssh/jenkins_git_lab.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
adminepn@srv-node02's password:

Number of key(s) added: 1

Now try logging into the machine, with: "ssh -i /home/adminepn/.ssh/jenkins_git_lab 'adminepn@srv-node02'"
and check to make sure that only the key(s) you wanted were added.

adminepn@svr-devops01:~/unidad3/portal$ ssh -i "$HOME/.ssh/jenkins_git_lab" -o IdentitODE_USER@$NODE_HOST" hostname
srv-node02
adminepn@svr-devops01:~/unidad3/portal$
adminepn@svr-devops01:~/unidad3/portal$
adminepn@svr-devops01:~/unidad3/portal$ ls "$HOME/.ssh/"
authorized_keys  jenkins_git_lab  jenkins_git_lab.pub  known_hosts  known_hosts.old
adminepn@svr-devops01:~/unidad3/portal$
adminepn@svr-devops01:~/unidad3/portal$
adminepn@svr-devops01:~/unidad3/portal$ ssh -i "$HOME/.ssh/jenkins_git_lab" "$NODE_USEmkdir -p ~/repos && git init --bare ~/repos/portal.git"
Reinitialized existing Git repository in /home/adminepn/repos/portal.git/
adminepn@svr-devops01:~/unidad3/portal$ ls
tests  web
adminepn@svr-devops01:~/unidad3/portal$ ls -la
total 20
drwxrwxr-x 5 adminepn adminepn 4096 Oct 15 16:57 .
drwxrwxr-x 3 adminepn adminepn 4096 Oct 15 16:48 ..
drwxrwxr-x 7 adminepn adminepn 4096 Oct 15 17:55 .git
drwxrwxr-x 2 adminepn adminepn 4096 Oct 15 16:53 tests
drwxrwxr-x 2 adminepn adminepn 4096 Oct 15 16:50 web


## En el nodo1 

$ cd "$HOME/unidad3/portal/"

$ NODE_HOME="$(ssh -i "$HOME/.ssh/jenkins_git_lab" -o IdentitiesOnly=yes "$NODE_USER@$NODE_HOST" 'printf %s "$HOME"')"
adminepn@srv-node02's password:
adminepn@svr-devops01:~/unidad3/portal$

$ echo $NODE_HOME
/home/adminepn


adminepn@svr-devops01:~/unidad3/portal$ printf "HOME_VM02=%s\n" "$NODE_HOME"
HOME_VM02=/home/adminepn



adminepn@svr-devops01:~/unidad3/portal$ git remote add origin "ssh://$NODE_USER@$NODE_HOST$NODE_HOME/repos/portal.git"

adminepn@svr-devops01:~/unidad3/portal$ GIT_SSH_COMMAND="ssh -i $HOME/.ssh/jenkins_gitiesOnly=yes" git push -u origin main
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 631 bytes | 631.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To ssh://srv-node02/home/adminepn/repos/portal.git
* [new branch]      main -> main
  branch 'main' set up to track 'origin/main'.
  adminepn@svr-devops01:~/unidad3/portal$

adminepn@svr-devops01:~/unidad3/portal$ git remote -v
origin  ssh://adminepn@srv-node02/home/adminepn/repos/portal.git (fetch)
origin  ssh://adminepn@srv-node02/home/adminepn/repos/portal.git (push)


En caso de querer borrar un repsitior se hace con 

git remote remove origin

## Jenkins File 

declara los pipelines, son basicamente conectores logicos a los diferentes agentes

adminepn@svr-devops01:~/unidad3/portal$ more $HOME/unidad3/env.sh
export NODE_USER="adminepn"
export NODE_HOST="srv-node02"
export LAB_REPO="$HOME/unidad3/portal"
adminepn@svr-devops01:~/unidad3/portal$ sudo vim Jenkinsfile

contenido
pipeline {
agent any
options { timestamps(); buildDiscarder(logRotator(numToKeepStr: '15')) }
stages {
stage('Validar sintaxis') {
steps { sh 'bash -n tests/test_content.sh' }
}
stage('Probar portal') {
steps { sh 'bash tests/test_content.sh' }
}
stage('Guardar evidencia') {
steps { archiveArtifacts artifacts: 'web/index.html', fingerprint: true }
}
}
}

adminepn@svr-devops01:~/unidad3/portal$ git add Jenkinsfile

adminepn@svr-devops01:~/unidad3/portal$ git commit -m "Agregar Pipeline cI declarativo TEST"
[main 1b88af9] Agregar Pipeline cI declarativo TEST
1 file changed, 15 insertions(+)
create mode 100644 Jenkinsfile

adminepn@svr-devops01:~/unidad3/portal$ git show HEAD:Jenkinsfile | head -n 16
pipeline {
agent any
options { timestamps(); buildDiscarder(logRotator(numToKeepStr: '15')) }
stages {
stage('Validar sintaxis') {
steps { sh 'bash -n tests/test_content.sh' }
}
stage('Probar portal') {
steps { sh 'bash tests/test_content.sh' }
}
stage('Guardar evidencia') {
steps { archiveArtifacts artifacts: 'web/index.html', fingerprint: true }
}
}
}

adminepn@svr-devops01:~/unidad3/portal$ ssh-keyscan -t ed25519 "$NODE_HOST" > "$HOME/unidad3/node02_hostkey"

adminepn@svr-devops01:~/unidad3/portal$ more "$HOME/unidad3/node02_hostkey"
# srv-node02:22 SSH-2.0-OpenSSH_10.2p1 Ubuntu-2ubuntu3.7
srv-node02 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG6C88izEO9LbYLhlqrm2RWzxbD5Frnejnr1nwE
WNK7i


adminepn@svr-devops01:~/unidad3/portal$ ssh-keygen -lf $HOME/unidad3/node02_hostkey
256 SHA256:AsqnmV5qx7Epeyg7R5jWqFaPUdPru4HaXYwdJOIpTiw srv-node02 (ED25519)

adminepn@svr-devops01:~/unidad3/portal$ ssh "$NODE_USER@$NODE_HOST" "ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub"
adminepn@srv-node02's password:
256 SHA256:AsqnmV5qx7Epeyg7R5jWqFaPUdPru4HaXYwdJOIpTiw root@srv-node02 (ED25519)

adminepn@svr-devops01:~/unidad3/portal$ sudo install -d -o jenkins -g jenkins -m 0700 /var/lib/jenkins/.ssh
[sudo: authenticate] Password:
adminepn@svr-devops01:~/unidad3/portal$


adminepn@svr-devops01:~/unidad3/portal$ sudo install -o jenkins -g jenkins -m 0600 "$HOME/unidad3/node02_hostkey" /var/lib/jenkins/.ssh/known_hosts


adminepn@svr-devops01:~/unidad3/portal$ sudo ls -la /var/lib/jenkins/.ssh/known_hosts
-rw------- 1 jenkins jenkins 149 Oct 15 22:47 /var/lib/jenkins/.ssh/known_hosts
adminepn@svr-devops01:~/unidad3/portal$
adminepn@svr-devops01:~/unidad3/portal$
adminepn@svr-devops01:~/unidad3/portal$ sudo more /var/lib/jenkins/.ssh/known_hosts
# srv-node02:22 SSH-2.0-OpenSSH_10.2p1 Ubuntu-2ubuntu3.7
srv-node02 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG6C88izEO9LbYLhlqrm2RWzxbD5Frnejnr1nwE
WNK7i
adminepn@svr-devops01:~/unidad3/portal$

adminepn@svr-devops01:~/unidad3/portal$ sudo -u jenkins test -r /var/lib/jenkins/.ssh/known_hosts & echo HOST_CONFIABLE
[1] 80967
HOST_CONFIABLE




