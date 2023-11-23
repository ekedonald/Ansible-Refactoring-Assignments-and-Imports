# Ansible Refactoring, Assignments & Imports
[Refactoring](https://en.wikipedia.org/wiki/Code_refactoring) means making changes to the source code without changing the expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexibility, add proper comments without affecting the logic.

In this case, I will move things around a little bit in the code but the overall state of the infrastructure remains the same.

In this project, I will continue working with the `ansible-config-mgt` repository and make some improvements of my code such as: 
1. Refactor Ansible Code
2. Create Assignments
3. Imports Functionality (i.e. allows organization of tasks and reuse when needed)

## How To Implement Ansible Refactoring, Assignments & Imports
The following steps are taken to implement Ansible Refactoring, Assignments & Imports on the [ansible-config-mgt repository](https://github.com/ekedonald/ansible-config-mgt):

### Step 1: Jenkins job enhancement

* SSH into your `Jenkins-Ansible` server and create a directory called `ansible-config-artifact` (all the artifacts will be stored here after each build).

```sh
sudo mkdir /home/ubuntu/ansible-config-artifact
```

* Change permissions to this directory so Jenkins could save files there using the following commands:

```sh
sudo chmod -R 777 /home/ubuntu/ansible-config-artifact/
sudo chown -R jenkins:jenkins /home/ubuntu/ansible-config-artifact/
```

* Log into your Jenkins Console, click on `Manage Jenkins` tab and click on `Plugins`

* Click on `Available plugins` and type `copy artifact` in the search bar then select **Copy Artifact** plugin and click on `Install` (_Note that restarting jenkins after installing the plugin isn't necessary_).

* Create a new freestyle job named `save_artifacts`, give it a maximum of 2 builds to keep.

* Configure the `build trigger` of the job to **Build after other projects are built** then link it to your `ansible` job and select **Trigger only if build is stable**.

* Configure a `build step` to **Copy artifacts from another project**, specify the following parameters then apply and save changes:
1. Project name: ansible
2. Latest successful build
3. Artifacts to copy: **
4. Target directory: /home/ubuntu/ansible-config-artifact

* You will notice `save-artifacts `job has an upstream project called `ansible` which means they are connected to each other, click on the `ansible` job drop-down icon and click on **Build Now** to test if the `ansible` job will successfully trigger the `save-artifacts` job.

* Verify if the artifacts of the `ansible` are present in your `ansible-config-artifacts` directory on your `Jenkins-Ansible` server by running the following command:

```sh
cd /home/ubuntu/ansible-config-artifact && ll
```

### Step 2: Refactor Ansible code by importing other playbooks into `site.yml`
* Before starting to refactor the codes, go to the ansible-config-mgt directory on your local machine and pull down the latest code from the `main` branch.

* Create a new branch `refactor` and switch into the branch using the command below:

```sh
git checkout -b refactor
```

* In the `playbooks` folder, create a new file named `site.yml` (_The `site.yml` will become a parent to all other playbooks that will be developed including `common.yml` that you created previously_).

```sh
cd playbooks && touch site.yml
```

* Create a new folder in root of the repository and name it `static-assignments` (_The **static-assignments** directory is where all other children playbooks will be stored_).

* Move the `common.yml` file into the `static-assignments` directory.

* In the `site.yml` file, import `common.yml` playbook.

```sh
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

* The folder structure of the `ansible-config-mgt` should look like this:

### Step 3: Merge the changes from the refactor branch into the main branch and run your playbook

* Run the following command to view the untracked files (i.e. the file and directory you just created):

```sh
git status
```

* Add the untracked files and commit the changes using the following command:

```sh
git add . && git commit -m "updated files and directories"
```

* Push all changes from the `refactor` branch to the main branch.

```sh
git push --set-upstream origin refactor
```

* Go to your `ansible-config-mgt` repository on GitHub and click on the `Compare & pull request` button.

* Click on the `Create pull request` button.

* Click on the `Merge pull request` button.

* Click on the `Confirm merge` button.

* Go to the `ansible-config-mgt` on your local machine and run the following command to switch to the `main` branch and pull the changes.

```sh
git chekout main && git pull
```

* On your Jenkins console, you will see a build has been triggered.

* Run `ansible-playbook` command against the `dev` environment using the command shown below:

```sh
ansible-playbook -i inventory/dev playbook/site.yml
```

### Step 4: Create a playbook configuration file `common-del.yml` that will be used to remove the wireshark package and import the configuration file to the playbook

* Create a `common-del.yml` file in the `static-assignments` directory.

```sh
cd static-assignments && touch common-del.yml
```

* Paste the code shown below into the `common-del.yml` file

```sh
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

* Import the playbook file `common-del.yml` into the `site.yml` file.

```sh
- import_playbook: ../static-assignments/common-del.yml
```

* Run the following command to view the changes and add the untracked files in the `ansible-config-mgt` repository:

```sh
git status
```

```sh
git add common-del.yml ../playbooks/site.yml
```

* Commit the changes made.

```sh
git commit -m "updates"
```

* Push the changes to the `ansible-config-mgt` repository.

```sh
git push
```

* Run the `ansible-playbook` command against the `dev` envinronment to remove wireshark on all your hosts.

```sh
ansible-playbook -i inventory/dev playbook/site.yml
```

* Run Ansible Adhoc command to check if wireshark has been removed from all the servers.

```sh
ansible all -i inventory/dev -m command -a "wireshark --version"
```

_**Note**: Stop the EC2 Instances (i.e. Web Server 1 & 2, Database Server and Load Balancer) after running the Ansible Adhoc command to save cost on your AWS Bill since they are no longer needed for this project_.

### Step 5: Provision 2 UAT Web Servers

Use the following parameters when provisioning the EC2 Instance for the 2 UAT Web Servers:
1. Name of the Instance: UAT_1
2. AMI: Red Hat Enterprise Linux 9 (HVM), SSD Volume Type
3. New Key Pair: web11
4. Key Pair Type: RSA
5. Private Key File Format: .pem
6. New Secruity Group: Web-Server SG
7. Inbound Rules: Allow Traffic From Anywhere On Port 80 & Port 22

*Instance Summary for UAT_1 Server*

*Instance Summary for UAT_2 Server*

### Step 6: Create a new branch and a role for User Acceptance Testing.

* Create and switch into a branch `uat-145` that will be used for User Acceptance Testingin your `ansible-config-mgt` repository.

```sh
git checkout -b uat-145
```

* To create a role, you must create a directory called roles in the `ansible-config-mgt` directory. There are two ways you can this folder structure:

1. Using an Ansible utility called `ansible-galaxy` inside `ansible-config-mgt/roles` directory (Note that you need to create a `roles` directory upfront).

```sh
mkdir roles
cd roles
ansible-galaxy init webserver
```

2. Create the directory/file structure manually.

* I will use the **second** option since all my codes are stored in GitHub instead of using the `Ansible-Galaxy` utility on the `Jenkins Server`

* To create the folder structure below, the following commands were used:

```sh
cd .. && mkdir roles && cd roles && mkdir webserver
cd webserver && touch README.md && mkdir defaults handlers meta tasks templates
cp main.yml meta && mv main.yml tasks
cd .. ** tree webserver
```

### Step 7: Update the `inventory/uat` file and configure the `tasks/main.yml` file

* Update the `ansible-config-mgt/inventory/uat` file with IP adresses of your 2 UAT Web Servers.

```sh
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'
```

* In the tasks directory, configure the `main.yml` file to have the following configurations:

1. Install and Configure Apache (`httpd` service)
2. Clone **Tooling Website** form GitHub `https://github.com/<your-name>/tooling.git`
3. Ensure the tolling website code is deployed to `/var/www/html` on the 2 UAT Web Servers
4. Make sure `httpd` service is started

The `main.yml` will consist of the following tasks:

```sh
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

### Step 8: Reference the `Webserver` role

* Within the `static-assignments` folder, create a new assignments for **uat-webservers**. Update the `uat-webservers.yml`, this is where you will reference the role:

```sh
---
- hosts: uat-webservers
  roles:
     - roles/webserver
```

* Remember that the entry point to your ansible configuration is the `site.yml` file. Therefore, you need to refer your `uat-webservers.yml` role inside `site.yml`. Update the `site.yml` to have the following configuration:

```sh
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```