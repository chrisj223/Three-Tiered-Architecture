# Three-Tiered-Architecture
This project, part of my college coursework, simulates the setup of a documentation management web server for a company. It features a three-tier architecture, incorporating redundancy with failover using HAProxy load balancing, ensuring server persistence, and enhancing security through encryption. This is done utilizing the provider Amazon Web Services.

Resources used
- Amazon Web Services (AWS)
- Amazon Linux 2
- HAProxy Load Balances
- MySQL/MariaDB
- SeedDMS 5.1


# Setting up EC2 Instance
For the first steps this is going to be the steps on creating your AWS insatnce (VM). This would be performed multiple times has there is a total of 5 instances.

- Log into your AWS account
- Navigate to the saearch and search for "EC2"
  
<img width="252" alt="111" src="https://github.com/user-attachments/assets/7ddfe43c-3df9-47cf-a859-8bb532100bf8">

- Near the top right click "Launch Instances" then the first option "Launches Instances" again

 <img width="224" alt="Picture2" src="https://github.com/user-attachments/assets/4ffba155-58c3-441c-a62f-9bfd820723f2">

- Enter in desired information you wish to choose

![Picture3](https://github.com/user-attachments/assets/d1e05f4d-33b1-412c-a1ec-da13ded24255)

- Be sure to create your key pair. _Ensure that you save this where you access it_

![image](https://github.com/user-attachments/assets/7e04785f-59aa-4c2a-bc44-43905b82089c)

- Configure your security rules to allow HTTPS, MySQL, and HTTP

![image](https://github.com/user-attachments/assets/ad00e61f-6f08-4dcf-b0e2-2024bda8e998)

- Now we can launch the instance


# Connecting to Instances
This step is to elaborate on how to connect to your instance via SSH client.

- Within the EC2 menu choose the desired instance you'll connect to and start it; then click the "Connect" option
- Choose the SSH Client tab, that is how we are connecting

![image](https://github.com/user-attachments/assets/10d4a7ed-7658-42ce-973e-1586bee8dbbf)

- Open your terminal and navigate to the key pair you saved earlier on, then log into your instance

![image](https://github.com/user-attachments/assets/4c70a367-9236-49ad-9155-15b92a3734a3)

- You are now logged into your instance

# Setting up Database Server
This step is to setup your database server. Ensure you connect to the instance that you want as your database.

- Connect to your instance you designated as your database server
- Once connected, installed MariaDB by running the following command `sudo yum install mariadb maria-dbserver -y`
- Now enable MariaDB to start on reboot by running the command `sudo systemctl enable mariadb` 
- Once all the above steps are completed, log into your database using the command `mysql -u root`

![image](https://github.com/user-attachments/assets/0583b030-52e9-499b-8f2d-e5e14a2a3e6f)

- First you'll create a database. Run the command `CREATE DATABASE [name.of.db];`
   
![image](https://github.com/user-attachments/assets/189af68c-ac4a-428c-90e8-29e685c8d1b9)

- Run `show databases;` to verify it was created

![image](https://github.com/user-attachments/assets/a0253187-63f0-400b-b642-07d2f8ee2dec)

- Lets now create a user that can login remotely. Run the command `create user '[username]'@'%' identified by '[password]';`

![image](https://github.com/user-attachments/assets/aeab7a91-8324-427f-84ed-39b5cd0cb97f)

- To verifiy the user created run `select * from mysql.user;` _(this way of checking isn't prefered but was utlized at the time; `SELECT user,host FROM mysql.user;` provides cleaner output)_  

![image](https://github.com/user-attachments/assets/86c46d0a-6b4b-4da5-acae-7bb8aa765d74)

- We will now assign privileges to the user of the database we created. _For simplicity, we assigned the user all privilegs, however in production environments you would give the privileges the users need to perform their task_
- Run the command `grant all privileges on [db.you.choose].* to '[user]'@'%';`

![image](https://github.com/user-attachments/assets/e081ca2a-37fb-40c6-9a05-3d9bf67b5c88)

- To verify if the grants where given run the command `show grants for '[user]'@'%';`

![image](https://github.com/user-attachments/assets/a11dbceb-d3dc-4ac8-a7ea-256e959ae3e7)

- We have now finished setting up the database


# Setting up Webservers
This is shows the steps of setting up the webserver. This step would be repeated 3 times as we have 3 active webservers running. So, launch one of your instaces and then repeat for other instances or copy/dupilcate instances, just ensure that the IPs and other networking configurations don't conflict one another.

### Installing Apache & PHP
- Once you've started/connected to the instance run the command `sudo yum update -y`
- Then install apache by running `sudo yum install httpd` then enable it `sudo systemctl enable httpd`
- Now a quick turn, we need php packages. However, since we are using a AWS Linux 2 Instance, it is done through amazon extras.
- To verify if you have Amazon extars installed run the command `which amazon-linux-extras` then it'll given a return output. If not installed run this command `sudo yum install -y amazon-linux-extras`
- Now once that is verified we will search for which versions are available for php through the Extras Library by running `sudo  amazon-linux-extras | grep php`

![image](https://github.com/user-attachments/assets/ce338cf2-8afd-412b-8d9b-0d1fe14fe6dd)

- Choose whichever you desire, we choose a stable one. After run this command to download/enable the package `sudo amazon-linux-extras enable php7.4`
- Now we will insatll by first running `yum clean metadata` and then run `sudo yum install php php-{pear,cgi,common,curl,mbstring,gd,mysqlnd,gettext,bcmath,json,xml,fpm,intl,zip,imap}`
- Verify the installation but running `php -version`

![image](https://github.com/user-attachments/assets/14f37461-b81b-479a-813a-aa99e55563b1)

### Installing SeedDMS
- Next we will install SeedDMS. Navigated to `/var/www/html` folder to simplify matters
- Once there used the wget command to download the package/file by running `wget https://sourceforge.net/projects/seeddms/files/seeddms-5.1.22/seeddms-quickstart-5.1.22.tar.gz` then extract the compressed file by running `tar xvf seeddms-quickstart-5.1.22.tar.gz`. Once extracted you may move or removed the original compressed file.
- Before we forget lets give apache ownership to the conf and data directory by running `sudo chown -R apache /var/www/html/seeddms51x/conf/` for the conf and then `sudo chown -R apache /var/www/html/seeddms51x/data/` for the data.
- Now navigate to the conf folder of the seeddms folder and delete the setting,xml file by running `sudo rm settings.xml` then create a new file by running `touch ENABLE_INSTALL_TOOL`. This is needed for the setup process of the webserver.
- Now open your webserver instance either via DNS or IP using http and attached `/seeddms` at the end. Then a setup screen shoudl appear like below

![image](https://github.com/user-attachments/assets/bcb4f707-7075-4a6b-b154-a97e8552dc53)

- Fill in the fields with the database settings we created earlier
- For the section "Extra PHP" include the path `/var/www/html/pear`
- Once completed, click "Apply"; upon the second page it directs you too, choose the first option
- Back to the terminal of the webserver, navigate to the apache conf file by running `cd /etc/httpd/conf`

![image](https://github.com/user-attachments/assets/e217eeb6-9381-41f8-8b3e-dcae14bfb281)

- Now edit the httpd.conf file with and text editing command, we used nano
- Within the httpd.conf file, fine "Document Root" and then alter the path to `/var/www/html/seeddms`

![image](https://github.com/user-attachments/assets/1138025d-fd53-47b9-8acd-6a384be1d094)

- Remind to repeat these steps for the other 2 webservers

  
# Setting up HAProxy Load Balancer
In this step we will setup one of the Instances to function as the load balancer and the incorporate SSL traffic for encryption. This was done with an Ubuntu OS.

- First, we will install HAProxy package by running `apt-get install --no-install-recommends software-properties-common` then run the command `add-apt-repository ppa:vbernat/haproxy-2.6`
- Lastly, we will now install HAProxy by running `apt-get install haproxy=2.6.\*` then to verify the installation run `haproxy -version`

![image](https://github.com/user-attachments/assets/e61d2b1d-5732-4bae-8b7b-55ec80a0462f)



### Setting up SSL Certificate
- Still within the HAProxy, we set up our SSL certificate with 2048 bit length. _Ensure you have openssl installed_
- First command we'll run is `sudo mkdir /etc/ssl/xip.io` this will be the directory of the certificate location
- We will now generate key by running this command `sudo openssl genrsa -out /etc/ssl/xip.io/xip.io.key 2048` then to generate the certificate signing reuqest run `sudo openssl req -new -key /etc/ssl/xip.io/xip.io.key \ -out /etc/ssl/xip.io/xip.io.csr` Fill in the information with what it prompts you with and give it a password
- Now we will sign the certificate and setting expiration parameters by running the command `sudo openssl x509 -req -days 365 -in /etc/ssl/xip.io/xip.io.csr \ -signkey /etc/ssl/xip.io/xip.io.key \ -out /etc/ssl/xip.io/xip.io.crt`
- Now we generate the .PEM format file to store everything the together by running the command `sudo cat /etc/ssl/xip.io/xip.io.crt /etc/ssl/xip.io/xip.io.key \ | sudo tee /etc/ssl/xip.io/xip.io.pem`
- Now that you have generated the certificate you need to bind it within the haproxy.cfg. We will also add the failover safe by adding the roundrobin. Navigate to `/etc/haproxy/haproxy.cfg` and use a desired text editor to edit the file
- Once your in the file, locate the "Frontend level" and "Backend level". To bind the ssl certiifcate, underneath "Frontend level" input next to "bind" `*:443 ssl crt /etc/ssl/[your .PEM file path]`
- Now to ensure fault tolerance, underneath the "Backend level" configure balance ensuring `roundrobin` is in place. Then underneath you'll add the servers that it'll route to format like `sever <sever namer> <IP Address>:80 check` or similar to below. Then save the file.

![image](https://github.com/user-attachments/assets/c208832f-a09e-49c6-82b6-0d38b5d4d605)

- Now verify, by accessing the HAProxy server through the broswer, by checking the certificate and swap between servers each refresh

![image](https://github.com/user-attachments/assets/b972ade2-d8a3-49b4-a6a3-901764e3d5a1)


### Setting up Cookies
This last step is short but helps add persistence to have the session stay remembered.

- Within the HAProxy navigate back to the haproxy.cfg file. Naviagte to "backend level" and insert the line `cookie SERVERID indirect nocache` underneath balance similar to below

![image](https://github.com/user-attachments/assets/23678de6-744f-4d24-a5b8-122e8f01e5f1)

- Once you save the file then you can go back to your server to verify
