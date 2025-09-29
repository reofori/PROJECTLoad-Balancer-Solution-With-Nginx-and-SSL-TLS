# Load-Balancer_Solution_With_Nginx

- By now we have learned what Load Balancing is used for and have configured an LB solution using Apache, but a DevOps engineer must be a versatile professional and know different alternative solutions for the same problem. That is why, in this project we will configure an Nginx Load Balancer solution.
 - It is also extremely important to ensure that connections to  Web solutions are secure and information is encrypted in transit - we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it. 
- When data is moving between a client (browser) and a Web Server over the Internet - it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.
- This threat is real - users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.
- SSL and its newer version, TSL - is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS - even though SSL was replaced by TLS, the term is still being widely used.
- SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.
- There are different types of SSL/TLS certificates - we can learn more about them here. We can also watch a tutorial on how SSL works here or an additional resource hereIn this project you will register website with LetsEnrcypt Certificate Authority, to automate certificate issuance we will use a shell client recommended by LetsEncrypt - cetrbot.

## Task
This project consists of two parts:
1. Configure Nginx as a Load Balancer.
2. Register a new domain name and configure secured connection using SSL/TLS certificates.

Target architecture will look like this:

![image](https://github.com/user-attachments/assets/ea5312f0-30c0-4184-87e5-fea548778276)

## Part 1 - Configure Nginx As A Load Balancer:

1. Create an EC2 VM based on Ubuntu Server 24.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections).

