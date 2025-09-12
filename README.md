# AWS-PROJECT

<h3><b>🔹Introduction</b></h3>
Le projet consiste à déployer une application PHP existante sur AWS, en respectant les bonnes pratiques de sécurité, de disponibilité et d'évolutivité.
L'objectif est de garantir l'accessibilité du site web au public tout en protégeant les systèmes back-end.

<h3><b>🔹Diagramme architechturale</b></h3>
<img width="465" height="452" alt="Capoiu" src="https://github.com/user-attachments/assets/8de22f52-a942-4938-9c1e-e6dbec7c8a1c" />
 
<h3><b>🔹Documentation technique </b></h3>
Nous allons décrire chaque composant et justifier leur choix.

✅ Rôle du VPC :

Le VPC permet de créer son propre réseau privé dans AWS, comme si on construisait notre propre centre de données dans le cloud. 
 Ce qu’on y fait :
 - Créer un réseau isolé avec des plages IP personnalisées
 - Définir des sous-réseaux(subnets) publics et privés
 - Contrôler l’accès à Internet (via Internet Gateway ou NAT Gateway)/Des passerelles
 - Gérer les routes et la communication entre les ressources
 - appliquer des groupes de sécurité/ règles de sécurité et ACLs

✅  Application Layer: Auto Scaling Group of EC2 instances (Amazon Linux 2023) in private subnets.

Ce sont les serveurs applicatifs qui contiennent le code métier (API, backend, site web, etc.).
Placés dans des private subnets pour les protéger d’Internet.
Seul l’ALB peut les contacter.

Auto Scaling Group (ASG) :
- Ajoute ou supprime des EC2 selon la charge.
- Garantit toujours un nombre minimal d’instances.

👉 Rôle : exécuter mon application web et traiter les requêtes.

✅ 	Public Layer: Application Load Balancer in public subnets.

Le Load Balancer (ALB) est en front door de notre application.
Placé dans des public subnets car il doit être accessible depuis Internet (HTTP/HTTPS). (Dans notre cas le  HTTPS ne sera pas utilisé, ni de domaine) 
Il distribue le trafic vers les instances EC2 dans les private subnets.

Avantages :
- Sécurité → les EC2 ne sont pas exposées directement au public.
- Haute disponibilité → le trafic est réparti automatiquement.
- Scalabilité → il s’adapte avec l’Auto Scaling Group.

👉 Rôle : recevoir les requêtes Internet et les rediriger vers tes serveurs applicatifs.

✅  	Connectivity: NAT Gateway for updates from private instances.

c’est un service managé (donc fourni et géré par AWS) qui utilise la technique du NAT (Network Address Translation) Située dans un public subnet.
Donne aux instances privées (EC2 App Layer) la possibilité de sortir sur Internet (par ex. pour télécharger des updates, paquets, librairies).
Mais Internet ne peut pas initier de connexion vers elles ( accès sortant uniquement à Internet pour tes instances privées).

<!--🎯 
NAT (la technique) :
C’est le mécanisme réseau qui permet de traduire des adresses IP privées en adresses IP publiques (et inversement) pour que des machines privées
puissent accéder à Internet, sans être directement exposées.
NAT Gateway (dans AWS) :
👉 C’est une ressource virtuelle (un service managé par AWS) qui applique cette technique de NAT.
👉 Tu le déploies dans un subnet public avec une Elastic IP (EIP).
👉 Les instances dans les subnets privés passent par lui pour sortir sur Internet (ex : télécharger des mises à jour, accéder à des dépôts, etc.),
 mais elles ne sont pas accessibles depuis l’extérieur.
Exemple :
mon serveur applicatif EC2 dans un subnet privé fait un yum update.
Il passe par la route → NAT Gateway → IGW → Internet.
🎯-->
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

