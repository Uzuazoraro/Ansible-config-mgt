# ansible-config-mgt

## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS: PROJECT 12 

### Step 1 – Jenkins job enhancement

1. Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact. we will store there all artifacts after each build.
`sudo mkdir /home/ubuntu/ansible-config-artifact`

2. Change permissions to this directory, so Jenkins could save files there.
`chmod -R 0777 /home/ubuntu/ansible-config-artifact`

3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

4. Create a new Freestyle project and name it save_artifacts.

5. Configure and complete your existing ansible project. This configuration will trigger this project.

6. The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory.

7. Test your set up by making some change in README.MD file inside your ansible-config-mgt repository 

ansible webhook trigger
<<<<<<< HEAD

copy artifact trigger

### Step 2 – Refactor Ansible code by importing other playbooks into site.yml

1. Within playbooks folder, create a new file and name it site.yml

2. Create a new folder in root of the repository and name it static-assignments

3. Move common.yml file into the newly created static-assignments folder.

4. Inside site.yml file, import common.yml playbook.

5. Run ansible-playbook command against the dev environment
Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.

`cd /home/ubuntu/ansible-config-mgt/`

`ansible-playbook -i inventory/dev.yml playbooks/site.yml`

### Step 3 – Configure UAT Webservers with a role ‘Webserver’

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT

2. Create the "roles" directory.

`mkdir roles`
`cd roles`
`ansible-galaxy init webserver`

Note that directory/files can also be done manually.

3. Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers

[uat-webservers]
172.31.84.175 ansible_ssh_user='ec2-user' 

172.31.86.187 ansible_ssh_user='ec2-user' 

4. In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.

5. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

Install and configure Apache (httpd service)
Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started

### Step 4 – Reference ‘Webserver’ role

Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.

---
- hosts: uat-webservers
  roles:
     - webserver

Update site.yml file]

---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml

### Step 5 – Commit & Test

`git status`
`git add`
`git commit`

Now run the playbook against your uat inventory and see what happens:

`sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml`

You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

`http://54.197.170.142/index.php`

or

`http://3.87.5.119/index.php`


=======
>>>>>>> b9de4ca3f4f8812c3f4aa14986f63bc9f22c15bd


# Project 13 Ansible Dynamic Assignments (Include) and Community Roles

## Step 1 - Introducing Dynamic Assignment Into Our structure

1. In your https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.

`git checkout -b dynamic-assignments`

2. Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml.

`mkdir dynamic-assignments`

`touch env-vars.yml`

3. Create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

`mkdir env-vars`
`touch dev.yml`
`touch stage.yml`
`touch uat.yml`
`touch prod.yml`

## Step 2 - Update site.yml with dynamic assignments

site.yml should now look like this.

---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml

## Step 3 - Community Roles

1. Create a role for MySQL database. It should install the MySQL package, create a database and configure users. 

Download Mysql Ansible Role (galaxy.ansible.com/home)

You can browse available community roles here and we'll be using a MySQL role developed by geerlingguy.

`cd roles`

`ansible-galaxy install geerlingguy.mysql`

2. Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql

mv geerlingguy.mysql/ mysql

Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

Now it is time to upload the changes into your GitHub:

git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin dynamic-assignments

Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.

## Step 4 - Load Balancer roles

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

1. Nginx
2. Apache

`cd roles`

`ansible-galaxy install geerlingguy.nginx`

`ansible-galaxy install geerlingguy.apache`

`mv geerlingguy.nginx/ nginx`

`mv geerlingguy.apache/ apache`

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

Update both assignment and site.yml files respectively


# Project 14 Experience Continuous Integration With Jenkins | Ansible | Artifactory | Sonarqube | PHP

## STEP 1 - ANSIBLE ROLES FOR CI ENVIRONMENT

## Install Java and JAVA path:

ubuntu@ip-172-31-84-49:~$ vi .bash_profile
ubuntu@ip-172-31-84-49:~$ source ~/.bash_profile
ubuntu@ip-172-31-84-49:~$ cat .bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH

export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVE_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools/jar

Now go ahead and Add two more roles to ansible:

1. SonarQube (Scroll down to the Sonarqube section to see instructions on how to set up and configure SonarQube manually)
2. Artifactory