![image](https://github.com/user-attachments/assets/406d1ccb-a73b-4b0c-bf7a-01f8b464bbaf)

![image](https://github.com/user-attachments/assets/bcacd6d4-ad3b-4d42-84b3-c47e647f1967)

2. Update /etc/hosts file for local DNS with Web Servers' names (e.g. Web1 and Web2) and their local IP addresses

```
sudo nano /etc/hosts
```
![image](https://github.com/user-attachments/assets/64c05f3e-f52f-4c69-9dcf-22e5fc03c28c)



3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers.
   
Update the instance and Install Nginx Install Nginx
```
sudo apt update
```
![image](https://github.com/user-attachments/assets/7e51133b-499f-42a2-9d6b-c58b27b0c661)


```
sudo apt install nginx -y
```
![image](https://github.com/user-attachments/assets/a8ed19bc-531d-4449-b206-ad7772609e40)


Open the default nginx configuration file

```
sudo nano /etc/nginx/nginx.conf
```
```
http {
    # Other configurations
    
    upstream myproject {
        server Web1 weight=5;
        server Web2 weight=5;
    }

    server {
        listen 80;
        server_name www.domain.com;

        location / {
            proxy_pass http://myproject;
        }

        # Comment out the following line
        # include /etc/nginx/sites-enabled/*;
    }
}

```
Replace your already purchased domain name in the section for domain name, save and exit

![image](https://github.com/user-attachments/assets/63164b98-ac07-4d5a-81ef-b853aabd434d)



```
sudo systemctl restart nginx
sudo systemctl status nginx
```

## Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates
- Let us make necessary configurations to make connections to our Tooling Web Solution secured.
- !In order to get a valid SSL certificate - you need to register a new domain name, you can do it using any Domain name registrar - a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

1. Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP.

![image](https://github.com/user-attachments/assets/d36a4e0d-52aa-4208-86ba-f7b2382ec074)


![image](https://github.com/user-attachments/assets/5fb15855-7976-4076-93c1-5fdbc906f839)


We might have noticed, that every time we restart or stop/start EC2 instance - We get a new public IP address. When we want to associate we domain name - it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.

3. Update A record in registrar to point to Nginx LB using Elastic IP address.


Adding records inside the Hosted Zone;

![image](https://github.com/user-attachments/assets/2e85b8e2-0ccd-4e4d-a800-99ec56d51abb)


4. Configure Nginx to recognize your new domain name.

```
sudo nano /etc/nginx/nginx.conf
```
![image](https://github.com/user-attachments/assets/2dcec4b1-5087-4ba9-a85f-3fda1008130f)


After do configuration insdie the nginx.conf need to check the syntax

```
sudo nginx -t
```
Reload the changes.

```
sudo systemctl reload nginx
```

Accessing the domain on web-browser;

![image](https://github.com/user-attachments/assets/6e0bcd17-6ec6-442c-8650-73507fa13668)



5. Install certbot and request for an SSL/TLS certificate.

- Ensure 'snapd' service is active and running

```
sudo systemctl status snapd
```

![image](https://github.com/user-attachments/assets/7a8157eb-ad9d-46d1-980a-7c3aa9768e1b)


- Install certbot using snapd package manager.
```
 sudo snap install --classic certbot
```

- Create a symlink for certbot.

````
     sudo ln -s /snap/bin/certbot /usr/bin/certbot
````

- Follow the prompt to configure and request for ssl certificate.
```
sudo certbot --nginx
```

Visit your website to confirm SSL has been successfully installed.
![image](https://github.com/user-attachments/assets/f15d1206-2073-4c33-ac6a-dca71c3fdb6c)


Check the certificate on Browser.
![image](https://github.com/user-attachments/assets/606f47bd-e19d-4a45-8fbb-cdfb5b7dfc94)



# Note: Letsencrypt ssl certificate is usually valid for 90 days, in order to make this continually renew itself, this can be achieved using a service known as Cron Job;

A cron job is a scheduled task on Unix-like operating systems, such as Linux. The cron service runs these scheduled tasks at specified times and intervals. Cron jobs are useful for automating repetitive tasks, such as system maintenance, backups, and running scripts.

First test the renewal command

```
sudo certbot renew --dry-run
```


![image](https://github.com/user-attachments/assets/d1f0f1e8-44b2-4300-acd4-f41e3d92a677)


Setting up a cron job to automate checking the server for ssl and renewal constantly.

#### Edit the cron tab.

```
    crontab -e
```
![image](https://github.com/user-attachments/assets/efea6030-989b-4a3a-8d59-b9c8ad870e8f)


Add the following line.

```
    * */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```
![image](https://github.com/user-attachments/assets/f602333b-15ff-4393-9879-f314e2787d80)



We have now successfuly configured a Nginx based Load Balancer for our webservers, ensured it can be accessed by a domain name and has SSL installed for security.

# Conclusion:

1. Project Overview:
   This project demonstrates how to set up a load balancer using Nginx to distribute traffic across multiple web servers. It's designed to improve the performance and reliability of web applications by evenly distributing incoming requests.

2. Architecture:
   The setup includes:
   - Two or more backend web servers (Apache in this case)
   - One Nginx server acting as a load balancer

3. Implementation Steps:
   a. Provisioning EC2 Instances:
      - Create EC2 instances for web servers and the load balancer
      - Configure security groups to allow necessary traffic

   b. Installing and Configuring Web Servers:
      - Install Apache web server on the backend instances
      - Configure Apache to serve a sample web page

   c. Setting up Nginx as Load Balancer:
      - Install Nginx on the load balancer instance
      - Configure Nginx to distribute traffic to the backend servers

   d. Testing the Setup:
      - Verify that the load balancer is correctly distributing requests
      - Test failover scenarios by stopping one of the web servers

4. Configuration Details:
   The README provides specific configuration snippets for Nginx, showing how to set up the upstream servers and the proxy pass directive.

5. Benefits:
   - Improved scalability and performance of web applications
   - Enhanced reliability through redundancy
   - Better resource utilization across multiple servers

6. Additional Considerations:
   The project mentions potential enhancements like SSL termination at the load balancer and session persistence configurations.

This project serves as a practical guide for implementing a basic load balancing solution, which is a fundamental concept in web architecture and DevOps practices. It's particularly useful for those learning about high availability and scalable web infrastructures.
