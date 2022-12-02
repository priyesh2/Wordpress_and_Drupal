# Wordpress_and_Drupal installtion on ansible-playbook using tags

### WHY USE ANSIBLE?
 
 Here are some important pros/benefits of using Ansible
 One of the most significant advantages of Ansible is that it is free to use by everyone.
 It does not need any special system administrator skills to install and use Ansible, and the official documentation is very comprehensive.
 Its modularity regarding plugins, modules, inventories, and playbooks make Ansible the perfect companion to orchestrate large environments.
 Ansible is very lightweight and consistent, and no constraints regarding the operating system or underlying hardware are present.
 It is also very secure due to its agentless capabilities and due to the use of OpenSSH security features.
 Another advantage that encourages the adoption of Ansible is its smooth learning curve determined by the comprehensive documentation and easy to learn structure and configuration
  

  * PREREQUISITIES FOR THIS PROJECT
       -- ANSIBLE MUST BE INSTALLED
    
  * Feature
    * Easy to install wordpress and drupal on an ubuntu machine
   
       
* This playbook is used for installing Wordpress and Drupal using ansible on UBUNTU with nginx,mariadb and php-fpm.

* Before running playbook  make changes to the hosts inventory , you can use the hosts from the default location or can be created on any location as per your need.

  * Sample hosts entry for configuring the ansible client in the ansible master server.
 
  ````
  [ubuntu]
  172.31.41.53 ansible_user="ubuntu" ansible_port=22 ansible_ssh_private_key_file="key.pem"
  
  ````
  
  * Make sure that the hosts are accessible from the ansible master server by issuing a ping command
   ````
   [ec2-user@ap-south-1 ~]$ ansible -i hosts all -m ping
   
   172.31.41.53 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    }, 
    "changed": false, 
    "ping": "pong"
   }
    ````
  * To list out tasks in the playbook
  ````
    [ec2-user@ap-south-1 ~]$ ansible-playbook -i hosts combined.yml --list-tasks
    
    playbook: combined.yml

  play #1 (ubantu): Wordpress or Drupal installation    TAGS: []
    tasks:
      Pre Installation  TAGS: [stack]
      Installing nginx and php-fpm      TAGS: [stack]
      Creating document root for drupal         TAGS: [drupal]
      Creating document root for wordpress      TAGS: [wordpress]
      Creating virtual host for drupal  TAGS: [drupal]
      Creating virtual host for wordpress       TAGS: [wordpress]
      removing deafault conf    TAGS: [stack]
      Restarting and enabling services  TAGS: [stack]
       installing needful packages      TAGS: [stack]
      Installing mysql client   TAGS: [stack]
      Installing mariadb-server TAGS: [stack]
      Restarting and enable mariadb-server      TAGS: [stack]
      Copying template for mysql        TAGS: [stack]
      Setting root password     TAGS: [stack]
      Remove anonymous user     TAGS: [stack]
       Remove test db   TAGS: [stack]
      Creating new database {{ extra_db }}      TAGS: [stack]
      Creating new user {{ extra_user }}        TAGS: [stack]
      Downloading drupal        TAGS: [drupal]
      Downloading wordpress     TAGS: [wordpress]
      Extracting drupal zip     TAGS: [drupal]
      Extracting Wordpress zip  TAGS: [wordpress]
      Copying files to documentroot /var/www/html/{{ drupal_domain }}   TAGS: [drupal]
      Copying files to documentroot /var/www/html/{{ wordpress_domain }}        TAGS: [wordpress]
      Creating wp-config.php from template      TAGS: [wordpress]
      Restarting services       TAGS: [stack]
   ````
 
# EXPLAINING THE PLAYBOOK
````
  ## TO SETUP MARIADB EXTRA DATABASE,USER AND PASSWORD.
  ---
