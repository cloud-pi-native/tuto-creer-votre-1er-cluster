# Créer votre cluster pour bien commencer
**( version pour commentaires )**

Pour permettre de mettre en place votre environnement de travail avec un cluster kubernetes. Un choix assez large d'options est disponible, cet article propose plusieurs approches pour créer le votre.
Il vous permet de construire un cluster et progressivement utiliser une chaine de livraison logicielle compatible avec l'offre Cloud Pi Native proposée par le ministère de l'Intérieur. Il vous faudra respecter quelques conditions ou adapter votre code au choix de processeur, rootless, liste des images de références. Cf. liste de compatibilité citée en annexes.

Pour découvrir la conteneurisation et Kubernetes, vous pouvez suivre des tutoriaux tels que ceux présents au sein de ce répertoire github, ainsi qu'à travers une offre de tutoriaux vidéo en ligne. Tel que ceux proposés par "Tech With Nana" (Activer la traduction des sous titre dans YouTube) :
https://www.youtube.com/watch?v=VnvRFRk_51k ( acculturation rapide de 15 min )
https://www.youtube.com/watch?v=s_o8dwzRlu4 ( 'crash course' en 1H )
https://www.youtube.com/watch?v=X48VuDVv0do ( court complet 4H )

Vous disposer de plusieurs possibilités pour monter votre environnement, dans le cloud ou en local. Vous commencerez probablement sur votre poste de travail et irez ensuite vers une combinaison hybride :

### 1. Monter votre cluster sur votre poste de travail : (commencez par celle-ci)
L'option généralement la plus confortable pour débuter ou pour un développeur.
Pré-requis : un ordinateur Mac ou Unix (PC possible) avec à minima 8Go de RAM et plusieurs processeurs.
Note: Cet article a été réalisé à partir de MacBook Pro 'M1' (processeur de type Arm) et testé sur des VM linux Ubuntu Arm.
Là vous disposer de plusieurs options également.

###### Option 1.1 : Installer **DockerDestop** : (peut nécessiter une licence)

Permet d'activer simultanément l'ensembles des outils docker et d'un minicluster kubernetes. ( à activer dans les préférences)
cf https://www.docker.com/.

###### Option 1.2: Installer un **cluster K3s**, version économe en ressource cf. https://k3s.io/

Sur Mac et PC vous devez disposer de la capacité d'exécution de VM linux, vous pouvez utiliser  https://multipass.run, suivez les instructions pour l'installation, cela s'installe comme un charme.

  ```
  # Lancer la commande suivate pour créer une VM d'accueil pour le mini cluster kubernetes : 8Go Ram, 4 vcpu, 60G de disque), vous pouvez modifier ces valeurs.

  multipass launch lts --name k3s-server  -m 8g -c 4 -d 60G

# Installer un master k3s avec les droits d'accès sur l'ensemble des utilisateurs
  multipass shell k3s-server
  sudo curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644

# Vérifier la disponibilité du worker node.
  ubuntu@k3s-server:~$   kubectl get nodes
  NAME         STATUS   ROLES                  AGE     VERSION
  k3s-server   Ready    control-plane,master   3m25s   v1.25.4+k3s1

  ubuntu@k3s-server:~$ kubectl get pods -A
  NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
  kube-system   local-path-provisioner-79f67d76f8-nkslg   1/1     Running     0          3m47s
  kube-system   coredns-597584b69b-g9h4m                  1/1     Running     0          3m47s
  kube-system   helm-install-traefik-crd-frxpb            0/1     Completed   0          3m47s
  kube-system   svclb-traefik-162592aa-zzfqb              2/2     Running     0          3m38s
  kube-system   helm-install-traefik-h276n                0/1     Completed   1          3m47s
  kube-system   traefik-bb69b68cd-x257d                   1/1     Running     0          3m38s
  kube-system   metrics-server-5c8978b444-g55zj           1/1     Running     0          3m47s
```
   ###### Préparer l'utilisation de votre cluster

```
 #Rechercher l'ip locale de votre machine dans le haut de la fenêtres en déroulant vers le haut pour retrouver dans les messages initiaux de la VM


   Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-56-generic aarch64)

  * Documentation:  https://help.ubuntu.com
  * Management:     https://landscape.canonical.com
  * Support:        https://ubuntu.com/advantage

   System information as of Wed Dec 28 18:43:32 CET 2022

   System load:             1.35009765625
   Usage of /:              4.0% of 57.97GB
   Memory usage:            9%
   Swap usage:              0%
   Processes:               157
   Users logged in:         1
   IPv4 address for cni0:   10.42.0.1
*> IPv4 address for enp0s1: 192.168.64.X <******* CETTE IP ICI
   IPv6 address for enp0s1: fd12:d3ea:b18f:437b:5054:ff:fecd:9738
```

  ###### Gérer son cluster avec l'application 'Lens'
  **Lens**  permet d'interagir facilement / monitorer votre cluster sans nécessiter d'installer des services dans le cluster. Pour installer le logiciel :  https://k8slens.dev/ Vous devrez vous créer un compte sur Lens, utiliser votre login github pour vous logguer
  .

