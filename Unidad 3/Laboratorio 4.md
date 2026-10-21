Servidor 1


adminepn@svr-devops01:~/unidad3/pipeline$ sudo systemctl is-active jenkins
[sudo: authenticate] Password:
active
adminepn@svr-devops01:~/unidad3/pipeline$ sudo -u jenkins git --version
git version 2.53.0
[Laboratorio 2.md](Laboratorio%202.md)
adminepn@svr-devops01:~/unidad3/pipeline$ sudo apt-get install -y openssh-server openssh-clien
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información del estado... Hecho

adminepn@svr-devops01:~/unidad3/pipeline$ sudo systemctl enable --now ssh

adminepn@svr-devops01:~/unidad3/pipeline$ getent passwd git || sudo useradd -m -s "$(command -v git-shell)" git
adminepn@svr-devops01:~/unidad3/pipeline$
adminepn@svr-devops01:~/unidad3/pipeline$ sudo usermod -s "$(command -v git-shell)" git
usermod: sin cambios
adminepn@svr-devops01:~/unidad3/pipeline$
adminepn@svr-devops01:~/unidad3/pipeline$ sudo install -d -o git -g git -m 0700 /home/git/.ssh
adminepn@svr-devops01:~/unidad3/pipeline$


adminepn@svr-devops01:~/unidad3/pipeline$ sudo touch /home/git/.ssh/authorized_keys
adminepn@svr-devops01:~/unidad3/pipeline$
adminepn@svr-devops01:~/unidad3/pipeline$ sudo chown git:git /home/git/.ssh/authorized_keys
adminepn@svr-devops01:~/unidad3/pipeline$
adminepn@svr-devops01:~/unidad3/pipeline$ sudo chmod 0600 /home/git/.ssh/authorized_keys
adminepn@svr-devops01:~/unidad3/pipeline$


adminepn@svr-devops01:~/unidad3/pipeline$ getent passwd git
git:x:1004:1008::/home/git:/usr/bin/git-shell
adminepn@svr-devops01:~/unidad3/pipeline$
adminepn@svr-devops01:~/unidad3/pipeline$ getent passwd adminepn
adminepn:x:1000:1000:admin:/home/adminepn:/bin/bash
adminepn@svr-devops01:~/unidad3/pipeline$


sudo install -d -o git -g git -m 0755 /srv/git


adminepn@svr-devops01:~$ sudo -u git git init --bare /srv/git/portal-html.git
hint: Using 'master' as the name for the initial branch. This default branch name
hint: will change to "main" in Git 3.0. To configure the initial branch name
hint: to use in all of your new repositories, which will suppress this warning,
hint: call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>
hint:
hint: Disable this message with "git config set advice.defaultBranchName false"
Initialized empty Git repository in /srv/git/portal-html.git/



adminepn@svr-devops01:~$ sudo -u git git --git-dir=/srv/git/portal-html.git symbolic-ref HEAD refs/heads/master
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$ sudo -u git git --git-dir=/srv/git/portal-html.git symbolic-ref HEAD
refs/heads/master


adminepn@svr-devops01:~$ ls -ld /srv/git/portal-html.git/
drwxr-xr-x 6 git git 4096 Oct 16 15:35 /srv/git/portal-html.git/



adminepn@svr-devops01:~$ sudo install -d -o desarrollador -g 1001 -m 0700 /home/desarrollador
adminepn@svr-devops01:~$
adminepn@svr-devops01:~$ ls -la /home/
total 32
drwxr-xr-x  8 root          root      4096 Oct 16 15:47 .
drwxr-xr-x 20 root          root      4096 Sep 26 22:46 ..
drwxr-x---  9 adminepn      adminepn  4096 Oct 16 15:42 adminepn
drwxr-x---  2 auditor1      auditor1  4096 Sep 30 16:17 auditor1
drwx------  2 desarrollador linuxops  4096 Oct 16 15:47 desarrollador
drwxr-x---  3 git           git       4096 Oct 16 15:30 git
drwxr-x---  2 lab_bc        lab_bc    4096 Oct  5 20:38 lab_bc
drwxr-x---  2 operador1     operador1 4096 Sep 30 16:16 operador1
adminepn@svr-devops01:~$ sudo -iu desarrollador