- name: "Wordpress or Drupal installation"
  hosts: ubuntu
  become: true
  vars_files:
    - combined.vars
  vars_prompt:
    - name: "install_mariadb"
      prompt: "Do you want to install MariaDB (yes/no)?"
      private: no

    - name: "extra_db"
      prompt: "Enter mysql extra database"
      private: no
      default: demo_db

    - name: "extra_user"
      prompt: "Enter mysql extra username"
      private: no
      default: demo_usr

    - name: "extra_pass"
      prompt: "Enter mysql extra password"
      private: yes
      salt_size: 15
      salt: "sample" 
  tasks:
  
  ## ADDING REPOSITORY FOR THE INSTALLATION OF PHP-FPM
  
  - name: "Pre Installation"
      become: yes
      apt_repository: repo=ppa:ondrej/php
      tags:
        - stack
        
  ## INSTALLING NGINX AND PHP-FPM
  
  - name: "Installing nginx and php-fpm"
    apt:
      name: "{{ packs }}"
      state: present  
    tags:
       
  ## CREATING DOCUMENTROOT FOR WORDPRESS AND DRUPAL INSTALLATION
  
    - name: "Creating document root for drupal "
      file:
         path: "{{ drupal_path }}"
         state: directory
         owner: "{{ nginx_owner }}"
         group: "{{ nginx_group }}"
      tags:
        - drupal
   
    - name: "Creating document root for wordpress "
      file:
         path: "{{ wordpress_path }}"
         state: directory
         owner: "{{ nginx_owner }}"
         group: "{{ nginx_group }}"
      tags:
        - wordpress

  ## CREATING VIRTUALHOST FOR CONFIGURING DRUAPAL AND WORDPRESS
   
     - name: "Creating virtual host for drupal"
       template:
        src: nginxconfub.tmpl (TEMPLATE WHICH CONTAINS THE NGINX CONFIGURATION REQUIRE FOR THE DRUPAL WEBSITE)
        dest: "{{ drupal_conf_dest }}"
        owner: "{{ nginx_conf_owner }}"
        group: "{{ nginx_conf_group }}"
      tags:
        - drupal

    - name: "Creating virtual host for wordpress"
      template:
        src: nginxconfub1.tmpl(TEMPLATE WHICH CONTAINS THE NGINX CONFIGURATION REQUIRE FOR THE WORDPRESS WEBSITE)
        dest: "{{ wordpress_conf_dest }}"
        owner: "{{ nginx_conf_owner }}"
        group: "{{ nginx_conf_group }}"
      tags:
        - wordpress

   ## REMOVING NGINX DEFAULT CONF
   
    - name: "removing deafault conf"
      ansible.builtin.file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      ignore_errors: true
      tags:
        - stack

  ## RESTARTING AND ENABLING NGINX AND PHP-FPM
     
    - name: "Restarting and enabling services"
      service:
        name: "{{ item }}"
        state: restarted
        enabled: true
      with_items:
        - "nginx"
        - "php8.1-fpm"
      tags:
        - stack

  ## INSTALLING SOME ADDITIONAL PACKAGES 
  
    - name: " installing needful packages"
      apt: 
        name: "{{ needs }}"
        state: present
      tags:
        - stack
   
  ## SETTING UP MARIADB SERVER
     
    - name: "Installing mysql client"
      pip:
        name: mysqlclient
        state: present
      tags:
        - stack
 
    - name: "Installing mariadb-server"
      apt:
        name: "{{ packages }}"
        state: present  
      tags:
        - stack

    - name: "Restarting and enable mariadb-server"
      service:
        name: mariadb     
        state: restarted
        enabled: true
      when: install_mariadb | bool
      tags:
        - stack
     
    - name: "Copying template for mysql"
      template:
        src: my.cnf.tmpl(MYSQL USER AND PASSWORD IS PREDEFINED IN HERE TO AVOID INSTALLATION ERROR WHEN RUNNING THE SCRIPT AS PER YOUR NEED)
        dest: /root/.my.cnf
        owner: root
        group: root
        mode: 0600
      tags:
        - stack

    - name: "Setting root password"
      mysql_user:
        user: "{{ mysql_user }}"
        password: "{{ mysql_pass }}"
        host: "%"
      tags:
        - stack

    - name: "Remove anonymous user"
      mysql_user:
        user: ""
        host_all: true
        state: absent
      tags:
        - stack

    - name: " Remove test db"
      mysql_db:
        name: "test"
        state: absent
      tags:
        - stack
        
    - name: "Creating new database {{ extra_db }}"
      mysql_db:
        name: "{{ extra_db }}"
        state: present
      tags:
        - stack
   
    - name: "Creating new user {{ extra_user }}"
      mysql_user:
        user: "{{ extra_user }}"
        host: "%"
        password: "{{ extra_pass }}"
        priv: '{{ extra_db }}.*:ALL'
      tags:
        - stack 
  
  ## DOWNLOADING AND EXTRACTING DRUPAL AND WORDPRESS TO A TEMPORARY PATH IN REMOTE SOURCE
  
     - name: "Downloading drupal"
       get_url:
        url: "{{ drupal_url }}"
        dest: /tmp/drupal-10.0.0-rc1.tar.gztar.gz
       tags:
        - drupal

    - name: "Downloading wordpress"
      get_url:
        url: "{{ wp_url }}"
        dest: /tmp/wordpress.tar.gz
      tags:
        - wordpress
     
    - name: "Extracting drupal zip"
      unarchive:
        src: /tmp/drupal-10.0.0-rc1.tar.gztar.gz
        dest: /tmp/
        remote_src: true
      tags:
        - drupal
    
    - name: "Extracting Wordpress zip"
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /tmp/
        remote_src: true
      tags:
        - wordpress

  ## COPYING FILES OF DRUPAL AND WORDPRESS TO THE DOCROOT
     
     - name: "Copying files to documentroot /var/www/html/{{ drupal_domain }}"
      copy:
        src: /tmp/drupal-10.0.0-rc1/
        dest: "/var/www/html/{{ drupal_domain }}"
        owner: "{{ nginx_owner }}"
        group: "{{ nginx_group }}"
        remote_src: true
      tags:
        - drupal
              
    - name: "Copying files to documentroot /var/www/html/{{ wordpress_domain }}"
      copy:
        src: /tmp/wordpress/
        dest: "/var/www/html/{{ wordpress_domain }}"
        owner: "{{ nginx_owner }}"
        group: "{{ nginx_group }}"
        remote_src: true
      tags:
        - wordpress

  ## LOADING wp-config.php as a template from ansible master server to remote source
    
    - name: "Creating wp-config.php from template"
      template:
        src: wordpress.config.tmpl
        dest: "/var/www/html/{{ wordpress_domain }}/wp-config.php"
        owner: "{{ nginx_owner }}"
        group: "{{ nginx_group }}"
      tags:
        - wordpress


  ## RESTARTING SERVICES

    - name: "Restarting services"
      service:
        name: "{{ item }}"
        state: restarted
        enabled: true
      with_items:
        - "mariadb"
        - "nginx"
        - "php8.1-fpm"
      tags:
        - lamp 
