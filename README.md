

 - The actual tool **logrotate**. (/usr/bin/logrotate) 

 - Logrotate's configuration file located at **/etc/logrotate.conf**. This file holds the configuration for all log files that Logrotate manages.

 - A daily cron job **/etc/cron.daily/logrotate** that issues the logrotate command to run based on settings in its configuration file. 
   (/usr/sbin/logrotate /etc/logrotate.conf)


Most Linux systems come with Logrotate installed by default. Check if you have it installed on your VMs by issuing the logrotate command. 

If you don't have it installed, perform the steps below to install logrotate:

```sh
sudo apt-get update
sudo apt-get install logrotate
```

Create a new test log file.

```sh
sudo mkdir /var/log/myapp
sudo touch /var/log/myapp/myapp.log
```

![myapp-log](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/myapp-log.png)

```sh
sudo nano log_gen_script.sh
```
```sh
#!/bin/bash
echo "$(date '+%b %d %H:%M:%S') $(hostname) MyApp[2022]: MyApp's log messages"  >> /var/log/myapp/myapp.log
```

```sh
sudo chmod 755 log_gen_script.sh
```

![loggenscript](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/loggenscript.png)

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
```sh
* * * * * /home/cloud_user/log_gen_script.sh
```

![syslog1](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/syslog1.png)


Create a new Logrotate configuration
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

The configuration options in the snippet above instruct Logrotate to:

size 1k: Rotate log file if size is greater than or equal to 1k.

missingok: Ignore error messages if testfile.log does not exist.

copytruncate: Create a copy of current log file and then truncate it. This comes in handy when an application cannot close its log file because it continuously appends to it.

rotate 5: limit the number of log file rotations to 5. This will delete old versions of log files greater than 5 days.




To ensure logrotate is running every 5 minutes. Add another line to your /etc/crontab file.
```sh
sudo crontab -e
```
```sh
*/5 * * * * /usr/sbin/logrotate /etc/logrotate.d/myapp
```
![cron](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/cron.png)

![syslog2](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/syslog2.png)

![myapp-log1](https://raw.githubusercontent.com/vottri/logrotate-notes/main/images/myapp-log1.png)


![syslog3](https://github.com/vottri/logrotate-notes/blob/main/images/syslog3.png)


