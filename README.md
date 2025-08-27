# AWS-PROJECT
Projet du cours sur le cloud AWS

Introduction

The project involves deploying an existing PHP application on AWS while adhering to security, availability, and scalability best practices. The goal is to ensure a publicly accessible website while protecting backend systems.

<b> Architecture </b>

✅ Rôle du VPC : <br>
Le VPC permet de créer ton propre réseau privé dans AWS, comme si tu construisais ton propre centre de données dans le cloud, 
 Ce qu’on y fait : 
 - Créer un réseau isolé avec des plages IP personnalisées
 - Définir des sous-réseaux(subnets) publics et privés
 - Contrôler l’accès à Internet (via Internet Gateway ou NAT Gateway)/Des passerelles
 - 	Gérer les routes et la communication entre les ressources
 - 		Appliquer des groupes de sécurité/ règles de sécurité et ACLs

👉 Tous les services AWS comme EC2, RDS, Lambda peuvent être déployés dans un VPC.

✅  Application Layer: Auto Scaling Group of EC2 instances (Amazon Linux 2023) in private subnets.

Ce sont les serveurs applicatifs qui contiennent ton code métier (API, backend, site web, etc.).
Placés dans des private subnets pour les protéger d’Internet.
Seul l’ALB peut les contacter.


Auto Scaling Group (ASG) :
- Ajoute ou supprime des EC2 selon la charge.
- Garantit toujours un nombre minimal d’instances.

👉 Rôle : exécuter ton application web et traiter les requêtes.


✅ 	Public Layer: Application Load Balancer in public subnets.

Le Load Balancer (ALB) est en front door de ton application.
Placé dans des public subnets car il doit être accessible depuis Internet (HTTP/HTTPS).
Il distribue le trafic vers tes instances EC2 dans les private subnets.

Avantages :

Sécurité → tes EC2 ne sont pas exposées directement au public.
Haute disponibilité → le trafic est réparti automatiquement.
Scalabilité → il s’adapte avec l’Auto Scaling Group.

👉 Rôle : recevoir les requêtes Internet et les rediriger vers tes serveurs applicatifs.


✅  	Connectivity: NAT Gateway for updates from private instances.

Située dans un public subnet.
Donne aux instances privées (EC2 App Layer) la possibilité de sortir sur Internet (par ex. pour télécharger des updates, paquets, librairies).
Mais Internet ne peut pas initier de connexion vers elles ( accès sortant uniquement à Internet pour tes instances privées).

Exemple :

Ton serveur applicatif EC2 dans un subnet privé fait un yum update.
Il passe par la route → NAT Gateway → IGW → Internet.


✅  IGW (Internet Gateway)

Attachée au VPC.
Nécessaire pour toute communication entre le VPC et Internet.

Sert à deux choses :
- L’ALB dans le public subnet → reçoit du trafic entrant depuis Internet.
- La NAT Gateway → envoie le trafic sortant vers Internet.

Rôle : la porte d’entrée/sortie du VPC vers Internet.



✅  Amazon S3 (Simple Storage Service)  Type de service: Stockage d’objets.

ça sert à
- Sauvegarder des fichiers (images, vidéos, PDF, backups, logs, fichiers zip, etc.).
- Héberger du contenu statique (par exemple un site statique HTML).
- Stocker des dumps SQL, du code, ou des fichiers à partager entre services AWS.
Accessible via HTTP/HTTPS (API REST).

✅  Data Layer: Amazon RDS (Relational Database Service). MySQL in DB subnets, with credentials stored in Secrets Manager.
  
Amazon RDS est un service de Base de données relationnelle managée avec lequel on peut:
- Stocker et gérer des données structurées , c 'est à dire Stocke les données en tables et colonnes (par exemple des utilisateurs, des commandes, des statistiques).
- Supporte plusieurs moteurs : MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora.
- Permet aux applications  de faire des requêtes SQL : SELECT, INSERT, UPDATE, etc.
- Gère les sauvegardes automatiques, la haute disponibilité, la réplication.
- il est Optimisé pour des applications transactionnelles (sites web, ERP, CRM).

 Accès uniquement via une connexion réseau (port 3306 pour MySQL).



