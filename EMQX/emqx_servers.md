# EMQX Server Node Main Content Steps
The following virtual machine(s) will be created using the PROXMOX Hypervisor Type 1 Software.   
___
1. Access the Proxmox hypervisor web interface using a web browser and enter the following url in the specified format:  
    **https://PROXMOX-Server-IP-Address:8006/** 
2. If a base ubuntu template (**base-ubuntu-template**) is available, then see the 
   [EMQX Server Node Setup ](#emqx-server-node-setup) section, if not continue in **this section** to **step 3**.
3. If no base Ubuntu template is available, then see the **base-ubuntu** build sheet. 
4. Create the EMQX HAProxy servers using the **emqx_haproxy** document. 

## EMQX Server Node Setup 
___
1. Right-click and perform a full clone of the base Ubuntu template (**base-ubuntu-template**) and set the following settings below:  

   > Mode = **Full Clone**  
   > Target Storage = **Same as source**  
   > Name = **emq-XX** (where XX is the server instance)  
   > All Other Settings = **Default**

   **NOTE**: If the virtual machine needs to be under a different PROXMOX node (pm-01, pm-02, ...pm-XX), 
   then initiate a **migration** to the necessary PROXMOX node before modifying or starting the virtual machine.  

2. Update the VM configuration settings by accessing the VM management interface and selecting on the VM:  
   1. **Hardware Settings:**  
      1. **Memory:**  
         > Memory (MiB) = **8192**   
           Minimum memory (MiB) = **1024**   
           Ballooning Device = **True**   
           All Other Settings = **Default**  
      
         See the image below for modifying the **Hardware Memory** settings:   
         ![](img/vm_hardware_memory.png)   
   
      2. **Processors:**  
         > Sockets = **2**   
           Cores = **2**      
           All Other Settings = **Default**  

         See the image below for modifying the **Hardware Processor** settings:    
         ![](img/vm_hardware_processors.png)   
   
   2. **Options Settings**:      
      1. **QEMU Guest Agent:**   
         > Use QEMU Guest Agent  = **True**  
           All other parameters = **Default**    
       
         See the image below for modifying the **Option QEMU Guest Agent** settings:    
         ![](img/vm_options_qemu_guest_agent.png)   

      2. **Start at boot:**   
         > Start at boot = **True**   
      
         See the image below for modifying the **Option Start At Boot** settings:    
         ![](img/vm_options_start_at_boot.png)   
 
3. Start the virtual machine using the **Start** button.  
4. Update and upgrade the operating system using the following commands:   
   ```shell
   sudo apt update && sudo apt upgrade -y
   ```
   **NOTE:** If prompted to select which daemon services should be restarted, then accept the default selections, 
   press the **tab** key to navigate between the selections. 
5. Update the hostname from **base_ubuntu** to **emq-XX** (where XX is the server instance) using the following command:
   ```shell
   sudo nano /etc/hostname
   ```
6. Update the hosts file using the following command:  
   ```shell
   sudo nano /etc/hosts
   ```
   Overwrite the existing configuration with the following text, replace XX with the server IP address and 
   instance number:    
   ```shell
   127.0.0.1 localhost
   10.20.XX.XX emq-XX.research.pemo emq-XX
   10.20.1.13 ad-01.research.pemo ad-01
   10.20.5.13 ad-02.research.pemo ad-02
   10.20.3.13 ad-03.research.pemo ad-03
   ```
   See image below for reference:  
   ![](img/emqx_ad_hosts_file.png)  
   **NOTE**: IP Address per node server should fall within the following subnets:  
   
   > emq-01 - 10.20.1.18/24 and gateway 10.20.1.1  
   > emq-02 - 10.20.5.18/24 and gateway 10.20.5.1  
   > emq-03 - 10.20.3.18/24 and gateway 10.20.3.1  
   
7. Change the network interface IP address from **DHCP** to **Static** by editing the **00-installer-config.yaml** 
   file, using the following command:   
    ```shell
    sudo nano /etc/netplan/00-installer-config.yaml
    ```
   See the image below for reference:  
   ![](img/emqx_netplan_config_static_ip.png)  
   **NOTE**: IP Address per node server should fall within the following subnets:  
   
   > emq-01 - 10.20.1.18/24 and gateway 10.20.1.1  
   > emq-02 - 10.20.5.18/24 and gateway 10.20.5.1  
   > emq-03 - 10.20.3.18/24 and gateway 10.20.3.1  
   
8. Reset the machine ID using the following commands:
   ```shell
   sudo  rm  -f  /etc/machine-id /var/lib/dbus/machine-id
   sudo dbus-uuidgen --ensure=/etc/machine-id
   sudo dbus-uuidgen --ensure
   ```
9. Regenerate ssh keys using the following commands:
   ```shell
   sudo rm /etc/ssh/ssh_host_*
   sudo dpkg-reconfigure openssh-server
   ```
10. Restart the machine using the following command:  
    ```shell
    sudo reboot
    ```
11. Allow incoming connections on the following ports, using the following commands: 
    ```shell
    sudo ufw allow 1883/tcp
    sudo ufw allow 4370/tcp
    sudo ufw allow 5370/tcp
    sudo ufw allow 8080/tcp
    sudo ufw allow 8084/tcp
    sudo ufw allow 8404/tcp
    sudo ufw allow 18083/tcp
    ```
    Verify the firewall rules were accepted using the following command:  
    ```shell
    sudo ufw status numbered
    ```
12. Goto any MariaDB server (mdb-01, mdb-02, or mdb-03) and create the mqtt database and tables.    
    Issue the following commands:    
    1. Access the MariaDB shell:  
       ```shell 
       sudo mariadb -u root -p
       ```
    2. Create the **mqtt** database:  
       ```mariadb 
       CREATE DATABASE mqtt;
       ```
    3. Create a user to access the **mqtt** database:   
       1. Use the new more secure password hashing method when creating or changing passwords.
          ```mariadb 
          SET old_passwords=0;
          ```
       2. Create a new user with a password.
          ```mariadb 
          CREATE USER 'emqx'@'10.20.%' identified by '<one_extra_rich_capital_cat>';
          ```
       3. Grant the newly created user access to the tables in the database. 
          ```mariadb 
          GRANT ALL PRIVILEGES ON mqtt.* TO 'emqx'@'10.20.%';
          ```
       4. Apply the changes.
          ```mariadb 
          FLUSH PRIVILEGES;
          ```
       5. Check that the user was created. 
          ```mariadb
          SELECT user, host FROM mysql.user;
          ```
    4. Access the **mqtt** database:  
       ```mariadb 
       USE mqtt;
       ```
    5. Create the **mqtt_user** table in the **mqtt** database:  
        1. Create the table.
           ```mariadb
           CREATE TABLE `mqtt_user` (
           `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
           `username` varchar(100) DEFAULT NULL,
           `password_hash` varchar(100) DEFAULT NULL,
           `salt` varchar(35) DEFAULT NULL,
           `is_superuser` tinyint(1) DEFAULT 0,
           `created` datetime DEFAULT NULL,
           PRIMARY KEY (`id`),
           UNIQUE KEY `mqtt_username` (`username`)
           ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
           ```
        2. Create a user in the **mqtt_user** table.
           ```mariadb
           INSERT INTO mqtt_user(username, password_hash, salt, is_superuser) 
           VALUES ('mqtt', SHA2('V#2Rs%2E7%vem8', 256), NULL, 0);
           ```
        3. Check that the user was created. 
           ```mariadb
           SELECT * FROM mqtt.mqtt_user;
           ```
    **NOTE**: **Step 12** only has to be executed once, when creating the first EMQX node. 
              If yes, skip **Step 12** and continue to **Step 13** below.    
13. Install EMQX on Ubuntu using the following commands: 
    1. Download the EMQX repository:  
       ```shell
       curl -s https://assets.emqx.com/scripts/install-emqx-deb.sh | sudo bash
       ```
    2. Install EMQX:
       ```shell
       sudo apt install emqx
       ```
    3. Start EMQX:
       ```shell
       sudo systemctl start emqx
       ```
    4. Access the EMQX dashboard with the default login password by using the domain name or IP address 
       of the host where EMQX is being configured, using the url below:  

       > **http://EMQX-Server-IP-Address:18083/**  
       
       > default username = admin  
         default password = public
       
       **NOTE**: Skip the password reset prompt. 
       
14. Edit the main EMQX broker configuration file using the following command:  
    ```shell 
    sudo nano /etc/emqx/emqx.conf
    ```
    Overwrite the existing configuration file with the configuration below, keep the existing comments and add the new
    comments from below:  
    ```shell
    # Human-Optimized Config Object Notation (HOCON) File
    # EMQX Docs for verison 5.4 can be found at https://www.emqx.io/docs/en/v5.4.1/hocon/
    node {
      # Check the cluster.static.seeds parameter for the IP addresses of the nodes in the cluster. 
      name = "emqx@10.20.XX.XX"
      cookie = "5u#k4UGe9nX#^9"
      data_dir = "/var/lib/emqx"
    }
    
    cluster {
      name = emqxcl
      # Auto clustering by static node list
      discovery_strategy = static
      static {
        seeds = ["emqx@10.20.1.18", "emqx@10.20.5.18", "emqx@10.20.3.18"]
      }
      autoheal = true
      autoclean = 5m
      # Allow the nodes to connect via TCP IPv4
      proto_dist = inet_tcp
    }
    
    # MariaDB Authentication
    authentication = [ 
      {
       enable = true
       backend = "mysql"
       mechanism = "password_based"
       server = "mdb.research.pemo:3306"
       database = "mqtt"
       username = "emqx"
       password = "<one_extra_rich_capital_cat>"
       pool_size = 8
       query = "SELECT password_hash FROM mqtt_user where username = ${username} LIMIT 1"
       query_timeout = "5s"
       password_hash_algorithm {
         name = sha256
         salt_position = disable
       }
      }
    ]
    
    dashboard {
        listeners.http {
          bind = 18083
          max_connections = 512
          # proxy_header = true
        }
    }
    
    listeners.tcp.default {
      enable = true
      bind = "0.0.0.0:1883"
      enable_authn = quick_deny_anonymous
      proxy_protocol = true
      proxy_protocol_timeout = "5s"
      max_connections = 1024000
      # max_connections = "infinity"
    }
    
    log {
      file_handlers.default {
      enable = true
      level = debug
      file = "log/emqx.log"
      rotation = 10
      rotation_size = 50MB
      formatter = json
      }
      console_handler {
         enable = true
         level = warning
         formatter = json
      }
    }
    ```
    **NOTE**: The **node.name** and **authentication.password** should be the only parameters that need to be updated in the
     configuration file for every node instance.  
15. Restart the **emqx** service and verify the activity status.  
    1. Restart the service: 
       ```shell
       sudo systemctl restart emqx
       ```
    2. Check the status of the service:  
       ```shell
       sudo systemctl is-active emqx
       ```
16. Reset the default password when logging in a second time using one of the following methods:    
    1. Access the EMQX Web Dashboard using the hostname or IP address of the machine.   
       See image below for reference:   
       ![](img/emqx_dashboard_password_change.png)    
    2. Access the EMQX command utility and enter the following command:  
       ```shell
       emqx ctl admins passwd admin <one_extra_rich_capital_cat> 
       ```
    3. The password reset will ONLY need to be initiated on first EMQX node created.  
       Once each node joins the cluster, the updated password will propagate to the node.   
       The updated password can then can be used to login to the Web Dashboard.     
    
    **NOTE:** See below credentials for default username and password with new password:     

    > default username = admin    
      default password = public  
      new password = <one_extra_rich_capital_cat>   

17. Verify the EMQX cluster status using one of the following methods    
    1. Access the EMQX Web Dashboard and check **Cluster Overview** page using the login credentials 
       from **Step 13.4** by entering the following url:   
                 
       > **http://EMQX-Server-IP-Address:18083/**
       
       The status of the cluster from checking the **Cluster Overview** should look similar to the image below:   
       ![](img/emqx_dashboard_cluster_status.png)  
       
    2. Access the EMQX command utility and issue the following command from any node in the cluster:  
       ```shell
       sudo emqx ctl cluster status
       ```
       Output from issuing the command should look similar to the image below:  
       ![](img/cmd_line_cluster_status.png)   
    
18. Verify the connection status to the MariaDB database for client authentication.  
    See the image below for reference:     
    ![](img/emqx_dashboard_db_auth.png)    

19. Join the EMQX server to the Active Directory:  
    1. Install the necessary Samba and Kerberos packages to integrate with a Windows OS network using the command below:  
       ```shell
       sudo apt install samba krb5-config krb5-user winbind libnss-winbind libpam-winbind -y 
       ```
       **NOTE**: If prompted for the kerberos default realm type **RESEARCH.PEMO** then highlight over **Ok** 
       and press enter as in the image below:  
       ![](img/default_kerberos_realm.png)  
    2. Edit the Kerberos configuration file using the **nano** command:   
       ```shell
       sudo nano /etc/krb5.conf
       ```
       Add the following to the end of **[realms]** section:  
       ```ini
       RESEARCH.PEMO = {
               kdc = AD-01.RESEARCH.PEMO
               kdc = AD-02.RESEARCH.PEMO
               kdc = AD-03.RESEARCH.PEMO
               default_domain = RESEARCH.PEMO
       }
       ```
       Add the following to the end of **[domain_realm]** section:  
       ```ini
       .research.pemo = .RESEARCH.PEMO
       research.pemo = RESEARCH.PEMO
       ```
    3. Edit the Samba configuration file using the following command:
       ```shell 
       sudo nano /etc/samba/smb.conf
       ```
    4. Add the following text to the **[global]** section:  
       ```ini
       workgroup = RESEARCH
       netbios name = EMQ-<XX>
       realm = RESEARCH.PEMO
       server string = 
       security = ads
       encrypt passwords = yes
       password server = AD-01.RESEARCH.PEMO
       log file = /var/log/samba/%m.log
       max log size = 50
       socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF=8192
       preferred master = False
       local master = No
       domain master = No
       dns proxy = No
       idmap uid = 10000-20000
       idmap gid = 10000-20000
       winbind enum users = yes
       winbind enum groups = yes
       winbind use default domain = yes
       client use spnego = yes
       template shell = /bin/bash
       template homedir = /home/%U
       ```
       **NOTE 1**: Comment out any existing variable names that are similar to the names in the new configuration 
       for the **[global]** section above.   
       Potential existing variables:  
       
       >  **workgroup**  
          **server string**  
          **log file**  
          **max log size**  
       
       **NOTE 2**: The **netbios name** parameter (`netbios name = EMQ-01, EMQ-02, or EMQ-03`)  
       should be the only changed parameter across each EMQX instance and configuration.   
       See the image below for reference:    
       ![](img/samba_server_config_emqx.png)   
    
    5. Edit the name service switch configuration file using the following command:  
       ```shell
        sudo nano /etc/nsswitch.conf
       ```
       Replace the existing text in the file with the following: 
       ```shell
       passwd: compat winbind files systemd
       group: compat winbind files systemd
       shadow: compat winbind files
       gshadow: files
       
       hosts: files dns
       networks: files
       
       protocols: db files
       services: db files
       ethers: db files
       rpc: db files
       
       netgroup: nis
       ```
    6. Edit the **sudoers (/etc/sudoers.tmp)** configuration using the command below:  
       ```shell
       sudo visudo
       ```
       Add the following line to the end of the file:  
       ```text
       %cansudo ALL=(ALL:ALL) ALL
       ```
    7. Join the machine to active directory domain using the following command:  
       ```shell
       sudo net ads join -S AD-01.RESEARCH.PEMO -U <user_in_ad_domain>
       ```
       **NOTE:** **<user_in_ad_domain>** is a user who has privileges in the AD domain to add a computer.  
    8. Restart the **Samba** service using the following command:   
       ```shell
       sudo systemctl restart smbd
       ```
    9. Restart the **winbind** service using the following command:  
       ```shell
       sudo systemctl restart winbind
       ```
       Verify that **winbind** service established a connection to the active directory domain by running the command below:  
       ```shell
       sudo wbinfo -u
       ```
       **NOTE:** This command will return a list of users from the domain that is connected via **winbind**.   
    10. Ensure a user's home directory is created upon their first login, using the following command:  
        ```shell
        sudo pam-auth-update --enable mkhomedir
        ```
    11. Verify AD login acceptance into the machine by logging out and logging in with an AD account.   
        Use the following command for reference:  
        ```shell
        ssh <user_in_ad_domain>@emq-XX.research.pemo
        ```
20. Install **SentinelOne** cybersecurity software.   
    
    > The following sub steps will explain how to install **SentinelOne** by using a NAS (network attached storage) 
      device, then accessing the installation files on the NAS.  
    
    1. Check that the latest **SentinelOne GA Version** is on the **scada** share drive using the following path:  
       
       > /Volumes/scada/program_install_files/sentinel_one  
      
       See the image below for finding the latest packages using the **SentinelOne Web Management Console**:   
       ![](./img/sentinelone_packages.png)   
    
    2. Make note and verify the site token for the site that the machine will join, the site token for a site can be found using
       the following image for reference, click the site to find the site token:  
       ![](./img/sentinelone_settings_sites.png)  
    3. Install the network file system packages if not already installed using the following command:   
       ```shell
       sudo apt install nfs-common -y
       ```
    4. Create a NFS directory on the local machine to share using a similar command to the following:  
       ```shell
       sudo mkdir -p /mnt/scada/nas
       ```
    5. Check that the correct NFS share is available on the NFS server using a similar command to the following:  
       ```shell
       showmount -e cnas-01.research.pemo
       ```
       The following image will show the NFS shares available, from issuing the above command:  
       ![](./img/nfs_shares_available_on_server.png)   
       If the NFS share is not available, then check the following on the NAS:  
       - Ensure the share folder is created.  
       - Check the location of the share folder.  
       - Check the NFS permission rules.

    6. Mount the external NFS share on machine using a similar command to the following:  
       ```shell
       sudo mount -t nfs cnas-01.research.pemo:/volume2/scada /mnt/scada/nas
       ```
    7. Allow full permissions (read, write, execute) for the owner, group and others using a similar command to the following:  
       ```shell
       sudo chmod 777 /mnt/scada/nas
       ```
    8. Change directories to the location where the files and shell script are located using a similar command to the following:  
       ```shell
       cd /mnt/scada/nas/program_install_files/sentinel_one
       ```
    9. Once in the **SentinelOne** directory execute the shell script **sentinelone_linux_agent_install.sh** using the following command:  
       ```shell
       sudo ./sentinelone_linux_agent_install.sh
       ```
       **NOTE:** Ensure that the latest packages from **Step 17.1** are in the directory and that the shell script 
       contains the correct path to the latest package and site token (with respect to the site that the machine will join).  
       Use the following command to open the shell script, if necessary:  
       ```shell
       sudo nano sentinelone_linux_agent_install.sh
       ```
    10. Open up the **SentinelOne** web management console and verify the machine joined the Sentinels endpoint list, check the image below:  
        ![](./img/sentinelone_endpoints.png)  
21. Shutdown the VM and remove all **CD/DVD Drives**, see the following image for reference:
    ![](./img/vm_remove_hardware_cd-dvd-drive.png)  
22. Start the VM.
23. Repeat **Steps 1–22** above, for every EMQX node created.  
24. Jump to **Step 4** in the [EMQX Server Node Main Content Setup](#emqx-server-node-main-content-steps) section.  
