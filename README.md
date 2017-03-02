# owl-opswork
![Packer] (https://cdn.worldvectorlogo.com/logos/hashicorp-packer.svg "Packer")

#Project

### What I have done
- I created an RDS database with MariaDB engine using the aws cli:
```sh
  aws rds create-db-instance --db-name RDSDiego \
  --allocated-storage 5 \
  --db-instance-class db.t2.micro \
  --engine mariadb \
  --master-username YOU_DB_MASTER_USER \
  --master-user-password YOUR_DB_PASSWORD \
  --vpc-security-group-ids sg-04e8fb78 \
  --backup-retention-period 0 \
  --db-instance-identifier wordpress
```
> I created that security group I specified to only allow inbound traffic to port 3306, to be able to connect to the mariadb database.

You can get the database endpoint/host with:
```sh
aws rds describe-db-instances|grep Address
```
- I created a packer file to build the container and push it to an AWS ECR repository.
- I created an ansible playbook for the provisioning and configuration of extra repositories, nginx, php, wordpress and supervisor.
- Wordpress is configured to use the remote RDS database, which has a security group to only allow incoming connections on port 3306.

### How to run the project
- Edit your remote database details on ansible-playbook/group_vars/all.
- Configure the AWS key, secret, and ECR repository on packer_aws.json or pass the variables as I will explain below.

- To first test the packer build with a local docker image you can do:
```sh
  docker run -ti -p 8083:80 dispera/packer:latest /usr/bin/supervisord
```
> (I am forwarding local port 8083 to the container port 80, so I can then connect from a web browser to: localhost:8083)

- To have Packer build a container image and push it to an AWS ECR repository:
```sh
packer build packer_aws.json
```
> (you must complete the aws secret, key, and ECR repository in this file to run it like this) or, you can run it passing the variables from the cli:

```sh
packer build -var ‘aws_access_key=KEY’ -var ‘aws_secret_key=SECRET’ -var ‘aws_ECR_repository=REPOSITORY‘ packer_aws.json
```

- At this point, the image is ready. You can go to AWS ECS, and create a task which uses this image and runs on and instance on your ECS cluster. You must make sure to configure the command to run: /usr/bin/supervisord

### How components interact between each other
- Using packer, you pull a centos image which you will provision using the specified ansible playbook.
- Then a resulting new container image will be tagged and pushed to the AWS ECR repository.

### What problems did I have
- I had never used Packer, Ansible, or ECS before this, so it was a nice opportunity to learn them, as well as refresh my docker skills.
- As I first tested this in virtualbox, I had then to adapt it for docker - meaning the nginx and php processes cannot run in the background, so I had to change the config files to the processes to run in the foreground. And I installed and configure supervisor (in the provisioning step through ansible) to start and manage them.

### How would I have done things to have the best HA/automated architecture
- As this is running on an ECS Cluster, I would configure the cluster to run on multiple availability zones with an Elastic Load Balancer. That would take care of the high availability. I would also configure Auto Scaling based on the application load.

### Ideas to improve this kind of infrastructure.
-

### Scenario Question
Tomorrow we want to put this project in production. What would be your advices and choices to achieve that.
Regarding the infrastructure itself and also external services like the monitoring.
- I believe this application, being in an ECS container and with very restricted security groups, can easily be put into production as I find this very secure already. For the monitoring, I would look for the best monitoring solutions to help me monitor if the Wordpress site is down (I would try CloudKick as it seems to cover that).