🔹. Résumé de la différence

S3 = Stockage de fichiers (non structuré).C’est du stockage d’objets → tu mets des fichiers (appelés objets) dans des buckets.

Chaque objet a un ID (clé) et des métadonnées, mais tu ne peux pas faire de requêtes comme SELECT ou WHERE.
Je pourrais l’utiliser comme une sorte de "base NoSQL" si tu ajoutes une autre couche (par exemple : Amazon Athena pour faire des requêtes SQL sur des fichiers CSV/Parquet dans S3, ou DynamoDB qui est le vrai service NoSQL d’AWS).

RDS = Base de données relationnelle (structurée en tables).

 Exemple concret avec ton projet :

Tu vas mettre ton code PHP et ton dump SQL dans S3 pour que tes instances EC2 puissent les récupérer facilement.
Tu vas créer une base MySQL dans RDS pour stocker toutes les données de ton site (tables, statistiques, comptes, etc.).




👉 High Availability
•	Traffic distribution via ALB.
•	Automatic scaling via ASG
•	RDS optionally in Multi-AZ.

👉 Security
•	Application and DB instances in private subnets.
•	Strictly configured security groups (public ALB → private EC2 → RDS).
•	Secrets management via AWS Secrets Manager.
•	Use of IAM Roles for controlled access to AWS services.
•	Scalability
•	Auto-scaling based on metrics (e.g., CPU > 70%).
•	Scalable architecture to integrate caches (ElastiCache) or a CDN (CloudFront).


👉 Data Import
•	Upload the SQL dump to S3.
•	Download via EC2.
•	Import into RDS with MySQL.


<b>🔹 Étapes pour évaluer le coût AWS</b>

<b>1. Lister les ressources utilisées</b>

Dans mon architecture on a:
- Amazon VPC : gratuit (seule la data transfer coûte).
- Subnets : gratuit.
- Internet Gateway (IGW) : gratuit (seule la bande passante est facturée).
- NAT Gateway : payant (par heure + par Go transféré).
- Elastic Load Balancer (ALB) : payant (par heure + par LCU [Load Capacity Unit]).
- EC2 instances (ASG) : facturation à l’heure/seconde + EBS (disque).
- RDS MySQL : payant (instance + stockage + IOPS).
- Amazon S3 : payant (stockage + requêtes).
- Secrets Manager : payant (par secret stocké + appels API).
- Route 53 : payant (zones hébergées + requêtes DNS).


<b> 2. AWS Pricing Calculator (outil officiel AWS)</b>


Service	                                         Prix_unitaire_mensuel (€)     	Quantité	                    Coût_total_estime (€)
EC2 (2x t3.medium, 24/7)	                                       30                  	2                                     	60
Elastic Load Balancer (ALB)	                                    25                  	1	                                     25
NAT Gateway                                                    	35	                  1	                                     35
RDS MySQL (db.t3.medium)	                                       70	                  1	                                     70
S3 (100 Go, Standard)	                                           2,3                	1                                      	2,3
Secrets Manager (5 secrets)                                     	2                  	1                                      	2
Route 53 (1 zone + requêtes)	                                    0,5                	1                                      	0,5

TOTAL			194,8<img width="611" height="180" alt="image" src="https://github.com/user-attachments/assets/26622681-2e9f-4c81-8769-6637871c2f47" />


<b>Diagram architechture</b>

<img width="465" height="452" alt="Capoiu" src="https://github.com/user-attachments/assets/8de22f52-a942-4938-9c1e-e6dbec7c8a1c" />

Conclusion 

The solution meets the project's objectives: it is highly available, secure, and scalable. The proposed design provides a solid foundation for deploying the PHP application on AWS.
