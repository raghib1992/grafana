Check nginx is install or not
nginx -v


If not, Install nginx
sudo apt install nginx -y


Check the status of the service
sudo systemctl status nginx


cd /etc/nginx/sites-enabled
vi HOSTNAME.com.conf
```
server {
    listen 80;
    listen [::]:80;
    server_name  YOUR-DOMAIN-NAME;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass           http://localhost:3000/;
    }
}
```

start nginx service
```
sudo service nginx restart
sudo service nginx status
```

Add SSL
https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal


I ensure snap is installed.


sudo snap list
Make sure I have the latest version of snap


sudo snap install core; sudo snap refresh core
I install the classic certbot


sudo snap install --classic certbot
Prepare the command so that it can be executed from the command line


sudo ln -s /snap/bin/certbot /usr/bin/certbot
Start the process of installing the SSL certificate for my domain name.


sudo certbot --nginx
Follow the prompts, and enter the domain name that you want to secure.

After completion, you should then be able to now visit your Grafana server using the url

https://YOUR-DOMAIN-NAME

Note that after running Certbot, it has changed the settings of your Nginx configuration file you created earlier.

You can see those changes by using the cat command.


cat /etc/nginx/sites-enabled/YOUR-DOMAIN-NAME

