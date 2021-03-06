# Gov-UK offline clone VM

OVF + VMDK hosting a clone of `www.gov.uk`
This hosting clone has the html of www.gov.uk recursively scraped + all files linked in www.gov.uk in assets.publishing.service.gov.uk.
This is all packed inside gov_uk.zip.

## Creating the VM

* Access esxi server with valid credentials   [esxi01.glasswall-icap.com](http://esxi01.glasswall-icap.com) or [esxi02.glasswall-icap.com](http://esxi02.glasswall-icap.com) 

* Create a new Linux, Ubuntu (64-bit) VM with the follwing hardware specs:
    * 1 CPU,
    * 1 GB RAM
    * 80 GB of Harddisk (set disk Provisioning to be thin provisioned)

* Set CD/DVD drive is connected at power on and choose the ISO to boot from (live ubuntu server iso)

  ![image](https://user-images.githubusercontent.com/58347752/100460151-66715900-30cf-11eb-914e-2f802acb5052.png)

* Finish installation and boot the machine with default configuration

* Once you "Power ON" your Machine start with setup. Pay attention to the steps below. 

* In the network configuration, under ens160, edit the IPV4 method to be manual and add the network configuration. Example on image below. This IP addresses are ESXi related and can be obtained from corresponding channel.

  ![Networkconnection](https://user-images.githubusercontent.com/70108899/100768735-82d90280-33fb-11eb-8e1d-f60164fad167.PNG)

* Set the username to be `glasswall` and the password `Gl@$$wall`

Once installation is done restart the VM and press enter when it asks to remove the CD

## Implementation

- Access the VM

- Clone the repo and run configure.sh from /home/glasswall/GW-proxy/OVAs-creation/govuk
  
  ```bash
  git clone https://github.com/k8-proxy/GW-proxy
  cd GW-proxy/OVAs-creation/govuk
  ./configure.sh
  ```

- Download the Gov.uk clone archive `wget -O ~/gov_uk.zip https://glasswall-sow-ova.s3-eu-west-1.amazonaws.com/vms/gov-uk/gov_uk.zip`

- Extract the archive into **`/var/www/html`**, but before that create directory www and html inside 

  ```bash
  sudo apt install -y unzip
  unzip ~/gov_uk.zip
  # This last command will take some time
  ```
- After extracting the archive correctly, delete the archive to save space `rm ~/gov_uk.zip`

- Power off the VM `sudo poweroff` then export the VM to OVA/OVF

## Import OVA in the VM

- Download **s3://glasswall-sow-ova/vms/gov-uk/GovUK.ova** (AWS S3 bucket), you can download it [here](https://glasswall-sow-ova.s3-eu-west-1.amazonaws.com/vms/gov-uk/GovUK.ova)

- Login to VMware ESXi with a privliged user (i.e: **root**)

- From sidebase choose **Virtual machines**

- Click **Create / Register VM**

- Choose **Deploy a virtual machine from OVF or  OVA file**

- Click **Click to select files or drag/drop** and select the **.ova** file downloaded from the S3 bucket 

- Select desired storage and *optionally* tweak VM configuration

- Wait for the import to finish

## Usage

- Start the VM and wait until it's fully booted

- login as **`glasswall`** using the password **`Gl@$$wall`**

* Unless your environment does **not** use DHCP for network configuration,  you should have everything running, you can get your VM ip by running `ip addr show`
  
  If DHCP isn't used or for some reason the VM could not fetch automatic network configuration, you will have to:
  
  - Configure network manually, this can be done by modifying `/etc/netplan/network.yaml` , for reference, you can use this example:
    
    ```yaml
    network:
      version: 2
        ethernets:
          eth0:
            addresses:
              - 192.168.0.3/24 # Replace this with desired IP address in CIDR format
            gateway4: 192.168.0.1 # Replace this with desired gateway
            nameservers:
              addresses: [8.8.8.8, 1.1.1.1] # Replace DNS servers if needed
    ```

- Run `sudo netplan apply` to apply new network configuration

- Edit the local computer hosts file (C:\Windows\System32\drivers\etc\hosts on WIN, /etc/hosts on MAC and Linux), append the following to the end of the file, replace the placeholder with the VM ip address
  
  ```
  <VM IP ADDRESS> www.gov.uk assets.publishing.service.gov.uk www.gov.uk.local assets.publishing.service.gov.uk.local
  ```

- Download ca.pem from the VM home folder, this can be done by running `scp`  from your client computer as in the following example, replace the placeholder IP with the VM IP
  
  ```bash
  scp glasswall@<VM IP ADDRESS>:/home/glasswall/ca.pem .
  ```

* Open firefox, From the menu choose ***Preferences*** and search for ***certificates*** and press of ***View certificates*** 

* From the ***Authorities** tab, Import the created file on your desktop ***ca.pem*** and make sure to trust it

![image](https://user-images.githubusercontent.com/58347752/101023030-de7dca00-357a-11eb-8335-78de7f89aee1.png)

![image](https://user-images.githubusercontent.com/58347752/101023218-21d83880-357b-11eb-9059-1911dd0b410d.png)

* Try to access gov uk website: **https://www.gov.uk**

