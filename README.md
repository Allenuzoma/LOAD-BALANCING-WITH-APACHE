# LOAD-BALANCING-WITH-APACHE


STEP 1: Install Apache:

The apache mod_proxy module is used for load balancing protocols including HTTP and HTTPS. This module is combined with other modules for effective load balancing. Install them with the following commands (comments included for explanation):

          # Install Apache
          sudo apt install apache2 -y
          
          # Install development package for the libxml2 library which may be required as a dependency for other modules
          sudo apt install libxml2-dev -y
          
          
          # Enable the mod_rewrite module. It can be useful for URL manipulation before passing requests to backend servers.
          sudo a2enmod rewrite
          
          # Enable loadbalancing module
          sudo a2enmod proxy
          
          # For HTTP protocol
          sudo a2enmod proxy_http
          
          # For maintaining stickiness
          sudo a2enmod proxy_balancer
          
          # load balance scheduling algorithm
          sudo a2enmod lbmethod_bytraffic
          
          # Enable headers. Allows manipulation of HTTP request and response headers.
          sudo a2enmod headers



We can see the history of the codes we entered in the terminal dor reference


![history of codes](https://github.com/user-attachments/assets/b4d3cf56-02b2-460c-87d3-f9eae0636ad3)


Verify the apache2 is running using the command:

        sudo systemctl status apache2
        
![sudo systemctl statuss apache2](https://github.com/user-attachments/assets/6bb91bbc-16b1-4359-a4a8-080204407a26)

**Step 2. Configure Apache as a Load balancer**

We will create a new configuration file in the sites-available folder named webserver-lb.conf



      sudo nano /etc/apache2/sites-available/webserver-lb.conf


Now copy the code below into the conf file:







  <VirtualHost *:80>
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/html
  
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  
      <Proxy "balancer://mycluster">
          BalancerMember "http://web1.example.com"
          BalancerMember "http://web2.example.com"
          BalancerMember "http://web2.example.com"
          # You can add more members here
          ProxySet lbmethod=byrequests
      </Proxy>
  
      ProxyPreserveHost On
      ProxyPass "/" "balancer://mycluster/"
      ProxyPassReverse "/" "balancer://mycluster/"
  </VirtualHost>



  
We will replace the http://web1.example.com with the actual public IP address of each webserver.       