```
# Une fois l'application installée pour accéder à votre cluster
# vous devez récupérer le fichier de configuration kubernetes.
# >Copier le résultat de la commande suivante dans Lens.

  cat /etc/rancher/k3s/k3s.yaml

    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0t
    .../... (réponse coupée)
```
  Dans Lens ajoutez un nouveau cluster :

![alt_text](img/1.png "menu")

 Copier le convenu du 'cat' dans la nouvelle fenêtre et changez l'ip par celle identifiée plus haut,

![alt_text](img/2.png "creation")

  Cela crée une nouvelle entrée et vous pouvez ensuite vous connecter à votre cluster localement

![alt_text](img/3.png "creation")


```
  # a tout fin utile vous pourrez récupérer le token avec la commande suivante : ( vous en aurez besoin pour plus tard )
    sudo cat /var/lib/rancher/k3s/server/token
```
Vous êtes opérationnel pour commencer :)


###### Option 2 : Installer Minishift ( version sur poste de travail d'openshift )
Openshift est l'une des technologies utilisées par le ministère de l'Intérieur.

Vous devrez créer un compte développeur (gratuit)

https://developers.redhat.com/products/openshift-local/overview

Sélection le type d'OS et de processeurs ( ici Arm car MacBook Pro M1)
Récupérez le secret (nécessaire pour l'installation )

![alt_text](img/siteopsl.png "menu") ![alt_text](img/dwlgoc.png "menu")

https://access.redhat.com/documentation/en-us/red_hat_openshift_local/2.12

et ( cf ligne de commande )

https://crc.dev/crc/#introducing_gsg

Lancement de l'installer, puis de l'application 'open shift local'
Suivez les demandes (vous devrez spécifier votre secret)

![alt_text](img/oclocal.png "menu")

Lancez ensuite l'application et attendez 2-3 minutes que l'ensemble des 'operators' du cluster soit 'up'.

![alt_text](img/opsl.png "menu")



Pour récupérer le mot de passe, lancer 'Open developer terminal' depuis le menu puis exécuter la commande suivante

```
crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p <<VOTRE MOT DE PASSE ICI>> https://api.crc.testing:6443'

```

Ensuite lancer la console depuis le menu 'openshift'
(comme le certificat est autosigné, vous devrez valider plusieures exception de sécurité)

![alt_text](img/opshmenu.png "openshift")

https://console-openshift-console.apps-crc.testing/dashboards

Ensuite vous pouvez également ouvrir votre cluster dans l'application 'Lens'

![alt_text](img/lens-opsh-setup.png "openshift")

Vous pouvez visualiser l'état de celui-ci

![alt_text](img/lens-opsh-run.png "openshift")

Vous êtes opérationnel :)

### 2. Utiliser un ou plusieurs clusters (managés) dans le cloud, la plus facile ) :
- Pour exposer votre application vers ses usagers ;
- Séparer l'environnement d'exécution du poste de travail du développeur ;
- Travailler en groupe ;
- Attention cela nécessite une souscription.

L'avantage est que vous n'avez pas à gérer votre cluster si à maitriser tous les concepts d'infratructure. Généralement ces services sont disponibles avec des processeurs x86-64 ou AMD-64.
Les offre sous processeurs Arm sont généralement assez rare sauf à prendre des Mac 'M1'.

Pour démarrer vous pouvez utiliser des comptes d'évaluation valable quelques mois, l'ensemble des 'hyperscalers' proposent cette option.

TODO : < A FAIRE>

### 3. Monter votre propre cluster 'on premise' (pour les plus curieux) ;

Idem point précédent. C'est l'approche idéale pour se 'faire la main' et comprendre les principes fondamentaux de kubernetes y compris l'approche 'baremetal'.

- Vous considèrerez recycler des anciens serveurs pour vous faire la main, si les disques sont un peu anciens changez les par des versions SSD plus rapide et fiable. Il faut généralement >16 Go de Ram et assurez-vous que les versions récentes des OS linux (debian, ubuntu) contiennent les drivers matériels appropriés pour votre serveur et que le/les processeurs est/sont 64bits. https://www.debian.org/ports/
Il faut idéalement en prévoir 2-3 pour une configuration 'pédagogique'.
- Vous pouvez également monter un cluster à partir de microcomputeurs de type RaspberryPi. Privilégiez des modules à 8 Go de Ram. Ces petits serveurs sont assez performants et surtout ils utilisent des processeurs
'Arm' économe en énergie. ( Ce tutoriel a été réalisé à partir de RaspBerryPi 4 )

