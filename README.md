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
