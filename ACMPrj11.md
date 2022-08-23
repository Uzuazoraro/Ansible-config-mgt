# ANSIBLE CONFIGURATION MANAGEMENT: PROJECT 11

## STEP 1: INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

2. In your GitHub account create a new repository and name it ansible-config-mgt.

3. Instal Ansible

`sudo apt update`
![Instance updated](./EC2 Instance updated.png)

`sudo apt upgrade`
`![Instance upgraded](./EC2 Instance upgraded.png)`

`sudo apt install ansible`

![ansible has been installed](./ansible installed.png)

Check your Ansible version by running ansible --version

![version of ansible](./ansible version.png)

4 -Configure Jenkins build job to save your repository content every time you change it.

Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
Configure Webhook in GitHub and set webhook to trigger ansible build.
Configure a Post-build job to save all (**) files.

![console output](./ansible 1st build.png)

5 - Test your setup by making some change in README.MD file

![console output2](./ansible triggered build.png)

## Step 2: Prepare your development environment using Visual Studio Code

After you have successfully installed VSC, configure it to connect to your newly created GitHub:

## STEP 3: BEGIN ANSIBLE DEVELOPMENT

1. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.
2. Checkout the newly created feature branch to your local machine and start building your code and directory structure
3. Create a directory and name it playbooks – it will be used to store all your playbook files.
   Create a file within the playbooks and name it common.yml
4. Create a directory and name it inventory.
   Within the inventory folder, create dev.yml, uat.yml, prod.yml, staging.yml.

## Step 4 – Set up an Ansible Inventory

`eval`ssh-agent -s``
`ssh-add <path-to-private-key>`

Confirm the key has been added with the command below, you should see the name of your key

`ssh-add -l`

![ssh keys confirmed(./ssh-add -l.png)
Now, ssh into your Jenkins-Ansible server using ssh-agent

`ssh -A ubuntu@public-ip`

## Step 5 – Create a Common Playbook

Update your playbooks/common.yml file with following code:

## Step 6 – Update GIT with the latest code

`git status`

`git add <selected files>`

`git commit -m "commit message"`

## Step 7 – Run first Ansible test

![ansible playbook common.yml](./run playbook.png)

<<<<<<< HEAD
ubuntu@ip-172-31-81-174:~/ansible-config-mgt$ ansible-playbook -i inventory/dev.yml playbooks/common.yml

PLAY [update web, nfs and db servers] ***************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************

ok: [172.31.89.251]

PLAY [update LB server] *****************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************ok: [172.31.93.79]

TASK [Update apt repo] ******************************************************************************************************************************changed: [172.31.93.79]

TASK [ensure wireshark is at the latest version] ****************************************************************************************************ok: [172.31.93.79]

PLAY RECAP ******************************************************************************************************************************************172.31.81.40               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.31.86.92               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.31.88.17               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.31.89.251              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.31.93.79               : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

=======
>>>>>>> 3e5b0079a72761f9fe4bb225d6c55daba96dd4c0


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