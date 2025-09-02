# AWS-PROJECT


<h3><b>Introduction</b></h3>

Le projet consiste à déployer une application PHP existante sur AWS, en respectant les bonnes pratiques de sécurité, de disponibilité et d'évolutivité. L'objectif est de garantir l'accessibilité du site web au public tout en protégeant les systèmes back-end.

<h3><b>Diagram architechture</b></h3>

<img width="465" height="452" alt="Capoiu" src="https://github.com/user-attachments/assets/8de22f52-a942-4938-9c1e-e6dbec7c8a1c" />

<br> 
<h3><b> Architecture </b></h3>

✅ Rôle du VPC : <br>
Le VPC permet de créer ton propre réseau privé dans AWS, comme si tu construisais ton propre centre de données dans le cloud, 
 Ce qu’on y fait : 
 - Créer un réseau isolé avec des plages IP personnalisées
 - Définir des sous-réseaux(subnets) publics et privés
 - Contrôler l’accès à Internet (via Internet Gateway ou NAT Gateway)/Des passerelles
 - Gérer les routes et la communication entre les ressources
 - appliquer des groupes de sécurité/ règles de sécurité et ACLs

👉 Tous les services AWS comme EC2, RDS, Lambda peuvent être déployés dans un VPC.

✅  Application Layer: Auto Scaling Group of EC2 instances (Amazon Linux 2023) in private subnets.

Ce sont les serveurs applicatifs qui contiennent ton code métier (API, backend, site web, etc.).
Placés dans des private subnets pour les protéger d’Internet.
Seul l’ALB peut les contacter.


Auto Scaling Group (ASG) :
- Ajoute ou supprime des EC2 selon la charge.
- Garantit toujours un nombre minimal d’instances.

👉 Rôle : exécuter mon application web et traiter les requêtes.


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


<!-- 
Exemple :
mon serveur applicatif EC2 dans un subnet privé fait un yum update.
Il passe par la route → NAT Gateway → IGW → Internet.
-->

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



<!-- 
🔹. Résumé de la différence

S3 = Stockage de fichiers (non structuré).C’est du stockage d’objets → tu mets des fichiers (appelés objets) dans des buckets.

Chaque objet a un ID (clé) et des métadonnées, mais tu ne peux pas faire de requêtes comme SELECT ou WHERE.
Je pourrais l’utiliser comme une sorte de "base NoSQL" si tu ajoutes une autre couche (par exemple : Amazon Athena pour faire des requêtes SQL sur des fichiers CSV/Parquet dans S3, ou DynamoDB qui est le vrai service NoSQL d’AWS).

RDS = Base de données relationnelle (structurée en tables).

 Exemple concret avec ton projet :

Tu vas mettre ton code PHP et ton dump SQL dans S3 pour que tes instances EC2 puissent les récupérer facilement.
Tu vas créer une base MySQL dans RDS pour stocker toutes les données de ton site (tables, statistiques, comptes, etc.).

-->


<br>
👉 Haute disponibilité
• Distribution du trafic via ALB.
• Mise à l'échelle automatique via ASG
• RDS en option en multi-AZ.
<br>


👉 Sécurité
• Instances d'application et de base de données dans des sous-réseaux privés.
• Groupes de sécurité strictement configurés (ALB public → EC2 privé → RDS).
• Gestion des secrets via AWS Secrets Manager.
• Utilisation des rôles IAM pour un accès contrôlé aux services AWS.
• Évolutivité
• Mise à l'échelle automatique basée sur des métriques (par exemple, CPU > 70 %).
• Architecture évolutive pour intégrer des caches (ElastiCache) ou un CDN (CloudFront).
<br>

👉 Importation de données
• Téléchargement du dump SQL sur S3.
• Téléchargement via EC2.
• Importation dans RDS avec MySQL.
<br>

<b>🔹 Étapes pour évaluer le coût AWS</b>

<b>1. Lister les ressources utilisées</b>


Dans mon architecture, nous avons :
- Amazon VPC : gratuit (seuls les coûts de transfert de données sont facturés).
- Sous-réseaux : gratuits.
- Passerelle Internet (IGW) : gratuite (seule la bande passante est facturée).
- Passerelle NAT : payante (par heure + par Go transféré).
- Équilibreur de charge élastique (ALB) : payante (par heure + par LCU [Load Capacity Unit]).
- Instances EC2 (ASG) : facturées à l’heure/seconde + EBS (disque).
- RDS MySQL : payante (instance + stockage + IOPS).
- Amazon S3 : payante (stockage + requêtes).
- Secrets Manager : payante (par secret stocké + appels API).
- Route 53 : payante (zones hébergées + requêtes DNS).

