# Standard Operating Procedure for Installation and Configuration of Opal/DataSHIELD in NFDI4Health 

This is the Standard Operating Procedure (SOP) for Installing and Configuring Opal/DataSHIELD for the NFDI4Health consortium. It is based on an earlier version that was developed within the ENPADASI consortium by DIfE.

Opal is a data management application that is integrated with R. It is implemented with DataSHIELD (a series of R packages) which allows advanced statistical data analysis across multiple studies without sharing and disclosing any individual-level data[^1]. 

If needed, you can find the official Opal documentation here: https://opaldoc.obiba.org/en/latest/. For troubleshooting and addressing software issues you can go here: https://groups.google.com/g/obiba-users or https://github.com/obiba/opal/issues. 

This SOP is for internal use within NFDI4Health and will get updated with the progression of NFDI4Health. If you have any feedback for the SOP or require support, please contact me at sofiamaria.siampani@mdc-berlin.de.

OBiBa software are open source and made available under the GPL3 licence. OBiBa software are free of charge.

The work presented herein was made possible using the OBiBa suite (www.obiba.org), a software suite developed by Maelstrom Research (www.maelstrom-research.org) and Epigeny (www.epigeny.io)

## Installation and Configuration of Opal/DataSHIELD
### Server Hardware Requirements

The following server hardware requirements need to be met:

Recommended Operating system: RedHat/CentOS
-  Recent server-grade or high-end consumer-grade processor
-	8GB or more disk space
-	4GB or more RAM

Since every institute has their own hardware preferences/requirements, we present the following options based on the official Opal documentation[^2]:
1)	Native installation
    - Debian package installation (Linux based operating systems) or
    - RPM package installation (Red Hat Linux based operating systems) 
2)	Installation using Docker

Both options are fine for the Federated Analysis to take place but we recommend that you choose the option that best meets your institute’s security needs.

The person who is setting up Opal/DataSHIELD needs to have administrative rights on the server(s). 

## Native Installation
### Server Software Requirements

1)	Install Java 8 if it’s not already installed. For installing JDK 8 for your operating system, reference: https://openjdk.org/install/.  
2)	Install and configure the database[^3] : Opal uses a database for storing data. Currently the supported SQL database engines are MySQL, MariaDB and PostgreSQL. Here we are suggesting MySQL >= 5.5.x but the other supported databases also work.
    - Reference for installing MySQL based on your operating system: https://dev.mysql.com/downloads/mysql/.
    - For Post-installation Set up and Testing follow this: https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/default-privileges.html
    - Create the databases: A fully operational Opal server requires to have at least two different databases registered for identifiers mapping storage and data storage (at least one is required).
      - Log into the database shell with the root password you set before:
        ```
        sudo mysql -u root -p
        ```
      - After successful login you will see the database prompt and can continue the set up.
        ```
        mysql>
        ```
      - Write these commands one by one in the database prompt in order to create the databases and user required by Opal. Once done, you can quit the cell.
        ```
        CREATE DATABASE opal_data;
        CREATE DATABASE opal_ids;
        CREATE USER 'opal'@'localhost' IDENTIFIED BY '<opal-user-password>';
        GRANT ALL ON opal_data.* TO 'opal'@'localhost'; 
        GRANT ALL ON opal_ids.* TO 'opal'@'localhost'; 
        FLUSH PRIVILEGES;
        QUIT;
        ```
3) Install R >= 4.x: Reference this for installing R and recommended packages (r-base, r-dev) based on your operating system: https://cran.r-project.org/.

### Installation of Opal & Rock (R Server configured for Opal)

4)	Install Opal and Rock[^4]. For Rock, both Java and R must be installed on the same host.
    - For Debian: https://www.obiba.org/pages/pkg/
    - For RPM package installation: https://www.obiba.org/pages/rpm/

During this step you will be asked to set a strong password, which you will need later to login as administrator into Opal’s web interface. The password must contain at least 8 characters, with at least one digit, one upper case alphabet, one lower case alphabet, one special character (which includes @#$%^&+=!) and no white space.

### Configuration
5)	Edit this file: OPAL_HOME/conf/opal-config.properties to match your server needs. (https://opaldoc.obiba.org/en/latest/admin/configuration.html). 
6)	Set up Reverse Proxy Configuration (e.g. Apache[^5])  and replace Opal’s default certificate with a valid one in order to match your institute’s security needs.
7)	Now Opal is ready for the first login. The Opal’s administrative web interface can be accessed with any modern web browser by addressing:
    ```
    https://<host>:8843
    ```
