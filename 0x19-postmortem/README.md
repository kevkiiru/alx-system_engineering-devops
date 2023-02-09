Postmortem — MySQL server fails to start in Ubuntu 20.04

/home/kev/Pictures/devops/1_IGn_mHFsIpl694u5fxXPqA.webp

Issue Summary
Duration of outage: 6th February, 2023, from 3:00pm to 4:00pm EAT.

After a successful re-installation of mysql client, which needed to take place due to an earlier vagrant crash and also in preparation for an upcoming AirBnB clone version 3 team project, the display of the version was okay and printed the installed version, but at the moment of starting the service, the following message was received:

Job for mysql.service failed because the control process exited with error code.
See "systemctl status mysql.service" and "journalctl -xe" for details.

Then check for error message (systemctl status mysql.service) displayed:

mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; disabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Wed 2023-01-11 15:05:34 UTC; 19s 
    Process: 40046 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=1/FAILURE)
    
Initially, we thought the error occurred in relation to the disc space or root privileges. After spending some minutes in debugging, we discovered that the issue was a previous installation error.

Impact
This impacted the process to deploy a new application and caused set back to our team on the AirBnB clone project. Fortunately, it was an internal error and was detected at the earliest stage of the project and not before getting out to production.

Timeline
** 3;00pm EAT ~ After successful installation of mysql.service, I discovered that the service couldn’t start when I ran the command to start the service.

** 3:07 pm EAT ~ I alerted my project partner and we started investigating the case. At first, we looked at the disk and the configuration file. The initial assumption was made that the issue was with disk space and root privileges.

** 3:42 pm EAT ~ After investing the disk and executing the commands with root privileges and not finding the root cause of the issue, we further investigated the error log file and discovered redundant dependency issues.

** 3:50 pm EAT ~ We thereafter resolved the issue by uninstalling, purging and completely cleaning the service from the system.

** 4:00 pm EAT ~ MySQL service was successfully installed and the service started successfully.

Root Cause and Resolution:
Ø Mysql service was previously installed but removed from the system because the vagrant crashed and was restored on the system by re-installation.

Ø The initial re-installation was neither successfully installed nor successfully uninstalled.

Ø This led to installation issues and dependency error, thereby causing the service to be completely shut down and to enter a ‘dead’ state.

Ø The issue was resolved by getting rid of all instances of the service before a new one was installed.

Corrective and Preventative Measures:
· Train the system administrator on proper uninstallation and installation of MySQL service

· For installation, ensure you have strong internal connections to avoid incomplete installation.

· Before installing the server, ensure there are no instances of the server on the system. If there are, purge them before proceeding with the installation. You can achieve this by taking the following steps:

1. Uninstall the program. Run:

$ apt autoremove
$ apt autoclean
$ sudo apt purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-*

2. Back up configuration files that might be needed on the next installation

$ sudo mv /var/lib/mysql /var/lib/mysql_bk
$ sudo mv /etc/mysql /etc/mysql_bk

and thereafter manually remove all residual files present in /etc and /var/lib/mysql directory.

$ sudo rm -r /etc/mysql /var/lib/mysql

3. Auto remove & clean leftover dependencies. Do:

$ sudo apt autoremove
$ apt autoclean

4. Safely re-installation of MySQL. Run:

$ sudo apt update
$ sudo apt install mysql-server -y
$ sudo mysql_secure_installation

5. Check for the status of the service

$ sudo systemctl status mysql.service

6. Start the service

$ sudo service mysql start

7. Don’t forget to set up a password for your SQL.