````

 - TAGS USED -- drupal , wordpress , stack (stack is required for installation of both wordpress and drupal)

````
   drupal - for drupal installation
   
   wordpress - for wordpress installation
   
   stack - for installing nginx,php-fpm and mariadb
 ````
 ### RUNNING THE PLAYBOOK...............

* FOR INSTALLING NGINX ,MARIADB AND PHP-FPM run the playbook as below .
````
 [ec2-user@ap-south-1 ~]$ ansible-playbook -i hosts combined.yml --tag=stack
  
Do you want to install MariaDB (yes/no)?: yes
Enter mysql extra database [demo_db]: demodb
Enter mysql extra username [demo_usr]: demousr
Enter mysql extra password: 

PLAY [Wordpress or Drupal installation] ************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Pre Installation] ****************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing nginx and php-fpm] ****************************************************************************************************************************************
ok: [172.31.41.53]

TASK [removing deafault conf] **********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Restarting and enabling services] ************************************************************************************************************************************
changed: [172.31.41.53] => (item=nginx)
changed: [172.31.41.53] => (item=php8.1-fpm)

TASK [installing needful packages] *****************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing mysql client] *********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing mariadb-server] *******************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Restarting and enable mariadb-server] ********************************************************************************************************************************
changed: [172.31.41.53]

TASK [Copying template for mysql] ******************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Setting root password] ***********************************************************************************************************************************************
[WARNING]: Module did not set no_log for update_password
ok: [172.31.41.53]

TASK [Remove anonymous user] ***********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Remove test db] ******************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Creating new database demodb] ****************************************************************************************************************************************
changed: [172.31.41.53]

TASK [Creating new user demousr] *******************************************************************************************************************************************
changed: [172.31.41.53]

TASK [Restarting services] *************************************************************************************************************************************************
changed: [172.31.41.53] => (item=mariadb)
changed: [172.31.41.53] => (item=nginx)
changed: [172.31.41.53] => (item=php8.1-fpm)

PLAY RECAP *****************************************************************************************************************************************************************
172.31.41.53               : ok=16   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

````  
   
* FOR WORDPRESS INSTALLTAION run the playbook as below.

````
 [ec2-user@ap-south-1 ~]$ ansible-playbook -i hosts combined.yml --tag=stack,wordpress
   
Do you want to install MariaDB (yes/no)?: yes
Enter mysql extra database [demo_db]: wp_db
Enter mysql extra username [demo_usr]: wp_usr
Enter mysql extra password: 

