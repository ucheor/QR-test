Introduction

To improve the reliability, visibility, and traceability of our infrastructure management process, we are implementing a CI/CD pipeline for running our Ansible playbooks. Currently, playbooks are executed manually, which makes it difficult to track execution history, review logs, or ensure consistency across environments. By integrating Ansible with GitHub Actions, this proof of concept (PoC) demonstrates an automated workflow that executes playbooks on code changes, captures detailed run logs, and provides centralized visibility into deployment activity.

Note: This is based off of May2025(17.4) project - setting up ansible servers and nodes - so might make more sense if you already went throug the class/video

Will go back and polish my work but in case anyone needs to refernce the steps, here you go!

Steps:

Created a new folder - ansible-github - and moved terraform files so I can deploy and set up inventory infrastructure and configure ansible files automatically. Will be working from this new folder.

![setting_up_directory](images/setting_up_dir.png)

```
terraform init
terraform apply
```
'yes' to approve

![terraform_output](images/terraform_output.png)

Save this output because, you might need it later. 

Move into ansible-dev (where we have our configuration and inventory files already set up) and create a playbook - install-playbook.yml

![create_playbook](images/create_playbook.png)

Add this text to created the playbook file

```
---
- name: install package
  hosts: all
  become: true

  tasks:
    - name: Install Apache on Ubuntu/Debian
      package:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"

    - name: Install Apache on CentOS/RHEL
      package:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"

    - name: Start and enable Apache on Ubuntu/Debian
      service:
        name: apache2
        state: started
        enabled: yes
      when: ansible_os_family == "Debian"

    - name: Start and enable Apache on CentOS/RHEL
      service:
        name: httpd
        state: started
        enabled: yes
      when: ansible_os_family == "RedHat"

    - name: Create index.html with custom text
      copy:
        dest: /var/www/html/index.html
        content: |
          <html>
          <body>
            <h1>Installed using Ansible - updated</h1>
            <p>Deployed via CI/CD pipeline - GitHub Actions</p>
          </body>
          </html>
        mode: '0644'

```
![install_playbook_file](images/install_playbook_content.png)

While in ansible-dev folder, create a gitHub repository and push ansible-dev folder to it

![create_git_repo](images/git_repo.png)

![create_gitHub_repo](images/git_commit.png)

Now, let's create a self-hosted runner for our GitHub repository using our Ansible control server. On your new gitHub repository, go to Settings, then Actions, then Runners. Follow the steps to configure a new runner. Remember that the self-hosted runner is your control Ansible server so this is the server you need to configure.

You will need to log into your ansible server to do this

![log_into_ansible_server](images/log_into_ansible_server_2.png)

![configure_runner](images/configure_runner.png)

![new_self_hosted_runner](images/new_self_hosted_runner.png)

![runner_set_up](images/download_and_configure_runner.png)

![ansible_runner_setup](images/ansible_runner_setup.png)

Once you are done with the self-hosted runner setup, check the runner page on your gitHub repository to make sure your runner has been set up correctly.

![runner_ready](images/runner_ready.png)

Next is to set up github actions yml file. 

![create_gitHub_action yml file](images/github_action_yml_file.png)

```
name: run install playbook
on: 
    push:
      branches: [ main ]
jobs:
    install:
      runs-on: self-hosted
      steps:
        - name: git checkout
          uses: actions/checkout@v4
        - name: run playbook
          run: ansible-playbook install-playbook.yml
```
![create_gitHub_action yml file content](images/github_action_yml_file2.png)

It's time to commit changes and push to gitHub....but first...now is the time to double-check your security group for your servers to make sure port 80 is open. Your ansible playbook is installing Apache2 and we need to be able to access it on Port 80.

![edit_security_group](images/edit_security_group.png)

Now, push to your repository and your job should run. If it doesn't start, reconfirm the path where you have your .github/workflows/cicd.yml file

Once your playbook has run successfully, you should be able to view your installation on your servers. Feel free to go back to your playbook, update text and push to github. The update should appear on the webserver once the job is complete.

![server](images/ubuntu_server.png)

![update_playbook](images/update_text.png)

![update_workflow](images/updated_text_workflow.png)

![server_updated](images/server_updated.png)

Let me know on discord if you run into any issues with this so I can review and update. I am also happy to get input and happy to help troubleshoot if needed.

Let's get after it!!!







