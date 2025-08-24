# AWS-PROJECT
Projet du cours sur le cloud AWS

Introduction
The project involves deploying an existing PHP application on AWS while adhering to security, availability, and scalability best practices. The goal is to ensure a publicly accessible website while protecting backend systems.

Architecture

✅ Rôle du VPC :
Le VPC permet de créer ton propre réseau privé dans AWS, comme si tu construisais ton propre centre de données dans le cloud, 
 Ce qu’on y fait :
 •	Créer un réseau isolé avec des plages IP personnalisées
 •	Définir des sous-réseaux(subnets) publics et privés
 •	Contrôler l’accès à Internet (via Internet Gateway ou NAT Gateway)/Des passerelles
 •	Gérer les routes et la communication entre les ressources
 •	Appliquer des groupes de sécurité/ règles de sécurité et ACLs

👉 Tous les services AWS comme EC2, RDS, Lambda peuvent être déployés dans un VPC.

✅ 	Public Layer: Application Load Balancer in public subnets.


✅  Application Layer: Auto Scaling Group of EC2 instances (Amazon Linux 2023) in private subnets.

✅  Data Layer: Amazon RDS MySQL in DB subnets, with credentials stored in Secrets Manager.
✅  	Connectivity: NAT Gateway for updates from private instances.
✅
✅
High Availability
•	Traffic distribution via ALB.
•	Automatic scaling via ASG
•	RDS optionally in Multi-AZ.

Security
•	Application and DB instances in private subnets.
•	Strictly configured security groups (public ALB → private EC2 → RDS).
•	Secrets management via AWS Secrets Manager.
•	Use of IAM Roles for controlled access to AWS services.
•	Scalability
•	Auto-scaling based on metrics (e.g., CPU > 70%).
•	Scalable architecture to integrate caches (ElastiCache) or a CDN (CloudFront).


Data Import
•	Upload the SQL dump to S3.
•	Download via EC2.
•	Import into RDS with MySQL.

Diagram architechture


Conclusion
The solution meets the project's objectives: it is highly available, secure, and scalable. The proposed design provides a solid foundation for deploying the PHP application on AWS.
