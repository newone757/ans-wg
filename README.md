# ans-wg
An **Ansible playbook** for setting up a provate **Wireguard mesh network** . The playbook is designed to be idempotent, meaning it can be run multiple times on the same systems without causing unintended changes. Please read in it's entirity as it implements very restrictive firewall rules

---

### Key Features

* ✅ **Wireguard Installation**: Automatically installs Wireguard and its dependencies on both **Ubuntu/Debian** and **CentOS/RHEL** systems.
* ✅ **Configurable Port**: A custom Wireguard port can be set using the **`wireguard_port`** variable.
* ✅ **Custom SSH Settings**: Allows for custom SSH ports and key locations to be defined for each host in the inventory.
* ✅ **Firewall Rules**: Automatically configures **UFW (Ubuntu/Debian)** and **firewalld (CentOS/RHEL)** to allow Wireguard traffic, creating a secure environment.
* ✅ **Control Node Option**: The option to include or exclude the Ansible control node from the network is controlled by the **`include_control_node`** variable.
* ✅ **Mesh Network**: Creates a **full mesh connectivity** between all specified nodes, ensuring every node can communicate directly with every other node.

---

### Usage Instructions

1.  **Create the directory structure**: First, set up the project's directory as follows:

    ```markdown
    wireguard-ansible/
    ├── wireguard.yml
    ├── inventory.yml
    ├── ansible.cfg
    ├── .vault_pass (optional)
    ├── templates/
    │   └── wg0.conf.j2
    └── group_vars/
        └── all/
            └── vault.yml (encrypted)
    ```

2.  **Save the files**: Save the main playbook as **`wireguard.yml`**, the Ansible configuration as **`ansible.cfg`**, and the Wireguard template at **`templates/wg0.conf.j2`**.
3.  **Configure the inventory**: Create your inventory file named **`inventory.yml`**, specifying your server IPs, custom SSH ports, and key locations.
4.  **Create the encrypted vault file**: Secure your sensitive data, like sudo passwords, by creating an encrypted vault file using the following command:
    ```bash
    ansible-vault create group_vars/all/vault.yml
    ```
5.  **Run the playbook**: Execute the playbook with the inventory file to begin the automated setup:
    ```bash
    ansible-playbook -i inventory.yml wireguard.yml
    ```

---

### Configuration Variables

* **`wireguard_port`**: The UDP port for Wireguard. The default is **`51820`**.
* **`include_control_node`**: A boolean variable that determines whether to include the Ansible control node in the network. The default is **`true`**.
* **`enable_ufw`**: A boolean variable to enable UFW firewall rules. The default is **`true`**.
* **`ansible_port`**: Specifies a custom SSH port for each host.
* **`ansible_ssh_private_key_file`**: Defines the location of the SSH key file for each host.

---

### Network Details

* The playbook creates a private network using the **`10.0.0.0/24`** subnet.
* Each node is assigned an IP address starting from **`10.0.0.1`**.
* IP forwarding is enabled to handle network routing.
* **Persistent keepalive** is set up to ensure the connections remain active, which is useful for **NAT traversal**.
* It establishes **full mesh connectivity** between all nodes.

---

### ⚠️ Security Note: Restrictive Firewall Configuration

This playbook configures a highly restrictive firewall that **denies all traffic by default**. The only traffic explicitly allowed is:

* SSH on the configured port.
* Wireguard VPN traffic on the specified UDP port.
* All traffic within the Wireguard network interface.
* Loopback traffic (localhost).
* Outgoing traffic initiated by the servers.

If you need to allow additional services (e.g., HTTP/HTTPS), you must add them manually to the firewall rules.
* For **UFW**: `ufw allow 80/tcp`
* For **firewalld**: `firewall-cmd --permanent --add-port=80/tcp && firewall-cmd --reload`

---

### Vault Management Commands

These commands are essential for managing your encrypted vault file:

* **`ansible-vault view group_vars/all/vault.yml`**: View the contents of the encrypted file.
* **`ansible-vault edit group_vars/all/vault.yml`**: Edit the encrypted file.
* **`ansible-vault rekey group_vars/all/vault.yml`**: Change the vault password.