PLAY [Wordpress or Drupal installation] ************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Pre Installation] ****************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing nginx and php-fpm] ****************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Creating document root for wordpress] ********************************************************************************************************************************
ok: [172.31.41.53]

TASK [Creating virtual host for wordpress] *********************************************************************************************************************************
ok: [172.31.41.53]

TASK [removing deafault conf] **********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Restarting and enabling services] ************************************************************************************************************************************
changed: [172.31.41.53] => (item=nginx)
changed: [172.31.41.53] => (item=php8.1-fpm)

TASK [installing needful packages] *****************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing mysql client] *********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing mariadb-server] *******************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Restarting and enable mariadb-server] ********************************************************************************************************************************
changed: [172.31.41.53]

TASK [Copying template for mysql] ******************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Setting root password] ***********************************************************************************************************************************************
[WARNING]: Module did not set no_log for update_password
ok: [172.31.41.53]

TASK [Remove anonymous user] ***********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Remove test db] ******************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Creating new database wp_db] *****************************************************************************************************************************************
changed: [172.31.41.53]

TASK [Creating new user wp_usr] ********************************************************************************************************************************************
changed: [172.31.41.53]

TASK [Downloading wordpress] ***********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Extracting Wordpress zip] ********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Copying files to documentroot /var/www/html/wordpress.sageos.tk] *****************************************************************************************************
ok: [172.31.41.53]

TASK [Creating wp-config.php from template] ********************************************************************************************************************************
changed: [172.31.41.53]

TASK [Restarting services] *************************************************************************************************************************************************
changed: [172.31.41.53] => (item=mariadb)
changed: [172.31.41.53] => (item=nginx)
changed: [172.31.41.53] => (item=php8.1-fpm)

PLAY RECAP *****************************************************************************************************************************************************************
172.31.41.53               : ok=22   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
 
 ```` 

* FOR DRUPAL INSTALLTAION run the playbook as below.
 ```` 
 [ec2-user@ap-south-1 ~]$ ansible-playbook -i hosts combined.yml --tag=stack,drupal
  
Do you want to install MariaDB (yes/no)?: yes
Enter mysql extra database [demo_db]: drupal_db
Enter mysql extra username [demo_usr]: drupal_usr
Enter mysql extra password: 

PLAY [Wordpress or Drupal installation] ************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Pre Installation] ****************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing nginx and php-fpm] ****************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Creating document root for drupal] ***********************************************************************************************************************************
ok: [172.31.41.53]

TASK [Creating virtual host for drupal] ************************************************************************************************************************************
ok: [172.31.41.53]

TASK [removing deafault conf] **********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Restarting and enabling services] ************************************************************************************************************************************
changed: [172.31.41.53] => (item=nginx)
changed: [172.31.41.53] => (item=php8.1-fpm)

TASK [installing needful packages] *****************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing mysql client] *********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Installing mariadb-server] *******************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Restarting and enable mariadb-server] ********************************************************************************************************************************
changed: [172.31.41.53]

TASK [Copying template for mysql] ******************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Setting root password] ***********************************************************************************************************************************************
[WARNING]: Module did not set no_log for update_password
ok: [172.31.41.53]

TASK [Remove anonymous user] ***********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Remove test db] ******************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Creating new database drupal_db] *************************************************************************************************************************************
changed: [172.31.41.53]

TASK [Creating new user drupal_usr] ****************************************************************************************************************************************
changed: [172.31.41.53]

TASK [Downloading drupal] **************************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Extracting drupal zip] ***********************************************************************************************************************************************
ok: [172.31.41.53]

TASK [Copying files to documentroot /var/www/html/drupal.sageos.tk] ********************************************************************************************************
ok: [172.31.41.53]

TASK [Restarting services] *************************************************************************************************************************************************
changed: [172.31.41.53] => (item=mariadb)
changed: [172.31.41.53] => (item=nginx)
changed: [172.31.41.53] => (item=php8.1-fpm)

PLAY RECAP *****************************************************************************************************************************************************************
172.31.41.53               : ok=21   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  

````

* You can configure the DNS records and complete the wordpress or drupal installation , I have used route53 to configure DNS records for my websites.

* What is route53?.

     URL: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html

  ----------------------------------------------------------------------------
  ## Conclusion
  {THIS PLAYBOOK IS USED FOR A FLEXIBLE INSTALLATION OF DRUPAL AND WORDPRESS ON UBUNTU}

 
  Note : In future i will add support for other distributions
          
### ⚙️ Connect with Me

<p align="center">
<a href="mailto:priyesh241997@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/priyesh-p-34359515a"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a> 
