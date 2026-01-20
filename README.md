# proxmox-ve-on-lxd

Install Proxmox as an LXD VM in 2026

Installing Proxmox inside LXD is best achieved by running it as a **Virtual Machine (VM)**. 

1. **Launch a Debian 12 VM in LXD:**  
   Proxmox is based on Debian. Use the `cloud` variant to simplify initial setup.  
    ``` bash
    lxc launch images:debian/12/cloud PROXMOX \--vm \-c limits.memory=8GB \-c limits.cpu=4
    ```
2. **Enable Nested Virtualization:**  
   To allow Proxmox to run its own VMs inside this LXD VM, you must disable secure boot and enable host CPU passthrough.  
    ``` bash
    lxc stop PROXMOX
    lxc config set PROXMOX security.secureboot=false  
    lxc config set PROXMOX raw.qemu \-- "-cpu host"  
    lxc start PROXMOX
    ```
3. **Install Proxmox Packages:**  
   Access the VM console and follow the standard "Install Proxmox on Debian" steps:  
   1. Set a static IP and update `/etc/hosts`.  
        Edit /etc/systemd/network/20-enp5s0.network:
        ```
        [Match]
        Name=enp5s0

        [Network]
        Address=10.171.117.144/24
        Gateway=10.171.117.1
        DNS=10.171.117.1
        ```
        Append to /etc/hosts `10.171.117.144 PROXMOX` then run `networkctl reload`.
   2. Add the Proxmox repository: `echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list`. 
   3. Install the kernel and environment: `apt update && apt full-upgrade && apt install proxmox-ve postfix open-iscsi`.  
        Fix for error "The repository 'http://download.proxmox.com/debian/pve bookworm InRelease' is not signed.":
        ```
        wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
        apt update
        ```
        Verify the checksum: `sha512sum /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg`.
        The ouput must match with: `7da6fe34168adc6e479327ba517796d4702fa2f8b4f0a9833f5ea6e6b48f6507a6da403a274fe201595edc86a84463d50383d07f64bdde2e3658108db7d6dc87`
4. **Access Web UI:**  
    Once installed, access the interface at `https://10.171.117.144:8006`.
    
