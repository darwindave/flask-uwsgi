Flask with uWSGI + Nginx
===
This tutorial shows you to set up a simple Flask app with uWSGI + Nginx.

At the end of this tutorial, you will be able to do the following:

- Start or stop the app using Upstart
- Visit the app at http://flask-uwsgi.ubuntu.local. (Throughout this tutorial, replace ubuntu.local with your machine's name.) 

Setting up the Python environment
---
First, install the following packages.
    
    sudo apt-get install build-essential python-dev python-pip git

`build-essential` and `python-dev` are used to compile C extensions. Pip is used to install system commands.

or if you are using Redhat or Amazon Linux /TODO Verify

	sudo yum groupinstall “Development Tools”
	
	sudo yum install git
	
	sudo curl -o /tmp/ez_setup.py https://sources.rhodecode.com/setuptools/raw/bootstrap/ez_setup.py 

	sudo /usr/bin/python27 /tmp/ez_setup.py 
	sudo /usr/bin/easy_install-2.7 pip 

Next, install the following Pip packages.

    sudo pip install virtualenv uwsgi 

Next, download the configuration using 

	cd /var/www/
	
	git clone https://github.com/mking/flask-uwsgi.git
	
Next, create the Virtualenv for this tutorial.

    cd flask-uwsgi
    virtualenv env
    source env/bin/activate
    pip install flask

Next, run the Flask app using uWSGI.

    uwsgi --http 0.0.0.0:8080 --home env --wsgi-file flask_uwsgi.py --callable app --master

You should be able to visit http://ubuntu.local:8080.

Creating an Upstart service for uWSGI
---
First, set up the directories the service will use.

	# Add group www-data user www-data
	sudo groupadd www-data
	sudo adduser www-data --system --no-create-home -g www-data

    # Create a directory for the UNIX sockets
    sudo mkdir /var/run/flask-uwsgi
    sudo chown www-data:www-data /var/run/flask-uwsgi

    # Create a directory for the logs
    sudo mkdir /var/log/flask-uwsgi
    sudo chown www-data:www-data /var/log/flask-uwsgi

    # Create a directory for the configs
    sudo mkdir /etc/flask-uwsgi

Next, create the init script.

- Copy the init script template to /etc/init/flask-uwsgi.conf.
- Replace /path/to/flask-uwsgi with the path to this directory.

Next, create the config file.

- Copy the config template to /etc/flask-uwsgi/flask-uwsgi.ini.
- Set uid and gid to the numeric uid and gid for www-data. Use the following commands to get the ids

	sudo id -u www-data
	sudo id -g www-data

Next, start uWSGI.

    sudo service flask-uwsgi start

Next, check that uWSGI is running.

    sudo less /var/log/flask-uwsgi/flask-uwsgi.log

You should see messages from uWSGI.

Connecting uWSGI to Nginx
---
First, install Nginx (if you haven't already).

    sudo apt-get install nginx    

Next, create the Nginx config.

- Copy the config template to /etc/nginx/sites-available/flask-uwsgi.
- Replace ubuntu.local with your machine's name.

Next, enable the config.

    sudo ln -sf /etc/nginx/sites-available/flask-uwsgi /etc/nginx/sites-enabled
    sudo service nginx reload

Next, check that the app is reachable through Nginx. You should be able to visit http://flask-uwsgi.ubuntu.local.

Congratulations! You have deployed an app to uWSGI + Nginx.

Code reloading
---
You can reload code (or reload the uWSGI config) by sending the HUP signal to uWSGI.

    sudo service flask-uwsgi reload

Adding or simulating DNS entries
---
If you are running this tutorial in a VM, you can simulate DNS names in the host by editing /etc/hosts. This allows you to visit your VM from your host's browser.

    # Replace <IP> with your VM's IP. 
    <IP> ubuntu.local flask-uwsgi.ubuntu.local

To check this, visit http://ubuntu.local (or http://flask-uwsgi.ubuntu.local) in your host's browser.

References
---
- [uWSGI Quickstart](http://uwsgi-docs.readthedocs.org/en/latest/WSGIquickstart.html)
- [uWSGI Configuration](http://uwsgi-docs.readthedocs.org/en/latest/Configuration.html)
- [uWSGI Upstart](http://uwsgi-docs.readthedocs.org/en/latest/Upstart.html)