✅  Data Layer: Amazon RDS (Relational Database Service). MySQL in DB subnets, with credentials stored in Secrets Manager.<br>
Amazon RDS est un service de Base de données relationnelle managée avec lequel on peut:
- Stocker et gérer des données structurées , c 'est à dire Stocke les données en tables et colonnes (par exemple des utilisateurs, 
des commandes, des statistiques).
- Supporte plusieurs moteurs : MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora.
- Permet aux applications  de faire des requêtes SQL : SELECT, INSERT, UPDATE, etc.
- Gère les sauvegardes automatiques, la haute disponibilité, la réplication.
- il est Optimisé pour des applications transactionnelles (sites web, ERP, CRM).

 Accès uniquement via une connexion réseau (port 3306 pour MySQL).

✅Rôle de Secrets Manager 

AWS Secrets Manager est un service géré qui stocke et protège les informations sensibles comme :
les identifiants de connexion MySQL (nom d’utilisateur, mot de passe, host, port, nom de la base), éventuellement d’autres secrets applicatifs (API keys, tokens, etc.)

Dans ce projet, il va:
-  sécurisé le Stockage:
 Au lieu d’écrire le mot de passe MySQL dans ton code PHP ou dans un fichier de config, tu le mets dans Secrets Manager.
-  contrôler l' Accès via IAM :
Les instances EC2 ont un IAM Role qui leur permet d’appeler Secrets Manager. Résultat? Seules Les EC2 peuvent récupérer ces infos,
 pas les utilisateurs finaux.
- faire une rotation automatique des credentials (optionnel mais recommandé) :
Secrets Manager peut changer régulièrement le mot de passe MySQL sans qu'on ne modifie son application. Ça améliore la sécurité contre les fuites.

<!-- 
🎯Résumé de la différence
S3 = Stockage de fichiers (non structuré).C’est du stockage d’objets → tu mets des fichiers (appelés objets) dans des buckets.

Chaque objet a un ID (clé) et des métadonnées, mais tu ne peux pas faire de requêtes comme SELECT ou WHERE.
Je pourrais l’utiliser comme une sorte de "base NoSQL" si tu ajoutes une autre couche (par exemple : Amazon Athena pour faire des requêtes SQL sur
 des fichiers CSV/Parquet dans S3, ou DynamoDB qui est le vrai service NoSQL d’AWS).

RDS = Base de données relationnelle (structurée en tables).
 Exemple concret avec ton projet :
Tu vas mettre ton code PHP et ton dump SQL dans S3 pour que tes instances EC2 puissent les récupérer facilement.
Tu vas créer une base MySQL dans RDS pour stocker toutes les données de ton site (tables, statistiques, comptes, etc.).
🎯
-->
<br>
👉 Haute disponibilité: 

-  Distribution du trafic via ALB.
- Mise à l'échelle automatique via ASG
- RDS en option en multi-AZ.

<br>
👉 Sécurité: 

-  Instances d'application et de base de données dans des sous-réseaux privés.
- Groupes de sécurité strictement configurés (ALB public → EC2 privé → RDS).
- Gestion des secrets via AWS Secrets Manager.
-  Utilisation des rôles IAM pour un accès contrôlé aux services AWS.
- Évolutivité
- Mise à l'échelle automatique basée sur des métriques (par exemple, CPU > 70 %).
- Architecture évolutive pour intégrer des caches (ElastiCache) ou un CDN (CloudFront).

<br>
👉 Importation de données:  
- Téléchargement du dump SQL sur S3.
- Téléchargement via EC2.
- Importation dans RDS avec MySQL.
<br>

<H2><b> Évaluation du coût AWS</b></H2>

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
  
<!-- 🎯<br>  
- Route 53 : payante (zones hébergées + requêtes DNS).
🎯-->

<b> 2. AWS Pricing Calculator (outil officiel AWS)monitoring</b>
 
<img width="611" height="180" alt="image" src="https://github.com/user-attachments/assets/26622681-2e9f-4c81-8769-6637871c2f47" />

<br>
<H2>🛠️<b>Monitoring et Observabilité</b></H2>

