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


### Step 2: Connect to Your EC2 Instance

1. **Get the Public IP Address**:
   - Once your instance is running, go to the Instances page and note the public IP address or DNS name of your instance.

2. **SSH into the Instance**:
   - Use an SSH client to connect to your instance.
   - Replace `your-key-pair.pem` with your key pair file, and `ec2-user@your-public-ip` with the appropriate user and IP address.
   
   ```sh
   ssh -i "your-key-pair.pem" ubuntu@your-public-ip
   ```

### Step 3: Install Nginx

1. **Update Package Lists**:
   ```sh
   sudo apt update
   ```

2. **Install Nginx**:
   ```sh
   sudo apt install nginx -y
   ```

3. **Start and Enable Nginx**:
   ```sh
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

### Step 4: Configure Nginx as a Load Balancer

1. **Edit Nginx Configuration**:
   - Open the default Nginx configuration file.
   
   ```sh
   sudo nano /etc/nginx/nginx.conf
   ```

2. **Configure Load Balancer Settings**:
   - Add a new `upstream` block and configure the backend servers. Replace `backend1.example.com` and `backend2.example.com` with your actual backend server addresses.

   ```nginx
   http {
       upstream backend_servers {
           server backend1.example.com;
           server backend2.example.com;
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

4. **Restart Nginx**:
   ```sh
   sudo systemctl restart nginx
   ```

### Step 5: Verify Load Balancer

1. **Access the Load Balancer**:
   - Open a web browser and navigate to the public IP address or DNS name of your Nginx load balancer.
   - You should be able to access your backend servers through the load balancer.

With these steps, you have successfully configured an Nginx load balancer on an EC2 instance running Ubuntu Server 24.04 LTS. Next, you can proceed to configure SSL/TLS to secure the connections.




