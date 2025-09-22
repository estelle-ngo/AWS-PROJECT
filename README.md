# AWS-PROJECT

<h3><b>🔹Introduction</b></h3>
Le projet consiste à déployer une application PHP existante sur AWS, en respectant les bonnes pratiques de sécurité, de disponibilité et d'évolutivité.
L'objectif est de garantir l'accessibilité du site web au public tout en protégeant les systèmes back-end.

<h3><b>🔹Diagramme architechturale</b></h3>
<img width="2900" height="1644" alt="archi final modifie" src="https://github.com/user-attachments/assets/1a61dc0b-d8f5-4930-bef3-327040466618" />

 
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
 
<!-- 🎯<img width="611" height="180" alt="image" src="https://github.com/user-attachments/assets/26622681-2e9f-4c81-8769-6637871c2f47" />  🎯-->

| Catégorie / ressource                   |  Quantité | Base tarifaire utilisée                                                                  | Coût/mois (est.) |
| --------------------------------------- | --------: | ---------------------------------------------------------------------------------------- | ---------------: |
| **NAT Gateway (heure)**                 | 1 × 730 h | \$0.045 / heure. ([1])                                                                   |      **\$32.85** |
| **NAT Gateway — traitement données**    |     10 GB | \$0.045 / GB (ex.) ([1])                                                                 |       **\$0.45** |
| **Application Load Balancer (horaire)** | 1 × 730 h | \$0.0225 / heure (base) + \$0.008 / LCU-hr (1 LCU).                            ([2])     |      **\$22.27** |
| **EC2 (t2.micro)**                      | 1 × 730 h | \$0.0116 / heure (on-demand).                       ([3])                                |       **\$8.47** |
| **RDS (db.t3.micro)**                   | 1 × 730 h | ≈ \$0.018 / heure (MySQL). ([[4])                                                          |      **\$13.14** |
| **RDS stockage (gp2)**                  |     20 GB | ≈ \$0.115 / GB-mo (approx.) ([5])                                                         |       **\$2.30** |
| **S3 Standard**                         |     10 GB | ≈ \$0.023 / GB-mo (first tier). ([6])                                                     |       **\$0.23** |
| **Secrets Manager**                     |  1 secret | \$0.40 / secret / mois. ([7])                                                            |       **\$0.40** |
| **CloudWatch (logs + 1 alarm)**         |         — | Estimation faible                                                                        |       **\$2.00** |
| **Elastic IP (assoc.)**                 |         1 | Gratuit si associée à ressource en usage (sinon facturation). ([8])                      |       **\$0.00** |
| **Autres (IAM, Route Tables, VPC)**     |         — | Gratuit / inclus                                                                         |       **\$0.00** |

<!-- 🎯
[1]: https://aws.amazon.com/vpc/pricing/?utm_source=chatgpt.com "Amazon VPC Pricing"
[2]: https://aws.amazon.com/elasticloadbalancing/pricing/?utm_source=chatgpt.com "Elastic Load Balancing pricing"
[3]: https://instances.vantage.sh/aws/ec2/t2.micro?utm_source=chatgpt.com "t2.micro pricing and specs - Amazon EC2 Instance Comparison"
[4]: https://instances.vantage.sh/aws/rds/db.t3.micro?utm_source=chatgpt.com "db.t3.micro pricing and specs - Vantage"
[5]: https://aws.amazon.com/rds/pricing/?utm_source=chatgpt.com "Managed Relational Database - Amazon RDS Pricing"
[6]: https://www.cloudzero.com/blog/s3-pricing/?utm_source=chatgpt.com "A 2025 Guide To Amazon S3 Pricing"
[7]: https://aws.amazon.com/secrets-manager/pricing/?utm_source=chatgpt.com "AWS Secrets Manager pricing"
[8]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html?utm_source=chatgpt.com "Elastic IP addresses"
🎯-->


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

👉Amazon SNS (Simple Notification Service) : <br> 
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

 provider "aws" {
  region = "us-east-1"
}

# ---------------------------
# VPC + Internet Gateway
# ---------------------------
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "my-vpc" }
}

resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id
  tags   = { Name = "my-igw" }
}

# ---------------------------
# Subnets
# ---------------------------
# Public subnets (ALB)
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = { Name = "public-a" }
}

resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = { Name = "public-b" }
}

# Private subnets (Application)
resource "aws_subnet" "private_app_a" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "private-app-a" }
}

resource "aws_subnet" "private_app_b" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "us-east-1b"
  tags = { Name = "private-app-b" }
}

# Private subnets (Database)
resource "aws_subnet" "private_db_a" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.5.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "private-db-a" }
}

resource "aws_subnet" "private_db_b" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.6.0/24"
  availability_zone = "us-east-1b"
  tags = { Name = "private-db-b" }
}

# ---------------------------
# Route Table publique
# ---------------------------
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.my_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.my_igw.id
  }
  tags = { Name = "public-rt" }
}

resource "aws_route_table_association" "public_a_assoc" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_route_table_association" "public_b_assoc" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public_rt.id
}

