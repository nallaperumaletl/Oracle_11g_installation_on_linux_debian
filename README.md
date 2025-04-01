# Oracle 11g Installation on Linux (Debian)

## Download Oracle XE 11g
Due to GitHub's file size limit, download the Oracle XE 11g installation file from the link below:

🔗 [Download oracle-xe-11.2.0-1.0.x86_64.rpm.zip](https://drive.google.com/file/d/1lnU8j-rsOjcPmkI7qz_RqXnkPs9gnwlU/view?usp=drive_link)

Follow the instructions in this repository to install Oracle 11g on Linux.


1. Download Oracle Database Express Edition.

2. Instructions before installing Oracle database
Copy the downloaded file and paste it in home directory.

Unzip using the downloaded .zip file:

```bash
 unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip 
```
Install required packages:

 sudo apt-get install alien libaio1 unixodbc # still works in 22.04
Change directories to the Disk1 directory:

```bash
 cd Disk1/
```
Convert .rpm package format to .deb package format (that is used by Ubuntu):

```bash
 sudo alien --scripts -d oracle-xe-11.2.0-1.0.x86_64.rpm
```
Create the required chkconfig script:

```bash
 sudo nano /sbin/chkconfig
```
The nano text editor is started and the commands are shown at the bottom of the screen. Now copy and paste the following into the file and save:

```bash
 #!/bin/bash
 # Oracle 11gR2 XE installer chkconfig hack for Ubuntu
 file=/etc/init.d/oracle-xe
 if [[ ! `tail -n1 $file | grep INIT` ]]; then
     echo >> $file
     echo '### BEGIN INIT INFO' >> $file
     echo '# Provides: OracleXE' >> $file
     echo '# Required-Start: $remote_fs $syslog' >> $file
     echo '# Required-Stop: $remote_fs $syslog' >> $file
     echo '# Default-Start: 2 3 4 5' >> $file
     echo '# Default-Stop: 0 1 6' >> $file
     echo '# Short-Description: Oracle 11g Express Edition' >> $file
     echo '### END INIT INFO' >> $file
 fi

 update-rc.d oracle-xe defaults 80 01
 ```
Change the permission of the chkconfig file:

```bash
 sudo chmod 755 /sbin/chkconfig  
```
Set kernel parameters. Oracle 11gR2 XE requires additional kernel parameters which you need to set using the command:

```bash
 sudo nano /etc/sysctl.d/60-oracle.conf
```
Copy the following into the file and save:

```bash
 # Oracle 11g XE kernel parameters 
 fs.file-max=6815744  
 net.ipv4.ip_local_port_range=9000 65000  
 kernel.sem=250 32000 100 128 
 kernel.shmmax=536870912 
 ```
Verify the changes:


```bash
sudo cat /etc/sysctl.d/60-oracle.conf 
```
You should see what you entered earlier. Now load the kernel parameters:

```bash
sudo service procps start # If this doesn't work try sudo systemctl start procps 
```
Verify the new parameters are loaded:

```bash
sudo sysctl -q fs.file-max
```
You should see the file-max value that you entered earlier.

Set up /dev/shm mount point for Oracle. Create the following file:

```bash
sudo nano /etc/rc2.d/S01shm_load
```
Copy the following into the file and save.

```bash
#!/bin/sh
case "$1" in
    start)
        mkdir -p /var/lock/subsys 2>/dev/null
        touch /var/lock/subsys/listener
        rm -rf /dev/shm 2>/dev/null
        mkdir -p /dev/shm 2>/dev/null
        mount -t tmpfs shmfs -o size=2048m /dev/shm ;;
    *)
        echo "Usage: $0 {start} error"
        exit 1
        ;;
esac


```
Change the permissions of the file:

```bash
sudo chmod 755 /etc/rc2.d/S01shm_load
```
Run the following commands:

```bash
sudo ln -s /usr/bin/awk /bin/awk 
sudo mkdir /var/lock/subsys 
sudo touch /var/lock/subsys/listener
sudo reboot
```
3. Install Oracle database
Install Oracle DBMS:

```bash
sudo dpkg --install oracle-xe_11.2.0-2_amd64.deb
```
Configure Oracle:

```bash
sudo /etc/init.d/oracle-xe configure 
```
Setup environment variables by editing your .bashrc file:

```bash
nano ~/.bashrc
```
Add the following lines to the end of the file:
```bash
 export ORACLE_HOME=/u01/app/oracle/product/11.2.0/xe
 export ORACLE_SID=XE
 export NLS_LANG=`$ORACLE_HOME/bin/nls_lang.sh`
 export ORACLE_BASE=/u01/app/oracle
 export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
 export PATH=$ORACLE_HOME/bin:$PATH
```
Load the changes by executing your profile:

```bash
. ~/.bashrc
```
Start Oracle 11gR2 XE:

```bash
 sudo service oracle-xe start
```
Add user YOURUSERNAME to group dba using the command:

```bash
sudo usermod -a -G dba YOURUSERNAME
```
4. Using the Oracle XE command shell
Start the Oracle XE 11gR2 server:

```bash
sudo service oracle-xe start
```
Start command-line shell as the system admin:

```sql
sqlplus sys as sysdba
```
Enter the password that you gave while configuring Oracle earlier. You will now be placed in a SQL environment that only understands SQL commands.

Create a regular user account in Oracle using the SQL command:

```sql
create user USERNAME identified by PASSWORD;
create user coreunix identified by core;
```
Replace USERNAME and PASSWORD with the username and password of your choice. Please remember this username and password. If you had error executing the above with a message about resetlogs, then execute the following SQL command and try again:

```sql
alter database open resetlogs;
```
Grant privileges to the user account using the SQL command:

```sql
grant connect, resource to coreunix;
```
Replace USERNAME and PASSWORD with the username and password of your choice. Please remember this username and password.

Exit the sys admin shell using the SQL command:

```bash
exit;
```
Start the command-line shell as a regular user using the command:

```sql
sqlplus
```