8)	The first site you will see is the login page. Sign in with ‘administrator’ as username and the Opal password you set previously.
![image](https://github.com/sofiasiamp/opal-datashield/assets/104575409/b51337cb-69c8-4d3f-b1f4-e8b7bcba0ad1)


#### Database Configuration

9)	Go to “Administration” > “Databases”. In the "Identifiers Database" section, click on "Register" and select the database type that you set up (“SQL” or “MongoDB”). Input information for the database: provide the name of the database along with connection information (URL, username, and password) and save the settings. Do the same for the “Data Databases”. Upon successful configuration a notification in blue box pop ups saying “Connection successful”.

![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/e1b0231b-2d61-4169-af6c-8822b4afee53)
![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/5b6f9943-fc18-478b-9b46-b4204ebde5a8)



#### Installing DataSHIELD packages

10) Go to “Adminstration”>”DataSHIELD”. Click on “Add Package”, choose the option ‘Install all DataSHIELD packages’ and click ‘Install’.The DataSHIELD packages should now appear in the list and the “Methods” section should be populated with entries. The initial set up is now complete.

![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/0068bf38-70d8-4049-b564-f0eee027dd90)


#### Upgrade
If you installed Opal via the Debian package, you may update it using the command:
```
apt-get install opal
```
If you installed Opal via the RPM package, you may update it using the command:
```
yum install opal-server
```

## Installation using Docker

OBiBa is an early adopter of the Docker technology, providing its own images from the Docker Hub repository. The Docker platform runs natively on Linux (on x86-64, ARM and many other CPU architectures) and on Windows (x86-64). Before proceeding with this option, make sure that this technology matches your institute’s security needs[^6].

### Docker Image Installation

1)	Install Docker[^7] and create an account. 
2)	Open a text editor (e.g. Notepad) on your machine and paste the following: 
```
version: '3' 
services: 
  opal: 
    image: obiba/opal:latest 
    ports: 
      - "8843:8443" 
      - "8880:8080" 
    links: 
      - mysqldata 
      - mysqlids 
      - rock 
    environment: 
      - JAVA_OPTS=-Xms1G -Xmx8G -XX:+UseG1GC 
      - OPAL_ADMINISTRATOR_PASSWORD=password 
      - MYSQLDATA_HOST=mysqldata 
      - MYSQLDATA_USER=opal 
      - MYSQLDATA_PASSWORD=password 
      - MYSQLIDS_HOST=mysqlids 
      - MYSQLIDS_USER=opal 
      - MYSQLIDS_PASSWORD=password 
      - ROCK_HOSTS=rock:8085 
      - ROCK_ADMINISTRATOR_USER=administrator 
      - ROCK_ADMINISTRATOR_PASSWORD=password 
    volumes: 
      - ./opal:/srv 
  mysqldata: 
    image: mysql:5 
    environment: 
      - MYSQL_DATABASE=opal 
      - MYSQL_ROOT_PASSWORD=password 
      - MYSQL_USER=opal 
      - MYSQL_PASSWORD=password 
    volumes: 
      - ./mysqldata:/var/lib/mysql 
  mysqlids: 
    image: mysql:5 
    environment: 
      - MYSQL_DATABASE=opal_ids 
      - MYSQL_ROOT_PASSWORD=password 
      - MYSQL_USER=opal 
      - MYSQL_PASSWORD=password 
    volumes: 
      - ./mysqlids:/var/lib/mysql 
  rock: 
    image: datashield/rock-base:latest 
    environment: 
      - ROCK_ADMINISTRATOR_NAME=administrator 
      - ROCK_ADMINISTRATOR_PASSWORD=password
```

3)	Reference this https://opaldoc.obiba.org/en/latest/admin/installation.html#docker-image-installation to check all environment variables. Adjust the values, making sure, you are using strong passwords. The password must contain at least 8 characters, with at least one digit, one upper case alphabet, one lower case alphabet, one special character (which includes @#$%^&+=!) and no white space. Also, make sure that the ROCK_ADMINISTRATOR_PASSWORD matches in both instances.
4)	Save the file as “docker-compose.yml”. 
5)	Open the command line in your machine and run the following command in the directory where the docker-compose.yml file is stored.
```
docker-compose -f docker-compose.yml up -d
```
This will create and configure the Opal server, the MySQL databases, a DataSHIELD ready R server and all useful plugins.
6)	Set up Reverse Proxy Configuration (e.g. Apache[^5]) and replace Opal’s default certificate with a valid one in order to match your institute’s security needs.
7)	Now Opal is ready for the first login. The Opal’s administrative web interface can be accessed with any modern web browser by addressing:
```
https://<host>:8843
```
8)	The first site you will see is the login page. Sign in with ‘administrator’ as username and the Opal password you set previously.
9)	Go to Go to “Administration” > “Databases” and confirm that the databases are successfully configured and connected.
10)	Go to “Adminstration”>”DataSHIELD” and confirm that “dsBase” is installed.

