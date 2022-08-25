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

