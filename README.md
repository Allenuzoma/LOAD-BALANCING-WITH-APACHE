# LOAD BALANCING WITH APACHE: Deploying and configuring a load balancer for a three-tier web application.

Setting up load balancing with Apache for a three-tier application improves availability,reliability and scalability.
As a follow up to the last project where we deployed three webservers for our tooling website solution, this project goes a step further by adding a load balancer to the configuration. 

Horizontal scaling allows to adapt to current load by adding (scale out) or
removing (scale in) Web servers. Adjustment of number of servers can be
done manually or automatically (for example, based on some monitored
metrics like CPU and Memory load).

Property of a system (in our case it is Web tier) to be able to handle
growing load by adding resources, is called "Scalability".

In our set up in Project-7 we had 3 Web Servers and each of them had its
own public IP address and public DNS name. 

A client has to access them by
using different URLs, which is not a nice user experience to remember
addresses/names of even 3 server, let alone millions of Google servers.

In order to hide all this complexity and to have a single point of access with
a single public IP address/name, a Load Balancer can be used. A Load
Balancer (LB) distributes clients' requests among underlying Web Servers
and makes sure that the load is distributed in an optimal way.

               
   ![project 8 with apache lb](https://github.com/user-attachments/assets/fb92c1cd-9817-4c57-bef5-2e78632b8784)




**STEP 0: Configure Apache As A Load Balancer**


1. Create an Ubuntu Server 20.04 EC2 instance and name it Webserver-lb, so your EC2 list will look like this:



   ![ec2 server list](https://github.com/user-attachments/assets/c438abf4-8682-4a56-ae5e-4f4594b5bb5d)



2. Open TCP port 80 on Webserver-lb by creating an Inbound Rule in
Security Group.

3. Install Apache Load Balancer on Webserver-lb server and
configure it to point traffic coming to LB to both Web Servers:


**STEP 1: Install Apache:**

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



We can see the history of the codes we entered in the terminal for reference


   ![history of codes](https://github.com/user-attachments/assets/b4d3cf56-02b2-460c-87d3-f9eae0636ad3)


Verify the apache2 is running using the command:

        sudo systemctl status apache2
        
   ![sudo systemctl statuss apache2](https://github.com/user-attachments/assets/6bb91bbc-16b1-4359-a4a8-080204407a26)

**Step 2: Configure Apache as a Load balancer**

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



  
We will replace the http://web1.example.com with the actual public IP address of each webserver as shown below.     





**Key Elements:**
Proxy and Load Balancer Setup: You're defining a proxy with balancer://mycluster, distributing traffic among the three backend servers using BalancerMember. You are also using the bytraffic method for load balancing, which distributes traffic based on server load.

ProxyPass and ProxyPassReverse: These directives ensure that incoming requests are passed to the load balancer and responses are routed back correctly to the client.

Logs: You're setting up error and access logs with ${APACHE_LOG_DIR}, which is a good practice for tracking errors and request logs.

Suggestions:
Timeout Values: You’ve set a timeout=1 for each BalancerMember. Depending on your network and server response times, this might be a bit low. You may want to increase it if you face timeouts or request failures.

Balancer Load Factor: All servers have the same loadfactor=5, which means they will receive an equal share of traffic. If any of the servers have different performance capacities, you might adjust the load factor accordingly.

Additional Security: If the balancer is accessible publicly, you might want to add security settings, such as restricting access by IP, or adding SSL if this is for production.

ProxyPreserveHost: Keeping ProxyPreserveHost On is generally a good idea if you want the backend servers to receive the original Host header. This is useful for applications that rely on the Host header for routing, logging, or other purposes.

   ![adding web servers with increased timeout](https://github.com/user-attachments/assets/8f8a5ac6-dad9-4a3e-a4a0-09aa83f6c3c1)

**Step 3: Enable the new configuration and disable the default**

Enable the new configuration and disable the default configuration:


          sudo a2ensite webserver-lb.conf
          sudo a2dissite 000-default.conf



**Step 4. Test the configuration and restart Apache**  

1. Check for any syntax error and restart apache using the commands: 


          sudo apache2ctl configtest
          sudo systemctl restart apache2
          sudo systemctl daemon-reload


   ![sudo apache2ctl configtest  syntax check ](https://github.com/user-attachments/assets/14887d97-07f3-4854-9232-d81047a2fc46)

   ![sudo systemctl reload apache2](https://github.com/user-attachments/assets/c3a40767-b085-4de2-b575-8097d078dfac)


**Step 5. Configure Firewall**

To ensure that Apache Load balance instance can communicate with the web servers and users can access the load balancer, the firewall rules should be configured:

          sudo ufw status # check if the firewall is enabled. If disabled, igrore it
          
          # if enabled allow Apache
          sudo ufw allow 'Apache Full'


   ![ufw status](https://github.com/user-attachments/assets/03ce0dd7-97e4-4192-8e19-377975c3bec0)


**Step 6. Test the Load balancer**

Access the load balancer's IP address in a web browser: My load balancer public IP is 18.171.158.99

   ![tooling login site using the webserver pub ip](https://github.com/user-attachments/assets/978606cf-b245-452f-aafe-7ebe1e46a421)


From the above, we can see we are able to access the website with the webserver IP address.

We can now access the webservers using **sudo df -h** to see the mount points and confirm the presence of the directory **var/log/httpd** mount point.


For Webserver 1

   ![df -h webs 1](https://github.com/user-attachments/assets/33d184b3-08cd-4c1c-bc92-f2781f6dada3)

For Webserver 2
   ![df -h web 2](https://github.com/user-attachments/assets/9e0f99c9-bd30-4f30-9c48-3373787a4f13)

For Webserver 3
   ![df -h web3](https://github.com/user-attachments/assets/455d1b14-651c-4b28-9974-8549814e82c1)


Note: In the previous project, we mounted /var/log/httpd/ from our Web Servers to the NFS server - we will have to unmount them and make sure that each Web Server has its own log directory.
Open two ssh/Putty consoles for both Web Servers and run following command:


          sudo tail -f /var/log/httpd/access_log




We would unmount the directory in each webserver:

          # unmount
          sudo umount /var/log/httpd
          
          # Optionally check the processes using the file with the lsof command
          sudo lsof +D /var/log/httpd
          
          # stop the services using the directory if it is busy
          sudo systemctl stop httpd
          
          # verify id /var/log/httpd is unmounted from nfs server
          df -h

          
Notice that the directory has been unmounted.


Run the following command on each terminal of the webservers:

          sudo tail -f /var/log/httpd/access_log
          
By refreshing our browser (load balancer IP) multiple times, we get the following results on each server: (Notice the access is from the load balancer IP- 35.178.187.79)


Web Server 2

   ![web 2 http get request from LB](https://github.com/user-attachments/assets/1afb668f-04da-4cd3-ae80-69e6f331aa30)



Web Server 3

   ![web 3 http get request from LB](https://github.com/user-attachments/assets/dae2fee0-2bfd-4924-bbfe-fa7002940e05)



Step 7. [Optional] Configure Local DNS names resolution
The local DNS name of our webservers can be configured in the /etc/hosts file of the loadbalancer server. This is an internal configuration that is local to our load balancer server and is used for testing purposes.

Open the file as follows:

sudo nano /etc/hosts


Add the following lines to resolve the IP address of our webserver1, webserver2 and webserver3 into web1, web2 and web3 respectively. 18.130.96.12, 18.171.59 and 13.40.57.37


          [Web1 Public IP] web1
          [Web2 Public IP] web2
          [Web1 Public IP] web3


          
          18.130.96.122 web1
          18.171.59 web2
          13.40.57.37 web3


          
The load balancer config files can be updated with the new names instead of IP addresses as shown:


When we curl the addresses locally from the load balancer server, they are accessible as shown in the images:

   ![curl web1 from lb](https://github.com/user-attachments/assets/f9f81f44-dd16-4f95-8153-246a4d8220f5)



Conclusion
We have successfuly configured an Apache loadbalancer to manage web traffic to our webservers. This setup enhances performance, reliability, and scalability in the three-tier architecture.
