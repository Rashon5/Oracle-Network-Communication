
# Controlled Deployment of Network Communication Control in Oracle Cloud Infrastructure

## Project Description
Deploying a Global Financing Portal app in the Finance subnet, then allow people from other departments, like Sales, to access it when needed. Each department has its own subnet, so the Security List feature needs to allow and control the communication and access between subnets.

## Video Representation
[![Watch the video on YouTube](https://img.youtube.com/vi/zFRANayOAg4/0.jpg)](https://www.youtube.com/watch?v=zFRANayOAg4)

## Steps

### 1. Create Compartments
Create two Compartments in Oracle called `resourceNetworking` for Networking (VCN, Subnets, etc.) and `resourceCompute` for Instances.

### 2. Download Scripts
Download the 'OCI Scripts (GitHub).zip' from the repository which includes all the bash scripts.
- The `Order of Operations.txt` shows the order in which the scripts should be run.

### 3. List of --compartment-ids in Each Script
- `Create-VCN.sh` -- `resourceNetworking`
- `Create-Subnet.sh` -- `resourceNetworking` -- for `finance-subnet` and `sales-subnet`
- `Create-SL.sh` -- `resourceNetworking` -- for `sl-finance` and `sl-sales`
- `Create-IGateway.sh` -- `resourceNetworking`
- `Create-RouteRule-IG.sh` -- `resourceNetworking`
- `Create-VM.sh` -- `resourceCompute`, `resourceNetworking` -- for `finance-vm` and `sales-vm`

### 4. SSH into Finance VM
1. Inside Oracle GUI, press Start on each instance 2-3 times so SSH can kick in.
2. `ssh -i <private-key.key> opc@<public-ip>`

#### Run Commands to Create Web App:
```bash
sudo yum install httpd -y
sudo apachectl start
sudo systemctl enable httpd
sudo firewall-cmd --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --reload
cd /var/www/html/
sudo wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/0GFffVSC0EF6-IAf1h6KiDDscR1izrV2q0mMZOeH9Gabm7dGGCj5U_Ybbr4a3MAs/n/idqfa2z2mift/b/bootcamp-oci/o/EN/oci-handson-module-networking-website-files.zip
sudo unzip oci-handson-module-networking-website-files.zip
sudo chown apache:apache *.*
sudo rm -rf __MACOSX oci-handson-module-networking-website-files.zip
```

### 5. SSH into Sales VM
#### Run Commands to Setup GUI Server:
```bash
sudo yum -y groups install "Server with GUI"
sudo yum group list
sudo yum install tigervnc-server -y
```

### 6. Execute Commands to Configure the VNC Server:
```bash
cd /lib/systemd/system/
sudo cp vncserver@.service /etc/systemd/system/vncserver@:1.service
sudo vi /etc/systemd/system/vncserver@:1.service
```
Inside `/etc/systemd/system/vncserver@:1.service`, replace `<USER>` with `opc`:
- Before: `ExecStart=/usr/bin/vncserver_wrapper <USER> %i`
- After: `ExecStart=/usr/bin/vncserver_wrapper opc %i`

#### Create VNC Password:
```bash
vncpasswd
Password: welcome1
Would you like to enter a view-only? (y/n)? Insert N
```

### 7. Setting Up and Loading VNC
```bash
sudo systemctl enable vncserver@:1.service
sudo systemctl daemon-reload
sudo systemctl start vncserver@:1.service
sudo systemctl status vncserver@:1.service
ssh -L 5901:localhost:5901 opc@<public-ip-sales-vm> -i <private-key-sales-vm>
```

### 8. Access Instance from RealVNC
1. Open RealVNC > Use without signing in
2. Right click > New Connection > VNC Server: `localhost:5901`
3. Enter password: `welcome1`, Press next until Linux Desktop shows
4. Open Firefox, enter private IP of finance-vm
5. Web app should show

## What I Learned
- How to create and manage compartments in Oracle Cloud Infrastructure.
- How to write and execute bash scripts for setting up cloud infrastructure.
- Setting up and configuring web applications on Oracle Cloud instances.
- Implementing security measures and controlling access between subnets using Security Lists.
- Configuring and using VNC to remotely access and manage cloud instances.
