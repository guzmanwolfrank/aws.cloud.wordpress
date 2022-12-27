### Wordpress Site using AWS Elastic Beanstalk, and Amazon RDS, EC2, using MySQL as database.  

## What you will need before starting:

-AWS Account: Create one if necessary //

-Skill Level: prior experience with Wordpress //

-AWS Experience: Intermediate level familiarity with AWS and its services is reccommended. //

Remember to delete all instances, services etc. to avoid recurring billing! 
#

#These instructions are for learning purposes only! 

This tutorial describes how to launch an Amazon RDS DB instance that is external to AWS Elastic Beanstalk, then how to configure a high-availability environment running a WordPress website to connect to it. The website uses Amazon Elastic File System (Amazon EFS) as the shared storage for uploaded files.
#
Running a DB instance external to Elastic Beanstalk decouples the database from the lifecycle of your environment. This lets you connect to the same database from multiple environments, swap out one database for another, or perform a blue/green deployment without affecting your database.
#
This tutorial was developed with WordPress version 4.9.5 and PHP 7.0.

#Prerequisites 

This tutorial assumes you have knowledge of the basic Elastic Beanstalk operations and the Elastic Beanstalk console. If you haven't already, follow the instructions in Getting started using Elastic Beanstalk to launch your first Elastic Beanstalk environment.

To follow the procedures in this guide, you will need a command line terminal or shell to run commands. Commands are shown in listings preceded by a prompt symbol ($) and the name of the current directory, when appropriate.

~/eb-project$ this is a command
this is output
On Linux and macOS, you can use your preferred shell and package manager. On Windows 10, you can install the Windows Subsystem for Linux to get a Windows-integrated version of Ubuntu and Bash.

Default VPC
The Amazon Relational Database Service (Amazon RDS) procedures in this tutorial assume that you are launching resources in a default Amazon Virtual Private Cloud (Amazon VPC). All new accounts include a default VPC in each AWS Region. If you don't have a default VPC, the procedures will vary. See Using Elastic Beanstalk with Amazon RDS for instructions for EC2-Classic and custom VPC platforms.

AWS Regions
The sample application uses Amazon EFS, which only works in AWS Regions that support Amazon EFS. To learn about supported AWS Regions, see Amazon Elastic File System Endpoints and Quotas in the AWS General Reference.

Launch a DB instance in Amazon RDS
When you launch an instance with Amazon RDS, it's completely independent of Elastic Beanstalk and your Elastic Beanstalk environments, and will not be terminated or monitored by Elastic Beanstalk.

In the following steps you'll use the Amazon RDS console to:

Launch a database with the MySQL engine.

Enable a Multi-AZ deployment. This creates a standby in a different Availability Zone (AZ) to provide data redundancy, eliminate I/O freezes, and minimize latency spikes during system backups.

To launch an RDS DB instance in a default VPC
Open the RDS console.

In the navigation pane, choose Databases.

Choose Create database.

Choose Standard Create.

Important
Do not choose Easy Create. If you choose it, you can't configure the necessary settings to launch this RDS DB.

Under Additional configuration, for Initial database name, type ebdb.

Review the default settings and adjust these settings according to your specific requirements. Pay attention to the following options:

DB instance class – Choose an instance size that has an appropriate amount of memory and CPU power for your workload.

Multi-AZ deployment – For high availability, set this to Create an Aurora Replica/Reader node in a different AZ.

Master username and Master password – The database username and password. Make a note of these settings because you will use them later.

Verify the default settings for the remaining options, and then choose Create database.

After your DB instance is created, modify the security group attached to it in order to allow inbound traffic on the appropriate port..

Note
This is the same security group that you'll attach to your Elastic Beanstalk environment later, so the rule that you add now will grant ingress permission to other resources in the same security group.

To modify the inbound rules on the security group that's attached to your RDS instance
Open the Amazon RDS console.

Choose Databases.

Choose the name of your DB instance to view its details.

In the Connectivity section, make a note of the Subnets, Security groups, and Endpoint that are displayed on this page. This is so you can use this information later.

Under Security, you can see the security group that's associated with the DB instance. Open the link to view the security group in the Amazon EC2 console.

![rds-securitygroup](https://user-images.githubusercontent.com/29739578/209721399-52dbcd42-991b-44e7-afd9-013d07366aab.png)

In the security group details, choose Inbound.

Choose Edit.

Choose Add Rule.

For Type, choose the DB engine that your application uses.

For Source, type sg- to view a list of available security groups. Choose the security group that's associated with the Auto Scaling group that's used with your Elastic Beanstalk environment. This is so that Amazon EC2 instances in the environment can have access to the database.

![ec2-securitygroup-rds](https://user-images.githubusercontent.com/29739578/209721597-11a78a4e-31c9-4636-a164-9a8e8c1e8518.png)

Choose Save.

Creating a DB instance takes about 10 minutes. In the meantime, download WordPress and create your Elastic Beanstalk environment.

Download WordPress
To prepare to deploy WordPress using AWS Elastic Beanstalk, you must copy the WordPress files to your computer and provide the correct configuration information.

To create a WordPress project
Download WordPress from wordpress.org
~$ curl https://wordpress.org/wordpress-4.9.5.tar.gz -o wordpress.tar.gz