## Configuring Ansible For Jenkins Deployment
In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

To do this,

1. Install or Navigate to Jenkins URL

`curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null`
`echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null`
`sudo apt-get update`
`sudo apt-get install jenkins`

2. Install & Open Blue Ocean Jenkins Plugin by clicking on the blue ocean after installation.

3. Create a new pipeline

4. Select GitHub

5. Connect Jenkins with GitHub

6. Login to GitHub & Generate an Access Tokenhttps://www.darey.io/wp-content/uploads/2021/07/Jenkins-Create-Access-Token-To-Github.png
Click on create-access-token under Connect to GitHub.

7. Copy Access Token

8. Paste the created token in Jenkins and click on connect.

9. Click or select the name of the organization (Uzuazoraro)
Click on the repository you're working on.
Click on "Create pipeline."

Click on Administration to exit the Blue Ocean console.

## Let us create our Jenkinsfile

Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.

Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}

Now go back into the Ansible pipeline in Jenkins, and select configure

Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile

First of all, click on the project repository. Then click configure. Click Build Configuration.
Configure. Note that your Jenkins file is in a repository. Configure it as "deploy/Jenkinsfile"

Click on main in your Jenkins repo. Then click "build now."

This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

Click on Blue Ocean

## 1. Create a new git branch and name it feature/jenkinspipeline-stages with the command below:
`git checkout -b feature/jenkinspipeline-stages`

2. pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}

3. To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

Click on the "Administration" button

4. Navigate to the Ansible project and click on "Scan repository now"

5. Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

## RUNNING ANSIBLE PLAYBOOK FROM JENKINS

1. Installing Ansible on Jenkins
2. Installing Ansible plugin in Jenkins UI
3. Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

### Jenkinsfile for quick task
======================================
---
 pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }

    stage('Package') {
      steps {
        script {
          sh 'echo "Package Stage"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Deploy Stage"'
        }
      }
    }

    stage("Clean up") {
      steps {
        cleanWs()
      }
    } 

    }
}
---

Note: Ensure that Ansible runs against the Dev environment successfully.

pipeline {
  agent any

  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }

  stages {
    stage("Initial cleanup") {
      steps {
        dir("${WORKSPACE}") {
          deleteDir()
        }
      }
    }

    stage('Checkout SCM') {
      steps {
        git branch: 'feature/jenkinspipeline-stages', url: 'https://github.com/Uzuazoraro/ansible-config-mgt.git'
      }
    }

    stage('Prepare Ansible For Execution') {
      steps {
        sh 'echo ${WORKSPACE}'
        sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        ansiblePlaybook become: true, colorized: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/dev.yml', playbook: 'playbooks/site.yml'
      }
    }

    stage('Clean Workspace after build') {
      steps {
        cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
      }
    }
  }

}

playbook
=========

StartInitial cleanupCheckout SCMPrepare Ansible For ExecutionRun Ansible PlaybookClean Workspace after buildEnd
Run Ansible Playbook
 - 1m 2s
Restart Run Ansible Playbook
Invoke an ansible playbook
1m 2s
[t_feature_jenkinspipeline-stages] $ /usr/bin/ansible-playbook playbooks/site.yml -i inventory/dev.yml -b --become-user root --private-key /var/lib/jenkins/workspace/t_feature_jenkinspipeline-stages/ssh169335226076353878.key -u ubuntu

[DEPRECATION WARNING]: ALLOW_WORLD_READABLE_TMPFILES option, moved to a per 

plugin approach that is more flexible, use mostly the same config will work, 

but now controlled from the plugin itself and not using the general constant. 

instead. This feature will be removed from ansible-base in version 2.14. 

Deprecation warnings can be disabled by setting deprecation_warnings=False in 

ansible.cfg.

PLAY [db] **********************************************************************

TASK [Gathering Facts] *********************************************************

Tuesday 20 September 2022  04:29:05 +0000 (0:00:00.023)       0:00:00.023 ***** 

ok: [172.31.86.92]

PLAY [db] **********************************************************************

TASK [mysql : include_tasks] ***************************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:02.221)       0:00:02.244 ***** 

included: /var/lib/jenkins/workspace/t_feature_jenkinspipeline-stages/roles/mysql/tasks/variables.yml for 172.31.86.92

