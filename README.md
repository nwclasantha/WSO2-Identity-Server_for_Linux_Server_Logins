## Integrating WSO2 Identity Server for Linux Server Logins

![image](https://github.com/user-attachments/assets/cdb4bb2f-a17f-431c-a147-29c24e2cde70)

### Introduction

With multiple Linux servers in enterprise environments, managing user credentials and access control can be challenging. Using WSO2 Identity Server (WSO2 IS) for Single Sign-On (SSO) enables centralized identity management and simplifies the authentication process. WSO2 IS acts as an Identity Provider (IdP), allowing users to log in to Linux servers with a single set of credentials. This setup enhances security by enforcing unified access controls and reduces the administrative load on IT teams.

### Objectives

This guide aims to:
1. Configure WSO2 IS to handle SSO for Linux server logins.
2. Set up Linux servers to authenticate users through WSO2 IS.
3. Implement SSH key-based authentication and configure client-side access for secure logins.
4. Facilitate a scalable, secure, and efficient user authentication process across Linux servers.

### Security Requirements

1. **WSO2 Identity Server Configuration**: Ensure WSO2 IS is configured as an IdP to handle authentication requests from the Linux servers.
2. **Secure SSH and SSSD Configuration**: Use `sssd` (System Security Services Daemon) on Linux servers to connect with WSO2 IS and manage authorized keys.
3. **Multi-Factor Authentication (MFA)**: Configure WSO2 IS to enforce MFA as an additional layer of security.
4. **Firewall and Network Security**: Limit SSH access to trusted networks and ensure secure, encrypted connections between the servers and WSO2 IS.

---

### Step-by-Step Guide

![35DWyHMSw24Bp2zr1x1bp2](https://github.com/user-attachments/assets/2e23dc2e-0336-4ebb-bb29-8e64c7ee5475)

#### Step 1: Install Required Packages on Each Linux Server

1. **Install SSSD and related packages**:
   - For **Debian/Ubuntu**:
     ```bash
     sudo apt update
     sudo apt install sssd sssd-tools realmd adcli -y
     ```
   - For **RHEL/CentOS**:
     ```bash
     sudo yum install sssd sssd-tools realmd adcli -y
     ```

2. **Install WSO2 CLI Tools (Optional)** for administration tasks if required.

#### Step 2: Configure WSO2 Identity Server as an Identity Provider for SSH Logins

1. **Access the WSO2 Identity Server Management Console**:
   - Go to `https://<wso2_is_ip>:9443/carbon` and log in with admin credentials.

2. **Create a Service Provider for SSH Access**:
   - Navigate to **Main** > **Identity** > **Service Providers** > **Add**.
   - Name the Service Provider (e.g., “Linux SSH SSO”) and save.

3. **Configure SAML2 Web SSO**:
   - In the newly created Service Provider, go to **Inbound Authentication Configuration** > **SAML2 Web SSO Configuration** > **Configure**.
   - Set the **Issuer** and **ACS URL** for your SSH authentication requirements. WSO2 IS will issue SAML tokens for users, allowing authentication.

4. **Configure Roles and Permissions**:
   - Under **Local & Outbound Authentication Configuration**, set **Authentication Type** to **Local Authentication** and select any multi-factor requirements if needed.
   - Map roles and permissions to define which users or groups can access the Linux servers.

#### Step 3: Configure Linux Servers to Authenticate Using WSO2 IS

1. **Configure SSSD on the Linux Server**:
   - Open the SSSD configuration file:
     ```bash
     sudo nano /etc/sssd/sssd.conf
     ```
   - Configure SSSD to communicate with WSO2 IS, replacing placeholders with actual WSO2 IS details:
     ```ini
     [sssd]
     services = nss, pam, ssh
     config_file_version = 2
     domains = example.com

     [domain/example.com]
     id_provider = ldap
     auth_provider = krb5
     chpass_provider = krb5
     ldap_uri = ldap://<wso2_is_ip>:<ldap_port>
     krb5_server = <wso2_is_ip>
     krb5_realm = EXAMPLE.COM
     cache_credentials = True
     use_fully_qualified_names = False
     fallback_homedir = /home/%u
     default_shell = /bin/bash
     ```

2. **Set Permissions on the SSSD Config File**:
   ```bash
   sudo chmod 600 /etc/sssd/sssd.conf
   sudo chown root:root /etc/sssd/sssd.conf
   ```

3. **Enable and Start SSSD**:
   ```bash
   sudo systemctl enable sssd
   sudo systemctl start sssd
   ```

4. **Configure SSH to Use SSSD**:
   - Open SSH configuration:
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
   - Set the following lines to let SSSD manage authorized keys:
     ```config
     AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys
     AuthorizedKeysCommandUser nobody
     ```

5. **Restart SSH Service**:
   ```bash
   sudo systemctl restart sshd
   ```

#### Step 4: Set Up SSH Key-Based Authentication

1. **Generate an SSH Key** (if you don’t have one) on the client machine:
   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. **Add the Public Key to WSO2 IS**:
   - Go to **WSO2 Management Console** > **Users and Roles** > **Add User** or select an existing user.
   - Add the SSH public key to the user’s account profile in the attribute field if supported.

#### Step 5: Test SSH Login

1. **SSH to the Linux Server Using WSO2 IS Credentials**:
   ```bash
   ssh 'your_username'@server_ip_address
   ```
2. **Troubleshoot if Necessary**:
   - Check SSSD status:
     ```bash
     sudo systemctl status sssd
     ```
   - Check logs in `/var/log/sssd/` for any errors.

#### Step 6: Connect to the Linux Server Using PuTTY (Optional for Windows Users)

1. **Convert SSH Private Key to PuTTY Format** (Optional):
   - Open **PuTTYgen**.
   - Load your private key file (e.g., `id_rsa`).
   - Save it as `.ppk` format for PuTTY compatibility.

2. **Configure PuTTY**:
   - Open PuTTY, enter the server’s IP address, and format the username:
     ```plaintext
     your_username@server_ip_address
     ```
   - Go to **Connection** > **SSH** > **Auth** and load the `.ppk` file.

3. **Connect**:
   - Click **Open** and accept the server’s SSH key if prompted.
   - Log in using your WSO2 IS credentials.

---

### Conclusion

Integrating WSO2 Identity Server with Linux servers enables centralized authentication, reducing the need for separate credentials and enhancing access control across servers. With SSO through WSO2 IS, users benefit from streamlined, secure logins, while IT teams gain centralized management and improved security oversight. Using SSSD and SSH configurations with WSO2 IS, this setup ensures that Linux servers are both accessible and secure, making WSO2 IS a powerful tool for identity and access management in Linux environments.
