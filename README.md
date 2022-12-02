# Wordpress_and_Drupal installtion using ansible-playbook with tags

### Prerequisites for this project

* INSTALLING ANSIBLE
  
  URL : https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

* This playbook is used for installing Wordpress and Drupal using ansible on ubuntu with nginx,mariadb and php-fpm.

* Before running playbook , make changes to the hosts inventory , you can use the hosts from the default location or can be created on any location as per your need.

  -Sample hosts file has been uploaded.
 
# EXPLAINING THE PLAYBOOK
````
  ## TO SETUP MARIADB EXTRA DATABASE,USER AND PASSWORD.
  ---
- name: "Wordpress or Drupal installation"
  hosts: ubantu
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
  
  ## ADDING REPOSITORY FOR THE INSTALLATION OF PHP_FPM
  
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
        - stack
        - 
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

  ## CREATING VIRTUALHOST FOR DRUAPAL AND WORDPRESS
   
     - name: "Creating virtual host for drupal"
       template:
        src: nginxconfub.tmpl
        dest: "{{ drupal_conf_dest }}"
        owner: "{{ nginx_conf_owner }}"
        group: "{{ nginx_conf_group }}"
      tags:
        - drupal

    - name: "Creating virtual host for wordpress"
      template:
        src: nginxconfub1.tmpl
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
        src: my.cnf.tmpl
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
  
  ## DOWNLOADIND AND EXTRACRTING DRUPAL AND WORDPRESS TO A TEMPORARY LOCATION
  
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

  ## LOADING wp-config.php as a template from local machine to remote source
    
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
* Here we are using ansible playbook for the installation process using tags

 - We have used tags like drupal , wordpress , stack (stack is required for installation of both wordpress and drupal)
   
   drupal - for drupal installation
   
   wordpress - for wordpress installation
   
   stack - for installing nginx,php-fpm and mariadb
   

* FOR INSTALLING NGINX ,MARIADB AND PHP_FPM 

  $ ansible-playbook -i hosts combined.yml --tag=stack
   
* FOR WORDPRESS INSTALLTAION run the playbook as below
  
   $ ansible-playbook -i hosts combined.yml --tag=stack,wordpress
   
* FOR DRUPAL INSTALLTAION run the playbook as below

  $ ansible-playbook -i hosts combined.yml --tag=stack,drupal
  
* I have configured the DNS Records in route53 after running the playbook

* What is route53?.

     URL: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html

   
  ----------------------------------------------------------------------------
  {THIS PLAYBOOK IS USED FOR A FLEXIBLE INSTALLATION OF DRUPAL AND WORDPRESS ON UBUNTU}

  
  
  
  Note : In future i will add support for other distributions
          