TASK [mysql : Include OS-specific variables.] **********************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.047)       0:00:02.291 ***** 

ok: [172.31.86.92] => (item=/var/lib/jenkins/workspace/t_feature_jenkinspipeline-stages/roles/mysql/vars/RedHat-8.yml)

TASK [mysql : Define mysql_packages.] ******************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.087)       0:00:02.379 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_daemon.] ********************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.047)       0:00:02.427 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_slow_query_log_file.] *******************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.047)       0:00:02.474 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_log_error.] *****************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.045)       0:00:02.519 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_syslog_tag.] ****************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.045)       0:00:02.565 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_pid_file.] ******************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.046)       0:00:02.611 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_config_file.] ***************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.045)       0:00:02.657 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_config_include_dir.] ********************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.045)       0:00:02.702 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_socket.] ********************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.047)       0:00:02.750 ***** 

ok: [172.31.86.92]

TASK [mysql : Define mysql_supports_innodb_large_prefix.] **********************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.046)       0:00:02.796 ***** 

ok: [172.31.86.92]

TASK [mysql : include_tasks] ***************************************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.045)       0:00:02.841 ***** 

included: /var/lib/jenkins/workspace/t_feature_jenkinspipeline-stages/roles/mysql/tasks/setup-RedHat.yml for 172.31.86.92

TASK [mysql : Ensure MySQL packages are installed.] ****************************

Tuesday 20 September 2022  04:29:07 +0000 (0:00:00.053)       0:00:02.895 ***** 

fatal: [172.31.86.92]: FAILED! => {"changed": false, "module_stderr": "Shared connection to 172.31.86.92 closed.\r\n", "module_stdout": "Out of memory allocating 125829120 bytes!\r\n/bin/sh: line 1:  2534 Aborted                 (core dumped) /usr/libexec/platform-python /home/ec2-user/.ansible/tmp/ansible-tmp-1663648147.9950013-2113-171507328398780/AnsiballZ_dnf.py\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 134}

PLAY RECAP *********************************************************************

172.31.86.92               : ok=14   changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   


Possible errors to watch out for:
=================================

Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub has discontinued the use of Master due to Black Lives Matter. You can read more here)

Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the .ansible.cfg file alongside Jenkinsfile in the deploy directory. This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.

## Parameterizing Jenkinsfile For Ansible Deployment
To deploy to other environments, we will need to use parameters.

Update sit inventory with new servers
[tooling]
<SIT-Tooling-Web-Server-Private-IP-Address>

[todo]
<SIT-Todo-Web-Server-Private-IP-Address>

[nginx]
<SIT-Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<SIT-DB-Server-Private-IP-Address>


## CI/CD PIPELINE FOR TODO APPLICATION

Our goal here is to deploy the application onto servers directly from Artifactory rather than from git. 

## Phase 1 – Prepare Jenkins

Fork the repository below into your GitHub account
https://github.com/darey-devops/php-todo.git

Then, copy the code form your github and clone it in your terminal. 
Note that you cannot clone one repository unto another repository.

`git clone https://github.com/Uzuazoraro/php-todo.git`

ubuntu@ip-172-31-84-49:~/ansible-config-mgt$ cd ..

ubuntu@ip-172-31-84-49:~$ git clone https://github.com/Uzuazoraro/php-todo.git
Cloning into 'php-todo'...
remote: Enumerating objects: 332, done.
remote: Counting objects: 100% (332/332), done.
remote: Compressing objects: 100% (161/161), done.
remote: Total 332 (delta 148), reused 320 (delta 147), pack-reused 0
Receiving objects: 100% (332/332), 68.04 KiB | 3.40 MiB/s, done.
Resolving deltas: 100% (148/148), done.

### Add php-todo folder to workspace
=====================================

Click on file
Click on add folder to workspace
Click on php-todo
Click ok

ubuntu@ip-172-31-89-129:~$ git clone git@github.com:Uzuazoraro/php-todo.git
Cloning into 'php-todo'...
remote: Enumerating objects: 341, done.
remote: Total 341 (delta 0), reused 0 (delta 0), pack-reused 341
Receiving objects: 100% (341/341), 68.95 KiB | 1.86 MiB/s, done.
Resolving deltas: 100% (154/154), done.
ubuntu@ip-172-31-89-129:~$ ls
ansible-config-mgt  artifactory  php-todo

