# Wordpress_and_Drupal using ansible-playbook with tags

IMPORTANT --FILES
(combined.yml , combined.vars , nginxconfub1.tmpl , nginxconfub.tmpl , wordpress.config.tmpl)

Installing Wordpress and drupal using ansible on ubuntu with nginx and php-fpm

Before that make changes to the hosts inventory , you can use the hosts from the default location or can be created as per your need

  *Sample hosts file
 
   "YOURIP" ansible_user="(username)" ansible_port=("SSH PORT") ansible_ssh_private_key_file="(PRIVATE KEY FILE)"
 

* Here we are using ansible playbook for the installation process using tags

 - We have used tags like drupal , wordpress , stack (stack is required for installation of both wordpress and drupal)
   
   drupal - for druapl installation
   
   wordpress - for wordpress installation
   
   stack - for installing nginx,php-fpm and ubuntu
   

* FOR INSTALLING NGINX ,MARIADB AND PHP_FPM 

  cmd : ansible-playbook -i hosts combined.yml --tag=stack
   
* FOR WORDPRESS INSTALLTAION run the playbook as below
  
   cmd : ansible-playbook -i hosts combined.yml --tag=stack,wordpress
   
* FOR WORDPRESS INSTALLTAION run the playbook as below

  cmd : ansible-playbook -i hosts combined.yml --tag=stack,drupal
  
* I have configured the DNS Records in route53 as a test(you can update it from the combined.vars file)
 -- Domains i have used
 
    For wordpress : wordpress.sageos.tk
    
    For drupal : drupal.sageos.tk
  
  {THIS IS USED FOR A FLEXIBLE INSTALLATION OF DRUPAL AND WORDPRESS ON UBUNTU}
  
  -------------------------------------------------------------------------------
  Note : In future i will add support for other distributions
