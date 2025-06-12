# Web Hosting using Raspberry Pi

This project demonstrates how to host a simple web server on a Raspberry Pi, making it accessible over the internet. It uses Apache as the web server, No-IP for dynamic DNS to manage the public IP, and port forwarding on a home router to expose the Raspberry Pi's local IP. The setup allows you to serve a basic website from your Raspberry Pi, accessible from anywhere using a domain name provided by No-IP.

## Project Goals

- Set up a web server on a Raspberry Pi using Apache.
- Use No-IP to handle dynamic IP addresses and provide a static domain name.
- Configure port forwarding on a home router to allow external access to the web server.
- Serve a basic website accessible over the internet.

## Tech Stack

### Hardware
- **Raspberry Pi**: Any model (e.g., Raspberry Pi 4) with Raspberry Pi OS installed.
- **Router**: A home router with port forwarding capabilities.

### Software
- **Raspberry Pi OS**: A Linux distribution (Debian-based) for the Raspberry Pi.
- **Apache2**: Web server to host the website.
- **No-IP DUC (Dynamic Update Client)**: To manage dynamic DNS and map the public IP to a domain.
- **nano** (or any text editor): To edit configuration files and HTML content.

### Other Tools
- **SSH**: For remote access to the Raspberry Pi (optional).
- **Git**: For version control (optional, if hosting project files).

## Prerequisites

- **Raspberry Pi**: Set up with Raspberry Pi OS (Lite or Desktop version) and connected to your home network.
- **Internet Connection**: A stable connection with a public IP (dynamic IP is fine with No-IP).
- **No-IP Account**: Sign up at [No-IP](https://www.noip.com/) and create a free hostname (e.g., `myraspberrypi.ddns.net`).
- **Router Access**: Admin access to your router to configure port forwarding.
- **Terminal Access**: Via direct connection (monitor/keyboard) or SSH.

## Implementation Steps

1. **Set Up Raspberry Pi OS**
   - Download and install Raspberry Pi OS using Raspberry Pi Imager.
   - Boot the Raspberry Pi, connect it to your network, and note its local IP address (e.g., `192.168.1.100`).
     ```bash
     hostname -I
     ```

2. **Install Apache Web Server**
   - Update the package list and install Apache2:
     ```bash
     sudo apt update
     sudo apt install apache2 -y
     ```
   - Start the Apache service and enable it on boot:
     ```bash
     sudo systemctl start apache2
     sudo systemctl enable apache2
     ```
   - Verify Apache is running by accessing `http://<Raspberry-Pi-IP>` in a browser on your local network (e.g., `http://192.168.1.100`). You should see the default Apache page.

3. **Create a Basic Website**
   - The default web directory for Apache is `/var/www/html`. Replace the default `index.html` with your own:
     ```bash
     sudo nano /var/www/html/index.html
     ```
   - Add a simple HTML page (example):
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>My Raspberry Pi Website</title>
     </head>
     <body>
         <h1>Welcome to My Raspberry Pi Web Server!</h1>
         <p>This is a simple website hosted on a Raspberry Pi.</p>
     </body>
     </html>
     ```
   - Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).
   - Set proper permissions:
     ```bash
     sudo chown -R www-data:www-data /var/www/html
     sudo chmod -R 755 /var/www/html
     ```

4. **Set Up No-IP for Dynamic DNS**
   - Sign up for a free No-IP account and create a hostname (e.g., `myraspberrypi.ddns.net`).
   - Install the No-IP Dynamic Update Client (DUC) on the Raspberry Pi:
     ```bash
     cd /usr/local/src
     sudo wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz
     sudo tar xzf noip-duc-linux.tar.gz
     cd noip-2.1.9-1
     sudo make
     sudo make install
     ```
   - During installation, enter your No-IP username, password, and hostname when prompted.
   - Start the No-IP client:
     ```bash
     sudo /usr/local/bin/noip2
     ```
   - Ensure it runs on boot by creating a systemd service:
     ```bash
     sudo nano /etc/systemd/system/noip2.service
     ```
     Add the following:
     ```ini
     [Unit]
     Description=No-IP Dynamic Update Client
     After=network.target

     [Service]
     Type=forking
     ExecStart=/usr/local/bin/noip2
     Restart=always

     [Install]
     WantedBy=multi-user.target
     ```
     Enable the service:
     ```bash
     sudo systemctl enable noip2.service
     sudo systemctl start noip2.service
     ```

5. **Configure Port Forwarding**
   - Log in to your router’s admin panel (e.g., via `http://192.168.1.1`).
   - Navigate to the Port Forwarding section.
   - Add a new rule:
     - **Service Port**: 80 (or your chosen port if different).
     - **Internal IP**: Raspberry Pi’s local IP (e.g., `192.168.1.100`).
     - **Internal Port**: 80.
     - **Protocol**: TCP.
   - Save and apply the settings.
   - Note: If your ISP blocks port 80, use a different port (e.g., 8080) and update Apache’s configuration (`/etc/apache2/ports.conf` and `/etc/apache2/sites-available/000-default.conf`).

6. **Test External Access**
   - From an external device (e.g., your phone on mobile data), visit your No-IP domain (e.g., `http://myraspberrypi.ddns.net`).
   - You should see the website hosted on your Raspberry Pi.

## Usage

- **Access the Website**: Open a browser and navigate to your No-IP domain (e.g., `http://myraspberrypi.ddns.net`).
- **Update Website Content**: Edit files in `/var/www/html/` on the Raspberry Pi to update your website.
- **Monitor Apache Logs**: Check for errors or access logs:
  ```bash
  sudo tail -f /var/log/apache2/access.log
  sudo tail -f /var/log/apache2/error.log
  ```

## Project Structure

```
web-hosting-raspberry-pi/
├── README.md               # Project documentation
└── (Raspberry Pi filesystem)
    └── /var/www/html/
        └── index.html      # Default website content
```

## Limitations and Future Improvements

- **Current Limitations**:
  - Basic setup with a static HTML page; no dynamic content or database.
  - Security risks with port forwarding (e.g., no HTTPS); Apache is exposed to the internet.
  - Dynamic IP changes are handled by No-IP, but router reboots may disrupt access temporarily.
  - Port 80 may be blocked by some ISPs, requiring a non-standard port.
- **Future Improvements**:
  - Enable HTTPS with a free SSL certificate (e.g., Let’s Encrypt).
  - Add a firewall (e.g., `ufw`) to secure the Raspberry Pi.
  - Host a dynamic website with PHP or a framework like Flask.
  - Use a reverse proxy (e.g., Nginx) for better performance and security.

## Troubleshooting

- **Website Not Accessible Externally**:
  - Verify port forwarding settings in your router.
  - Check if your ISP blocks port 80; try a different port (e.g., 8080).
  - Ensure the No-IP client is running (`sudo systemctl status noip2`).
- **Apache Not Running**:
  - Check Apache status:
    ```bash
    sudo systemctl status apache2
    ```
  - Restart if needed:
    ```bash
    sudo systemctl restart apache2
    ```
- **No-IP Domain Not Updating**:
  - Confirm your No-IP credentials and hostname configuration.
  - Check No-IP client logs for errors.

## Contributing

1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/your-feature`).
3. Commit changes (`git commit -m 'Add your feature'`).
4. Push to the branch (`git push origin feature/your-feature`).
5. Open a Pull Request.



## Contact

For questions or feedback, open an issue on GitHub or contact the repository owner.
