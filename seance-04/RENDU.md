# Rendu Séance 4 - Infrastructure as Code avec Terraform

## Résumé de la séance
L'objectif était d'automatiser le déploiement de l'infrastructure Docker via Terraform. Nous avons appris à gérer le cycle de vie complet (init, plan, apply, destroy) et à industrialiser le code en utilisant des variables et une gestion sécurisée des secrets.

## Étapes accomplies
1. **Initialisation** : Installation de Terraform et configuration du provider Docker.
2. **Infrastructure** : Création d'un réseau, d'un volume persistant et du conteneur MinIO.
3. **Paramétrage** : Refactoring pour utiliser `variables.tf` et isoler les secrets dans `terraform.tfvars`.
4. **Idempotence** : Vérification que Terraform ne recrée pas les ressources inutilement.

## Captures d'écran
![alt text](captures/terraform-plan2.png)
- `terraform-plan.png` : Vérification du plan avant déploiement.

![alt text](captures/terraform-apply2.png)
- `terraform-apply.png` : Déploiement réussi de la stack.

![alt text](captures/console-minio-tf.png)
- `console-minio-tf.png` : Accès à l'interface de gestion MinIO.

![alt text](captures/terraform-destroy2.png)
- `terraform-destroy.png` : Nettoyage propre de l'infrastructure.

## Réponses aux questions (optionnel du TP)
* **Pourquoi ne pas commit le .tfstate ?** Parce qu'il contient des informations sensibles en clair sur les ressources (mots de passe, adresses IP).
* **Utilité des variables ?** Elles permettent de rendre le code réutilisable (ex: changer le port ou le nom du conteneur) sans modifier le cœur de la logique métier.


## Réponses aux exercices d'application

### Exercice 1 - QCM conceptuel
* **1.1 :** B (L'IaC ne remplace pas la compréhension de l'infrastructure, elle l'automatise).
* **1.2 :** B (Déclaratif = état final souhaité ; Impératif = étapes à suivre).
* **1.3 :** B (Idempotence = résultat identique peu importe le nombre d'exécutions).
* **1.4 :** B (Le provider est le traducteur entre Terraform et l'API d'un service/cloud).
* **1.5 :** B (Terraform compare le code au state et ne fait rien si tout est conforme).
* **1.6 :** C (Le state mémorise le mapping entre le code et les ressources réelles).
* **1.7 :** B (Le fichier contient des secrets en clair et peut corrompre le déploiement en équipe).
* **1.8 :** C (`terraform plan`).
* **1.9 :** B (Fork open source suite au changement de licence HashiCorp).
* **1.10 :** B (Complémentaires : Terraform provisionne les ressources, Ansible les configure).

### Exercice 2 - Lecture et interprétation
* **2.1 :** - `docker_network.back` : crée le réseau privé.
  - `docker_volume.data` : crée le volume de stockage persistant.
  - `docker_image.postgres` : télécharge l'image Docker de Postgres.
  - `docker_container.db` : crée et lance le conteneur de base de données.
* **2.2 :** C'est une référence dynamique à l'ID de l'image. Cela garantit que Terraform attend que l'image soit téléchargée avant de lancer le conteneur.
* **2.3 :** Réseau, Volume, Image, puis Conteneur. Terraform analyse les dépendances (`depends_on` implicite via les références).
* **2.4 :** Le mot de passe (`secret123`) est en clair. **Correction :** utiliser une variable `var.db_password` avec `sensitive = true`.
* **2.5 :** Terraform va supprimer l'ancien conteneur et en recréer un nouveau car les ports d'un conteneur existant ne sont pas modifiables à chaud sur Docker.

### Exercice 3 - Diagnostic
* **3.1 :** Cycle de dépendance. A a besoin de B pour démarrer, et B a besoin de A. Résolution : extraire les noms/identifiants ou utiliser des valeurs fixes.
* **3.2 :** - a. La modification des variables d'environnement force la recréation du conteneur par Docker.
  - b. Non, car le volume est monté à l'extérieur. Les données persistent tant que le volume n'est pas détruit.
  - c. Non, car cela cause une interruption de service (downtime) lors du cycle stop/start.
* **3.3 :** - a. Fuite de secrets (mots de passe, clés).
  - b. Corruption du state (si Awa applique des changements, elle peut écraser ceux de l'autre).
  - c. Utiliser un **Backend distant** (S3, Terraform Cloud) avec gestion de verrouillage (locking).

### Exercice 4 - Adaptation Compose -> Terraform

resource "docker_image" "jupyter" {

 name = "jupyter/scipy-notebook:latest"

}

resource "docker_container" "jupyter" {

  name  = "anfa-jupyter"
  image = docker_image.jupyter.image_id
  
  ##### Utilisation d'une variable ou valeur directe pour le token
  env = ["JUPYTER_TOKEN=anfa-token"]

  ports {
    
    internal = 8888
    external = 8888
  }

  networks_advanced {
    
    name = docker_network.anfa_net.name
  }
}

### Exercice 5 - Mini-cas d'architecture
* **5.1 :** 1. Bucket de stockage (S3/OVH Object Storage), 2. Cluster Kubernetes managé, 3. Base de données SQL managée, 4. Réseau virtuel (VPC).
* **5.2 :** Recommandation B (Plusieurs fichiers). Cela facilite la maintenance, la lecture et la collaboration en équipe.
* **5.3 :** 1. Utilisation de variables (`variables.tf` + fichiers `.tfvars` distincts par env). 2. Workspaces Terraform.
* **5.4 :** Non, ce ne sera pas trivial. Il faudra réécrire la définition des ressources (providers différents), ce qui prouve l'importance de bien abstraire son code Terraform.

