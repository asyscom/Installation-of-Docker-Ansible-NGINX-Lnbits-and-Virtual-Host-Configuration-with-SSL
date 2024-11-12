
---

# Installation of Docker, Ansible, NGINX, Lnbits, and Virtual Host Configuration with SSL

This repository provides instructions for installing Docker, Ansible, NGINX, and Lnbits on an Ubuntu or Rocky Linux system, and configuring a **Virtual Host** with **SSL** to route traffic to Lnbits.

## Prerequisites

- A machine running Ubuntu 20.04 or later, or Rocky Linux 8 or later.
- Root access or sudo privileges.
- Internet connection.

## 1. Install Docker

### On Ubuntu:

1. Update the system:
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2. Add the Docker repository:
    ```bash
    sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

3. Install Docker:
    ```bash
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io -y
    ```

4. Verify the installation:
    ```bash
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo docker --version
    ```

### On Rocky Linux:

1. Install the Docker repository:
    ```bash
    sudo dnf install https://download.docker.com/linux/centos/docker-ce.repo -y
    ```

2. Install Docker:
    ```bash
    sudo dnf install docker-ce docker-ce-cli containerd.io -y
    ```

3. Start and enable Docker:
    ```bash
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo docker --version
    ```

## 2. Install Ansible

### On Ubuntu:

1. Update the system:
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

2. Install the required dependencies:
    ```bash
    sudo apt install software-properties-common -y
    ```

3. Add the Ansible repository:
    ```bash
    sudo add-apt-repository ppa:ansible/ansible
    ```

4. Install Ansible:
    ```bash
    sudo apt update
    sudo apt install ansible -y
    ```

### On Rocky Linux:

1. Update the system:
    ```bash
    sudo dnf update -y
    ```

2. Install EPEL:
    ```bash
    sudo dnf install epel-release -y
    ```

3. Install Ansible:
    ```bash
    sudo dnf install ansible -y
    ```

## 3. Create a Custom Docker Network

To enable communication between the containers, we will create a custom Docker network:

```bash
sudo docker network create lnbits_network
```

This network will be used for both the **Lnbits** and **NGINX** containers.

## 4. Install NGINX Manager in Docker using Ansible

To install **NGINX Manager** in Docker via Ansible, follow these steps:

1. Create a file named `nginx_manager.yml` with the following content:

    ```yaml
    ---
    - name: Install NGINX Manager in Docker
      hosts: localhost
      become: yes

      tasks:
        - name: Pull the NGINX Manager Docker image
          docker_image:
            name: jc21/nginx-proxy-manager
            source: pull

        - name: Start the NGINX Manager Docker container
          docker_container:
            name: nginx-proxy-manager
            image: jc21/nginx-proxy-manager
            state: started
            restart_policy: always
            published_ports:
              - "80:80"
              - "443:443"
              - "81:81"  # Access port to the manager
            networks:
              - name: lnbits_network
            volumes:
              - /path/to/nginx_data:/data
              - /path/to/nginx_config:/config
    ```

2. Modify `/path/to/nginx_data` and `/path/to/nginx_config` with the appropriate paths on your system.

3. Run the Ansible playbook:

    ```bash
    ansible-playbook nginx_manager.yml
    ```

4. NGINX Manager will be accessible at `http://<server_ip>:81`.

## 5. Install Lnbits in Docker using Ansible

1. Create a file named `lnbits.yml` with the following content:

    ```yaml
    ---
    - name: Install Lnbits in Docker
      hosts: localhost
      become: yes

      tasks:
        - name: Pull the Lnbits Docker image
          docker_image:
            name: lnbits/lnbits
            source: pull

        - name: Start the Lnbits Docker container
          docker_container:
            name: lnbits
            image: lnbits/lnbits
            state: started
            restart_policy: always
            published_ports:
              - "5000:5000"
            networks:
              - name: lnbits_network
            volumes:
              - /path/to/lnbits_data:/lnbits/data
    ```

2. Modify `/path/to/lnbits_data` with the appropriate path on your system.

3. Run the Ansible playbook:

    ```bash
    ansible-playbook lnbits.yml
    ```

4. Lnbits will be accessible at `http://<server_ip>:5000`.

## 6. Configure the Virtual Host in NGINX with SSL

1. Access the NGINX Manager interface at `http://<server_ip>:81`.

2. Create a **Proxy Host** for your domain pointing to the **Lnbits** container:
    - **Domain Names**: `lnbits.yourdomain.com` (replace with your domain).
    - **Scheme**: `http`.
    - **Forward Hostname / IP**: `lnbits` (this is the name of the Lnbits container on the Docker network, which will resolve automatically).
    - **Forward Port**: `5000`.
    - **Block Common Exploits**: Enabled.
    - **SSL**: Enabled.
        - **Let's Encrypt**: Check the box to automatically obtain the SSL certificate.
        - **Email Address**: Provide a valid email for the SSL certificate.
        - **Force SSL**: Enabled (to ensure traffic is redirected to HTTPS).

3. Save the configuration. NGINX Manager will now route traffic from `lnbits.yourdomain.com` to the **Lnbits** container.

4. After configuring the proxy host, the Lnbits site will be accessible via **HTTPS** at `https://lnbits.yourdomain.com`.

---

## Donations

If you appreciate my work, you can make a donation in BTC or via Lightning Network.

- For **BTC donations**, bc1qrvlwjutnztfxkqtnjdwwqa6uzd9ljdv8asdknn
- For **Lightning Network donations**, you can use: `tips@davidebtc.me`.

Your support is greatly appreciated!

---

### Summary of Steps:

1. **Install Docker** on Ubuntu or Rocky Linux.
2. **Install Ansible**.
3. **Create a Docker network** for communication between containers.
4. **Install NGINX Manager** in Docker.
5. **Install Lnbits** in Docker.
6. **Configure a Virtual Host** for Lnbits in NGINX with SSL enabled.

Now you have a complete setup with Docker, NGINX, and Lnbits, all managed through Ansible, with a secure Virtual Host accessible via HTTPS.

---
