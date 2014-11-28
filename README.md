XUC/bluemind/XiVO installation
==============================

Install XiVO
------------

Follow the procedure on the official documentation : http://documentation.xivo.io/production/installation/installation.html

Install Bluemind
----------------

Follow the official documentation on : http://blue-mind.net/communaute/installer-bluemind-3-0/

Install XiVO/bluemind connector
-------------------------------

The connector is avaiblable here : https://marketplace.blue-mind.net/addons/38/

After you have installed the package you need to create the config file for bluemind and xuc.

    cat /etc/bm/xivo.ini

    api.key=<your_xuc_api_key>
    xuc.host=<you_xuc_host>

By default the xuc port is 9000.

Now go to the bluemind webi console (superuser) and configure the application server and activate it on your domain supervised on the telecom tab.

On XiVO
-------

Create a webservice user with the ip address of your xuc server.
Authorized postgres bd with the good IP address in pg_hba.conf and restart your postgres server.

Create a xuc user with a cti profile like "Client". This user don't need a line or other specific telephony information.

On coreOS
---------

If you have coreos follow this procedure.

Add this file /etc/systemd/system/xuc.service.

    [Unit]
    Description=XuC
    After=docker.service
    Requires=docker.service
    
    [Service]
    TimeoutStartSec=0
    ExecStartPre=-/usr/bin/docker kill xuc
    ExecStartPre=-/usr/bin/docker rm xuc
    ExecStartPre=/usr/bin/docker pull jlebleu/xuc
    ExecStart=/usr/bin/docker run --name xuc -v /conf/xuc:/conf -e CONFIG_FILE=/conf/xuc.conf -e NO_DAEMON=Y -p 9000:9000 jlebleu/xuc
    ExecStop=/usr/bin/docker stop xuc
    
    [Install]
    WantedBy=multi-user.target

Create the config directory in your coreos

    mkdir /conf/xuc

Configure your xuc with the good xivo hostname, good password.

Enable the container.

    systemctl enable /etc/systemd/system/xuc.service

And start it :

    systemctl start xuc

You can check the log with this commmand.

    journalctl -f -u xuc

Only Docker
-----------

Create the config directory

    mkdir /conf/xuc

Pull the image.

    docker pull jlebleu/xuc

Launch container.

     docker run --name xuc -d -v /conf/xuc:/conf/ -e CONFIG_FILE=“/conf/xuc.conf” -e NO_DAEMON=“y” -p 9000:9000 jlebleu/xuc

To check the log.

    docker logs xuc
