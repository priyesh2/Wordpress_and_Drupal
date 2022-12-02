# Wordpress_and_Drupal installtion using ansible-playbook with tags

### Prerequisites for this project

* INSTALLING ANSIBLE
  
  URL : https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

* This playbook is used for installing Wordpress and Drupal using ansible on ubuntu with nginx,mariadb and php-fpm.

* Before running playbook , make changes to the hosts inventory , you can use the hosts from the default location or can be created on any location as per your need.

  -Sample hosts file has been uploaded.
 

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
  {THIS IS USED FOR A FLEXIBLE INSTALLATION OF DRUPAL AND WORDPRESS ON UBUNTU}

  
  
  
  Note : In future i will add support for other distributions
          