### On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)

`sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}`

ubuntu@ip-172-31-84-49:~$ sudo su
root@ip-172-31-84-49:/home/ubuntu# sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils libapache2-mod-php8.1 libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.3-0 libonig5 libzip4
  php-common php-file-iterator php-json php8.1-bcmath php8.1-bz2 php8.1-cli php8.1-common php8.1-gd php8.1-intl php8.1-mbstring php8.1-mysql php8.1-opcache
  php8.1-readline php8.1-xml php8.1-zip phpunit-cli-parser phpunit-version ssl-cert unzip
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser php-pear
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils libapache2-mod-php libapache2-mod-php8.1 libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.3-0
  libonig5 libzip4 php-bcmath php-bz2 php-common php-file-iterator php-gd php-intl php-json php-mbstring php-mysql php-xml php-zip php8.1-bcmath php8.1-bz2
  php8.1-cli php8.1-common php8.1-gd php8.1-intl php8.1-mbstring php8.1-mysql php8.1-opcache php8.1-readline php8.1-xml php8.1-zip phploc phpunit-cli-parser
  phpunit-version ssl-cert unzip zip
0 upgraded, 42 newly installed, 0 to remove and 2 not upgraded.
Need to get 8778 kB of archives.
After this operation, 34.5 MB of additional disk space will be used.

Install PHP Composer
=====================

`sudo apt update`
`apt install php-cli unzip`
`curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php`
`HASH=`curl -sS https://composer.github.io/installer.sig``
`php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"`
`php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer`

root@ip-172-31-84-49:/home/ubuntu# curl -sS https://getcomposer.org/installer | php
root@ip-172-31-84-49:/home/ubuntu# HASH=`curl -sS https://composer.github.io/installer.sig`
root@ip-172-31-84-49:/home/ubuntu# php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
Installer verified
root@ip-172-31-84-49:/home/ubuntu# php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer


All settings correct for using Composer
Downloading...

Composer (version 2.4.2) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer

`sudo apt install php8.1-mysql php8.1-mbstring php8.1-xml php8.1-curl`

root@ip-172-31-84-49:/home/ubuntu# php -v
PHP 8.1.10 (cli) (built: Sep 14 2022 10:21:36) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.10, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.10, Copyright (c), by Zend Technologies

### Install Jenkins plugins
============================

## Plot plugin
## Artifactory plugin

We will use plot plugin to display tests reports, and code coverage information.
The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.

Installing Plugins/Upgrades
Preparation	
Checking internet connectivity
Checking update center connectivity
Success
Plot	 Success
Loading plugin extensions	 Success
Config File Provider	 Success
Ivy	 Success
Javadoc	 Success
Maven Integration	 Success
Artifactory	 Success
Loading plugin extensions	 Success

## In Jenkins UI configure Artifactory

In Jenckins UI:

Click on Dashboard - Configure system (System configuration)
Then, Configure the server ID, URL and Credentials, run Test Connection.

### – Integrate Artifactory repository with Jenkins

Create a dummy Jenkinsfile in the repository

Using Blue Ocean, create a multibranch Jenkins pipeline
On the database server, create database and user

Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';

Update the database connectivity requirements in the file .env.sample

Update Jenkinsfile with proper pipeline configuration

On the database server, create database and user
================================================

Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';


Create a jenkinsfile inside php-todo
=====================================

Update Jenkinsfile with proper pipeline configuration

For Database Connection
===========================

DB_CONNECTION=mysql
DB_PORT=3306

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
`sudo yum install mysql -y` 

`mysql -h 172.31.86.92 -u homestead -p`
This is used to connect Mysql with Jenkins.

`use homestead`

Update the Jenkinsfile to include Unit tests step
===================================================

    stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      } 


## Phase 3 – Code Quality Analysis

1. Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file.

stage('Code Analysis') {
  steps {
        sh 'phploc app/ --log-csv build/logs/phploc.csv'

  }
}

2. Plot the data using plot Jenkins plugin.

Install phploc on Ubuntu
==========================

