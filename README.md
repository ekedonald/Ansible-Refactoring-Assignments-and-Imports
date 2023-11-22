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

* Create a new folder in root of the repository and name it `static-assignments` (_The **static-assignments** directory is where all other children playboojs will be stored_).

* Move the `common.yml` file into the `static-assignments` directory.

* In the `site.yml` file, import `common.yml` playbook.

```sh
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

* The folder structure of the `ansible-config-mgt` should look like this:

### Step 3: Merge the changes from the refactor branch into the main branch

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