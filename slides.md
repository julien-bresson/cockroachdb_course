---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki

# default: 980
# since the canvas gets smaller, the visual size will become larger
canvasWidth: 950

# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
---

# Bases de données distribuées

---

# Système distribué
<br>
Qu’est-ce qu’un système distribué ?<br><br>
Aujourd’hui le business doit être agile pour survivre.<br><br>
Et donc les applications qui soutiennent le business doivent être agiles.<br><br>
Pour qu’un système supporte les changements, il doit être :<br><br>

- Disponible
- Scalable
- Facilement déployable
- Testable
- Maintenable

---

# Bénéfices d'un système distribué
<br>
Bénéfices d’un système distributé<br><br>
Composé de plusieurs ressources, serveurs physiques, machines virtuelles et containers.<br><br> 
N’importe quel composant avec des capacités de calcul disponibles sur le réseau.<br><br>
Ce type d’architecture a 4 principaux objectifs :

- Agilité
- Rapidité
- Compétitivité
- Speed to market


---
# CockroachDB

Une base de données distribuée

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# Présentation de CockroachDB

La tendance actuelle est de passer d'une infrastructure on-premise à une infrastructure cloud.<br>
Pour ce faire il est nécessaire d'avoir des systèmes cloud ready.<br>
C'est-à-dire des systèmes qui tirent les avantages du cloud. Et donc des capacités de scaling.

Traditionnellement les environnements onpremise fournissent des ressources de manière spécifique.<br>
Chaque datacenter disposait de services qui reliaient étroitement les applications à des environnements spécifiques, en s'appuyant souvent le provisionnement manuel de l'infrastructure, comme les machines virtuelles.<br>
Les développeurs et leurs applications étaient donc limités à ce datacenter.<br>
Les applications qui n'ont pas été conçues pour le cloud ne tirent pas parti des capacités de résilience et d'évolutivité d'un environnement cloud.<br>
- Ex1 : si une application n'est pas conçue pour être déployée sur plusieurs instances en parallèle. La gestion du cache distribué.
- Ex2 : une application qui doit nécessite une intervention manuelle pour démarrer correctement, ou qui ne peut pas redémarrer automatiquement si un incident se produit.

<br>

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

---

# Architecture de CockroachDB
<br>

CockroachDB est une base de donnée relationnelle, cloud-native, cohérente et hautement scalable.<br>

Les principaux objectifs sont de fournir une forte cohérence, la geo-distribution des données, la haute-disponibilité, le support SQL, un déploiement facile et une maintenance réduite.

CockroachDB peut être divisé en 5 couches fonctionnelles :
- SQL
- Transaction
- Distribution
- Replication
- Storage


---

# Termes et concepts
<br>

Un <ins>**cluster**</ins> CockroachDB fait référence à un groupe de nodes agissant comme une unité logique unique.<br><br>
Un <ins>**node**</ins> est une machine unique qui exécute une instance de CockroachDB.<br><br>
CockroachDB stocke toutes les données sous forme de paires key-value triées. <br><br>
Ces clés sont divisées en <ins>**ranges**</ins> (plages). <br><br>

---

# Termes et concepts - Ranges
<br>

CockroachDB réplique chaque range et stocke chaque replica sur un nœud différent. <br><br>
Pour chaque range, il y a un <ins>**leaseholder**</ins>, qui agit en tant que propriétaire principal d'un range donné et reçoit et coordonne tout le trafic pour ce range. <br><br>
Pour chaque range, l'un des replicas joue le rôle de <ins>**leader**</ins> pour les demandes d'écriture et s'assure que la majorité des replica ont le corum (consensus) avant d'engager une écriture de donnée. <br><br>
Pour chaque range, il y aura un log des écritures ordonné dans le temps, appelé un <ins>**raft log**</ins>, pour lequel la majorité des replica se sont mis d'accord.

---

# KV Store

Les données sont stockées dans une seule liste triée de clées-valeurs.

Toutes les tables ont une clé primaire.

Il y a une paire clés-valeurs par colonne. Les clés et les valeurs sont des string.

Exemple 

<div grid="~ cols-2 gap-2" m="-t-2">

|id|name|weight|
|-|-|-|
|34|carl|10.1|
|7A|dagne|13.4|
|94|figmen|65.8|

|key|value|
|-|-|
|dog/34/name|carl|
|dog/34/weight|10.1|
|dog/7A/name|dagne|
|dog/7A/weight|13.4|
|dog/94/name|figmen|
|dog/94/weight|65.8|

</div>


---

# Les 5 couches fonctionnelles

- **SQL Layer** est chargée de recevoir les requêtes SQL et les convertir en opérations key-value.
- **Transaction Layer** garantit que les opérations CRUD qui interagissent sur plusieurs paires key-value sont exécutées de manière transactionnelle.
- **Distribution Layer** est responsable de veiller à ce que les ranges soient répartis uniformément entre tous les nodes disponibles d'un cluster.
- **Replication Layer** garantit que les ranges sont répliqués de manière synchrone, chaque fois qu'il y a un changement.
- **Storage Layer** est responsable de la gestion des données key-value sur le disque.

---

# SQL Layer
<br>
Cette couche expose une API SQL et converti les requêtes SQL en requêtes read et write dans la couche key-value.<br>
Ces requêtes sont ensuite passées à la couche Transaction layer.
<br>
<br>

- API SQL : interface utilisateur exposée
- Parser : converti le SQL en AST (abstract syntax tree)
- Optimizer : converti l'AST en un plan d'exécution logique optimisé
- Physical planner : converti le plan logique en un plan d'exécution physique pour être exécuté sur un ou plusieurs nodes du cluster
- SQL execution engine : exécute le plan physique en effectuant des opérations de lecture/écriture

---

# Transaction Layer
<br>

Pour gérer la cohérence des données, CockroachDB exécute toutes les requêtes comme des <ins>transactions</ins>.<br><br>
Une transaction correspond à une ou plusieurs instructions SQL.<br><br>
Ces instructions SQL sont "entourées" des mots-clés **BEGIN** et **COMMIT**.<br><br>
**BEGIN** initialise la transaction, on peut notamment spécifier la priorité<br><br>
**COMMIT** valide la transaction
<br>

```sql
BEGIN;

SELECT id, firstname 
FROM person;

COMMIT;
```

---

# Transaction Layer
<br>

Quand la Transaction Layer exécute une opération d'écriture, elle n'écrit pas directement les informations sur le disque.<br><br>
Elle utilise un système de lock pour "protéger" les enregistrements.<br><br>
Les transactions peuvent avoir un statut : PENDING, STAGING, COMMITTED ou ABORTED.

---

# Distribution layer

Cette couche est responsable de gérer la localisation des données, les différents ranges et leur splitting.

Pour rendre les données accessibles depuis n'importe quel node, CockroachDB stocke les données dans une liste unique de clés-valeurs.

Cette liste contient la description de toutes les données du cluster.

Egalement la localisation des données puisque les données sont stockées dans des ranges.

Cette liste est constituée de 2 éléments :

- meta ranges : la localisation des ranges dans le cluster
- data ranges : les données à proprement parler



---

# Replication layer

Cette couche copie les données entre les nodes et garanti la cohérence entre ces copies.

La haute disponibilité nécessite que la base de données tolère que certains nodes ne soient pas fonctionnels.
Cela sans interruption de service. 
Le fait de répliquer les données entre les nodes permet d'assurer que les données restent accessibles.

