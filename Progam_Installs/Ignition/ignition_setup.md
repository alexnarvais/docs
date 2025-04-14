s# Updating/Installing Ignition
___
## Linux Machines
Open a local terminal and SSH into the ignition server that's needing an upgrade see the command below:
```commandline
ssh {username}@{ignition server name} 

# EXAMPLE: ssh automation@igs-cafe-01.alcon.pemo
```
Change the current working directory to last ignition install directory which is located on local network drive cnas-01.  
The path to the Ignition directory will look similar to the below path.
```commandline
cd /mnt/scada/Admin/Program_Install_Files/InductiveAutomation/'Ignition 8.X.X'
```
If an issue occurs finding the network share path above, then you may need to mount the network share drive using the following command:
```commandline
sudo mount //cnas-01.alcon.pemo/scada -o username=admin, password=1richcat
```
If none of the above options work then download the latest ignition version onto your local machine    
and use the `scp` command to transfer the file to the ignition gateway needing an upgrade, use the following   
command for reference:     
```commandline
scp ignition-x.x.x-linux-64-installer.run automation@{ignition server name}.alcon.pemo:/home/automation

# EXAMPLE: scp ignition-8.1.47-linux-64-installer.run automation@igs-cafe-01.alcon.pemo:/home/automation
```
Remove the previous ignition installer file using the following command:
```commandline
rm -rf ignition-x.x.x-linux-64-installer.run
```
If the installer doesn't run then you may need to make the installer an executable using the following command:
```commandline
chmod +x ignition-x.x.x-linux-64-installer.run
```
Run the ignition installer using the following command where "x.x.x" is ignition version to be installed:
```commandline
./ignition-x.x.x-linux-64-installer.run
```

Don't start ignition yet but instead go through the default options.
After verifying the default options then change directories to where ignition is installed and start ignition using the following commands:
```commandline
/usr/local/bin/ignition/ignition.sh start
```


