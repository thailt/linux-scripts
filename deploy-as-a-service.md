```bash
#!/bin/bash
#usage: sudo ./deploy-as-a-service.sh /path/to/service/execution/file service-name which-user-run-this-service
chmod a+x  $1

sudo ln -fs $1 /etc/init.d/$2

sudo touch /etc/systemd/system/$2.service

sudo sh -c "echo '[Unit]                           ' >  /etc/systemd/system/$2.service"
sudo sh -c "echo 'Description=$2                   ' >> /etc/systemd/system/$2.service"
sudo sh -c "echo 'After=syslog.target              ' >> /etc/systemd/system/$2.service"
sudo sh -c "echo '[Service]                        ' >> /etc/systemd/system/$2.service"
sudo sh -c "echo 'User=$3                          ' >> /etc/systemd/system/$2.service"
sudo sh -c "echo 'ExecStart=/etc/init.d/$2         ' >> /etc/systemd/system/$2.service"
sudo sh -c "echo 'SuccessExitStatus=143            ' >> /etc/systemd/system/$2.service"
sudo sh -c "echo '[Install]                        ' >> /etc/systemd/system/$2.service"
sudo sh -c "echo 'WantedBy=multi-user.target       ' >> /etc/systemd/system/$2.service"

sudo systemctl enable $2.service

sudo touch /var/log/$2.log
sudo chown $3 /var/log/$2.log

sudo service $2 start

tail -1000 /var/log/$2.log
```





