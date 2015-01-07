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

By default the xuc port is 9000 (TCP). On bluemind the port 9091 (TCP) need to be open to get the notification from XiVO.

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
    Description=Xuc
    After=docker.service
    Requires=docker.service

    [Service]
    TimeoutStartSec=0
    ExecStartPre=-/usr/bin/docker kill xuc
    ExecStartPre=-/usr/bin/docker rm  xuc
    ExecStartPre=/usr/bin/docker pull xivo/xuc
    ExecStart=/usr/bin/docker run -rm --name xuc  -p 8080:9000 -v /conf/xuc:/conf xivo/xuc -Dconfig.file=/conf/xuc.conf  -mem 256
    ExecStop=/usr/bin/docker stop xuc

    [Install]
    WantedBy=multi-user.target

    [Unit]
    Description=XuC
    After=docker.service
    Requires=docker.service
    

Create the config directory in your coreos

    mkdir /conf/xuc

Configuration
-------------

You can find a sample of the configuration here : https://github.com/sboily/docker-xuc/blob/master/config/xuc.conf
Detailed xuc configuration can be found here : http://xuc.readthedocs.org/en/latest/

Configure your xuc with the good xivo hostname, good password.

Minimal configuration
---------------------

Provided you configured your xivo with default user and password

    #
    # bluemind xivo connector configuration file
    #
    include "application.conf"

    xivohost="192.168.51.252"

    XivoWs {
        enabled=true
    }

    api {
        eventUrl = "http://bluemind.avencall.com:9091/xivo/1.0/event/avencall.com/dropbox/"
    }

    xucami {
        enabled = false
    }

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

    docker pull xivo/xuc

Launch container.

     docker run --rm --name xuc  -p 8080:9000 -v /conf/xuc:/conf xivo/xuc -Dconfig.file=/conf/xuc.conf  -mem 256

To check the log.

    docker logs xuc

