# Mini-projet Ansible 

## Création d'un role pour le déploiement d'une webapp avec docker et apache

Il est question d'écrire un *role* ansible et que ce *role* soit appelé dans un *PlayBook* et exécuter ce *role*.

### Consignes

- Il est question d'installer l'application **apache**. 
- Pas de l'installer en *bare metal* mais de la mettre dans un *conteneur*.
- Donc il est question de déployer un conteneur docker d'apache.
- Il n'est question de *builder* l'image et ensuite de déployer le conteneur. **Non**.
- Nous avons une application *apache*, lancer cette application dans un conteneur mais à l'aide de *Ansible**.
- On aura un *role*.
- Lorsque *Ansible* va faire appel à ce *role*.
- Ce que ce *role* sait faire c'est de déployer *apache* dans un conteneur.

### Petite illustration shématique du process.

![Image roe](/images/illustRole.PNG "Image Process")

# Réalisation du mini projet k8s

## Les outils utilisés et fichiers du projet

- Visual Studio Code.
- L'hyperviseur de type 2 Oracle VM VirtualBox.
- VM CentOS 7
- - **Vagrantfile** pour déployer mes VM *Ansible*, *client1* et *client2*.
- **install_ansible.sh** fichier de provisionning qui va automatiser l'intallation des outils nécessaire dans les VM.
- Je vais déployer un serveur **Ansible** dans une VM appelée *ansible*.
- Je vais déployer deux VM  **client** qui seront pilotés par le serveur *Ansible*.
- **client1** est la machine de *Prod*
- **client2** est dans un environnement développement *Staging* de pré-production.

On aura deux **groupes**.
- **Prod** qui contient la machine *client1*.
- **Staging** qui contient la machine *client2*.

On va installer dans **client1** via le serveur **ansible**:
- *Docker*.
- *Python*.
- *Pip* qui est une librairie de python.
- Ainsi que tous les librairies python qui nous permettent de piloter *docker* à l'aide de *ansible*. En particulier **docker-py**.

Il est bon de savoir qu'à l'aide de **docker-py**, *ansible* pourra piloter le *daemon docker* pour déployer nos conteneurs applicatifs *apache*.

# Illustration graphique de notre solution

![Image Depl](/images/illusDeployment.jpeg "Image Deployment")

## Mise en place des VMs ansible, client1 et clients

Résultat du déploiement de nos trois VMs.

![Image Depl1](/images/illusDeployment1.PNG "Image Deployment1")

1. On lance le déploiement des VMs
```
vagrant up --provision
```

2. Je me connecte uniquement à la VM serveur ansible via ssh. Et c'est *ansible* qui va établir les connexions avec les autres VMs.

```
vagrant ssh ansible
```

Résultat:

```

```

Pour faciliter le process de connexion avec *client1* et *client2*, je vais établir une communication par échange des clés. Entre le serveur *ansible* et les autres machines.

### Rappel

- Le login de toutes ces VMs est *vagrant*.
- Leur mot de passe est *vagrant*.

1. Je génère un couple de clés (publique et privée) pour mon utilisateur *vagrant*.

```
ssh-keygen -t rsa
```

![Image ssh](/images/ssh.PNG "Image ssh")

2. Je copy cette clé *ssh* dans *client1*.

```
ssh-copy-id vagrant@192.168.99.11
```
Résultat:
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host '192.168.99.11 (192.168.99.11)' can't be established.
ECDSA key fingerprint is SHA256:3pcHGz3LYQ5fH/P2kqlgbhnuKk3KYRJIDMhBGvFwaKU.
ECDSA key fingerprint is MD5:1e:d8:b7:73:38:8d:54:76:25:52:80:5c:dc:74:d6:77.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@192.168.99.11's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@192.168.99.11'"
and check to make sure that only the key(s) you wanted were added.
```

3. Je copy cette clé *ssh* dans *client*.

```
ssh-copy-id vagrant@192.168.99.12
```
Résultat:
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host '192.168.99.12 (192.168.99.12)' can't be established.
ECDSA key fingerprint is SHA256:3pcHGz3LYQ5fH/P2kqlgbhnuKk3KYRJIDMhBGvFwaKU.  
ECDSA key fingerprint is MD5:1e:d8:b7:73:38:8d:54:76:25:52:80:5c:dc:74:d6:77. 
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@192.168.99.12's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@192.168.99.12'"
and check to make sure that only the key(s) you wanted were added. 
```

4. Je peux maintenant me connecter à *client1 ou *client* sans demande de mot de passe.

```
sssh vagrant@192.168.99.11
```
ou

```
sssh vagrant@192.168.99.12
```

5. Pour se déconnecter d'un machine *client*

```
exit
```
6. Je vérifie que mon serveur *ansible* est bien installé.

```
ansible --version
```

# Préparation des manifestes de notre projet ansible.

1. Je créé un repertoire et je me déplace dans ce dernier

```
mkdir -p ansible-project
```
```
cd ansible-project/
```
2. Etant donné que je suis dans ma machine *vagrant*, je copie tous éléménts dans **ansible-project**.

```
cp -r /vagrant/* .
```

3. Je vérifie que la copie a été effectué avec succès.

```
ll
```
Résultat: J'ai ici tous les fichiers de mon projet.

![Image cp](/images/cp.PNG "Image copie")

# Lancer la lecture de mon playbook en mode check -C

```
ansible-playbook deploy-app.yml -C
```
Résultat du outup en mode check: Tout est ok

![Image check](/images/check.PNG "Image -C")

# Lancer la lecture de mon playbook de façon réelle.

```
ansible-playbook deploy-app.yml
```

Résultat du outup: Tout est ok.

![Image Play](/images/play.PNG "Image PlayBook OK")

Tout s'est bien passé dans la machine *client2*.

1. Je vais aller consulter.

```
curl -I 192.168.99.12
```
Résultat:
```
HTTP/1.1 200 OK
Date: Sun, 24 Apr 2022 14:01:19 GMT
Server: Apache/2.4.53 (Unix)
Last-Modified: Sun, 24 Apr 2022 13:50:48 GMT
ETag: "1d-5dd66bf3a4dde"
Accept-Ranges: bytes
Content-Length: 29
Content-Type: text/html
```

2. Je vais vérifier que *client2* répond aussi bien.

```
ansible client2 -m shell -a "sudo docker ps -a"
```
Résultat:

![Image Client2](/images/respond.PNG "Image client2 check")

Sur la machine, j'ai un conteneur qui s'appelle **webapp**, qui tourne depuis 10min et écoute au port 80.

3. Je vérifie dans le navigateur.

![Image Navigateur](/images/nav.PNG "Image Navigateur check")

# Lancer le déploiment sur la Prod.

Pour ce faire, on va juste modifier le manifeste **deploy-app.yml** dans sa première *hosts* en mettant **all**.

Puis je relance le **playbook. 

Pour cette nouvelle lecture du play, il va juste faire la actions déjà faites au *client2* dans *client1*.

Donc le **changed** s'effectue seulement au **client1**.

```
ansible-playbook deploy-app.yml
```