<b> 2. AWS Pricing Calculator (outil officiel AWS)monitoring</b>
 


TOTAL			194,8<img width="611" height="180" alt="image" src="https://github.com/user-attachments/assets/26622681-2e9f-4c81-8769-6637871c2f47" />


<br>
🛠️ <b>Monitoring et Observabilité</b>

<br>Objectif du Monitoring

- Surveiller l’état des ressources (RDS, EC2, ALB, Auto Scaling).

- Alerter en cas de problème (ex : CPU trop haut, DB en panne, instance non healthy).

- Analyser la performance et les logs pour l’optimisation.


<br>
Afin de garantir la disponibilité, la performance et la sécurité de l’application, une solution de monitoring a été intégrée à l’architecture à l’aide des services Amazon CloudWatch et Amazon SNS .

CloudWatch Metrics :

Suivi de l’utilisation CPU, mémoire, trafic réseau et état de santé des instances EC2 dans l’Auto Scaling Group.

Suivi des connexions et performances de la base de données RDS (latence, nombre de connexions, espace disque, IOPS).

Suivi des requêtes et latence de l’Application Load Balancer (ALB).

CloudWatch Alarms :

Création d’alarmes sur des seuils critiques (ex. CPU > 80% pendant 5 minutes, latence ALB élevée, échec de l’état de santé RDS).

Déclenchement automatique de notifications.

Amazon SNS (Simple Notification Service) :

Les alarmes CloudWatch envoient des alertes email/SMS via un SNS Topic configuré pour notifier l’administrateur système.

CloudWatch Logs :

Collecte des journaux d’accès Apache/PHP depuis les instances EC2.

Stockage et analyse centralisée pour faciliter le dépannage.

Mise en place de log groups par service (Application, RDS, ALB).

Bénéfices :

Détection proactive des incidents (surconsommation CPU, panne DB, instance EC2 non disponible).

Automatisation des actions grâce au couplage Auto Scaling + CloudWatch.

Meilleure visibilité sur la santé globale du système.


<br>
<H2>CODE TERRAFORM</H2>
creation vpc t ses sous reseaux

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # Compatible et disponible sur toutes les plateformes courantes
    }
  }

  required_version = ">= 1.3.0"  # Assurez-vous que votre Terraform est à jour
}



provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "hello-vpc"
  }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = {
    Name = "hello-igw"
  }
}

resource "aws_subnet" "public_subnet_a" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "public-subnet-a"
  }
}

resource "aws_subnet" "public_subnet_b" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name = "public-subnet-b"
  }
}

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  tags = {
    Name = "public-route-table"
  }
}

resource "aws_route_table_association" "public_rta_a" {
  subnet_id      = aws_subnet.public_subnet_a.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_route_table_association" "public_rta_b" {
  subnet_id      = aws_subnet.public_subnet_b.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow HTTP"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "web-sg"
  }
}

resource "aws_instance" "web" {
  ami                         = "ami-0c2b8ca1dad447f8a"
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public_subnet_a.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]
  associate_public_ip_address = true
  key_name                    = "my-keypair"  # Replace with your Key pair
  user_data                   = file("userdata.sh")

  tags = {
    Name = "hello-web"
  }
}

resource "aws_lb" "alb" {
  name               = "hello-alb"
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web_sg.id]
  subnets            = [
    aws_subnet.public_subnet_a.id,
    aws_subnet.public_subnet_b.id
  ]
  tags = {
    Name = "hello-alb"
  }
}

resource "aws_lb_target_group" "tg" {
  name     = "hello-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main_vpc.id
  health_check {
    path = "/"
  }
}

resource "aws_lb_target_group_attachment" "web_attach" {
  target_group_arn = aws_lb_target_group.tg.arn
  target_id        = aws_instance.web.id
  port             = 80
}

resource "aws_lb_listener" "listener" {
  load_balancer_arn = aws_lb.alb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tg.arn
  }
}

<br>
<b><h2>Conclusion </h2></b>

The solution meets the project's objectives: it is highly available, secure, and scalable. The proposed design provides a solid foundation for deploying the PHP application on AWS.
