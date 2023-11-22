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

