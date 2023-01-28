
Logrotate is used in all Linux distribution like Ubuntu, RHEL, CentOS for managing logs that are generated by the Linux machine.

Logrotate manages logs by rotating, compressing, or removing logs automatically on daily, weekly, monthly basis or as per the size of the logs (as it grows).

The moving parts that make logrotate run on any Linux machine are:

 - The actual tool **logrotate**. (/usr/bin/logrotate) 

 - Logrotate's configuration file located at **/etc/logrotate.conf**. This file holds the configuration for all log files that Logrotate manages.

 - A daily cron job **/etc/cron.daily/logrotate** that issues the logrotate command to run based on settings in its configuration file. 
   (/usr/sbin/logrotate /etc/logrotate.conf)
   
 - If you look inside **/etc/logrotate.conf**, you will see that it has the line **include /etc/logrotate.d** in it. What this line does is tell **Logrotate** to look inside the **/etc/logrotate.d** directory and run every configuration file in it. This directory is typically where applications installed on your linux system will add their logrotate configurations. For example, Apache2 will typically create a /etc/logrotate.d/apache configuration file upon installation.


Setting Up an example config for Logrotate 

### Install Logrotate.

Most Linux systems come with Logrotate installed by default. Check if you have it installed on your VMs by issuing the **logrotate** command. 

If you don't have it installed, perform the steps below to install logrotate:

```sh
sudo apt-get update
sudo apt-get install logrotate
```

### Create a new test log file for logrotate demonstration.

```sh
sudo mkdir /var/log/myapp
sudo touch /var/log/myapp/myapp.log
```

![myapp-log](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/myapp-log.png)

### Create a script to generate logs for your test log file.

Create a script.

```sh
sudo nano log_gen_script.sh
```
```sh
#!/bin/bash
echo "$(date '+%b %d %H:%M:%S') $(hostname) MyApp[2022]: MyApp's log messages"  >> /var/log/myapp/myapp.log
```

Make your script executable.

```sh
sudo chmod 755 log_gen_script.sh
```

![loggenscript](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/loggenscript.png)

Open an editor to edit the **root** user crontab

```sh
sudo crontab -e
no crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 1
```

This cron job will schedule the script to run every minute.

```sh
* * * * * /home/cloud_user/log_gen_script.sh
```

![syslog1](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/syslog1.png)

### Create a new Logrotate configuration

To manage log files using logrotate for applications outside of the pre-packaged and pre-configured system services, create a new Logrotate configuration file and place it in **/etc/logrotate.d/**. This will be run daily as the **root** user along with all the other standard Logrotate jobs.

```sh
sudo nano /etc/logrotate.d/myapp
```
The name of the file doesn't really matter but the contents of the file should look something like this.

```sh
/var/log/myapp/myapp.log {
        size 100
        missingok
        rotate 3
        dateext
        dateformat -%Y-%m-%d_%H:%M:%S
        create 644 root root
}
```

The configuration options in the content above instruct Logrotate to:

 - **size 100**: Rotate log file if size is greater than or equal to 1k.

 - **missingok**: Ignore error messages if testfile.log does not exist.

copytruncate: Create a copy of current log file and then truncate it. This comes in handy when an application cannot close its log file because it continuously appends to it.

 - **rotate 3**: limit the number of log file rotations to 5. This will delete old versions of log files greater than 5 days.

 - **dateext**:
 
 - **dateformat -%Y-%m-%d_%H:%M:%S**:
 
 - **create 644 root root**:

If you left everything as default, logrotate will be running everyday.

To ensure logrotate is running every 5 minutes. Add another line to your **root** crontab file.

```sh
sudo crontab -e
```

This will force the logrotate command to run every five minute.

```sh
*/5 * * * * /usr/sbin/logrotate /etc/logrotate.d/myapp
```

![cron](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/cron.png)

![syslog2](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/syslog2.png)

![myapp-log1](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/myapp-log1.png)

Logrotate removes an older log file "**myapp.log-2022-12-30_02:00:01**" to add a new one "**myapp.log-2022-12-30_02:15:01**"

![syslog3](https://github.com/vottri/logrotate-notes/blob/main/images/syslog3.png)


