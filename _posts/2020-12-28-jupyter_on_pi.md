---
title: 树莓派上安装jupyter并连接到vscode
data: 2020-12-28 20:22:00 +0800
categories: [Develop]
tags: [jupyter,vscode]
---
Jupyter notebooks and jupyter labs are preferred workbenches for many data science projects. For IoT projects, it’s always not possible to connect to the Raspberry PI desktop to access the notebooks. We can access Raspberry PI via console using SSH but for some applications especially for the Computer Vision, we still need some graphical user interface.
In this article, I wish to share some of the techniques I found to run the Jupyterlab on a raspberry PI and access it from a remote machine.

# Install Jupyter Lab
Connect to your Raspberry PI using SSH. Install all the dependencies. We will use pip3 to install jupyter lab later.

```shell
$ sudo apt-get update
$ sudo apt-get install python3-pip
$ sudo pip3 install setuptools
$ sudo apt install libffi-dev
$ sudo pip3 install cffi
```
Install jupyter using pip3. If you are using virtualenv then follow the instructions from the jupyter documentation here.
```shell
$ pip3 install jupyterlab
```
Create a directory for your notebooks and start jupyter lab (optional)
```shell
$ mkdir notebooks
$ jupyter lab --notebook-dir=~/notebooks
```
Alternately you can also start the jupyterlab from any directory as below.
```shell
$ jupyter lab
```
This should will start the jupyterlab running on https://localhost:8888 on raspberry pi but its not accessible from the your local machine. Also once you close your SSH session, the jupyterlab instance will be terminated.

To resolve this issue, we need to start the jupyterlab as a service.

# Setup Jupyter lab as a service

Run below command to locate your jupyter lab binary:
```shell
$ which jupyter-lab
/home/pi/.local/bin/jupyter-lab
```

Create a file `/etc/systemd/system/jupyter.service`
```shell
$ sudo nano /etc/systemd/system/jupyter.service
```

With below content. Make sure to replace the path to jupyter-lab binary which was returned by which jupyter-lab command if its different for your system.
```
[Unit]
Description=Jupyter Lab
[Service]
Type=simple
PIDFile=/run/jupyter.pid
ExecStart=/bin/bash -c "/home/pi/.local/bin/jupyter-lab --ip="0.0.0.0" --no-browser --notebook-dir=/home/pi/notebooks"
User=pi
Group=pi
WorkingDirectory=/home/pi/notebooks
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

Enable the service to start it whenever the raspberry pi boots
```shell
$ sudo systemctl enable jupyter.service
$ sudo systemctl daemon-reload
```

Start the service:
```shell
$ sudo systemctl start jupyter.service
```

You can also stop the service using command:
```shell
$ sudo systemctl stop jupyter.service
```

Reboot the raspberry pi and make sure that the service is running by checking the service status
```shell
$ sudo systemctl status jupyter.service
jupyter.service - Jupyter Notebook
   Loaded: loaded (/etc/systemd/system/jupyter.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-08-30 16:12:27 PDT; 2s ago
 Main PID: 4864 (jupyter-lab)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/jupyter.service
           └─4864 /usr/bin/python3 /usr/local/bin/jupyter-lab --ip=0.0.0.0 --no-browser --notebook-dir=/home/pi/notebooks
```

The service should be up and running and you can now access it from your local machine by going to `http://<rpi ip address>:8888`

However, this will be also accessible from anywhere on internet (depending on your router settings, firewall etc). We can secure it using one of the below three methods

1.Add password (easy)

2.Using SSL for encrypted communication (recommended)

3.SSH port forwarding (recommended)

# Add password
Check to see if you have a notebook configuration file,`jupyter_notebook_config.py`. The default location for this file is your Jupyter folder located in your home directory:
```shell
/home/pi/.jupyter/jupyter_notebook_config.py
```

If you don’t already have a Jupyter folder, or if your Jupyter folder doesn’t contain a notebook configuration file, run the following command:
```shell
$ jupyter notebook --generate-config
Writing default config to: /home/pi/.jupyter/jupyter_notebook_config.py

$ jupyter notebook password
Enter password:  ****
Verify password: ****
[NotebookPasswordApp] Wrote hashed password to /Users/you/.jupyter/jupyter_notebook_config.json
```

This can be used to reset a lost password; or if you believe your credentials have been leaked and desire to change your password. Changing your password will invalidate all logged-in sessions after a server restart.

# Using SSL for encrypted communication
If you need to secure using SSL, you can follow the instruction here

# SSH port forwarding (tunneling)
This is the most secure way but you need to keep a session alive in your local terminal as long as you want to access the jupyterlab. If you prefer this approach then make sure that you update the `/home/pi/.jupyter/jupyter_notebook_config.py` and replace `ExecStart` as below by removing the `--ip="0.0.0.0"`
```
<file:/home/pi/.jupyter/jupyter_notebook_config.py>
...
ExecStart=/bin/bash -c "/usr/local/bin/jupyter-lab --no-browser --notebook-dir=/home/pi/notebooks"
...
```

After updating the file. Stop the service, reload config and start the service again.

On your raspberry pi,retrieve the IP address of the raspberry pi using either `ifconfig` or `hostname -I` command

SSH to raspberry pi with ssh port forwarding as below:
```
ssh -L 8888:localhost:8888 pi@192.168.xx.xx
```

Now you can access the jupyterlab running on raspberry pi from you local computer by going to `http://localhost:8888`


转载自<https://medium.com/analytics-vidhya/jupyter-lab-on-raspberry-pi-22876591b227>