L'objectif du Monitoring est: 
- Surveiller l’état des ressources (RDS, EC2, ALB, Auto Scaling).
- Alerter en cas de problème (ex : CPU trop haut, DB en panne, instance non healthy).
- Analyser la performance et les logs pour l’optimisation.
 
Afin de garantir la disponibilité, la performance et la sécurité de l’application, une solution de monitoring a été intégrée à l’architecture à l’aide des services Amazon CloudWatch et Amazon SNS .

👉CloudWatch Metrics :
- Suivi de l’utilisation CPU, mémoire, trafic réseau et état de santé des instances EC2 dans l’Auto Scaling Group.
- Suivi des connexions et performances de la base de données RDS (latence, nombre de connexions, espace disque, IOPS).
- Suivi des requêtes et latence de l’Application Load Balancer (ALB).

👉CloudWatch Alarms :
- Création d’alarmes sur des seuils critiques (ex. CPU > 80% pendant 5 minutes, latence ALB élevée, échec de l’état de santé RDS).
- Déclenchement automatique de notifications.

👉Amazon SNS (Simple Notification Service) : 
Les alarmes CloudWatch envoient des alertes email/SMS via un SNS Topic configuré pour notifier l’administrateur système.

👉CloudWatch Logs :
- Collecte des journaux d’accès Apache/PHP depuis les instances EC2.
- Stockage et analyse centralisée pour faciliter le dépannage.
- Mise en place de log groups par service (Application, RDS, ALB).

Bénéfices :
- Détection proactive des incidents (surconsommation CPU, panne DB, instance EC2 non disponible).
- Automatisation des actions grâce au couplage Auto Scaling + CloudWatch.
- Meilleure visibilité sur la santé globale du système.

<br>
<H2>CODE TERRAFORM</H2>
 <pre>
