# Ansible Configuration Management - Automate Project 7 to 10

### Tasks
- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

### Step 1 - Install and Configure Ansible on EC2 Instance

1. Update the Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
2. In your GitHub account create a new repository and name it ansible-config-mgt.
3. Install Ansible (see:[install Ansible with pip](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-with-pip))

![Jenkins-Ansible](./images/server-rename.png)

![Ansible Config Mgt Repo](./images/ansible-config-repo.png)

```
sudo apt update
sudo apt install ansible
```

![Ansible Installation](./images/install-ansible.png)

### Check your Ansible version by running `ansible --version`

![Ansible Version](./images/ansible-version.png)

4. Configure Jenkins build job to archive your repository content every time you change it - this will solidify your Jenkins configuration skills acquired in Project 9.

- Create a new Freestyle project ansible in Jenkins and point it to your 'ansible-config-mgt' repository.

![Ansible Version](./images/project-ansible.png)

![Point Project Ansible to GitHub repo](./images/point-repo.png)

- Configure a webhook in GitHub and set the webhook to trigger ansible build.

![Webhook Trigger](./images/build-trigger.png)

- Configure a Post-build job to save all (**) files, like you did it in Project 9.

![Post build](./images/post-build.png)

5. Test your setup by making some change in README.md file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/

![Automatic build](./images/automatic-build.png)

`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

![build artefacts](./images/artefacts.png)

## Note: Trigger Jenkins project execution only for main (or master) branch.

## Now, the setup will look like this:

![setup](./images/setup.png)

## Tip: Every time you stop/start your Jenkins-Ansible server - you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your Jenkins-Ansible server (you have done it before to your LB server in Project 10). Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.

![Elastic](./images/elastic.png)

### Step 2 - Prepare your development environment using Visual Studio Code

1. First part of 'DevOps' is 'Dev', which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable - you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs - Visual Studio Code (VSC), you can get it here.

![Visual Studio Code](./images/VSC.png)

2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

![Visual Studio Code](./images/new-repo.png)
![Visual Studio Code](./images/in-workspace.png)

3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

`git clone <ansible-config-mgt repo link>`

![Cloning Ansible repo to Jenkins Instance](./images/clone-repo-jenkins.png)

## Step 3 - Begin Ansible Development

1. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

### Tip: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool - include ticket number (e.g. PRJ-145) in the name of your branch and add a topic and a brief description what this branch is about - a bugfix, hotfix, feature, release (e.g. feature/prj-145-lvm)

![feature/prj-11](./images/new-branch.png)


2. Checkout the newly created feature branch to your local machine and start building your code and directory structure

![feature/prj-11](./images/prj-11.png)

3. Create a directory and name it playbooks - it will be used to store all your playbook files.

![playbooks](./images/playbooks.png)

4. Create a directory and name it inventory - it will be used to keep your hosts organised.

![inventory](./images/inventory.png)

5. Within the playbooks folder, create your first playbook, and name it common.yml

![yml](./images/common.png)

6. Within the inventory folder, create an inventory file () for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively. These inventory files use .ini languages style to configure Ansible hosts.

![ini files](./images/ini-files.png)

## Step 4 - Set up an Ansible Inventory

### An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

### Save the below inventory structure in the inventory/dev file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

### Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers  from Jenkins-Ansible host - for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

![SSH Key](./images/ssh-add.png)
### Confirm the key has been added with the command below, you should see the name of your key

`ssh-add -l`

![SSH](./images/ssh-l.png)
### Now, ssh into your Jenkins-Ansible server using ssh-agent

`ssh -A ubuntu@public-ip`

![SSH](./images/ssh-a.png)
### Also notice, that your Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

### Update your inventory/dev.yml file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user=ec2-user

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user=ec2-user
<Web-Server2-Private-IP-Address> ansible_ssh_user=ec2-user

[db]
<Database-Private-IP-Address> ansible_ssh_user=ec2-user 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user=ubuntu
```

![dev](./images/devini.png)
### Step 5 - Create a Common Playbook

### It is time to start giving Ansible the instructions on what you need to be performed on all servers listed in inventory/dev.
### In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

### Update your playbooks/common.yml file with following code:

```
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  become: yes
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest
   

- name: update LB server
  hosts: lb
  become: yes
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```
![Common YML](./images/commonyml.png)

### Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

### Step 6 - Update GIT with the latest code

### Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.
### In the real world, you will be working within a team of other DevOps engineers and developers. It is important to learn how to collaborate with help of GIT. In many organisations there is a development rule that do not allow to deploy any code before it has been reviewed by an extra pair of eyes - it is also called "Four eyes principle".
### Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the master branch.

### Commit your code into GitHub:

1. Use git commands to add, commit and push your branch to GitHub.

```
git status

git add <selected files>

git commit -m "commit message"
```

![GIT status](./images/git-status.png)
![GIT commit](./images/git-commit.png)

2. Create a Pull request (PR)

![GIT commit](./images/successfully-merged.png)

3. Wear the hat of another developer for a second, and act as a reviewer.

4. If the reviewer is happy with your new feature development, merge the code to the master branch

![GIT commit](./images/merged.png)

5.  Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.
![GIT pull](./images/gitpull1.png)

### Once your code changes appear in master branch - Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

![Build](./images/builds345.png)

![Build](./images/artefact5.png)

![Build](./images/artefactbuild5storage.png)

![Build](./images/build5-directory.png)

### Step 7 - Run first Ansible test

### Now, it is time to execute ansible-playbook command and verify if your playbook actually works:

1. Setup your VSCode to connect to your instance as demonstrated by the video above. Now run your playbook using the command:

![Build](./images/ssh-JA.png)

`cd ansible-config-mgt`

`ansible-playbook -i inventory/dev.yml playbooks/common.yml`

![Build](./images/ansible-3.png)
![Build](./images/ansible-4.png)
![Build](./images/ansible-5.png)
![Build](./images/ansible-6.png)
![Build](./images/ansible-7.png)
### Note: Make sure you're in your ansible-config-mgt directory before you run the above command.

### You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version

![WireShark](./images/wireshark-version.png)
![WireShark](./images/wireshark-version-lb.png)
![WireShark](./images/wireshark-nfs.png)
![WireShark](./images/wireshark-db.png)
![WireShark](./images/wireshark-web2.png)


## Updated with Ansible architecture now looks like this:

![Updated architecture](./images/updated-archi.png)

### Optional step:
### Update your ansible playbook with some new Ansible tasks and go through the full checkout -> change codes -> commit -> PR -> merge -> build -> ansible-playbook cycle again to see how easily you can manage a servers fleet of any size with just one command!

# I have just automated my routine tasks by implementing my first Ansible project! There are more exciting projects ahead, so lets keep it moving!