The initial set up is now complete.

#### Upgrade
Change the docker image version and restart the docker container. If the opal-home directory was mounted in user space, it will be reused.

# Importing the dataset
After the harmonization process, you will end up having two files:
1.	a data dictionary in an xls/xlsx format
2.	a harmonized dataset in a CSV format

## Uploading the dataset and data dictionary
These two files need to be uploaded before they can be imported. The files need to be accessible from the Opal server. For that purpose, Opal has a "file system" where the data files can be uploaded.

From the "Dashboard" page, click on "Manage Files": Navigate to the directory where the data file will be uploaded. Click on the "Upload" button; this will open a "File Upload" window which allows you to select a file to upload from your computer. Click on "Choose File" and select the data dictionary file (*-dictionary.xlsx). Once done, click on the "Upload" button. Do the same with the data source file (*.csv).

![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/f4c29ae7-a3ae-4905-8298-6a527446e162)

![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/702ea792-d022-497b-ad50-49c20e3e42a8)

## Creating a project
In Opal, a project is a workspace for managing data. It is required to create a project before importing data into Opal.

1)	Go to the “Projects” page.
2)	Click on “Add Project” button. 
3)	In the “Add Project” popup window:
    - Enter the name of the project “NFDI4Health”
    - Select name of database created by the installation for default storage.
    - Optionally enter the project title and description
    - “Save” the project.
    - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/d7fd77fa-ea1b-4e0e-bcc4-8d306de038d3)
    - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/f5b24676-1e68-49bd-99e2-4fff2fc442d5)


## Importing the dataset
4)	Go to this project’s table section.
5)	Click on “Add Table” and select “Add/update” from dictionary…”.                                                                                                                                                                                                                                                     ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/b6c1b02f-865a-4ff0-b7ac-9e00ce8cfb65)

6)	In the “Add/Update Tables” popup window:
    - Click on “Browse”. In the “File Selector” pop-up window:
    - Go to the projects section and click on the project name.
    - Choose the previously uploaded dictionary file *-dictionary.xlsx by ticking the checkbox to the left of the file name and click on “Select”. 
    - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/8ad2c5aa-dd99-4a00-9966-0581bbae4eb9)
    - Click on “Next”. 
7) In the window choose the dataset file (in csv format) and click on “Finish”.
    ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/e0478894-133b-4d6a-83e0-13f4defee313)
    The imported table should appear now in the project’s table section. 
    - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/c5b23002-6f6b-4395-8e48-e29cd32f3b85)

8)	Click on “Import” (the second one in the button group on the right) in tables section.
9)	In the “Import Data” pop-up window:
    - Select CSV data format and click on “Next”.
    - Click on “Browse”.
    - In the “File Selector” popup window:
      - Go to the projects section and click on the project name.
      - Choose the previously uploaded CSV data file by ticking the checkbox to the left of the file name and click on “Select”.
      - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/e3b48151-6aca-43a3-8101-07e51f4c08f1)

    - Enter the name of the table (as described in the data dictionary) in destination table and click on “Next”.
    - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/c0c6d821-14b2-442b-8c2e-a4da003c6f39)

    - Click on “Next “in the next window with default settings. 
    - Tick the checkbox to the left of the test table and click on “Next”. 
    - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/7479c03d-0c8d-474d-87fc-66b8c1a90c81)

    - Click on “Finish”.
    - ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/9db4efee-5642-4771-950a-5a9fbbe4b49a)

  10) Data importation is completed. You can see the data by clicking on the table test in the table section of the projects page.
      ![image](https://github.com/sofiasiamp/opal-datashield-sop/assets/104575409/779c338c-1f9d-417a-995e-de38d97bd2ab)

# Next steps
Once the initial set up Opal/DataSHIELD and you have imported the dataset successfully, the next step is “Creating Users and Setting DataSHIELD permissions”. This section will be updated here with the progression of the pilot projects that are planned in NFDI4Health.



[^1]: https://www.obiba.org/pages/products/opal/
[^2]: https://opaldoc.obiba.org/en/latest/
[^3]: https://opaldoc.obiba.org/en/latest/web-user-guide/administration/databases.html#databases
[^4]: https://rockdoc.obiba.org/en/latest/ 
[^5]: https://opaldoc.obiba.org/en/latest/admin/configuration.html#reverse-proxy-configuration
[^6]: https://docs.docker.com/engine/security/security/
[^7]: https://www.docker.com/products/docker-desktop

