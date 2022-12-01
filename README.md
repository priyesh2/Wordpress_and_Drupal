# Wordpress_and_Drupal using ansible playbook with tags

IMPORTANT --FILES
(combined.yml , combined.vars , nginxconfub1.tmpl , nginxconfub.tmpl , wordpress.config.tmpl)

Installing Wordpress and drupal using ansible on ubuntu with nginx and php-fpm
Before that make changes to the hosts inventory , you can use the hosts from the default location or can be created as per your need

--Sample hosts file
 
 "YOURIP" ansible_user="(username)" ansible_port=("SSH PORT") ansible_ssh_private_key_file="(PRIVATE KEY FILE)"
 

* Here we are using ansible playbook string method for the installation process using tags

 - We have used tags like drupal , wordpress , lamp(lamp is required for installation of both wordpress and drupal)
   
   drupal - for druapl installation
   
   wordpress - for wordpress installation
   
   lamp - for installing nginx,php-fpm and ubuntu
   

* FOR INSTALLING NGINX AND PHP_FPM ONLY

  cmd : ansible-playbook -i hosts combined.yml --tag=lamp
   
* FOR WORDPRESS INSTALLTAION run the playbook as below
  
   cmd : ansible-playbook -i hosts combined.yml --tag=lamp,wordpress
   
* FOR WORDPRESS INSTALLTAION run the playbook as below

  cmd : ansible-playbook -i hosts combined.yml --tag=lamp,drupal
  
  {THIS IS USED AS A FLEXIBLE INSTALLATION OF DRUPAL AND WORDPRESS ON UBUNTU}
  
  -------------------------------------------------------------------------------
  Note : In future i will add support for other distributions
