### Wordpress Site using AWS Elastic Beanstalk, and Amazon RDS, EC2, using MySQL as database.  
#
![wordpress-arch-v2 f065678e8a2d45a770dc192747d49f939ccd5ac9](https://user-images.githubusercontent.com/29739578/209723105-aeaeac8f-c56b-4b02-8244-6aec454c2ca0.png)
#

This tutorial describes how to launch an Amazon RDS DB instance that is external to AWS Elastic Beanstalk, then how to configure a high-availability environment running a WordPress website to connect to it. The site uses Amazon Elastic File System (Amazon EFS) as the shared storage for uploads.


#### What you will need before starting:

    -AWS Account: Create one if necessary 

    -Skill Level: prior experience with Wordpress 

    -AWS Experience: Intermediate level familiarity with AWS and its services is reccommended. 


#
 Remember to delete all instances, services etc. to avoid recurring billing! 
 These instructions are for learning purposes only! 
#


## Prerequisites 

 If you need to catchup on your Elastic Beanstalk skills, head to [https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.html] to launch your first Beanstalk environment. Get acquainted, if necessary-- and head back to this tutorial. 

#
#### Tools required for this project:
        Command Line Terminal or Shell>> to run commands
        Default VPC >> Default when launching Amazon RDS.  All new accounts include a default VPC in each Region. 
        AWS Regions >> This project uses EFS, which only works in certain AWS Regions.  To find supported Regions, see the AWS General                                         Reference for Amazon Elastic File System. 


*Commands* are shown in listings preceded by a prompt symbol ($) and the name of the current directory, when appropriate.

![ebproj](https://user-images.githubusercontent.com/29739578/209729897-2e651309-ef16-499b-b522-b8facf33b686.PNG)


#### Shell or Command Line Terminal 
*If you are using Linux and macOS, you can use your preferred shell and package manager. If you don't have access to Linux or macOS, use Windows 10. Install the Windows Subsystem for Linux to get a Windows-integrated application for Bash and Ubuntu.

#### Default VPC
 All new accounts include a default VPC in each AWS Region. If you don't have a default VPC, the procedures will vary. 

#### AWS Regions
The sample application uses Amazon EFS, which only works in AWS Regions that support Amazon EFS. To learn about supported AWS Regions, see Amazon Elastic File System Endpoints and Quotas in the AWS General Reference.

#
## I. Launch a DB instance in Amazon RDS

Building an instance with Amazon RDS will not be terminated or monitored by Elastic Beanstalk.  This is because launcing this instance is independent of Elastic Beanstalk environments. 

In the following steps you'll use the Amazon RDS console to:

                Launch a database with the MySQL engine.

                Enable a Multi-AZ deployment. This creates a standby in a different Availability Zone (AZ) to provide data redundancy, eliminate I/O freezes, and                       minimize latency spikes during system backups.


                ### To launch an RDS DB instance in a default VPC
                      1. Open the RDS console.

                      2. In the navigation pane, choose Databases.

                      3. Choose Create database.

                      4. Choose Standard Create.

                *Important :Do not choose Easy Create. If you choose it, you can't
                configure the necessary settings to launch this RDS DB.*

                      5.  Under Additional configuration, for Initial database name, type ebdb.

                      6.  Review the default settings and adjust these settings according to your specific requirements. Pay attention to the following options:

                          *DB instance class* ??? Choose an instance size that has an appropriate amount of memory and CPU power for your workload.

                          *Multi-AZ deployment* ??? For high availability, set this to Create an Aurora Replica/Reader node in a different AZ.

                          *Master username and Master password* ??? The database username and password. Make a note of these settings because you will use them later.

                      7. Verify the default settings for the remaining options, and then choose Create database.

After your DB instance is created, modify the security group attached to it in order to allow inbound traffic on the appropriate port..

### To modify the inbound rules on the security group that's attached to your RDS instance
                        1. Open the Amazon RDS console.

                        2. Choose Databases.

                        3. Choose the name of your DB instance to view its details.

                        4. In the Connectivity section, make a note of the Subnets, Security groups, and Endpoint that are displayed on this page. This is so you can use this information later.
                        
                        5. Under Security, you can see the security group that's associated with the DB instance. Open the link to view the security group in the Amazon EC2 console.

![rds-securitygroup](https://user-images.githubusercontent.com/29739578/209721399-52dbcd42-991b-44e7-afd9-013d07366aab.png)

                        6. In the security group details, choose Inbound.

                        7. Choose Edit.

                        8. Choose Add Rule.

                        9. For Type, choose the DB engine that your application uses.

                        10. For Source, type sg- to view a list of available security groups. Choose the security group that's associated with the Auto Scaling group that's used with your Elastic Beanstalk environment. This is so that Amazon EC2 instances in the environment can have access to the database.

![ec2-securitygroup-rds](https://user-images.githubusercontent.com/29739578/209721597-11a78a4e-31c9-4636-a164-9a8e8c1e8518.png)

                        11. Choose Save.

                        Creating a DB instance takes about 10 minutes. In the meantime, download WordPress and create your Elastic Beanstalk environment.
#
## II. Download WordPress

  To prepare to deploy WordPress using AWS Elastic Beanstalk, you must copy the WordPress files to your computer and provide the correct configuration information.

To create a WordPress project

    1. Download WordPress from wordpress.org

    ~$ curl https://wordpress.org/wordpress-4.9.5.tar.gz -o wordpress.tar.gz

    2. Download the configuration files from the sample repository.

    ~$ wget https://github.com/aws-samples/eb-php-wordpress/releases/download/v1.1/eb-php-wordpress-v1.zip

    3. Extract WordPress and change the name of the folder.

    ~$ tar -xvf wordpress.tar.gz
    ~$ mv wordpress wordpress-beanstalk
    ~$ cd wordpress-beanstalk
 
    4. Extract the configuration files over the WordPress installation.

     ~/wordpress-beanstalk$ unzip ../eb-php-wordpress-v1.zip
     creating: .ebextensions/
    inflating: .ebextensions/dev.config
    inflating: .ebextensions/efs-create.config
    inflating: .ebextensions/efs-mount.config
    inflating: .ebextensions/loadbalancer-sg.config
    inflating: .ebextensions/wordpress.config
    inflating: LICENSE
    inflating: README.md
    inflating: wp-config.php

#
## III. Launch an Elastic Beanstalk environment

Use the Elastic Beanstalk console to create an Elastic Beanstalk environment. After you launch the environment, you can configure it to connect to the database, then deploy the WordPress code to the environment.

In the following steps, you'll use the Elastic Beanstalk console to:

            Create an Elastic Beanstalk application using the managed PHP platform.

            Accept the default settings and sample code.

#### To launch an environment (console)
    1. Open the Elastic Beanstalk console using this preconfigured link: console.aws.amazon.com/elasticbeanstalk/home#/newApplication?applicationName=tutorials&environmentType=LoadBalanced

    2. For Platform, select the platform and platform branch that match the language used by your application.

    3. For Application code, choose Sample application.

    4. Choose Review and launch.
    
    5. Review the available options. Choose the available option you want to use, and when you're ready, choose Create app.

Environment creation takes about five minutes and creates the following resources.
#
![wordpresscloud](https://user-images.githubusercontent.com/29739578/209721981-61548c4e-0ecf-4bc8-9f65-204d753f53bd.PNG)
#
All of these resources are managed by Elastic Beanstalk. When you terminate your environment, Elastic Beanstalk terminates all the resources that it contains.

Because the Amazon RDS instance that you launched is outside of your environment, you are responsible for managing its lifecycle.

Note
The Amazon S3 bucket that Elastic Beanstalk creates is shared between environments and is not deleted during environment termination. For more information, see Using Elastic Beanstalk with Amazon S3.
#
### IV. Configure security groups and environment properties
#
Add the security group of your DB instance to your running environment. This procedure causes Elastic Beanstalk to reprovision all instances in your environment with the additional security group attached.

#### To add a security group to your environment
Do one of the following:
To add a security group using the Elastic Beanstalk console

    1.Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

    2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.

    3. In the navigation pane, choose Configuration.

    4. In the Instances configuration category, choose Edit.

    5.  Under EC2 security groups, choose the security group to attach to the instances, in addition to the instance security group that Elastic Beanstalk creates.

    6. Choose Apply.

    7. Read the warning, and then choose Confirm.

    To add a security group using a configuration file, use the securitygroup-addexisting.config example file.

Next, use environment properties to pass the connection information to your environment.

The WordPress application uses a default set of properties that match the ones that Elastic Beanstalk configures when you provision a database within your environment.

**To configure environment properties for an Amazon RDS DB instance**

    1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

    2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.

    3. In the navigation pane, choose Configuration.

    4. In the Software configuration category, choose Edit.
    
    5. In the Environment properties section, define the variables that your application reads to construct a connection string. For compatibility with environments that have an integrated RDS DB instance, use the following names and values. You can find all values, except for your password, in the RDS console.

![rdsimg](https://user-images.githubusercontent.com/29739578/209722751-5bdbb998-2a2b-4a14-9ddb-62185b3e40b6.PNG)

#
![environment-cfg-envprops-rds](https://user-images.githubusercontent.com/29739578/209722769-84012cf8-86ec-4c30-9b81-8f1d4ada5aa9.png)

    6. Choose Apply. 
#
### V. Configure and deploy your application

Verify that the structure of your wordpress-beanstalk folder is as shown.
#
![trckbak](https://user-images.githubusercontent.com/29739578/209722992-245ccbea-6199-4458-9731-d3870db8022d.PNG)
#


The customized wp-config.php file from the project repo uses the environment variables that you defined in the previous step to configure the database connection. The .ebextensions folder contains configuration files that create additional resources within your Elastic Beanstalk environment.

The configuration files require modification to work with your account. Replace the placeholder values in the files with the appropriate IDs and create a source bundle.

**To update configuration files and create a source bundle**
    1. Modify the configuration files as follows.

        .ebextensions/dev.config ??? Restricts access to your environment to protect it during the WordPress installation process. Replace the placeholder IP address near the top of the file with the public IP address of the computer you'll use to access your environment's website to complete your WordPress installation.

*Note
Depending on your network, you might need to use an IP address block.*

    .ebextensions/efs-create.config ??? Creates an EFS file system and mount points in each Availability Zone/subnet in your VPC. Identify your default VPC and subnet IDs in the Amazon VPC console.

    2. Create a source bundle containing the files in your project folder. The following command creates a source bundle named wordpress-beanstalk.zip.

    ~/eb-wordpress$ zip ../wordpress-beanstalk.zip -r * .[^.]*
    Upload the source bundle to Elastic Beanstalk to deploy WordPress to your environment.

#
    To deploy a source bundle
        1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

        2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.



        3. On the environment overview page, choose Upload and deploy.

        4. Use the on-screen dialog box to upload the source bundle.

        5. Choose Deploy.

        6. When the deployment completes, you can choose the site URL to open your website in a new tab.

#
### VI. Install WordPress
#
        1. To complete your WordPress installation open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

        2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.



        3. Choose the environment URL to open your site in a browser. You are redirected to a WordPress installation wizard because you haven't configured the site yet.

        4. Perform a standard installation. The wp-config.php file is already present in the source code and configured to read the database connection information from the environment. You shouldn't be prompted to configure the connection.

#
### VII. Update keys and salts
#
The WordPress configuration file wp-config.php also reads values for keys and salts from environment properties. Currently, these properties are all set to test by the wordpress.config file in the .ebextensions folder.

The hash salt can be any value that meets the environment property requirements, but you should not store it in source control. Use the Elastic Beanstalk console to set these properties directly on the environment.

#### To update environment properties
            1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

            2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.

            3. On the navigation pane, choose Configuration.

            4. Under Software, choose Edit.

            5. For Environment properties, modify the following properties:

                    AUTH_KEY ??? The value chosen for AUTH_KEY.

                    SECURE_AUTH_KEY ??? The value chosen for SECURE_AUTH_KEY.

                    LOGGED_IN_KEY ??? The value chosen for LOGGED_IN_KEY.

                    NONCE_KEY ??? The value chosen for NONCE_KEY.

                    AUTH_SALT ??? The value chosen for AUTH_SALT.

                    SECURE_AUTH_SALT ??? The value chosen for SECURE_AUTH_SALT.

                    LOGGED_IN_SALT ??? The value chosen for LOGGED_IN_SALT.

                    NONCE_SALT ??? The value chosen for NONCE_SALT.

            6. Choose Apply.

*Note
Setting the properties on the environment directly overrides the values in wordpress.config.

#
### VIII. Remove access restrictions
#
The sample project includes the configuration file loadbalancer-sg.config. It creates a security group and assigns it to the environment's load balancer, using the IP address that you configured in dev.config. It restricts HTTP access on port 80 to connections from your network. Otherwise, an outside party could potentially connect to your site before you have installed WordPress and configured your admin account.

Now that you've installed WordPress, remove the configuration file to open the site to the world.

#### To remove the restriction and update your environment
    1. Delete the .ebextensions/loadbalancer-sg.config file from your project directory.

        ~/wordpress-beanstalk$ rm .ebextensions/loadbalancer-sg.config
        
    2. Create a source bundle.

        ~/eb-wordpress$ zip ../wordpress-beanstalk-v2.zip -r * .[^.]*
Upload the source bundle to Elastic Beanstalk to deploy WordPress to your environment.

#### To deploy a source bundle
    1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

    2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.


    3. On the environment overview page, choose Upload and deploy.

    4. Use the on-screen dialog box to upload the source bundle.

    5. Choose Deploy.

    6. When the deployment completes, you can choose the site URL to open your website in a new tab.

#
### IX. Configure your Auto Scaling group
#
Finally, configure your environment's Auto Scaling group with a higher minimum instance count. Run at least two instances at all times to prevent the web servers in your environment from being a single point of failure. This also allows you to deploy changes without taking your site out of service.

####To configure your environment's Auto Scaling group for high availability
    
    1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

    2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.


    3. In the navigation pane, choose Configuration.

    4. In the Capacity configuration category, choose Edit.

    5. In the Auto Scaling group section, set Min instances to 2.

    6. Choose Apply.

To support content uploads across multiple instances, the sample project uses Amazon EFS to create a shared file system. Create a post on the site and upload content to store it on the shared file system. View the post and refresh the page multiple times to hit both instances and verify that the shared file system is working.

#
### X. Upgrade WordPress
To upgrade to a new version of WordPress, back up your site and deploy it to a new environment.
            
            Important
            Do not use the update functionality within WordPress or update your source files to use a new version. Both of these actions can result in your post URLs               returning 404 errors even though they are still in the database and file system.

#### To upgrade WordPress
    1. In the WordPress admin console, use the export tool to export your posts to an XML file.

    2. Deploy and install the new version of WordPress to Elastic Beanstalk with the same steps that you used to install the previous version. To avoid downtime, you can create an environment with the new version.

    3. On the new version, install the WordPress Importer tool in the admin console and use it to import the XML file containing your posts. If the posts were created by the admin user on the old version, assign them to the admin user on the new site instead of trying to import the admin user.

    4. If you deployed the new version to a separate environment, do a CNAME swap to redirect users from the old site to the new site.

#
### XI. Clean up
#
When you finish working with Elastic Beanstalk, you can terminate your environment. Elastic Beanstalk terminates all AWS resources associated with your environment, such as Amazon EC2 instances, database instances, load balancers, security groups, and alarms.

####To terminate your Elastic Beanstalk environment
                
                1. Open the Elastic Beanstalk console, and in the Regions list, select your AWS Region.

                2. In the navigation pane, choose Environments, and then choose the name of your environment from the list.

                *Note
                If you have many environments, use the search bar to filter the environment list.

                3. Choose Environment actions, and then choose Terminate environment.

                4. Use the on-screen dialog box to confirm environment termination.

With Elastic Beanstalk, you can easily create a new environment for your application at any time.

In addition, you can terminate database resources that you created outside of your Elastic Beanstalk environment. When you terminate an Amazon RDS DB instance, you can take a snapshot and restore the data to another instance later.

#### To terminate your RDS DB instance
                   
                    1. Open the Amazon RDS console.

                    2. Choose Databases.

                    3. Choose your DB instance.

                    4. Choose Actions, and then choose Delete.

                    5. Choose whether to create a snapshot, and then choose Delete.
#
### Congratulations! 
#
You built the infrastructure and environments along with the pertinent instances.  You now know how to deploy a wordpress site using Amazon RDS Console and Services. 


Finally, if you decide to use your application in an active production environment, you will need to configure a custom domain name for your environment along with enabling HTTPS for more secure connections.