```hcl

  terraform {
  required_version = ">= 1.3.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

########################
# Basic Network (VPC) ##
########################

resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = { Name = "gdi-vpc" }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = { Name = "gdi-igw" }
}

# Public subnets (2 AZs)
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = { Name = "public-a" }
}
resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = { Name = "public-b" }
}

# Private subnets (where EC2 will live)
resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.11.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "private-a" }
}
resource "aws_subnet" "private_b" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.12.0/24"
  availability_zone = "us-east-1b"
  tags = { Name = "private-b" }
}

# Database subnets
resource "aws_subnet" "db_a" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.21.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "db-a" }
}
resource "aws_subnet" "db_b" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.22.0/24"
  availability_zone = "us-east-1b"
  tags = { Name = "db-b" }
}

# Public route table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
  tags = { Name = "public-rt" }
}
resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public_rt.id
}
resource "aws_route_table_association" "public_b" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public_rt.id
}

# NAT Gateway (for private instances to reach internet for updates)
resource "aws_eip" "nat_eip" {
  vpc = true
  tags = { Name = "nat-eip" }
}
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_a.id
  tags = { Name = "gdi-nat" }
}

# Private route table with NAT for internet access (outbound)
resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main_vpc.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
  tags = { Name = "private-rt" }
}
resource "aws_route_table_association" "private_a" {
  subnet_id      = aws_subnet.private_a.id
  route_table_id = aws_route_table.private_rt.id
}
resource "aws_route_table_association" "private_b" {
  subnet_id      = aws_subnet.private_b.id
  route_table_id = aws_route_table.private_rt.id
}

# DB route table - no direct internet (no association to IGW or NAT) -> isolated
resource "aws_route_table" "db_rt" {
  vpc_id = aws_vpc.main_vpc.id
  tags = { Name = "db-rt" }
}
resource "aws_route_table_association" "db_a" {
  subnet_id      = aws_subnet.db_a.id
  route_table_id = aws_route_table.db_rt.id
}
resource "aws_route_table_association" "db_b" {
  subnet_id      = aws_subnet.db_b.id
  route_table_id = aws_route_table.db_rt.id
}

########################
# Security Groups ######
########################

# ALB SG: allow HTTP from internet
resource "aws_security_group" "alb_sg" {
  name        = "alb-sg"
  description = "Allow HTTP from internet"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTP public"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = { Name = "alb-sg" }
}

# EC2 SG: allow HTTP from ALB only, allow outbound to anywhere (including to RDS)
resource "aws_security_group" "ec2_sg" {
  name        = "ec2-sg"
  description = "Allow traffic from ALB and access to RDS"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb_sg.id]
    description     = "Allow HTTP from ALB"
  }

  # (optional) allow SSH from your office IP - change CIDR to your IP or remove in lab
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "SSH - CHANGE for production"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = { Name = "ec2-sg" }
}

# RDS SG: allow MySQL from EC2 SG only
resource "aws_security_group" "rds_sg" {
  name        = "rds-sg"
  description = "Allow MySQL from EC2"
  vpc_id      = aws_vpc.main_vpc.id

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.ec2_sg.id]
    description     = "Allow MySQL from web servers"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = { Name = "rds-sg" }
}

########################
# ALB & Target Group ###
########################

resource "aws_lb" "alb" {
  name               = "gdi-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public_a.id, aws_subnet.public_b.id]
  tags = { Name = "gdi-alb" }
}

resource "aws_lb_target_group" "tg" {
  name     = "gdi-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main_vpc.id
  health_check {
    path                = "/"
    healthy_threshold   = 2
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
  }
  tags = { Name = "gdi-tg" }
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.alb.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tg.arn
  }
}

########################
# IAM Role for EC2 (to access Secrets Manager etc.)
########################

resource "aws_iam_role" "ec2_role" {
  name = "gdi-ec2-role"
  assume_role_policy = data.aws_iam_policy_document.ec2_assume_role.json
  tags = { Name = "gdi-ec2-role" }
}

data "aws_iam_policy_document" "ec2_assume_role" {
  statement {
    actions = ["sts:AssumeRole"]
    principals {
      type        = "Service"
      identifiers = ["ec2.amazonaws.com"]
    }
  }
}

# Policy: minimal SecretsManager read + CloudWatch logs (if needed)
resource "aws_iam_policy" "ec2_policy" {
  name = "gdi-ec2-policy"
  policy = data.aws_iam_policy_document.ec2_policy_doc.json
}

data "aws_iam_policy_document" "ec2_policy_doc" {
  statement {
    actions = [
      "secretsmanager:GetSecretValue",
      "secretsmanager:DescribeSecret"
    ]
    resources = ["*"] # pour un labo on peut mettre *, en prod restreindre au secret ARN
  }
  statement {
    actions = [
      "logs:CreateLogStream",
      "logs:PutLogEvents",
      "logs:CreateLogGroup"
    ]
    resources = ["*"]
  }
}

resource "aws_iam_role_policy_attachment" "attach_policy" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = aws_iam_policy.ec2_policy.arn
}

resource "aws_iam_instance_profile" "ec2_profile" {
  name = "gdi-ec2-instance-profile"
  role = aws_iam_role.ec2_role.name
}

########################
# Launch Template + ASG
########################

# Generate DB password for RDS (random)
resource "random_password" "db_password" {
  length  = 16
  override_characters = "!@#"
  special = true
}

# Simple user_data that installs Apache, PHP and uses AWS CLI to fetch secret
# Note: instance profile & role allow access to Secrets Manager
data "template_file" "user_data" {
  template = file("${path.module}/userdata.sh.tpl")
  vars = {
    secret_name = "gdi/mysql/credentials"
  }
}

resource "aws_launch_template" "lt" {
  name_prefix   = "gdi-lt-"
  image_id      = "ami-0c2b8ca1dad447f8a"  # change to Amazon Linux 2023 AMI for your region
  instance_type = "t2.micro"

  iam_instance_profile {
    name = aws_iam_instance_profile.ec2_profile.name
  }

  vpc_security_group_ids = [aws_security_group.ec2_sg.id]

  user_data = base64encode(data.template_file.user_data.rendered)

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "gdi-web-instance"
    }
  }
}

resource "aws_autoscaling_group" "asg" {
  name                      = "gdi-asg"
  max_size                  = 3
  min_size                  = 1
  desired_capacity          = 1
  health_check_type         = "ELB"
  health_check_grace_period = 120
  vpc_zone_identifier       = [aws_subnet.private_a.id, aws_subnet.private_b.id]

  launch_template {
    id      = aws_launch_template.lt.id
    version = "$Latest"
  }

  target_group_arns = [aws_lb_target_group.tg.arn]

  tag {
    key                 = "Name"
    value               = "gdi-asg-instance"
    propagate_at_launch = true
  }
}

########################
# RDS MySQL
########################

resource "aws_db_subnet_group" "rds_subnets" {
  name       = "gdi-rds-subnet-group"
  subnet_ids = [aws_subnet.db_a.id, aws_subnet.db_b.id]
  tags = { Name = "gdi-rds-subnet-group" }
}

resource "aws_db_instance" "db" {
  identifier = "gdi-mysql"
  engine     = "mysql"
  engine_version = "8.0"
  instance_class = "db.t3.micro"
  allocated_storage = 20
  username = "gdi_admin"
  password = random_password.db_password.result
  skip_final_snapshot = true           # in lab environment; in prod set false and configure final snapshot
  db_subnet_group_name = aws_db_subnet_group.rds_subnets.name
  vpc_security_group_ids = [aws_security_group.rds_sg.id]
  publicly_accessible = false
  multi_az = false
  tags = { Name = "gdi-mysql" }

  # optional parameter group / option group can be added if needed
}

########################
# Secrets Manager - store DB credentials (create version after DB so we include endpoint)
########################

resource "aws_secretsmanager_secret" "db_secret" {
  name = "gdi/mysql/credentials"
  description = "Credentials for GDI MySQL"
  tags = { Name = "gdi-db-secret" }
}

resource "aws_secretsmanager_secret_version" "db_secret_ver" {
  secret_id = aws_secretsmanager_secret.db_secret.id

  # we set depends_on to ensure DB instance has an address to include
  secret_string = jsonencode({
    username = aws_db_instance.db.username
    password = random_password.db_password.result
    host     = aws_db_instance.db.address
    port     = aws_db_instance.db.port
    dbname   = "mysql" # adapt if your app needs specific DB name
  })

  depends_on = [aws_db_instance.db]
}

########################
# S3 Bucket for backups / SQL dump / static files
########################

resource "aws_s3_bucket" "app_bucket" {
  bucket = "gdi-app-bucket-${random_id.bucket_suffix.hex}"
  acl    = "private"

  versioning {
    enabled = true
  }

  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }

  tags = { Name = "gdi-s3" }
}

resource "random_id" "bucket_suffix" {
  byte_length = 4
}

########################
# CloudWatch alarms (basic example for RDS CPU)
########################

resource "aws_cloudwatch_metric_alarm" "rds_cpu_high" {
  alarm_name          = "gdi-rds-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/RDS"
  period              = 300
  statistic           = "Average"
  threshold           = 70
  dimensions = {
    DBInstanceIdentifier = aws_db_instance.db.id
  }
  alarm_description = "Alarm when RDS CPU > 70% (avg)"
}

########################
# Outputs
########################

output "alb_dns_name" {
  description = "Public DNS for ALB (use to reach the website)"
  value       = aws_lb.alb.dns_name
}

output "rds_endpoint" {
  description = "RDS endpoint (hostname)"
  value       = aws_db_instance.db.address
  sensitive   = false
}

output "secrets_arn" {
  description = "Secrets Manager ARN storing DB credentials"
  value       = aws_secretsmanager_secret.db_secret.arn
  sensitive   = false
}
```
 </pre>

<br>
<b><h2>Conclusion </h2></b>

The solution meets the project's objectives: it is highly available, secure, and scalable. The proposed design provides a solid foundation for deploying the PHP application on AWS.
