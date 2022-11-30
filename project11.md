## ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

In Projects 7 to 10 you had to perform a lot of manual operations to seet up virtual servers, install and configure required software, deploy your web application.

This Project will make you appreciate DevOps tools even more by making most of the routine tasks automated with Ansible

## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE

1. Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.

![](./images/Jenkins-Ansible.PNG)

2. In your GitHub account create a new repository and name it ansible-config-mgt.

![](./images/ansible%20config%20mgt.PNG)

3. Install Ansible

```
sudo apt update

sudo apt install ansible
```
![](./images/update%20%26%26%20install%20ansible.PNG)

Check your Ansible version by running ansible --version

![](./images/Ansible%20version.PNG)

4. Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.

Create a new Freestyle project `ansible` in Jenkins and point it to your ‘ansible-config-mgt’ repository.

![](./images/project%20ansible.PNG)

Configure Webhook in GitHub and set webhook to trigger `ansible` build.

Configure a Post-build job to save all (`**`) files, like you did it in Project 9.

![](./images/source%20code%20mgt.PNG)

![](./images/github%20webhook.PNG)

![](./images/jenkins%20build%20trigger.PNG)

![](./images/test%20config.PNG)

![](./images/jenkins%20branches%20build.PNG)



5. Test your setup by making some change in README.MD file in `master` branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

**Note**: Trigger Jenkins project execution only for /main (master) branch.

![](./images/update%20readme%20file.PNG)

![](./images/automatic%20push%20from%20Jenkins.PNG)

![](./images/confirm%20archieve.PNG)

Now your setup will look like this:

![](./images/jenkins_ansible_architech.png)

Tip Every time you stop/start your `Jenkins-Ansible` server – you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your `Jenkins-Ansible` server (you have done it before to your LB server in Project 10). Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.

## Prepare your development environment using Visual Studio Code

1. First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC).

2. After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

3. Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

`git clone <ansible-config-mgt repo link>`

![](./images/connect%20vscode%20to%20ec2instance01.PNG)

![](./images/connect%20vscode%20to%20ec2instance02.PNG)

![](./images/connect%20vscode%20to%20ec2instance03.PNG)

![](./images/connect%20vscode%20to%20ec2instance04.PNG)

![](./images/connect%20vscode%20to%20ec2instance05.PNG)

![](./images/connect%20vscode%20to%20ec2instance06.PNG)


## BEGIN ANSIBLE DEVELOPMENT

1. In your ansible-config-mgt GitHub repository, create a new branch that will be used for development of a new feature.

**Tip**: Give your branches descriptive and comprehensive names, for example, if you use Jira or Trello as a project management tool – include ticket number (e.g. `PRJ-145`) in the name of your branch and add a topic and a brief description what this branch is about – a `bugfix, hotfix, feature, release` (e.g. `feature/prj-145-lvm`)

![](./images/connect%20%26%20create%20new%20branch.PNG)

2. Checkout the newly created feature branch to your local machine and start building your code and directory structure
3. Create a directory and name it `playbooks` – it will be used to store all your playbook files.
4. Create a directory and name it `inventory` – it will be used to keep your hosts organised.
5. Within the playbooks folder, create your first playbook, and name it `common.yml`
6. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) `dev, staging, uat,` and `prod` respectively.

![](./images/create%20folders%20and%20files.PNG)

## Set up an Ansible 

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

Save below inventory structure in the `inventory/dev` file to start configuring your development servers. Ensure to replace the IP addresses according to your own setup.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from `Jenkins-Ansible` host – for this you can implement the concept of `ssh-agent`. Now you need to import your key into `ssh-agent`:

```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```

![](./images/upload%20key%20to%20ssh%20agent.PNG)

Confirm the key has been added with the command below, you should see the name of your key

`ssh-add -l`

Now, ssh into your `Jenkins-Ansible` server using ssh-agent

`ssh -A ubuntu@public-ip`


![](./images/connect%20via%20ssh%20tunnelling.PNG)

![](./images/testing%20ssh.PNG)

Also notice, that your Load Balancer user is `ubuntu` and `user` for RHEL-based servers is ec2-user.

Update your `inventory/dev.yml` file with this snippet of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```

![](./images/update%20files2.PNG)

## CREATE A COMMON PLAYBOOK

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in `inventory/dev`.

In `common.yml` playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your `playbooks/common.yml` file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest

```

![](./images/update%20files.PNG)

Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install `wireshark` utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses `root` user to perform this task and respective package manager: `yum` for RHEL 8 and `apt` for Ubuntu.

Feel free to update this playbook with following tasks:

Create a directory and a file inside it

Change timezone on all servers

Run some shell script

## Update GIT with the latest code

Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.

In the real world, you will be working within a team of other DevOps engineers and developers. It is important to learn how to collaborate with help of `GIT`. In many organisations there is a development rule that do not allow to deploy any code before it has been reviewed by an extra pair of eyes – it is also called "Four eyes principle".

Now you have a separate branch, you will need to know how to raise a `Pull Request (PR)`, get your branch peer reviewed and merged to the `master` branch.

Commit your code into GitHub:

1. use git commands to add, commit and push your branch to GitHub.

```
git status

git add <selected files>

git commit -m "commit message"

```

![](./images/commit%20changes.PNG)

2. Create a Pull request (PR)

3. Wear a hat of another developer for a second, and act as a reviewer.

4. If the reviewer is happy with your new feature development, merge the code to the `master` branch.

![](./images/create%20pull%20request01.PNG)

![](./images/create%20pull%20request02.PNG)

![](./images/create%20pull%20request03.PNG)

![](./images/create%20pull%20request04.PNG)

![](./images/create%20pull%20request05.PNG)

![](./images/create%20pull%20request06.PNG)

5. Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

![](./images/update%20main%20branch.PNG)

Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` directory on Jenkins-Ansible server.

![](./images/jenkins%20automatically%20updated%20files.PNG)

`/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`

![](./images/confirm%20build%20on%20artifact.PNG)

## RUN FIRST ANSIBLE TEST

Now, it is time to execute `ansible-playbook` command and verify if your playbook actually works:

`cd ansible-config-mgt`

`ansible-playbook -i inventory/dev.yml playbooks/common.yml`

![](./images/ansible%20playbook%20success.PNG)

You can go to each of the servers and check if `wireshark` has been installed by running `which wireshark` or `wireshark --version`

![](./images/wireshark%20installed.PNG)

Your updated with Ansible architecture now looks like this:

![](./images/ansible_architecture_final.png)

### Optional step – Repeat once again

Update your ansible playbook with some new Ansible tasks and go through the full `checkout -> change codes -> commit -> PR -> merge -> build -> ansible-playbook` cycle again to see how easily you can manage a servers fleet of any size with just one command!

# Congratulations
You have just automated your routine tasks by implementing your first Ansible project! There is more exciting projects ahead, so lets keep it moving!