```
# note : si raspeberyPi penser à ajouter cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 en fin du fichier /boot/cmdline.txt avec la cmd suivante :
sudo vi /boot/cmdline.txt

#update/upgrade du systeme
sudo apt update && sudo apt -y upgrade

# Pour monter un cluster k3s resistant aux pannes, il est recommandé de
- installer un cluster MariaDB sur 3 noeuds ( nombre impair ) avec réplication.
cf. https://www.atlantic.net/vps-hosting/how-to-install-and-configure-mariadb-galera-cluster-on-ubuntu-18-04/ ( tips : passez les premières étapes afin de gardez la version de mariadb maintenue par la communauté Débian )
- Puis suivre la procédure https://docs.k3s.io/installation (avec la base de donnée mariadb partagée)

mysql -u root -p ( saisissez <<votre-mot-de-passe-root-db>> )
#dans votre base de données >
create database k3s_cluster;
CREATE USER k3s@localhost IDENTIFIED BY <<votre-mot-de-passe-k3s>>;
GRANT ALL PRIVILEGES ON k3s_cluster.* TO k3s@localhost;
FLUSH PRIVILEGES;
<ctrl-c>

# vérifier la réplication des bases de données en vous connectant sur les autres noeuds
mysql -u root -p ( saisissez <<votre-mot-de-passe-root-db>> )
show databases;

show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| k3s_cluster        |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.001 sec)

SELECT User FROM mysql.user;
+-------------+
| User        |
+-------------+
| k3s         |
| mariadb.sys |
| mysql       |
| root        |
+-------------+
4 rows in set (0.004 sec)

```

#executer la commande suivante sur le premier node :
```
curl -sfL https://get.k3s.io | sh -s - server \
--datastore-endpoint="mysql://k3s:<<votre-mot-de-passe-k3s>>@tcp(localhost:3306)/k3s_cluster" \
--write-kubeconfig-mode 644

#récuperez le token ( pour que les autres nodes prennent la même clés)
sudo cat /var/lib/rancher/k3s/server/token

curl -sfL https://get.k3s.io | sh -s - server \
--datastore-endpoint="mysql://k3s:oF4Tz6QcY48HHiQx@tcp(localhost:3306)/k3s_cluster" \
--write-kubeconfig-mode 644 \
--token="<<VOTRE-TOKEN_ICI>>"

# installer le client HELM ( packager de kubernetes, lien avec l'installation pour debian)
https://helm.sh/docs/intro/install/#from-apt-debianubuntu

#pensez à taper la ligne de commande suivante
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

En option pour le reverse proxy d'entrée -> utilisation de MetalLB: https://metallb.universe.tf/installation/#installation-with-helm,  load balancing via ARP sur l'ensemble des noeuds

kubectl apply -f metalbl-pool.yaml -n default

voir fichier : metalbl-pool.yaml ( pensez à changer les IPs )
```

```
# Pour installer chacun des worker nodes :
# Installer un load balancer sur l'API kubernetes, une option consiste à placer le load-balanceur sur les worker-nodes directement afin d'éviter d'installer un lB externe qu'il faut redonder lui-même etc...

# Sur chaque node installer Nginx (tout autre load balancer https/http fera l'affaire.)
sudo apt-get install nginx

# deployer le fichier nginx.conf ( écoute en local sur le port 6443 )

# tester la connection au master

curl -k https://localhost:6443
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
  }

# Deployer le node K3S client ( tips : si RaspberryPi pensez à mettre à jour le fichier /boot/cmdline.txt )
sudo curl -sfL https://get.k3s.io | K3S_URL=http://localhost:6443 \
K3S_TOKEN=<<mynodetoken>>\
sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.25.4+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.25.4+k3s1/sha256sum-arm64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.25.4+k3s1/k3s-arm64
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent

# pour vérifier votre cluster : ( ici 3 nodes master 3 agent)

kubectl get nodes
node1   Ready    control-plane,master   33h     v1.25.4+k3s1
node3   Ready    <none>                 20m     v1.25.4+k3s1
node0   Ready    control-plane,master   34h     v1.25.4+k3s1
node4   Ready    <none>                 2m56s   v1.25.4+k3s1
node5   Ready    <none>                 106s    v1.25.4+k3s1
node2   Ready    control-plane,master   33h     v1.25.4+k3s1

# Vérifier que les nodes répondent aux requêtes (à faire sur tout les nodes );
curl -k https://<<IP-D'UN-NODE>>
404 page not found # veux dire que le node réponds
```

```
Uninstalling
If you installed K3s with the help of the install.sh script, an uninstall script is generated during installation.
 The script is created on your node at /usr/local/bin/k3s-uninstall.sh (or as /usr/local/bin/k3s-agent-uninstall.sh).
```

TODO : ajouter prometheus et configuration du metalLB
