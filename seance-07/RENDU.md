# Rendu TP : Séance 07 - Streaming de données avec Apache Kafka et Spark Structured Streaming

**Étudiant :** NGARTOBAYE OUMAROU BILLY
**Cours :** AWS & Big Data
**Date :** 06 Juillet 2026

---

# Objectifs de la séance

L'objectif principal de cette séance était de mettre en place une architecture de **traitement de données en temps réel** à l'aide d'**Apache Kafka** et **Apache Spark Structured Streaming**.

Les principaux objectifs étaient :

* Déployer un cluster Kafka composé de **trois brokers** en mode **KRaft**.
* Mettre en place un système de streaming simulant une flotte de bus envoyant leurs positions GPS.
* Vérifier la tolérance aux pannes du cluster Kafka.
* Traiter les données en temps réel avec Spark Structured Streaming.
* Stocker les résultats agrégés dans un Data Lake MinIO.

---

# Déploiement et configuration

L'infrastructure a été déployée à l'aide de **Docker Compose**.

Les services mis en place comprennent :

* un cluster Kafka composé de **3 brokers** ;
* Kafka UI pour l'administration et la supervision ;
* un producteur Python simulant les positions GPS des bus ;
* un consommateur de test ;
* Apache Spark pour le traitement temps réel ;
* MinIO pour le stockage des résultats.

---

## Configuration du cluster Kafka

Le cluster a été configuré avec :

* **3 brokers**
* Mode **KRaft** (sans ZooKeeper)
* Un topic nommé :

```text
anfa-positions-bus
```

avec les caractéristiques suivantes :

* **3 partitions**
* **Facteur de réplication : 3**

Cette configuration garantit une haute disponibilité des données même en cas de défaillance d'un broker.

---

## Validation du fonctionnement

Le bon fonctionnement du cluster a été vérifié grâce à des scripts Python :

* Producteur (Producer)
* Consommateur (Consumer)

Les messages produits sont correctement reçus par Kafka puis consommés en temps réel.

---

## Simulation de la flotte de bus

Un script Python simule une flotte de bus envoyant en continu :

* identifiant du bus ;
* latitude ;
* longitude ;
* vitesse ;
* horodatage.

Le débit des messages est observable directement dans **Kafka UI**.

---

## Tolérance aux pannes

Afin de tester la résilience du cluster, le broker :

```text
anfa-kafka-2
```

a été volontairement arrêté.

Grâce au facteur de réplication de **3**, le cluster continue de fonctionner normalement sans interruption des producteurs ni des consommateurs.

---

## Traitement avec Apache Spark

Deux traitements Spark ont été réalisés :

* affichage des micro-batchs dans la console ;
* agrégation des données en temps réel puis écriture dans MinIO.

Les résultats sont enregistrés automatiquement dans le bucket prévu à cet effet.

---

# Captures d'écran

## 1. Cluster Kafka avec 3 brokers actifs

![Kafka UI](captures/kafka-ui-3-brokers.png)

Cette capture montre le cluster Kafka fonctionnant avec ses trois brokers disponibles.

---

## 2. Débit des messages

![Débit Kafka](captures/kafka-throughput.png)

Le débit des messages augmente progressivement à mesure que les positions GPS sont envoyées par les producteurs.

---

## 3. Test de tolérance aux pannes

![Deux brokers actifs](captures/kafka-2-brokers.png)

Cette capture montre le cluster après l'arrêt volontaire du broker **anfa-kafka-2**.

Le cluster continue néanmoins de traiter les données grâce à la réplication.

---

## 4. Exécution de Spark Structured Streaming

![Spark Streaming](captures/spark-console-stream.png)

Les micro-batchs générés par Spark Structured Streaming sont affichés en temps réel dans la console.

---

## 5. Résultats enregistrés dans MinIO

![Résultats MinIO](captures/minio-stream-results.png)

Les données agrégées sont correctement enregistrées dans le Data Lake MinIO.

---

# Réponses aux exercices d'application

*(À compléter avec les réponses aux exercices demandés durant la séance.)*

---

# Difficultés rencontrées

## Gestion des dépendances Spark

Lors de l'exécution de :

```bash
spark-submit
```

une erreur est apparue concernant les dépendances Maven.

La cause provenait d'espaces incorrects dans l'option :

```text
--packages
```

Le problème a été résolu en corrigeant la syntaxe de la commande.

---

## Connexion à MinIO

Une configuration supplémentaire du client :

```text
mc
```

a été nécessaire afin de permettre à Spark d'écrire correctement dans le bucket MinIO avec les bonnes clés d'accès.

---

# Réflexion personnelle

Cette séance m'a permis de comprendre les avantages d'une architecture orientée **Streaming**.

Le couple **Apache Kafka + Spark Structured Streaming** est particulièrement adapté aux applications nécessitant un traitement immédiat des données, comme :

* le suivi d'une flotte de bus ;
* la surveillance de capteurs IoT ;
* la détection d'anomalies en temps réel.

À l'inverse, une architecture **Batch** reposant sur **Apache Airflow** et **Apache Spark** reste plus adaptée aux traitements lourds, historiques ou planifiés, comme la génération de rapports ou les analyses périodiques.

Enfin, le test de défaillance d'un broker a démontré concrètement l'intérêt de la réplication dans Kafka. Malgré l'arrêt volontaire d'un nœud, le cluster est resté entièrement opérationnel, illustrant la robustesse et la haute disponibilité offertes par cette architecture distribuée.

---

# Conclusion

Au cours de cette séance, nous avons construit une chaîne complète de traitement de données en temps réel en combinant **Apache Kafka**, **Apache Spark Structured Streaming** et **MinIO**.

Les principaux acquis sont :

* déploiement d'un cluster Kafka haute disponibilité ;
* utilisation d'un producteur et d'un consommateur Kafka ;
* mise en œuvre d'un pipeline de streaming temps réel ;
* validation de la tolérance aux pannes grâce à la réplication ;
* intégration de Spark Structured Streaming avec MinIO pour le stockage des résultats.

Cette architecture constitue une base solide pour le développement d'applications Big Data capables de traiter des flux continus de données avec une forte disponibilité et une excellente scalabilité.