phploc is a tool for quickly measuring the size and analyzing the structure of a PHP project.

Install on server.

`wget https://phar.phpunit.de/phploc.phar`
`chmod +x phpunit`
`sudo apt install php-phpunit-phploc`
`cd php-todo`

Go to Jenkins, click on Restart code analysis.

Bundle the application code for into an artifact (archived package) upload to Artifactory
=======================================================================================

Install zip on php-todo with:
`sudo apt install zip -y`

ubuntu@ip-172-31-89-129:~/php-todo$ sudo apt install zip -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  unzip
The following NEW packages will be installed:
  unzip zip
0 upgraded, 2 newly installed, 0 to remove and 19 not upgraded.
Need to get 350 kB of archives.
After this operation, 929 kB of additional disk space will be used.

stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }


### Publish the resulted artifact into Artifactory

stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifactory-server'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }

        }


### Deploy the application to the dev environment by launching Ansible pipeline

stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }


## SONARQUBE INSTALLATION

`sudo apt-get update`
`sudo apt-get upgrade`

Install wget and unzip packages
----------------------------------

`sudo apt-get install wget unzip -y`

ubuntu@ip-172-31-90-99:~$ sudo apt-get install wget unzip -y
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Suggested packages:
  zip
The following NEW packages will be installed:
  unzip
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 169 kB of archives.
After this operation, 593 kB of additional disk space will be used.
Get:1 http://us-east-1.ec2.archive.ubuntu.com/ubuntu focal/main amd64 unzip amd64 6.0-25ubuntu1 [169 kB]
Fetched 169 kB in 0s (6376 kB/s)
Selecting previously unselected package unzip.
(Reading database ... 90477 files and directories currently installed.)
Preparing to unpack .../unzip_6.0-25ubuntu1_amd64.deb ...
Unpacking unzip (6.0-25ubuntu1) ...
Setting up unzip (6.0-25ubuntu1) ...
Processing triggers for mime-support (3.64ubuntu1) ...
Processing triggers for man-db (2.9.1-1) ...
ubuntu@ip-172-31-90-99:~$


Install OpenJDK and Java Runtime Environment (JRE) 11
------------------------------------------------------

` sudo apt-get install openjdk-11-jdk -y`
 `sudo apt-get install openjdk-11-jre -y`

 Processing triggers for mime-support (3.64ubuntu1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.9) ...
Processing triggers for systemd (245.4-4ubuntu3.18) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for ca-certificates (20211016~20.04.1) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...

done.
done.
ubuntu@ip-172-31-90-99:~$ sudo apt-get install openjdk-11-jre -y
Reading package lists... Done
Building dependency tree       
Reading state information... Done
openjdk-11-jre is already the newest version (11.0.16+8-0ubuntu1~20.04).
openjdk-11-jre set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
ubuntu@ip-172-31-90-99:~$ 

Set default JDK – To set default JDK or switch to OpenJDK enter below command:
------------------------------------------------------------------------------

`sudo update-alternatives --config java`

ubuntu@ip-172-31-90-99:~$ sudo update-alternatives --config java
There is only one alternative in link group java (providing /usr/bin/java): /usr/lib/jvm/java-11-openjdk-amd64/bin/java
Nothing to configure.

`java --version`

ubuntu@ip-172-31-90-99:~$ java -version
openjdk version "11.0.16" 2022-07-19
OpenJDK Runtime Environment (build 11.0.16+8-post-Ubuntu-0ubuntu120.04)
OpenJDK 64-Bit Server VM (build 11.0.16+8-post-Ubuntu-0ubuntu120.04, mixed mode, sharing)

Install and Setup PostgreSQL 10 Database for SonarQube
=======================================================

`sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'`

Download PostgreSQL software

`wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -`

Install PostgreSQL Database Server

`sudo apt-get -y install postgresql postgresql-contrib`

Start PostgreSQL Database Server

`sudo systemctl start postgresql`

Enable it to start automatically at boot time

`sudo systemctl enable postgresql`

ubuntu@ip-172-31-90-99:~$ sudo systemctl start postgresql
ubuntu@ip-172-31-90-99:~$ sudo systemctl enable postgresql
Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable postgresql

Set up password

`sudo passwd postgres`

Switch to the postgres user

`su - postgres`

