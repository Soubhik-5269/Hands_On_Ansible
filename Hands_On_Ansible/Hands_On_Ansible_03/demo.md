Certainly! I'll walk you through a simple demo using the ansistrano.deploy and ansistrano.rollback roles. This demo will show you how to deploy a sample web application and then roll back to a previous version using these roles.

Prerequisites
Ansible Installed: Ensure Ansible is installed on your control machine.
Target Server: A remote server (e.g., CentOS, Ubuntu) where you can deploy a simple web application.
Basic Web Server: The target server should have a basic web server installed (like Apache or Nginx).
Step 1: Install the Ansistrano Roles
First, install the ansistrano.deploy and ansistrano.rollback roles using Ansible Galaxy:

bash
Copy code
ansible-galaxy install ansistrano.deploy ansistrano.rollback
Step 2: Create a Simple Web Application
For this demo, we'll create two versions of a simple web page. These files will be deployed to the target server.

Version 1: index_v1.html

html
Copy code
<!DOCTYPE html>
<html>
<head>
    <title>Version 1</title>
</head>
<body>
    <h1>Welcome to Version 1</h1>
</body>
</html>
Version 2: index_v2.html

html
Copy code
<!DOCTYPE html>
<html>
<head>
    <title>Version 2</title>
</head>
<body>
    <h1>Welcome to Version 2</h1>
</body>
</html>
Step 3: Create the Ansible Playbook for Deployment
Create an Ansible playbook deploy_app.yml to deploy the web application.

yaml
Copy code
---
- name: Deploy Application with Ansistrano
  hosts: webservers
  become: yes

  vars:
    ansistrano_deploy_to: /var/www/myapp
    ansistrano_repo_path: /tmp/myapp
    ansistrano_version: v1.0

  tasks:
    - name: Ensure deployment directory exists
      file:
        path: "{{ ansistrano_deploy_to }}"
        state: directory

    - name: Copy application files to the repository path
      copy:
        src: ./index_v1.html
        dest: "{{ ansistrano_repo_path }}/index.html"

    - name: Deploy application using Ansistrano
      include_role:
        name: ansistrano.deploy
      vars:
        ansistrano_deploy_via: copy
        ansistrano_copy_source_path: "{{ ansistrano_repo_path }}"
        ansistrano_shared_paths:
          - uploads

    - name: Restart web server
      service:
        name: httpd  # Use 'nginx' if you're using Nginx
        state: restarted
Step 4: Run the Deployment
Run the playbook to deploy Version 1 of the application:

bash
Copy code
ansible-playbook -i inventory deploy_app.yml
Step 5: Update the Application
Now, let's update the playbook to deploy Version 2. Modify the deploy_app.yml to use index_v2.html and update the version number.

Updated Playbook: deploy_app.yml

yaml
Copy code
---
- name: Deploy Application with Ansistrano
  hosts: webservers
  become: yes

  vars:
    ansistrano_deploy_to: /var/www/myapp
    ansistrano_repo_path: /tmp/myapp
    ansistrano_version: v2.0

  tasks:
    - name: Ensure deployment directory exists
      file:
        path: "{{ ansistrano_deploy_to }}"
        state: directory

    - name: Copy application files to the repository path
      copy:
        src: ./index_v2.html
        dest: "{{ ansistrano_repo_path }}/index.html"

    - name: Deploy application using Ansistrano
      include_role:
        name: ansistrano.deploy
      vars:
        ansistrano_deploy_via: copy
        ansistrano_copy_source_path: "{{ ansistrano_repo_path }}"
        ansistrano_shared_paths:
          - uploads

    - name: Restart web server
      service:
        name: httpd  # Use 'nginx' if you're using Nginx
        state: restarted
Step 6: Deploy Version 2
Run the playbook again to deploy Version 2:

bash
Copy code
ansible-playbook -i inventory deploy_app.yml
Step 7: Rollback to Version 1
Now, let's simulate a situation where Version 2 has issues, and you need to roll back to Version 1.

Rollback Playbook: rollback_app.yml

yaml
Copy code
---
- name: Rollback Application with Ansistrano
  hosts: webservers
  become: yes

  tasks:
    - name: Rollback to the previous version using Ansistrano
      include_role:
        name: ansistrano.rollback

    - name: Restart web server
      service:
        name: httpd  # Use 'nginx' if you're using Nginx
        state: restarted
Run the rollback playbook:

bash
Copy code
ansible-playbook -i inventory rollback_app.yml
Step 8: Verify the Rollback
After running the rollback playbook, visit your web application in a browser or use curl to check that it has reverted to Version 1:

bash
Copy code
curl http://your_server_ip/
You should see the content from Version 1:

html
Copy code
<h1>Welcome to Version 1</h1>
Summary
This demo showed how to deploy a simple web application using the ansistrano.deploy role and how to roll back to a previous version using the ansistrano.rollback role. This workflow is particularly useful in scenarios where you need a reliable way to revert to a previous release in case of deployment issues.