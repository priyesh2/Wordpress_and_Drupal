# Wordpress_and_Drupal installtion using ansible-playbook with tags

* IMPORTANT-FILES
(combined.yml , combined.vars , nginxconfub1.tmpl , nginxconfub.tmpl , wordpress.config.tmpl , my.cnf.tmpl)

* Installing Wordpress and drupal using ansible on ubuntu with nginx,mariadb and php-fpm *

* Before that make changes to the hosts inventory , you can use the hosts from the default location or can be created on any location as per your need

  * Sample hosts file
 
   "YOURIP" ansible_user="(username)" ansible_port=("SSH PORT") ansible_ssh_private_key_file="(PRIVATE KEY FILE)"
 

* Here we are using ansible playbook for the installation process using tags

 - We have used tags like drupal , wordpress , stack (stack is required for installation of both wordpress and drupal)
   
   drupal - for drupal installation
   
   wordpress - for wordpress installation
   
   stack - for installing nginx,php-fpm and ubuntu
   

* FOR INSTALLING NGINX ,MARIADB AND PHP_FPM 

  $ ansible-playbook -i hosts combined.yml --tag=stack
   
* FOR WORDPRESS INSTALLTAION run the playbook as below
  
   $ ansible-playbook -i hosts combined.yml --tag=stack,wordpress
   
* FOR DRUPAL INSTALLTAION run the playbook as below

  $ ansible-playbook -i hosts combined.yml --tag=stack,drupal
  
* I have configured the DNS Records in route53 

-- Domains i have used
 
    For wordpress : wordpress.sageos.tk
    
    For drupal : drupal.sageos.tk
    
    FROM the combined.vars file , you can update the domain names i have used and change it according to your need.
  
  {THIS IS USED FOR A FLEXIBLE INSTALLATION OF DRUPAL AND WORDPRESS ON UBUNTU}
  
  -------------------------------------------------------------------------------
  Notes : In future i will add support for other distributions
  
          The outputs are not attached in here cause i don't want to mess up the readme file.
          
          
          
  ------------------------- USE IT ON UBUNTU DISTRIBUTIONS---------------------------------
          