Create a new user by typing

`createuser sonar`

Switch to the PostgreSQL shell

`psql`

Set a password for the newly created user for SonarQube database

`ALTER USER sonar WITH ENCRYPTED password 'passWord.1';`

Create a new database for PostgreSQL database by running:

`CREATE DATABASE sonarqube OWNER sonar;`

Grant all privileges to sonar user on sonarqube Database.

`grant all privileges on DATABASE sonarqube to sonar;`

Exit from the psql shell:

`\q`
Switch back to the sudo user by running the exit command.

`exit`

ubuntu@ip-172-31-90-99:~$ sudo systemctl start postgresql
ubuntu@ip-172-31-90-99:~$ sudo systemctl enable postgresql
Synchronizing state of postgresql.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable postgresql
ubuntu@ip-172-31-90-99:~$ sudo passwd postgres
New password: 
Retype new password: 
passwd: password updated successfully
ubuntu@ip-172-31-90-99:~$ su - postgres
Password: 
postgres@ip-172-31-90-99:~$ createuser sonar
postgres@ip-172-31-90-99:~$ psql
psql (12.12 (Ubuntu 12.12-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# ALTER USER sonar WITH ENCRYPTED password 'passWord.1';
ALTER ROLE
postgres=# CREATE DATABASE sonarqube OWNER sonar
postgres-# grant all privileges on DATABASE sonarqube to sonar;
GRANT
postgres=# \q
postgres@ip-172-31-90-99:~$ exit
logout
ubuntu@ip-172-31-90-99:~$


Install SonarQube on Ubuntu 20.04 LTS

Navigate to the tmp directory to temporarily download the installation files

`cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip`

Unzip the archive setup to /opt directory

`sudo unzip sonarqube-7.9.3.zip -d /opt`

Move extracted setup to /opt/sonarqube directory

`sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube`

   creating: /opt/sonarqube-7.9.3/lib/jdbc/h2/
  inflating: /opt/sonarqube-7.9.3/lib/jdbc/h2/h2-1.3.176.jar
  inflating: /opt/sonarqube-7.9.3/lib/sonar-shutdowner-7.9.3.jar
   creating: /opt/sonarqube-7.9.3/elasticsearch/plugins/
ubuntu@ip-172-31-90-99:/tmp$ sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube
ubuntu@ip-172-31-90-99:/tmp$

## CONFIGURE SONARQUBE
=======================

We cannot run SonarQube as a root user, if you run using root user it will stop automatically. The ideal approach will be to create a separate group and a user to run SonarQube

Create a group sonar

`sudo groupadd sonar`

Now add a user with control over the /opt/sonarqube directory

`sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar` 
`sudo chown sonar:sonar /opt/sonarqube -R`

ubuntu@ip-172-31-90-99:~$ sudo groupadd sonar
ubuntu@ip-172-31-90-99:~$ sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
ubuntu@ip-172-31-90-99:~$ sudo chown sonar:sonar /opt/sonarqube -R

Open SonarQube configuration file using your favourite text editor (e.g., nano or vim)

`sudo vim /opt/sonarqube/conf/sonar.properties`

Find the following lines and uncomment them. Provide the username and password.

#sonar.jdbc.username=sonar
#sonar.jdbc.password=passWord.1

Edit the sonar script file and set RUN_AS_USER

`sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh`

#IGNORE_SIGNALS=true

# If specified, the Wrapper will be run as the specified user.
# IMPORTANT - Make sure that the user has the required privileges to write
#  the PID file and wrapper.log files.  Failure to be able to write the log
#  file will cause the Wrapper to exit without any way to write out an error
#  message.
# NOTE - This will set the user which is used to run the Wrapper as well as
#  the JVM and is not useful in situations where a privileged resource or
#  port needs to be allocated prior to the user being changed.
RUN_AS_USER=sonar

# The following two lines are used by the chkconfig command. Change as is
#  appropriate for your application.  They should remain commented.
# chkconfig: 2345 20 80
# description: Test Wrapper Sample Application

# Do not modify anything beyond this point
#-----------------------------------------------------------------------------

# Get the fully qualified path to the script

^G Get Help     ^O Write Out    ^W Where Is     ^K Cut Text     ^J Justify      ^C Cur Pos      M-U Undo        M-A Mark Text   M-] To Bracket  M-Q Previous    ^B Back         ^◀ Prev Word
^X Exit         ^R Read File    ^\ Replace      ^U Paste Text   ^T To Spell     ^_ Go To Line   M-E Redo        M-6 Copy Text   ^Q Where Was    M-W Next        ^F Forward      ^▶ Next Word

### Now, to start SonarQube we need to do following:
-----------------------------------------------------

Switch to sonar user

`sudo su sonar`

Move to the script directory

`cd /opt/sonarqube/bin/linux-x86-64/`

Run the script to start SonarQube

`./sonar.sh start`

ubuntu@ip-172-31-90-99:~$ sudo vim /opt/sonarqube/conf/sonar.propertiesubuntu@ip-172-31-90-99:~$ sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
ubuntu@ip-172-31-90-99:~$ sudo su sonar
$ cd /opt/sonarqube/bin/linux-x86-64/
$ ./sonar.sh start
Starting SonarQube...
Started SonarQube.
$

Check SonarQube running status:

`./sonar.sh status`

ubuntu@ip-172-31-90-99:~$ sudo su sonar
$ cd /opt/sonarqube/bin/linux-x86-64/
$ ./sonar.sh start
Starting SonarQube...
Removed stale pid file: /opt/sonarqube/bin/linux-x86-64/./SonarQube.pid
Started SonarQube.
$ ./sonar.sh status
SonarQube is running (1046).
$

To check SonarQube logs, navigate to /opt/sonarqube/logs/sonar.log directory

`tail /opt/sonarqube/logs/sonar.log`

$ tail /opt/sonarqube/logs/sonar.log
2022.10.09 00:41:33 INFO  app[][o.s.a.SchedulerImpl] Waiting for Elasticsearch to be up and running
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
2022.10.09 00:41:35 INFO  app[][o.e.p.PluginsService] no modules loaded
2022.10.09 00:41:35 INFO  app[][o.e.p.PluginsService] loaded plugin [org.elasticsearch.transport.Netty4Plugin]
2022.10.09 00:41:58 INFO  app[][o.s.a.SchedulerImpl] Process[es] is up
2022.10.09 00:41:58 INFO  app[][o.s.a.ProcessLauncherImpl] Launch process[[key='web', ipcIndex=2, logFilenamePrefix=web]] from [/opt/sonarqube]: /usr/lib/jvm/java-11-openjdk-amd64/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/opt/sonarqube/temp --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED -Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Dhttp.nonProxyHosts=localhost|127.*|[::1] -cp ./lib/common/*:/opt/sonarqube/lib/jdbc/postgresql/postgresql-42.2.5.jar org.sonar.server.app.WebServer /opt/sonarqube/temp/sq-process12064081722208917461properties
2022.10.09 00:42:16 INFO  app[][o.s.a.SchedulerImpl] Process[web] is up
2022.10.09 00:42:16 INFO  app[][o.s.a.ProcessLauncherImpl] Launch process[[key='ce', ipcIndex=3, logFilenamePrefix=ce]] from [/opt/sonarqube]: /usr/lib/jvm/java-11-openjdk-amd64/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/opt/sonarqube/temp --add-opens=java.base/java.util=ALL-UNNAMED -Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Dhttp.nonProxyHosts=localhost|127.*|[::1] -cp ./lib/common/*:/opt/sonarqube/lib/jdbc/postgresql/postgresql-42.2.5.jar org.sonar.ce.app.CeServer /opt/sonarqube/temp/sq-process4910143770100196201properties
2022.10.09 00:42:24 INFO  app[][o.s.a.SchedulerImpl] Process[ce] is up
2022.10.09 00:42:24 INFO  app[][o.s.a.SchedulerImpl] SonarQube is up   

### Configure SonarQube to run as a systemd service
===================================================

Stop the currently running SonarQube service

`cd /opt/sonarqube/bin/linux-x86-64/`

Run the script to stop SonarQube

`./sonar.sh stop`

$ cd /opt/sonarqube/bin/linux-x86-64/
$ ./sonar.sh stop
Gracefully stopping SonarQube...
Waiting for SonarQube to exit...
Waiting for SonarQube to exit...
Waiting for SonarQube to exit...
Waiting for SonarQube to exit...
Waiting for SonarQube to exit...
Waiting for SonarQube to exit...
Stopped SonarQube.

Type exit to exit.

Create a systemd service file for SonarQube to run as System Startup.

`sudo nano /etc/systemd/system/sonar.service`

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target

Save the file and control the service with systemctl

sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar

ubuntu@ip-172-31-90-99:/$ sudo nano /etc/systemd/system/sonar.service
ubuntu@ip-172-31-90-99:/$ sudo systemctl start sonar
ubuntu@ip-172-31-90-99:/$ sudo systemctl enable sonar
Created symlink /etc/systemd/system/multi-user.target.wants/sonar.service → /etc/systemd/system/sonar.service.
ubuntu@ip-172-31-90-99:/$ sudo systemctl status sonar
● sonar.service - SonarQube service
     Loaded: loaded (/etc/systemd/system/sonar.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-10-08 03:31:24 UTC; 46s ago
   Main PID: 4436 (wrapper)
      Tasks: 154 (limit: 2309)
     Memory: 1.3G
     CGroup: /system.slice/sonar.service
             ├─4436 /opt/sonarqube/bin/linux-x86-64/./wrapper /opt/sonarqube/bin/linux-x86-64/../../conf/wrapper.conf wrapper.syslog.ident=SonarQube wrapper.pidfile=/opt/sonarqube/bin/linux-x86-64/./S>
             ├─4438 java -Dsonar.wrapped=true -Djava.awt.headless=true -Xms8m -Xmx32m -Djava.library.path=./lib -classpath ../../lib/jsw/wrapper-3.2.3.jar:../../lib/common/sonar-plugin-api-7.9.3-all.j>
             ├─4466 /usr/lib/jvm/java-11-openjdk-amd64/bin/java -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.n>
             ├─4563 /usr/lib/jvm/java-11-openjdk-amd64/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/opt/sonarqube/temp --add-opens=java.base/java.util=ALL-UNNAMED --add-op>
             └─5094 /usr/lib/jvm/java-11-openjdk-amd64/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/opt/sonarqube/temp --add-opens=java.base/java.util=ALL-UNNAMED -Xmx512m>

Oct 08 03:31:23 ip-172-31-90-99 systemd[1]: Starting SonarQube service...
Oct 08 03:31:23 ip-172-31-90-99 sonar.sh[4386]: Starting SonarQube...
Oct 08 03:31:24 ip-172-31-90-99 sonar.sh[4386]: Started SonarQube.
Oct 08 03:31:24 ip-172-31-90-99 systemd[1]: Started SonarQube service.
lines 1-17/17 (END)


Access SonarQube
To access SonarQube using browser, type server’s IP address followed by port 9000

http://server_IP:9000 OR http://54.172.169.206:9000

Default username: admin
Default password: admin

## CONFIGURE SONARQUBE AND JENKINS FOR QUALITY GATE
=================================================

In Jenkins, install SonarScanner plugin

Jenkins > Manage Jenkins > Manage plugins > Click Available > Type SonarQube Scanner > Click Install without Restart

### Configure SonarQube Scanner

Navigate to configure system in Jenkins.

### Generate authentication token in SonarQube

User > My Account > Security > Generate Tokens

Generated token: bd52583d665d254a6524a66ebda2dcb04716dc2e

### Configure Quality Gate Jenkins Webhook in SonarQube – The URL should point to your Jenkins server http://{JENKINS_HOST}/sonarqube-webhook/

Administration > Configuration > Webhooks > Create

Continuous Code Quality
Projects
Issues
Rules
Quality Profiles
Quality Gates
Administration

Search for projects and files...
A
Administration
Configuration
Security
Projects
System
Marketplace
WebhooksCreateWebhooks are used to notify external services when a project analysis is done. An HTTP POST request including a JSON payload is sent to each of the provided URLs. Learn more in the Webhooks documentation.
Name	URL	Secret?	Last delivery	
Jenkins	http://34.202.65.97:8080/sonarqube-webhooks	Yes	Never	

### Setup SonarQube scanner from Jenkins – Global Tool Configuration

Manage Jenkins > Global Tool Configuration



