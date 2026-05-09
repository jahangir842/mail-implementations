### Resources:
- https://dixmata.medium.com/install-zimbra-ubuntu-20-04-6d5c47df0498
- https://www.habibza.in/install-zimbra-10-ubuntu-20-04-from-scratch/
- https://medium.com/@nadjibammour80/install-zimbra-on-ubuntu-20-04-step-by-step-d3f1508b9302

### Download:
- https://www.zimbra.com/product/download/zimbra-collaboration-network-edition/

---

### Installation:

To install **Zimbra Collaboration Suite** on an **Ubuntu Server** for local network use, you'll follow a process that involves setting up your Ubuntu server, downloading Zimbra, configuring it, and then deploying it. Below is a step-by-step guide to help you with the installation.

### Step 1: Prepare Your Ubuntu Server

1. **Update the Server**:
   Start by updating your Ubuntu Server to ensure it has the latest patches and updates.
   
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Set the Hostname**:
   Set the hostname for your server. This will be used by Zimbra for domain setup.
   
   ```bash
   sudo hostnamectl set-hostname mail.yourdomain.local
   ```

   Update `/etc/hosts` with the server's IP and hostname:
   
   ```bash
   sudo nano /etc/hosts
   ```

   Add the following line (replace with your server's actual IP address and domain):

   ```
   192.168.1.100 mail.yourdomain.local mail
   ```

   Save and close the file.

3. **Install Dependencies**:
   Zimbra requires several dependencies such as `curl`, `dnsmasq`, and `libaio1`. Install them:

   ```bash
   sudo apt install netcat-openbsd sudo libperl5.30 pax sysstat sqlite3 libaio1 -y
   ```

4. **Disable Unnecessary Services** (if running):
   Disable `AppArmor` and any local firewall that may block Zimbra's ports:

   ```bash
   sudo systemctl stop apparmor
   sudo systemctl disable apparmor
   ```

5. **DNS Configuration**:
   Zimbra requires a DNS server to resolve its domain name. For local network usage, you can use **dnsmasq** for local DNS resolution or set up a local DNS server to ensure correct mail delivery. Install and configure `dnsmasq` as follows:
   
   ```bash
   sudo apt install dnsmasq -y
   ```

   Configure `dnsmasq` to resolve the mail server's domain locally:
   
   ```bash
   sudo nano /etc/dnsmasq.conf
   ```

   Add this line (replace IP and domain with your actual values):

   ```
   address=/yourdomain.local/192.168.1.100
   ```

   Restart `dnsmasq`:

   ```bash
   sudo systemctl restart dnsmasq
   ```

   Test DNS resolution:

   ```bash
   dig mail.yourdomain.local
   ```

### Step 2: Download and Install Zimbra

1. **Download Zimbra**:
   Navigate to the Zimbra downloads page and download the latest release for Ubuntu, or use the command below to download it directly:

   ```bash
   wget https://files.zimbra.com/downloads/9.0.0_GA/zcs-9.0.0_GA_UBUNTU20_64.tgz
   ```

2. **Extract the Zimbra Package**:

   ```bash
   tar xzvf zcs-9.0.0_GA_UBUNTU20_64.tgz
   ```

3. **Run the Zimbra Installer**:
   Move into the extracted directory and start the installation process:

   ```bash
   cd zcs-9.0.0_GA_UBUNTU20_64
   sudo ./install.sh
   ```

4. **Agree to the License Agreement**:
   During the installation, you will be asked to accept the software license agreement. Press `Y` to accept.

5. **Package Selection**:
   When prompted to select packages, choose the default settings unless you know what specific components you need. In most cases, you will want to select the following:

   - Zimbra LDAP
   - Zimbra MTA (Mail Transfer Agent)
   - Zimbra Mailbox
   - Zimbra SNMP
   - Zimbra Logger
   - Zimbra Spell

6. **DNS Configuration**:
   If Zimbra warns about DNS resolution issues, confirm that your DNS server is correctly configured. For a local network, use the IP address as a substitute for the external DNS.

7. **Admin Password Setup**:
   During the installation, you'll be prompted to set an administrator password for the Zimbra admin panel. Make sure to set a secure password.

8. **Installation Finalization**:
   The installer will proceed to install all necessary components. Once the process is complete, Zimbra will automatically configure itself.

### Step 3: Access Zimbra Admin Panel and Mail Client

1. **Access the Admin Console**:
   Once Zimbra is installed, you can access the Admin Console using your web browser. In a local network setup, you would use the server's local IP:

   ```
   https://192.168.1.100:7071
   ```

   Log in with the `admin@yourdomain.local` email and the password you set during the installation.

2. **Access the Zimbra Webmail Client**:
   For end-users to access the email client:

   ```
   https://192.168.1.100
   ```

   Users can log in with their respective email credentials.

### Step 4: Configure Zimbra for Local Network

Since you're using Zimbra on a local network without internet access, make sure the server's **DNS configuration** is properly resolving your domain locally and that the firewall rules are set to allow local traffic.

### Step 5: (Optional) Nginx Proxy for HTTPS (Self-Signed Certificate)

If you need HTTPS for internal use, you can set up **Nginx** as a reverse proxy with a self-signed certificate:

1. **Install Nginx**:

   ```bash
   sudo apt install nginx -y
   ```

2. **Generate Self-Signed SSL Certificate**:

   ```bash
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/zimbra.key -out /etc/ssl/certs/zimbra.crt
   ```

   Fill in the required fields for certificate details.

3. **Configure Nginx as Reverse Proxy**:

   ```bash
   sudo nano /etc/nginx/sites-available/zimbra
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name mail.yourdomain.local;
       return 301 https://$host$request_uri;
   }

   server {
       listen 443 ssl;
       server_name mail.yourdomain.local;

       ssl_certificate /etc/ssl/certs/zimbra.crt;
       ssl_certificate_key /etc/ssl/private/zimbra.key;

       location / {
           proxy_pass http://127.0.0.1:8080;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

4. **Enable the Nginx Configuration**:

   ```bash
   sudo ln -s /etc/nginx/sites-available/zimbra /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

Now you can access Zimbra over HTTPS with the self-signed certificate.

---

This guide should help you get Zimbra running on your local network. Let me know if you need additional details on any step!
