# Flask-on-EC2

This repository contains a Flask "Hello World" web app that can quickly be deployed on an Amazon EC2 instance. Instructions on how to configure nginx and uWSGI to work with Flask are below.

# Instructions

Instructions adapted from [vladikk's blog post](http://vladikk.com/2013/09/12/serving-flask-with-nginx-on-ubuntu). More detailed commentary is available at his site.

####These first steps just set up the server – we don't touch the git repo yet.

1. Launch a new EC2 instance from the [AWS console](https://console.aws.amazon.com), running Ubuntu (tested on 14.04.2).

2. After it finishes setting up, SSH into the instance from your local terminal.

 ```ssh -i private-key.pem ubuntu@instance-ip``` for a one-time session.

 ```ssh -i private-key.pem ubuntu@instance-ip "echo `cat ~/.ssh/id_rsa.pub` >> ~/.ssh/authorized_keys"``` to copy your public key to the server for permanent authentication.

 ```ssh ubuntu@instance-ip``` after your public key is copied to the server.

3. Add nginx to apt-get’s sources, update and upgrade apt-get, and install some packages.

 ```
 sudo add-apt-repository ppa:nginx/stable
 sudo apt-get update && sudo apt-get upgrade
 sudo apt-get install build-essential python python-dev python-virtualenv git nginx
 ```

4. Start nginx with `sudo /etc/init.d/nginx start`. Navigate to the instance's public IP address (make sure the instance's Security Group allows HTTP). You should see an nginx welcome page.

####The following steps will set up Flask using this repository's content.

1. Clone this repository to your home folder, or some downloads folder.

 ```
git clone https://github.com/kennysong/Flask-on-EC2/
```

2. Move the `demoapp` subdirectory to `/var/www/` and chown it.

 ```
 sudo mv Flask-on-EC2/demoapp /var/www/demoapp
 sudo chown -R ubuntu:ubuntu /var/www/demoapp
```

3. Go into the directory, start virtualenv, and install some modules with pip.

 ```
cd /var/www/demoapp
virtualenv venv
. venv/bin/activate
pip install flask uwsgi
```

4. If you run `python server.py` and navigate to http://instance-ip:8080, you should see the "Hello World" page generated by our Flask code.

5. Now we configure uWSGI to connect to nginx. We replace nginx's default site configuration with our own, and make a directory for uWSGI log files.

 ```
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /var/www/demoapp/demoapp_nginx.conf /etc/nginx/conf.d/
sudo mkdir -p /var/log/uwsgi
sudo chown -R ubuntu:ubuntu /var/log/uwsgi
```

6. Restart nginx, and start uWSGI with our config file.

 ```
 sudo /etc/init.d/nginx restart
 uwsgi --ini /var/www/demoapp/demoapp_uwsgi.ini
```

7. Now, visiting the IP address should show our Flask app. However, we want uWSGI to run as a background service. We set up new directories and config files to run uWSGI Emperor in the background.

 ```
sudo mv uwsgi.conf /etc/init/uwsgi.conf
sudo mkdir /etc/uwsgi && sudo mkdir /etc/uwsgi/vassals
sudo ln -s /var/www/demoapp/demoapp_uwsgi.ini /etc/uwsgi/vassals
sudo chown -R www-data:www-data /var/www/demoapp/
sudo chown -R www-data:www-data /var/log/uwsgi/
```

8. Finally, run `sudo start uwsgi` to connect everything together. Visiting the IP address will show our Flask app without having to manually run uWSGI.
