# Load Balancer Solution With Nginx and SSL/TLS

As a versatile DevOps engineer, it's essential to understand different load balancing solutions. In this project, we focus on configuring an Nginx Load Balancer solution. Additionally, ensuring secure connections to web applications via HTTPS is crucial to protect data in transit.

### Key Concepts:

1. **Load Balancing**:
   - Distributes traffic across multiple servers for improved performance and reliability.

2. **Nginx**:
   - A high-performance web server that can also function as a load balancer.

3. **HTTPS**:
   - Encrypts data between clients (browsers) and web servers to prevent Man-In-The-Middle (MITM) attacks.
   - Protects sensitive information like bank details and social media credentials.

#### Importance of SSL/TLS:
- **Security Technology**: Protects data from MITM attacks by encrypting the session between the browser and the web server.
- **Terminology**: SSL (Secure Sockets Layer) and its newer version, TLS (Transport Layer Security), are referred to as SSL/TLS. Though SSL has been replaced by TLS, the term SSL is still widely used.

By implementing an Nginx Load Balancer with SSL/TLS, you ensure secure, efficient, and reliable web traffic management.

## Task

This project consist of two parts:

- Configure Nginx as a Load Balancer
- Register a new domain name and configure secure connection


Our target architecture of the solution

<img width="568" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/20dbbde5-bdf6-43cf-9cbc-75f34fae08d9">

### Part 1 - Configure Nginx As A Load Balancer
To configure Nginx as a Load Balancer, follow these steps. First, you'll need to create an EC2 instance running Ubuntu Server 24.04 LTS. Here's how you can do it:

### Step 1: Launch an EC2 Instance

1. **Log in to AWS Management Console**: Go to [AWS Management Console](https://aws.amazon.com/console/).

2. **Navigate to EC2 Dashboard**:
   - In the Services menu, select **EC2** under Compute.

3. **Launch Instance**:
   - Click on the **Launch Instance** button.
   
4. **Choose an Amazon Machine Image (AMI)**:
   - Search for and select **Ubuntu Server 24.04 LTS**.

5. **Choose an Instance Type**:
   - Select an instance type based on your needs (e.g., `t2.micro` for testing purposes).

6. **Configure Instance Details**:
   - Configure the instance details as per your requirements. Ensure that you have selected the correct VPC and subnet.
   - Enable auto-assign Public IP.

7. **Add Storage**:
   - Add storage if needed. The default storage is typically sufficient for a basic load balancer setup.

8. **Add Tags**:
   - Add a tag to name your instance (e.g., `Name: nginx LB`).

9. **Configure Security Group**:
   - Create a new security group or select an existing one.
   - Ensure the security group allows inbound traffic on port 22 (SSH) for remote access.
   - Allow inbound traffic on port 80 (HTTP) and port 443 (HTTPS) for web traffic.

10. **Review and Launch**:
    - Review your settings and click **Launch**.
    - Select an existing key pair or create a new one to SSH into your instance, and then launch the instance.
   
   <img width="725" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/e78ac6d3-4e91-490a-bfb0-72ef16a67629">


To update the `/etc/hosts` file on your Nginx load balancer for local DNS with the web servers' names and their local IP addresses, follow these steps:

### Step 2: Update `/etc/hosts` File

1. **SSH into the Nginx Load Balancer Instance**:
   - If you are not already connected, use SSH to connect to your EC2 instance.

   ```sh
   ssh -i "your-key-pair.pem" ubuntu@your-public-ip
   ```

2. **Open the `/etc/hosts` File**:
   - Use a text editor like `nano` to edit the `/etc/hosts` file.

   ```sh
   sudo vi /etc/hosts
   ```

3. **Add Web Servers' Names and IP Addresses**:
   - Add the local IP addresses and names of your web servers. For example, if your web servers have IP addresses `172.31.47.234` and `172.31.38.199`, and you want to name them `web1` and `web2`, you would add the following lines to the file:

   ```sh
   172.31.47.234   web1
   172.31.38.199   web2
   ```
   <img width="441" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/6bd9a119-4ae7-43ee-adf3-9c5d994f01b8">


4. **Save and Exit**:
   - Save the changes and exit the text editor. In `nano`, you can do this by pressing `Ctrl+O` to save and `Ctrl+X` to exit.

5. **Verify the Changes**:
   - You can verify that the changes have been applied correctly by pinging the web server names.

   ```sh
   ping web1
   ping web2
   ```

   - You should see replies from the respective IP addresses.

### Example `/etc/hosts` File

Here is how the `/etc/hosts` file might look after the changes:

```sh
127.0.0.1       localhost
127.0.1.1       nginx-lb

# Web servers
192.168.1.10    web1
192.168.1.11    web2
```

This configuration allows your Nginx load balancer to resolve the names `web1` and `web2` to their respective IP addresses.

### Next Steps

With the `/etc/hosts` file updated, you can now proceed to configure Nginx to use these names in its load balancing configuration. Update the `nginx.conf` file to use `web1` and `web2` as the upstream servers.

### Update Nginx Configuration

1. **Edit Nginx Configuration**:
   ```sh
   sudo nano /etc/nginx/nginx.conf
   ```

2. **Configure Load Balancer Settings**:
   - Use the names `web1` and `web2` in the `upstream` block.

   ```nginx
   http {
       upstream backend_servers {
           server web1;
           server web2;
       }

       server {
           listen 80;
           server_name example.com;

           location / {
               proxy_pass http://backend_servers;
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;
           }
       }
   }
   ```

3. **Test Nginx Configuration**:
   ```sh
   sudo nginx -t
   ```
   <img width="377" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/03827c9a-f054-4e19-99b0-9762f259e4af">


4. **Restart Nginx**:
   ```sh
   sudo systemctl restart nginx
   ```

   <img width="724" alt="image" src="https://github.com/MabelOlivia/Devops-Cloud-Engineering/assets/70368706/f0ad9ae0-f9e2-4e1e-9da4-ebebceffb36e">

Now, your Nginx load balancer should be able to route traffic to `web1` and `web2` using the names you defined in the `/etc/hosts` file.