# ---------------------------
# Security Groups
# ---------------------------
# ALB SG
resource "aws_security_group" "alb_sg" {
  name        = "alb-sg"
  description = "Allow HTTP/HTTPS from Internet"
  vpc_id      = aws_vpc.my_vpc.id

  ingress {
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
}

# Application SG (allow HTTP from ALB)
resource "aws_security_group" "app_sg" {
  name        = "app-sg"
  description = "Allow HTTP from ALB"
  vpc_id      = aws_vpc.my_vpc.id

  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# DB SG (allow MySQL from App)
resource "aws_security_group" "db_sg" {
  name        = "db-sg"
  description = "Allow MySQL from App tier"
  vpc_id      = aws_vpc.my_vpc.id

  ingress {
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.app_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ---------------------------
# Application Load Balancer
# ---------------------------
resource "aws_lb" "alb" {
  name               = "my-alb"
  load_balancer_type = "application"
  subnets            = [aws_subnet.public_a.id, aws_subnet.public_b.id]
  security_groups    = [aws_security_group.alb_sg.id]
}

resource "aws_lb_target_group" "app_tg" {
  name     = "app-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.my_vpc.id
}

resource "aws_lb_listener" "http_listener" {
  load_balancer_arn = aws_lb.alb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app_tg.arn
  }
}

# ---------------------------
# Application EC2 (Private subnets) - 2 instances
# ---------------------------
resource "aws_instance" "app1" {
  ami                    = "ami-0c02fb55956c7d316"
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.private_app_a.id
  vpc_security_group_ids = [aws_security_group.app_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Hello from App1 EC2</h1>" > /var/www/html/index.html
              EOF

  tags = { Name = "app1-ec2" }
}

resource "aws_instance" "app2" {
  ami                    = "ami-0c02fb55956c7d316"
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.private_app_b.id
  vpc_security_group_ids = [aws_security_group.app_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Hello from App2 EC2</h1>" > /var/www/html/index.html
              EOF

  tags = { Name = "app2-ec2" }
}

# Attachment des 2 instances au Target Group
resource "aws_lb_target_group_attachment" "app1_attach" {
  target_group_arn = aws_lb_target_group.app_tg.arn
  target_id        = aws_instance.app1.id
  port             = 80
}

resource "aws_lb_target_group_attachment" "app2_attach" {
  target_group_arn = aws_lb_target_group.app_tg.arn
  target_id        = aws_instance.app2.id
  port             = 80
}

# ---------------------------
# Secrets Manager pour RDS
# ---------------------------
resource "aws_secretsmanager_secret" "db_secret" {
  name = "rds-credentials"
}

resource "aws_secretsmanager_secret_version" "db_secret_version" {
  secret_id     = aws_secretsmanager_secret.db_secret.id
  secret_string = jsonencode({
    username = "rds-user"
    password = "password"
  })
}

# ---------------------------
# DB Subnet Group
# ---------------------------
resource "aws_db_subnet_group" "db_subnet_group" {
  name       = "db-subnet-group"
  subnet_ids = [aws_subnet.private_db_a.id, aws_subnet.private_db_b.id]
}

# ---------------------------
# RDS MySQL
# ---------------------------
resource "aws_db_instance" "mydb" {
  allocated_storage      = 20
  engine                 = "mysql"
  engine_version         = "8.0"
  instance_class         = "db.t3.micro"
  db_name                = "myappdb"

  username = jsondecode(aws_secretsmanager_secret_version.db_secret_version.secret_string)["username"]
  password = jsondecode(aws_secretsmanager_secret_version.db_secret_version.secret_string)["password"]

  skip_final_snapshot    = true
  vpc_security_group_ids = [aws_security_group.db_sg.id]
  db_subnet_group_name   = aws_db_subnet_group.db_subnet_group.name
  multi_az               = false # mettre true si tu veux HA DB Multi-AZ
}

# ---------------------------
# S3 bucket (static files)
# ---------------------------
resource "aws_s3_bucket" "static_files" {
  bucket = "3tier-static-files-terraform"
}

# Gestion des permissions du bucket via aws_s3_bucket_acl
resource "aws_s3_bucket_acl" "static_files_acl" {
  bucket = aws_s3_bucket.static_files.id
  acl    = "private"
}


# ---------------------------
# CloudWatch Monitoring
# ---------------------------
resource "aws_cloudwatch_metric_alarm" "cpu_alarm" {
  alarm_name          = "EC2HighCPU"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300
  statistic           = "Average"
  threshold           = 80
  alarm_description   = "Alarm when CPU exceeds 80%"
  dimensions = {
    InstanceId = aws_instance.app1.id
  }
}



```
 </pre>

<br>
<b><h2>Conclusion </h2></b><br>
La solution proposée satisfait pleinement aux objectifs du projet : elle garantit une haute disponibilité, un niveau de sécurité renforcé ainsi qu’une capacité d’évolution optimale. L’architecture retenue constitue ainsi une base robuste et fiable pour le déploiement de l’application PHP sur AWS.