CockroachDB utilise un système de consensus / corum pour répondre au besoin de haute-disponibilité.
L'algorithme nécessite qu'un corum soit atteint pour toute modification avant de considérer le changement comme validé.

Un cluster CockroachDB nécessite au moins 3 nodes pour être considéré comme hautement disponible.
Simplement parce que le plus petit nombre pour qu'un corum soit atteint est 3 (ex: 2 sur 3). 

Le nombre de nodes défaillants pouvant être toléré dépend du Replication factor.
La formule est : (Replication factor - 1) / 2.
Avec un Replication factor de 3, on peut avoir 1 node en panne. (3-1)/2 = 1
Avec un Replication factor de 5, on peut avoir 2 nodes en panne. (5-1)/2 = 2

---

# Storage layer

Cette couche écrit et lit les données sur le disque.

Chaque node CockroahDB contient au moins un <ins>**store**</ins>, qui contient les informations lues et écrites sur le disque par CockroachDB.<br><br>
CockroachDB utilise le moteur de stockage Pebble, qui est un peu une black-box...<br><br>
Les données sont stockées sous forme de key-value.<br><br>
Sans rentrer dans le détail, cette couche supporte la gestion des accès concurrents aux données, le <ins>**SQL:2011 standard**</ins> et un système de garbage collector.<br><br>
Pour des informations beaucoup plus détaillées : [Github Pebble](https://github.com/cockroachdb/pebble).

---

# Exécution d'une requête SQL

- Parsing
- Logical planning
- Physical planning: cette phase détermine les nodes sur lesquels les données sont présentes, en particulier le primary owner d'un range. C'est ce qu'on appelle le leaseholder.

| Leaseholder node | Key range | Names |
|-|-|-|
|node 1| [A-G]| Ben, Darnell |
|node 2| [G-L]| Jordan, Kimball |
|node 3| [L-Q]| Peter, Mattis |
|node 4| [Q-Z]| Spencer, Zoey |


---

# Cluster CockroachDB - single node

<br>
docker-compose.yaml

```docker
version: '3.8'

services:
  crdb:
    container_name: crdb
    hostname: crdb
    image: cockroachdb/cockroach:v22.2.7
    ports:
      - "8080:8080"
      - "26257:26257"
    volumes:
      - ../data:/cockroach/cockroach-data
    command: start --cluster-name=cockroach-cluster --insecure --join=crdb
```
<br> 
https://github.com/julien-bresson/cockroachdb

---

# Cluster CockroachDB - healthcheck + initialization
<br>

Vérifier la version du serveur 
```shell
docker exec -it crdb ./cockroach version
```

Initialiser le cluster
```shell
docker exec -it crdb ./cockroach init --insecure --cluster-name=cockroach-cluster
```

Lister les nodes et leurs status
```shell
docker exec -it crdb ./cockroach node ls --insecure
docker exec -it crdb ./cockroach node status --insecure
```

---

# Cluster CockroachDB - Console d'administration
<br>

Afficher la console d'administration CockroachDB<br>
http://localhost:8080

Elle donne une vue d'ensemble sur le cluster et les nodes.<br>

Exploration de cette console

---

# Exploration de la Console d'administration
<br>

- Overview Dashboard : le status des nodes, de la réplication des données, l'uptime et quelques informations hardware (cpu et memory)<br><br>
- Metrics Dashboard : informations sur les métriques, hardware, le node, les sessions et requêtes SQL, le stockage<br><br>
- Databases : les différentes bases de données du cluster, leur taille et le nombre de tables...<br><br>
- SQL Activity : les requêtes fréquemment exécutées, les transactions en cours, les sessions ouvertes<br><br>
- Insights : les problèmes rencontrées comme les requêtes SQL qui échouent et provoquent des "retry", les requêtes lentes...

---

# Exploration de la Console d'administration - suite
<br>


- Hot Ranges : affiche des informations sur les ranges qui sont fortement sollicités (Queries Per Second: read/write), aide au troubleshooting<br><br>
- Jobs : détaille les tâches "long-running" comme des modifications du schema, un import, un backup/restore...<br><br>
- Schedules : informations sur les jobs cadencés comme les bakcup, la récolte des statistiques<br><br>
- Advanced debug : informations pour le monitoring et le troubleshooting

---

# Installer un client

Installation en local de la CLI cockroachDB

[Download the executable](https://www.cockroachlabs.com/docs/v22.2/install-cockroachdb-windows.html#install-binary)

Script powershell a exécuter<br>
Télécharge CockroachDB et ajoute le dossier dézippé au PATH<br>
```shell
$ErrorActionPreference = "Stop"
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$ProgressPreference = 'SilentlyContinue'
$null = New-Item -Type Directory -Force $env:appdata/cockroach
Invoke-WebRequest -Uri https://binaries.cockroachdb.com/cockroach-v22.2.7.windows-6.2-amd64.zip -OutFile cockroach.zip
Expand-Archive -Force -Path cockroach.zip
Copy-Item -Force "cockroach/cockroach-v22.2.7.windows-6.2-amd64/cockroach.exe" -Destination $env:appdata/cockroach
$Env:PATH += ";$env:appdata/cockroach"
```

Ajouter cockroach au PATH
```
$Env:PATH += ";$env:appdata/cockroach"
```

---

# Utiliser le client
<br>

Lister les nodes et leurs status
```shell
cockroach node ls --insecure
cockroach node status --insecure
```

Se connecter au client SQL
```shell
cockroach sql --insecure
```

Exécuter une requête directement
```shell
cockroach sql --execute="CREATE DATABASE mynewdb;" --insecure
cockroach sql --execute="SHOW DATABASES;" --insecure
```

Exécuter un script sql
```shell
cockroach sql --insecure --file myscript.sql
```

---     

# Quelques manipulations
Environnement équivalent à PostgreSQL
<br>

Ajouter une table
```shell
cockroach sql --database=mynewdb --execute="CREATE TABLE person (id serial, firstname varchar(255));" --insecure
```

Insérer des data
```shell
cockroach sql --database=mynewdb --execute="INSERT INTO person (firstname) SELECT md5(random()::text) 
FROM generate_series(1, 5);" --insecure --echo-sql
```

```shell
cockroach sql --database=mynewdb --execute="SELECT * FROM person;" --insecure
```

---     

# EXPLAIN
Plus simple et plus complet que le résultat obtenu avec PostgreSQL
```shell
cockroach sql --database=mynewdb --execute="EXPLAIN SELECT * from person order by firstname desc;" --insecure

```

```sql {all} {maxHeight:'360px'}
EXPLAIN SELECT * FROM person ORDER BY firstname DESC;
                                          info
-----------------------------------------------------------------------------------------
  distribution: full
  vectorized: true

  • sort
  │ estimated row count: 110,008
  │ order: -firstname
  │
  └── • scan
        estimated row count: 110,008 (100% of the table; stats collected 5 minutes ago)
        table: person@person_pkey
        spans: FULL SCAN

  index recommendations: 1
  1. type: index creation
     SQL command: CREATE INDEX ON person (firstname DESC) STORING (id);
(15 rows)


Time: 1ms total (execution 1ms / network 0ms)
```


---     

# EXPLAIN ANALYSE
Juste après avoir inséré 50.000 enregistrements
<br>

```sql
INSERT INTO person (firstname) SELECT md5(random()::text) FROM generate_series(1, 50000)
```

```shell
cockroach sql --database=mynewdb --execute="EXPLAIN ANALYZE SELECT * from person order by firstname desc;" --insecure
```

```sql {all} {maxHeight:'320px'}
                                                                    info
---------------------------------------------------------------------------------------------------------------------------------------------
  planning time: 320µs
  execution time: 195ms
  distribution: full
  vectorized: true
  rows read from KV: 50,005 (4.0 MiB, 1 gRPC calls)
  cumulative time spent in KV: 98ms
  maximum memory usage: 8.4 MiB
  network usage: 2.1 MiB (51 messages)

  • sort
  │ nodes: n3
  │ actual row count: 50,005
  │ estimated max memory allocated: 5.2 MiB
  │ estimated max sql temp disk usage: 0 B
  │ estimated row count: 1
  │ order: -firstname
  │
  └── • scan
        nodes: n3
        actual row count: 50,005
        KV time: 98ms
        KV contention time: 0µs
        KV rows read: 50,005
        KV bytes read: 4.0 MiB
        KV gRPC calls: 1
        estimated max memory allocated: 4.0 MiB
        estimated row count: 1 (100% of the table; stats collected 13 minutes ago)
        table: person@person_pkey  ----------------------  WARNING: the row count estimate is inaccurate, consider running 'ANALYZE person'
        spans: FULL SCAN

  WARNING: the row count estimate on table "person" is inaccurate, consider running 'ANALYZE person'
(31 rows)


Time: 198ms
```
---     

# EXPLAIN ANALYSE
Quelques secondes plus tard, les données statistiques sont à jour
<br>

```sql {all} {maxHeight:'360px'}
                                          info
-----------------------------------------------------------------------------------------
  planning time: 428µs
  execution time: 119ms
  distribution: full
  vectorized: true
  rows read from KV: 50,005 (4.0 MiB, 1 gRPC calls)
  cumulative time spent in KV: 35ms
  maximum memory usage: 8.6 MiB
  network usage: 0 B (0 messages)

  • sort
  │ nodes: n3
  │ actual row count: 50,005
  │ estimated max memory allocated: 5.2 MiB
  │ estimated max sql temp disk usage: 0 B
  │ estimated row count: 50,005
  │ order: -firstname
  │
  └── • scan
        nodes: n3
        actual row count: 50,005
        KV time: 35ms
        KV contention time: 0µs
        KV rows read: 50,005
        KV bytes read: 4.0 MiB
        KV gRPC calls: 1
        estimated max memory allocated: 4.1 MiB
        estimated row count: 50,005 (100% of the table; stats collected 18 seconds ago)
        table: person@person_pkey
        spans: FULL SCAN
(29 rows)


Time: 124ms
```

---

# Variable d'environnement
Permet de variabiliser et factoriser certaines valeurs
<br>

Fichier .env
```shell
CRDB_VERSION=v22.2.7
```

Fichier docker-compose.yaml
```shell {7,8} 
version: '3.8'

services:
  crdb:
    container_name: crdb
    hostname: crdb
    #image: cockroachdb/cockroach:v22.2.7
    image: cockroachdb/cockroach:${CRDB_VERSION}
    ports:
      - "8080:8080"
      - "26257:26257"
    volumes:
      - ../data:/cockroach/cockroach-data
    command: start --cluster-name=cockroach-cluster --insecure --join=crdb
```

---

# Variable d'environnement
<br>
Vue du dossier<br><br>

```
oneNode
│   .env
│   docker-compose.yaml
│
└───data
```
Le dossier data sera crée automatiquement.<br><br>

---

# Cluster CockroachDB - 2 nodes

```shell {all|5,16|8,19|6-7,14|17-18,25|21-22,24,26-27|all} {maxHeight:'420px'}
version: '3.8'

services:

  crdb-1:
    container_name: crdb-1
    hostname: crdb-1
    image: cockroachdb/cockroach:${CRDB_VERSION}
    ports:
      - "8080:8080"
      - "26257:26257"
    volumes:
      - ../data01:/cockroach/cockroach-data
    command: start --cluster-name=cockroach-cluster --insecure --join=crdb-1,crdb-2

  crdb-2:
    container_name: crdb-2
    hostname: crdb-2
    image: cockroachdb/cockroach:${CRDB_VERSION}
    ports:
      - "8081:8080"
      - "26258:26257"
    volumes:
      - ../data02:/cockroach/cockroach-data
    command: start --cluster-name=cockroach-cluster --insecure --join=crdb-1,crdb-2
```

---

# Cluster CockroachDB - Exercice
<br>

- Déployer un cluster CockroachDB avec 3 nodes<br>
- Vérifier qu'il fonctionne correctement<br>
- Créer une base de données<br>
- Ajouter une table<br>
- Insérer des données dans cette table<br>
- Accéder aux données de cette table depuis chacun des 3 containers<br>
- Explorer la console d'administration<br>
- Faire tomber un node et vérifier le comportement (connexion, requête, console, logs...)<br>


https://github.com/julien-bresson/cockroachdb
---

# Cluster CockroachDB - 3 nodes (une solution)
<br>

```shell {all|12,22,34|23-25,28-29,31|all} {maxHeight:'420px'}
version: '3.8'
services:
  crdb-1:
    container_name: crdb-1
    hostname: crdb-1
    image: cockroachdb/cockroach:${CRDB_VERSION}
    ports:
      - "8080:8080"
      - "26257:26257"
    volumes:
      - ../data01:/cockroach/cockroach-data
    command: start --cluster-name=cockroach-cluster --insecure --join=crdb-1,crdb-2,crdb-3
  crdb-2:
    container_name: crdb-2
    hostname: crdb-2
    image: cockroachdb/cockroach:${CRDB_VERSION}
    ports:
      - "8081:8080"
      - "26258:26257"
    volumes:
      - ../data02:/cockroach/cockroach-data
    command: start --cluster-name=cockroach-cluster --insecure --join=crdb-1,crdb-2,crdb-3
  crdb-3:
    container_name: crdb-3
    hostname: crdb-3
    image: cockroachdb/cockroach:${CRDB_VERSION}
    ports:
      - "8082:8080"
      - "26259:26257"
    volumes:
      - ../data03:/cockroach/cockroach-data
    command: start --cluster-name=cockroach-cluster --insecure --join=crdb-1,crdb-2,crdb-3
```


---

# Cluster CockroachDB - 3 nodes
Manipuler la base de données

<br>
Via le client cockroachDB installé (pas recommandé en production)
```shell
cockroach sql --database=mynewdb --execute="SELECT count(*) FROM person;" --insecure
```

En utilisant le client SQL depuis un container
```shell
exec -it crdb-1 ./cockroach cockroach sql --database=mynewdb --execute="SELECT count(*) FROM person;" --insecure

exec -it crdb-2 ./cockroach cockroach sql --database=mynewdb --execute="SELECT count(*) FROM person;" --insecure

exec -it crdb-3 ./cockroach cockroach sql --database=mynewdb --execute="SELECT count(*) FROM person;" --insecure
```

En utilisant un client SQL (ex: DBeaver, Datagrip...)

En utilisant une connection depuis une application, en qualifiant un connection string
```shell
postgresql://<username>@<host>:<port>/<database>?sslmode=disable
```

---

# Cluster CockroachDB - 3 nodes
<br>
Y a-t-il un problème ?

Si oui, lequel ou lesquels ?

---

# Cluster CockroachDB - 3 nodes (problème)
<br>
Les nodes sont exposés avec des ports différents.<br> 
<br>
```shell {3,6,9}
  crdb-1:
    ports:
      - "26257:26257"
  crdb-2:
    ports:
      - "26258:26257"
  crdb-3:
    ports:
      - "26259:26257"
```
<br>
Si on veut se connecter avec un client SQL standard, on se connecte forcément à un node particulier.<br><br>
Si ce node tombe, l'application ne pourra plus accéder à la base de données alors qu'elle fonctionne encore.

---

# Cluster CockroachDB - Load balancer
<br>
La solution est d'utiliser un load balancer.<br><br>

Un load balancer est un service qui distribue le trafic du réseau ou de l'application sur un certain nombre de serveurs.<br><br>
Les load balancer sont utilisés pour augmenter la capacité (utilisateurs simultanés) et la fiabilité des applications.<br><br>
Ils améliorent les performances globales des applications en réduisant la charge des serveurs associés à la gestion et au maintien des applications et des sessions réseau, ainsi qu'en effectuant des tâches spécifiques à l'application.


---

# Cluster CockroachDB - Load balancer
<br>

Les requêtes sont reçues par le load balancer et sont distribuées à un serveur particulier sur la base d'un algorithme configuré.<br><br>
Voici quelques algorithmes standards :
- Round robin
- Round robin pondéré
- Moins de connexions
- Temps de réponse le plus court

Dans le cas présent nous allons utiliser nginx.
Il en existe d'autres : haproxy, traefik...


---

# Cluster CockroachDB - Architecture loadbalancer

<img src="/schema-nginx.svg" width="700" height="600" />


---

# Load balancer - NGINX
<br>

Complément au fichier .env
```shell
NGINX_VERSION=1.23.4
```

Complément au docker-compose.yaml
```shell
  nginx:
    container_name: nginx
    hostname: nginx
    image: nginx:${NGINX_VERSION}
    volumes:
      - ../nginx/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "8080:8080"
      - "26257:26257"
    depends_on:
      - crdb-1
      - crdb-2
      - crdb-3
```

---

# Load balancer - Configuration NGINX

Fichier nginx.conf

<div grid="~ cols-2 gap-2" m="-t-2">

```shell {all}   {maxHeight:'400px'}
events { worker_connections 4096; }
stream {
    upstream crdb-backend {
        server crdb-1:26257 weight=5;
        server crdb-2:26257;
        server crdb-3:26257;
    }
    server {
        listen 26257;
        proxy_pass crdb-backend;
    }
    upstream crdb-ui {
        server crdb-1:8080;
        server crdb-2:8080;
        server crdb-3:8080;
    }
    server {
        listen 8080;
        proxy_pass crdb-ui;
    }
}
```
Les requêtes qui arrivent sur le port 26257 est transmis au composant crdb-backend<br><br>
Sur 7 requêtes :<br>
- 5 iront vers le node 1<br>
- 1 vers le node 2<br>
- 1 vers le node 3<br><br>
Ce composant correspond aux 3 nodes exposés sur le port 26257 (par défaut)<br><br>
Les requêtes qui arrivent sur le port 8080 sont transmises au composant crdb-ui<br><br>
Ce composant correspond aux 3 consoles d'admin (DB Console) exposées sur le port 8080 (par défaut)<br><br>

</div>

---

# Cluster CockroachDB - 3 nodes avec un load balancer
<br>
Vue du dossier<br><br>

```
threeNode
│   .env
│   docker-compose.yaml
│
└───nginx
│   │   nginx.conf
│
└───data01
│
└───data02
│
└───data03
```

---

# Analyse des données de la console 
<br>
Après avoir effectué quelques requêtes SQL, on peut analyser le comportement du cluster depuis la console d'administration.<br><br>
Vu précédemment, la console d'administration CockroachDB est accessible via l'url http://localhost:8080<br><br>
Depuis la page d'accueil vérifier que le cluster fonctionne normalement (tous les nodes ont le statut 'live').<br><br>
Depuis l'onglet Metrics, par défaut on voit le dashboard Overview, sélectionner le dashboard SQL.<br><br>
On peut voir les différents types de requêtes dans le graphique SQL Statements.<br><br>
Par défaut les informations du cluster sont affichées, il est possible de filtrer par node.<br><br>
Dans le Dashboard Distributed, on voit les requêtes en fonction des nodes.<br><br>

---

# Metrics - Dashboard: SQL
Sur ce dashboard SQL, on voit les différents types de requêtes exécutées sur le cluster.<br><br>

<img src="/02dashboardSQL.png" width="700" height="600" />

---

# Metrics - Dashboard: Distributed
Sur ce dashboard Distributed, on voit toutes les requêtes (qui sont des transactions) sur le cluster.<br><br>

<img src="/03dashboardDistributedcluster.png" width="700" height="600" />

---

# Metrics - Dashboard: Distributed node 1
Sur ce dashboard Distributed, on voit uniquement les requêtes (qui sont des transactions) sur le 1er node.<br><br>

<img src="/04dashboardDistributednode1.png" width="700" height="600" />


---

# Metrics - Dashboard: Distributed node 3
Sur ce dashboard Distributed, on voit uniquement les requêtes (qui sont des transactions) sur le 3ème node.<br><br>

<img src="/05dashboardDistributednode3.png" width="700" height="600" />



---

# Workload
<br>
Le workload est une fonctionnalité fournie par CockroachDB (non supportée) permettant d'effectuer des opérations sur une base de données fictive.

Il existe différents workloads.

|Nom|Description|
|-|-|
|intro|Base de données 'intro' avec un message caché|
|movr|Simule un workload pour l'application exemple [MOVR](https://www.cockroachlabs.com/docs/v22.2/movr)|
|kv|Lit et écrit une table key value|
|tpcc|Basé sur le protocole [TPC-C](https://www.tpc.org/tpcc/)|


---

# Workload - Utilisation
<br>

Il faut initialiser la base de données via la commande <ins>"workload init"</ins>
```shell
cockroach workload init <name>
```

Puis lancer des requêtes SQL via la commande <ins>"workload run"</ins>
```shell
cockroach workload run <name> <connection string>
```

---

# Workload MOVR init

Initialisation du workload (via docker ou la CLI)

```shell
cockroach workload init movr 'postgresql://root@localhost:26257?sslmode=disable'
```

Log d'exécution
```
1 workload/cli/run.go:622  [-] 1  random seed: 3604441918269522183
1 ccl/workloadccl/fixture.go:318  [-] 2  starting import of 6 tables
43 ccl/workloadccl/fixture.go:481  [-] 3  imported 411 B in user_promo_codes table (5 rows, 0 index entries, took 2.351408044s, 0.00 MiB/s)
42 ccl/workloadccl/fixture.go:481  [-] 4  imported 220 KiB in promo_codes table (1000 rows, 0 index entries, took 2.366369474s, 0.09 MiB/s)
38 ccl/workloadccl/fixture.go:481  [-] 5  imported 4.8 KiB in users table (50 rows, 0 index entries, took 2.368735817s, 0.00 MiB/s)
39 ccl/workloadccl/fixture.go:481  [-] 6  imported 3.2 KiB in vehicles table (15 rows, 15 index entries, took 2.57014203s, 0.00 MiB/s)
41 ccl/workloadccl/fixture.go:481  [-] 7  imported 72 KiB in vehicle_location_histories table (1000 rows, 0 index entries, took 2.570020797s, 0.03 MiB/s)
40 ccl/workloadccl/fixture.go:481  [-] 8  imported 153 KiB in rides table (500 rows, 1000 index entries, took 2.660990282s, 0.06 MiB/s)
1 ccl/workloadccl/fixture.go:326  [-] 9  imported 453 KiB bytes in 6 tables (took 2.920366453s, 0.15 MiB/s)
1 workload/workloadsql/workloadsql.go:136  [-] 10  starting 8 splits
1 workload/workloadsql/workloadsql.go:136  [-] 11  starting 8 splits
1 workload/workloadsql/workloadsql.go:136  [-] 12  starting 8 splits
```


---

# Workload MOVR - Modèle de données

<img src="/movr-model.png" width="650" height="600" />

---

# Workload MOVR - Contenu

```sql
cockroach sql --database=movr --insecure

root@localhost:26257/movr> \dt
  schema_name |         table_name         | type  | owner | estimated_row_count | locality
--------------+----------------------------+-------+-------+---------------------+-----------
  public      | promo_codes                | table | root  |                1000 | NULL
  public      | rides                      | table | root  |                 500 | NULL
  public      | user_promo_codes           | table | root  |                  13 | NULL
  public      | users                      | table | root  |                  83 | NULL
  public      | vehicle_location_histories | table | root  |                1830 | NULL
  public      | vehicles                   | table | root  |                  30 | NULL
(6 rows)


Time: 28ms total (execution 27ms / network 1ms)
```

---

# Workload MOVR - Fonctionnement
Source: https://github.com/cockroachdb/cockroach/blob/master/pkg/workload/movr/workload.go

> Our workload is as follows: with 95% chance, do a simple read operation.<br>
Else, update all active vehicle locations, then pick a random "write" operation<br>
weighted by the weights in movrWorkloadFns.
<br>

La configuration des différents Weights
```shell
readPercentage = 0.95
createPromoCode: 0.03
applyPromoCode: 0.1
addUser: 0.3
addVehicle: 0.1
startRide: 0.4
endRide: 0.07
```

---

# Workload MOVR - Fonctionnement

Requêtes SQL exécutées
```sql
addUser: INSERT INTO users VALUES ($1, $2, $3, $4, $5)

applyPromoCode: INSERT INTO user_promo_codes VALUES ($1, $2, $3, now(), 1)

readVehicles: SELECT city, id FROM vehicles WHERE city = $1

startRide: INSERT INTO rides VALUES ($1, $2, $2, $3, $4, $5, NULL, now(), NULL, $6)

updateActiveRides: UPSERT INTO vehicle_location_histories VALUES ($1, $2, now(), $3, $4)

addUser: INSERT INTO users VALUES ($1, $2, $3, $4, $5)

endRide: UPDATE rides SET end_address = $3, end_time = now() WHERE city = $1 AND id = $2
```

---

# Workload MOVR - Exécution


```shell
cockroach workload run movr --duration=30s 'postgresql://root@localhost:26257?sslmode=disable'

cockroach workload run movr --duration=10s --num-vehicles=100 'postgresql://root@localhost:26257?sslmode=disable'
```

Log d'exécution
```shell  
 elapsed   errors  ops/sec(inst)   ops/sec(cum)  p50(ms)  p95(ms)  p99(ms) pMax(ms)
    1.0s        0            4.0            4.0      6.0      7.1      7.1      7.1 addUser
    1.0s        0            1.0            1.0    100.7    100.7    100.7    100.7 applyPromoCode
    1.0s        0          186.8          186.9      0.8      1.8     58.7     79.7 readVehicles
    1.0s        0            4.0            4.0     58.7    104.9    104.9    104.9 startRide
    1.0s        0            9.0            9.0      0.1     92.3     92.3     92.3 updateActiveRides
    2.0s        0            2.0            3.0     10.0     14.7     14.7     14.7 addUser
    2.0s        0            0.0            0.5      0.0      0.0      0.0      0.0 applyPromoCode
    2.0s        0            0.5            0.5      9.4      9.4      9.4      9.4 endRide
    2.0s        0          178.9          182.9      1.0      1.6     46.1     67.1 readVehicles
    2.0s        0            2.0            3.0     17.8     60.8     60.8     60.8 startRide
    2.0s        0            5.0            7.0     96.5    192.9    192.9    192.9 updateActiveRides
```

---

# Gestion des logs
<br>
Afficher le paramètre d'activation des logs (à false par défaut)
```sql
SHOW CLUSTER SETTING sql.trace.log_statement_execute;
```
<br><br>

Activer les logs
```sql
SET CLUSTER SETTING sql.trace.log_statement_execute = true;
```
<br><br>

Les fichiers de log seront écrits dans un sous-dossier "/logs" du dossier "/data" d'un des nodes.<br>
Template du nom des fichiers de log:<br>
cockroach-sql-exec._nodename_._user_._timestamp_._number_.log

---

# Gestion des slow query
<br>
Afficher le seuil d'une slow query
```sql
SHOW CLUSTER SETTING sql.log.slow_query.latency_threshold;
```
<br><br>

Modifier le seuil d'une slow query
```sql
SET CLUSTER SETTING sql.log.slow_query.latency_threshold = '100ms';
```
<br><br>
Les requêtes qui dépasseront ce seuil seront tracées dans un fichier de log spécifique.<br>
Template du nom des fichiers de log pour les slow query:<br>
cockroach-sql-slow._nodename_._user_._timestamp_._number_.log


---

# Cluster CockroachDB - Monitoring
<br>

Comme vu précédemment, la console d'administration CockroachDB permet de visualiser le comportement du cluster.<br>

Dans la page Advanced Debug (http://localhost:8080/#/debug), il y a une section <ins>Metrics</ins> et <ins>Prometheus</ins>

Cette page affiche la liste des <ins>metrics</ins> Prometheus exposées par le node
(http://localhost:8080/_status/vars)

Prometheus est un solution Open source de monitoring et alerting.


---

# Monitoring - Metrics
<br>

Quelques métriques sur un premier node - http://localhost:8080/_status/vars
```shell {all} {maxHeight:'200px'}
HELP node_id node ID with labels for advertised RPC and HTTP addresses
# TYPE node_id gauge
node_id{advertise_addr="crdb-2:26257",http_addr="crdb-2:8080",sql_addr="crdb-2:26257"} 3

# HELP sql_select_count Number of SQL SELECT statements successfully executed
# TYPE sql_select_count counter
sql_select_count 11

# HELP sys_uptime Process uptime
# TYPE sys_uptime gauge
sys_uptime 10273

# HELP ranges Number of ranges
# TYPE ranges gauge
ranges{store="3"} 45
```

Les mêmes métriques sur un 2ème node - http://localhost:8080/_status/vars (avec un autre browser)
```shell
node_id{advertise_addr="crdb-1:26257",http_addr="crdb-1:8080",sql_addr="crdb-1:26257"} 1
sql_select_count 10
sys_uptime 10867
ranges{store="1"} 45
```


---

# Inconvénients d'un système distribué
<br>
Par principe il est réparti sur plusieurs ressources, ce qui augmente la complexité de sa gestion.<br><br>
Les systèmes distribués sont constitués de multiples composants basés sur des cycles de vies différents, utilisant des technologies qui évoluent constamment.<br><br>
Et donc les équipes en charge de leur exploitation ont besoin de connaissances approfondies de ces systèmes pour les exploiter correctement en production.<br><br>
Le dépannage d’un système distribué peut souvent être une phase compliquée pour les OPS et les DEV.<br><br>
Parce qu’il faut diagnostiquer plusieurs systèmes en simultané, pour pouvoir détecter les anomalies.<br><br>
Un autre inconvénient est le coût associé à la plate-forme.<br><br>
Qu’il s’agisse des ressources informatiques nécessaires mais aussi les ressources humaines.

---

# Besoins d'un système distribué
<br>


Il est donc crucial de disposer d’une plate-forme d’observabilité fiable afin d’accéder rapidement à toutes les données nécessaires au diagnostic et à l’optimisation dans tous les domaines et à plusieurs niveaux du système distribué.<br><br>
De nouvelles méthodes pour trouver et résoudre efficacement les problèmes pour limiter les temps d’arrêt et atténuer le risque d’impact négatif sur l’entreprise.<br><br>
Une mauvaise expérience utilisateur, par exemple, peut nuire à la réputation de l’entreprise et entrainer une perte de revenus ou à la perte de clients potentiels, voire de clients existants.<br><br>
L’observabilité est un élément essentiel de toute architecture pour gérer efficacement un système, déterminer s’il fonctionne correctement et décider ce qui doit être corrigé, modifié ou amélioré.

---

# Monitoring et Observability
<br>

Les termes monitoring et observability sont souvent utilisés de manière interchangeable.<br><br>
Cependant, ils ont des significations distinctes et des objectifs différents dans leur utilisations en entreprise.<br>
- Une plateforme de **Monitoring** obtient un état d'un système sur la base d'un ensemble prédéfini de mesures afin de détecter un ensemble de problèmes connus.
- L'**observability** vise à mesurer la compréhension de l'état d'un système sur la base de multiples sources, ce qui signifie qu'il s'agit d'une brique qui doit être conçue et mise en œuvre dès de la phase initiale du projet, avant même de coder, builder et tester.

Dans une logique DEVSECOPS, l’Observability est la responsabilité de tous.<br><br>
Elle nécessite la collaboration de tous les acteurs d’un projet.



---

# Prometheus

<br>
Prometheus est un outil de monitoring qui va récupérer des mesures fournies par différents services (db, linux, docker, application...).<br><br>
Les mesures sont scrapées à intervalles réguliers pour former une time series.<br><br>
Concrètement on a un serveur Prometheus et des "prometheus exporter" embarqués dans les services à monitorer (ou placés à coté de ces services, on parle de sidecar).<br><br>
Il est très fréquemment associé à Grafana pour exploiter ces mesures.<br><br>

Il a donc 3 fonctions : 
- Collecter les données
- Stocker les données
- Alerting
<br>

Site officiel : prometheus.io<br>
Github : github.com/prometheus/prometheus<br>

---

# Grafana
<br>
Grafana est un outil de visualisation des données.<br><br>
A l'origine pour visualiser des métriques.
Il est très fréquemment associé à Prometheus.<br><br>
Il existe des extensions pour visualiser des logs (Loki) ou des traces (Tempo).<br><br>

Les principes de Grafana :
- Datasources
- Dashboards
- Alerting
<br>

Site officiel : grafana.com/grafana<br>
Github : github.com/grafana/grafana<br>

---

# Cluster CockroachDB - Architecture 

<img src="/schemaCRDB.svg" width="700" height="600" />

---

# Cluster CockroachDB - Exposition des ports

<img src="/schema-prometheus-01.svg" width="700" height="600" />

---

# Prometheus + Grafana - Configuration
<br>
Prometheus est configuré via un fichier prometheus.yml qui est monté en volume. Ce fichier est lu au démarrage.<br><br>

Pour Grafana, nous allons charger la datasource pour lui indiquer de lire les données de Prometheus.<br><br>
Nous allons également charger automatiquement des dashboards pré-configurés.<br><br>
Documentation : [Visualize metrics in grafana](www.cockroachlabs.com/docs/stable/monitor-cockroachdb-with-prometheus.html#step-5-visualize-metrics-in-grafana)<br><br>
Les dashboards sont disponibles sur le [github de cockroachdb](github.com/cockroachdb/cockroach/tree/master/monitoring/grafana-dashboards).<br><br>


---

# Cluster 3 nodes + load balancer + monitoring
<br>
Vue du dossier<br><br>

```
threeNode
│   .env
│   docker-compose.yaml
└───nginx
│   │   nginx.conf
└───prometheus
│   │   prometheus.yml
└───grafana
│   └───provisioning
│       └───dashboards
│           │   CRDB overview.json
│           │   hardware.json
│           │   runtime.json
│           │   sql.json
│           │   CockroachDB Custom.json
│       └───datasources
│           │   datasource.yml
└───data01
└───data02
└───data03
```

---

# Monitoring - Prometheus + Grafana
<br>
Fichier .env
```shell
PROMETHEUS_VERSION=v2.43.0
GRAFANA_VERSION=9.5.1
```

Complément au docker-compose.yaml
```shell {all} {maxHeight:'300px'}
  prometheus:
    container_name: prometheus
    hostname: prometheus
    image: prom/prometheus:${PROMETHEUS_VERSION}
    command: --config.file=/etc/prometheus/prometheus.yml
    volumes:
      - ../prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ../prometheus-data:/prometheus/data
    ports:
      - 9090:9090

  grafana:
    container_name: grafana 
    hostname: grafana
    image: grafana/grafana-oss:${GRAFANA_VERSION}
    volumes:
      - ../grafana/provisioning:/etc/grafana/provisioning/
      - ../grafana/data:/var/lib/grafana
    ports:
      - 3000:3000
    depends_on:
      - prometheus
```

---

# Monitoring - Configuration Prometheus
<br>

Fichier prometheus.yml
```shell {all} {maxHeight:'350px'}
global:
  scrape_interval: 1s
  scrape_timeout: 1s
  evaluation_interval: 1m
  
scrape_configs:
  - job_name: 'cockroachdb'
    metrics_path: '/_status/vars'
    scheme: 'http'
    tls_config:
      insecure_skip_verify: true
    static_configs:
    - targets: ['crdb-1:8080', 'crdb-2:8080', 'crdb-3:8080']
      labels:
        cluster: 'my-cluster'
```


---

# Grafana - datasources
<br>
Fichier datasource.yml<br><br>

>Penser à changer l'adresse IP pour l'url de Prometheus à scraper.


```shell {all} {maxHeight:'350px'}
# config file version
apiVersion: 1

# list of datasources that should be deleted from the database
deleteDatasources:
  - name: Prometheus
    orgId: 1

# list of datasources to insert/update depending
# whats available in the database
datasources:

    # <string, required> name of the datasource. Required
  - name: Prometheus
    # <string, required> datasource type. Required
    type: prometheus
    # <string, required> access mode. direct or proxy. Required
    access: proxy
    # <int> org id. will default to orgId 1 if not specified
    orgId: 1
    # <string> url
    url: http://172.18.32.1:9090
    # <bool> enable/disable basic auth
    basicAuth: false
    # <bool> enable/disable with credentials headers
    withCredentials: false
    # <bool> mark as default datasource. Max one per org
    isDefault: true
    # <map> fields that will be converted to json and stored in json_data
    jsonData:
      httpMethod: "POST"
      prometheusType: "Prometheus"
      prometheusVersion: "2.40.1"
    # <string> json object of data that will be encrypted.
    secureJsonFields: {}
    version: 1
    # <bool> allow users to edit datasources from the UI.
    editable: true
```


---

# Prometheus - Healthcheck
<br>
Pour vérifier que Prometheus récupère bien les informations.<br><br>
Se rendre sur l'url http://localhost:9090<br><br>
Puis dans l'onglet Status et sélectionner Targets.<br><br>
Ou bien se rendre sur l'url http://localhost:9090/targets
Cela permet de voir les endpoints qui sont configurés pour être scrappé par Prometheus.<br><br>

Les nodes CockroachDB doivent apparaître, avec un state UP.

---

# Prometheus - Healthcheck

<img src="/prometheus-targets.png" width="1000" height="600" />

---

# Prometheus - Requêtes
<br>
Pour exécuter une requête sur prometheus, aller dans l'onglet Graph<br><br>
Se rendre sur l'url http://localhost:9090/graph<br><br>
Dans le champ de recherche, saisir la métrique ci-dessous et exécuter la requête.<br><br>
L'auto-complétion peut aider...<br><br>
```shell
liveness_livenodes
```
<br><br>
Ajouter un panel est chercher la métrique suivante<br><br>
```shell
sql_select_count
```
<br><br>
Exécuter plusieurs requêtes avec la commande cockroach sql puis vérifier ce qu'il se passe dans prometheus<br><br>

---

# Prometheus - Requêtes

<img src="/prometheus-queries.png" width="1000" height="600" />

---

# Monitoring - Dashboards Grafana
<br> 
Liste disponible sur Github :<br> 

(https://github.com/cockroachdb/cockroach/tree/master/monitoring/grafana-dashboards)

Les dashboards intéressants :
 - CRDB Console : Overview
 - CRDB Console : Hardware 
 - CRDB Console : Runtime
 - CRDB Console : SQL

<br>
Pour accéder à notre serveur Grafana : localhost:3000<br>
Se connecter avec admin/admin.<br>

---

# Dashboards Grafana - CRDB Console : Overview
<br> 

SQL Queries<br>
>A ten-second moving average of the # of SELECT, INSERT, UPDATE, and DELETE statements successfully executed per second across all nodes.<br>
```shell
sum(rate(sql_select_count{job="cockroachdb",cluster=~"$cluster",instance=~"$node"}[$__rate_interval]))
sum(rate(sql_update_count{job="cockroachdb",cluster=~"$cluster",instance=~"$node"}[$__rate_interval]))
sum(rate(sql_delete_count{job="cockroachdb",cluster=~"$cluster",instance=~"$node"}[$__rate_interval]))
sum(rate(sql_insert_count{job="cockroachdb",cluster=~"$cluster",instance=~"$node"}[$__rate_interval]))
```
<br>

Les requêtes écritent dans Grafana contiennent des paramètres pour filtrer l'affichage. A retenir:
```shell
rate(sql_select_count[rate_interval])
rate(sql_update_count[rate_interval])
rate(sql_delete_count[rate_interval])
rate(sql_insert_count[rate_interval])
```

---

# Dashboards Grafana - Explications des métriques utilisées

<br> 
On trouve plus d'info sur les métriques sur le endpoint où sont exposées les métriques Prometheus<br><br>
http://localhost:8080/_status/vars<br><br>

```shell
# HELP sql_select_count 
Number of SQL SELECT statements successfully executed
# TYPE sql_select_count counter

# HELP sql_update_count 
Number of SQL UPDATE statements successfully executed
# TYPE sql_update_count counter
...
```

---

# Dashboards Grafana - CRDB Console : Overview
<br>

Replicas per Node<br>
>The number of range replicas stored on this node. Ranges are subsets of your data, which are replicated to ensure survivability.<br>
```shell
replicas
```
<br><br>
Capacity<br>

> Usage of disk space

```shell
capacity by (cluster)
capacity_available by (cluster)
capacity_used by (instance)
```

---

# Dashboards Grafana - CRDB Console : Hardware 
<br>
 
CPU Percent<br>
```shell
sys_cpu_combined_percent_normalized
```
<br><br>

Memory Usage<br>
>Memory in use across all nodes<br>
```shell
sys_rss
```
<br><br>

Disk Read B/s<br>
```shell
rate(sys_host_disk_read_bytes[rate_interval])
```

---

# Dashboards Grafana - CRDB Console : Runtime
<br>

Live Node Count<br>
>The number of live nodes in the cluster.<br>
```shell
liveness_livenodes
```

---

# Dashboards Grafana - CRDB Console : SQL
<br>
SQL Statements<br>

>A moving average of the # of SELECT, INSERT, UPDATE, and DELETE statements successfully executed per second.<br>

```shell
rate(sql_select_count[rate_interval])
rate(sql_update_count[rate_interval])
rate(sql_insert_count[rate_interval])
rate(sql_delete_count[rate_interval])
```

---

# Workload TPCC init

Initialisation du workload (via la CLI)

```shell
cockroach workload init tpcc
```

Log d'exécution
```
1 workload\cli\run.go:622  [-] 1  random seed: 10050546446357955770
1 ccl\workloadccl\fixture.go:318  [-] 2  starting import of 9 tables
22 ccl\workloadccl\fixture.go:481  [-] 3  imported 1020 B in district table (10 rows, 0 index entries, took 2.4017399s, 0.00 MiB/s)
21 ccl\workloadccl\fixture.go:481  [-] 4  imported 54 B in warehouse table (1 rows, 0 index entries, took 4.9855528s, 0.00 MiB/s)
26 ccl\workloadccl\fixture.go:481  [-] 5  imported 123 KiB in new_order table (9000 rows, 0 index entries, took 22.7979856s, 0.01 MiB/s)
24 ccl\workloadccl\fixture.go:481  [-] 6  imported 2.2 MiB in history table (30000 rows, 0 index entries, took 22.9980645s, 0.10 MiB/s)
25 ccl\workloadccl\fixture.go:481  [-] 7  imported 1.6 MiB in order table (30000 rows, 30000 index entries, took 25.0875549s, 0.06 MiB/s)
29 ccl\workloadccl\fixture.go:481  [-] 8  imported 17 MiB in order_line table (300704 rows, 0 index entries, took 28.7927678s, 0.58 MiB/s)
23 ccl\workloadccl\fixture.go:481  [-] 9  imported 18 MiB in customer table (30000 rows, 30000 index entries, took 33.9559023s, 0.52 MiB/s)
28 ccl\workloadccl\fixture.go:481  [-] 10  imported 31 MiB in stock table (100000 rows, 0 index entries, took 36.0887094s, 0.85 MiB/s)
27 ccl\workloadccl\fixture.go:481  [-] 11  imported 7.9 MiB in item table (100000 rows, 0 index entries, took 36.583392s, 0.22 MiB/s)
1 ccl\workloadccl\fixture.go:326  [-] 12  imported 77 MiB bytes in 9 tables (took 37.142483s, 2.07 MiB/s)
```

---

# Workload TPCC - Modèle de données

<img src="/tpcc-model.png" width="800" height="600" />

---

# Workload TPCC - Contenu

```sql
cockroach sql --database=tpcc --insecure

root@localhost:26257/tpcc> \dt
  schema_name | table_name | type  | owner | estimated_row_count | locality
--------------+------------+-------+-------+---------------------+-----------
  public      | customer   | table | root  |               30000 | NULL
  public      | district   | table | root  |                  10 | NULL
  public      | history    | table | root  |               30000 | NULL
  public      | item       | table | root  |              100000 | NULL
  public      | new_order  | table | root  |                9000 | NULL
  public      | order      | table | root  |               30000 | NULL
  public      | order_line | table | root  |              300704 | NULL
  public      | stock      | table | root  |              100000 | NULL
  public      | warehouse  | table | root  |                   1 | NULL
(9 rows)

Time: 25ms total (execution 35ms / network 25ms)
```

---

# Workload TPCC init - avec plus de data

On ajoute le paramètre --warehouse

```shell
cockroach workload init tpcc --warehouses=3
```

Log d'exécution
```
1 workload\cli\run.go:622  [-] 1  random seed: 9953192450608402710
1 ccl\workloadccl\fixture.go:318  [-] 2  starting import of 9 tables
66 ccl\workloadccl\fixture.go:481  [-] 3  imported 92 MiB in stock table (300000 rows, 0 index entries, took 28.5929474s, 3.22 MiB/s)
62 ccl\workloadccl\fixture.go:481  [-] 4  imported 6.5 MiB in history table (90000 rows, 0 index entries, took 28.6888709s, 0.23 MiB/s)
65 ccl\workloadccl\fixture.go:481  [-] 5  imported 7.9 MiB in item table (100000 rows, 0 index entries, took 28.906299s, 0.27 MiB/s)
63 ccl\workloadccl\fixture.go:481  [-] 6  imported 4.7 MiB in order table (90000 rows, 90000 index entries, took 28.9110498s, 0.16 MiB/s)
64 ccl\workloadccl\fixture.go:481  [-] 7  imported 369 KiB in new_order table (27000 rows, 0 index entries, took 28.982853s, 0.01 MiB/s)
60 ccl\workloadccl\fixture.go:481  [-] 8  imported 3.0 KiB in district table (30 rows, 0 index entries, took 28.9831229s, 0.00 MiB/s)
61 ccl\workloadccl\fixture.go:481  [-] 9  imported 53 MiB in customer table (90000 rows, 90000 index entries, took 29.4151398s, 1.79 MiB/s)
59 ccl\workloadccl\fixture.go:481  [-] 10  imported 158 B in warehouse table (3 rows, 0 index entries, took 29.5829191s, 0.00 MiB/s)
67 ccl\workloadccl\fixture.go:481  [-] 11  imported 50 MiB in order_line table (900385 rows, 0 index entries, took 29.9137225s, 1.68 MiB/s)
1 ccl\workloadccl\fixture.go:326  [-] 12  imported 214 MiB bytes in 9 tables (took 33.00921s, 6.50 MiB/s)
```


---

# Workload TPCC - Contenu
Le contenu est plus conséquent

```sql
cockroach sql --database=tpcc --insecure

root@localhost:26257/tpcc> \dt
  schema_name | table_name | type  | owner | estimated_row_count | locality
--------------+------------+-------+-------+---------------------+-----------
  public      | customer   | table | root  |               90000 | NULL
  public      | district   | table | root  |                  30 | NULL
  public      | history    | table | root  |               90000 | NULL
  public      | item       | table | root  |              100000 | NULL
  public      | new_order  | table | root  |               27000 | NULL
  public      | order      | table | root  |               90000 | NULL
  public      | order_line | table | root  |              900385 | NULL
  public      | stock      | table | root  |              300000 | NULL
  public      | warehouse  | table | root  |                   3 | NULL
(9 rows)

Time: 641ms total (execution 636ms / network 5ms)
```

---

# Exercice
<br>

- Tester les workload movr et tpcc
- Exécuter le workload pendant plusieurs minutes
- Faire tomber un ou plusieurs node pendant que le workload tourne
  - Penser au paramètre : server.time_until_store_dead
- Analyser le comportement du cluster
- Parcourir l'url http://localhost:8080/_status/vars pour identifier des métriques intéressantes

- Créer un dashboard Grafana avec le nombre de nodes alive

---

# EXPLAIN ANALYZE DEBUG
<br>

Dans la db MOVR, afficher le EXPLAIN ANALYZE de la requête suivante :
```sql
EXPLAIN ANALYZE (DEBUG) SELECT * 
FROM public.rides a LEFT JOIN public.USERS b ON a.rider_id = b.id
LEFT JOIN public.vehicles c ON a.vehicle_id = c.id
WHERE b.city = 'san francisco'
ORDER BY b.name;
                                      info
--------------------------------------------------------------------------------
  Statement diagnostics bundle generated. Download from the Admin UI (Advanced
  Debug -> Statement Diagnostics History), via the direct link below, or using
  the SQL shell or command line.
  Admin UI: http://crdb-3:8080
  Direct link: http://crdb-3:8080/_admin/v1/stmtbundle/861616635918876675
  SQL shell: \statement-diag download 861616635918876675
  Command line: cockroach statement-diag download 861616635918876675
(7 rows)
```
---

# EXPLAIN ANALYZE DEBUG
<br>

```sql
                                      info
--------------------------------------------------------------------------------
  Statement diagnostics bundle generated. Download from the Admin UI (Advanced
  Debug -> Statement Diagnostics History), via the direct link below, or using
  the SQL shell or command line.
  Admin UI: http://crdb-3:8080
  Direct link: http://crdb-3:8080/_admin/v1/stmtbundle/861616635918876675
  SQL shell: \statement-diag download 861616635918876675
  Command line: cockroach statement-diag download 861616635918876675
(7 rows)
```

Utiliser le direct link en remplaçant crdb-3 par localhost.<br>
Afficher la page distsql.html.<br>
Puis Heatmap and Layered Graph View et enfin Graph.<br>
On voit ici que les différentes parties du plan d'exécution ont été exécutées sur plusieurs nodes.<br>
Ceci est géré automatiquement par CockroadhDB (et pas par NGINX), afin de répartir la charge.<br>


---

# END

---

# See more
<br>

[Multi-Region Capabilities](https://www.cockroachlabs.com/docs/v22.2/multiregion-overview.html)
cockroachlabs.com/docs/v22.2/multiregion-overview.html
Geo-partitioning
Geo-replication
...

[Alerts rules in Prometheus](github.com/cockroachdb/cockroach/blob/master/monitoring/rules/alerts.rules.yml)<br>
github.com/cockroachdb/cockroach/blob/master/monitoring/rules/alerts.rules.yml
<br><br>
[Try k6 SQL](github.com/grafana/xk6-sql)<br>
github.com/grafana/xk6-sql
<br><br>
[Quarkus - Simplified Hibernate ORM with Panache](quarkus.io/guides/hibernate-orm-panache)<br>
quarkus.io/guides/hibernate-orm-panache
<br><br>
Jaeger tracing
<br><br>
CockroachDB + Jaeger
