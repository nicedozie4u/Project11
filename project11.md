## ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

In Projects 7 to 10 you had to perform a lot of manual operations to seet up virtual servers, install and configure required software, deploy your web application.

This Project will make you appreciate DevOps tools even more by making most of the routine tasks automated with Ansible

## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

![](./images/Jenkins-Ansible.PNG)

2. In your GitHub account create a new repository and name it ansible-config-mgt.

![](./images/ansible%20config%20mgt.PNG)

3. Instal Ansible

```
sudo apt update

sudo apt install ansible
```
![](./images/update%20%26%26%20install%20ansible.PNG)

Check your Ansible version by running ansible --version

![](./images/Ansible%20version.PNG)

4. Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.

![](./images/source%20code%20mgt.PNG)

![](./images/github%20webhook.PNG)

![](./images/jenkins%20build%20trigger.PNG)

![](./images/test%20config.PNG)

![](./images/jenkins%20branches%20build.PNG)

![](./images/update%20readme%20file.PNG)

![](./images/automatic%20push%20from%20Jenkins.PNG)

![](./images/confirm%20archieve.PNG)