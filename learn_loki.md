

## Install

```
wget https://github.com/grafana/loki/releases/download/v2.9.8/logcli-linux-amd64.zip

wget https://github.com/grafana/loki/releases/download/v2.9.8/loki-linux-amd64.zip

wget https://github.com/grafana/loki/releases/download/v2.9.8/promtail-linux-amd64.zip
```

```
sudo adduser --system promtail
$ cd /var
$ var/ sudo setfacl -R -m u:promtail:rX log
$ sudo chown promtail:promtail /tmp/positions.yaml
$ sudo usermod -a -G systemd-journal promtail
I had the same problem, I wanted to tail logs from four “.log” files in a folder inside /var/log.
Giving 755 permission to the folder that contains the log files, saved my day.

$ sudo adduser --system promtail

$ sudo setfacl -R -m u:promtail:rX /var/log

OBS: You may have to install the acl package to run setfacl.

$ sudo usermod -a -G systemd-journal promtail

$ sudo usermod -a -G adm promtail


$ sudo firewall-cmd --zone=public --add-port=3100/tcp

$ sudo firewall-cmd --zone=public --list-all

###
Troubleshooting

Permission denied for log-files

Dec  7 15:51:40 server01 promtail-linux-amd64[2266]: level=error ts=2021-12-07T14:51:40.709501865Z caller=filetarget.go:287 msg="failed to start tailer" error="open /var/log/fail2ban.log: permission denied" filename=/var/log/fail2ban.log

The logfile's access rights:

$ getfacl /var/log/fail2ban.log
getfacl: Removing leading '/' from absolute path names
# file: var/log/fail2ban.log
# owner: root
# group: adm
user::rw-
group::r--
other::---

This is typically he result of adding a log after running setfacl above. Running setfacl again gives the promtail-user read access to the log:

sudo setfacl -R -m u:promtail:rX /var/log

Account not available

If you try to run promtail with runuser

$ sudo runuser -l promtail -c "promtail -log.level=debug -config.file=/etc/loki/promtail.yaml"

and you get

This account is currently not available.

there is no valid shell for the promtail user. Add a shell with the chsh-command:

$ sudo chsh -s /bin/bash promtail

Connection refused

This is usually a firewall-related problem. Check that you have opened the right ports.

Failed to locate executable

If you get something like this:

[root@localhost ~]# systemctl status promtail.service
× promtail.service - Promtail for Loki
     Loaded: loaded (/etc/systemd/system/promtail.service; enabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2024-01-09 05:21:23 EST; 5s ago
   Duration: 22ms
    Process: 3633351 ExecStart=/usr/local/bin/promtail-linux-amd64 -config.file /etc/loki/promtail.yaml (code=exited, status=203/EXEC)
   Main PID: 3633351 (code=exited, status=203/EXEC)
        CPU: 21ms

Jan 09 05:21:23 localhost.localdomain systemd[1]: Started Promtail for Loki.
Jan 09 05:21:23 localhost.localdomain systemd[3633351]: promtail.service: Failed to locate executable /usr/local/bin/promtail-linux-amd64: Permission denied
Jan 09 05:21:23 localhost.localdomain systemd[3633351]: promtail.service: Failed at step EXEC spawning /usr/local/bin/promtail-linux-amd64: Permission denied
Jan 09 05:21:23 localhost.localdomain systemd[1]: promtail.service: Main process exited, code=exited, status=203/EXEC
Jan 09 05:21:23 localhost.localdomain systemd[1]: promtail.service: Failed with result 'exit-code'.
it either due to the executable not being in the right place or to file permissions. For the latter, do:

[root@localhost ~]# ls -al /usr/local/bin/
total 165048
drwxr-xr-x+  3 root     root          128 Jan  2 12:35 .
drwxr-xr-x. 12 root     root          131 May 30  2023 ..
-rwxr-xr-x+  1 loki     loki     59424768 May  3  2023 loki-linux-amd64
-rw-r--r--+  1 root     root     18930096 May 31  2023 loki-linux-amd64.zip
-rwxr-xr-x.  1 root     root          233 Nov  6 11:53 normalizer
-rwxrwxr--+  1 promtail promtail 90640576 May  3  2023 promtail-linux-amd64
drwxr-xr-x.  7 root     root         4096 Jan  4 06:47 server_heartbeat
We can see that promtail is the user for the executable, and it appears to have the correct permissions. Let's take a look at the ACL:

[root@localhost ~]# getfacl /usr/local/bin/promtail-linux-amd64 
getfacl: Removing leading '/' from absolute path names
# file: usr/local/bin/promtail-linux-amd64
# owner: promtail
# group: promtail
user::rwx
group::r-x
other::r--
That looks good as well. However, we can see from the directory listing that SELinux is controlling access (some of the files has "." at the end). Is SELinux active and what SELinux context does the file have?

[root@localhost ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

[root@localhost ~]# ls -Z /usr/local/bin/loki-linux-amd64
unconfined_u:object_r:admin_home_t:s0 /usr/local/bin/loki-linux-amd64

Here is the problem. promtail cannot run files in the admin_home_t. Other running services have bin_t. Let's change that:

semanage fcontext -a -t bin_t /usr/local/bin/promtail-linux-amd64
restorecon -v /usr/local/bin/promtail-linux-amd64

Restart the service and look at the status.

References

Introduction:

https://grafana.com/docs/loki/latest/getting-started/get-logs-into-loki/

Configuring Promtail as a service:

https://sbcode.net/grafana/install-promtail-service/

General configuration of promtail:

https://grafana.com/docs/loki/latest/clients/promtail/configuration/

Troubleshooting:

https://grafana.com/docs/loki/latest/clients/promtail/troubleshooting/

Grafana Community:

https://community.grafana.com/



With service definition
consul services register 


{
  "service": {
    "id": "",
    "name": "",
    "tags": [],
    "address": "",
    "port": 8080,
    "checks": [
      {
        "args": ["/usr/local/bin/check_mem.py"],
        "internal": "30s"
      }
    ]
  }

}


```


