+++
title = "Setup a fresh server"

date = 2020-12-23T21:00:00
draft = false

authors = ["Gabriel Teotonio"]

tags = ["Server", "Linux", "Cloud"]

summary = "Some important configuration are needed when creating a new server (Ubuntu in our case). This document will go through steps to achieve a good initial configuration, keeping your server safe and reliable."

# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
# Caption (optional)
caption = ""

# Focal point (optional)
# Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
focal_point = ""

# Show image only in page previews?
preview_only = false

+++

# Initial (and necessary)

### Logging in

First verify if you have a SSH client installed to proceed.

The first step is to connect with your server. There will be different ways depending the cloud you chose to host your server. But in general you can simply type the following command

```bash
ssh root@my-server-ip
```

Where the root is the administrative user in Linux and `my-server-ip` is the IP of your server that might be available in your cloud. Clouds like AWS already provides a private key and a public DNS for connecting in a safer way when you create a server. In this case the command is

```bash
ssh -i /path/my-key-pair.pem my-instance-user-name@my-instance-public-dns-name
```

Where the `.pem` file is your private key. This file is available to download on the AWS EC2 console. Your private key file should be in `~/.ssh/` directory and to keep things safe, your private key file should be readable only by your user. To do so you can type the following command

`chmod 400 ~/.ssh/my-key-pair.pem`. The `chmod` command helps us to set the permissions about read, write, and execute files. To understand a lit bit more about the combination you have to pass as argument, please checkout [chmodcommand](https://chmodcommand.com/chmod-400/). Basically you have a sequence of bits that determines the level of access of a specific file in relation to e owner of this file, its group user, and others users in general . You can type `ll` in a directory to see the permission description of each file inside, like

![Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled.png](Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled.png)

Which means

![Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled%201.png](Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled%201.png)

After the initial login, run a `apt-get update` and `apt-get upgrade` to keep things Up-To-Date.

### Creating a new user

Note that the simplest way to make a initial login is using the root user. This approach is discouraged once the root has very broad privileges and this power can be cause very destructive changes. Besides there are many attacks to servers that assume the existence of a root user as door for breaking into your server. To avoid such attack We are going to create a new user and disable the root user.

Once you are logged in with the `root` user, you can use the command

```bash
adduser username
```

That command creates a new user called by `username` (here you should provide the name you want). The system will require to type some information about this new user as password and full name. The only required field is the password.

Our interest is to disable the root user to make logins in our server. However, the new user should be able to run commands and execute actions with a superuser or `root` privileges. Otherwise We won't have permission to execute simple commands (install packages, add new users) in the server and keep it running. To give these privilegies to our new user, We have to add it to the **sudo** group by

```bash
usermod -aG sudo username
```

The flag `a` corresponds to append and `G` declares which group. Now, when logged in as your regular user, you can type `sudo` before commands to perform actions with superuser privilegies.

Note that when using AWS you already have a user called `ubuntu` to initiate the actions in the server and run commands as a superuser. 

### Blocking root user

Read all topic before executing commands!

Once We have a new user with superuser privilegies, we have to disable the `root` user. In the ideal world, We are note able to login in our server by using passwords and only use private keys to access, like the way We perform to login in AWS. There is a config file called by `sshd_config` with a lot of definitions and guidelines  about the ssh environment.

We can see it by

```bash
su - username
vi /etc/ssh/sshd_config
```

In the **Authentication** section, there are the following definition, for example

```bash
PermitRootLogin yes
```

That is, the root user can login into our server. We can change this definition by only setting

```bash
PermitRootLogin no
```

As AWS works by default when creating a new server, We want to permit only login by ssh key. This definition is set up by changing to `no` the password authentication option.

```bash
PasswordAuthentication no
```

This will force the user to provide a ssh key to login into our server. Observe that We only created a new user with super privilegies, but didn't create a key for him. So, before We disable the root user, is necessarily to create this key or be locked out of server in the first logout. 

We are going to use `ssh-keygen` to create our ssh key

```bash
cd ~/.ssh
ssh-keygen
```

The `ssh-keygen` will require a file name to store the key and passphrase (optional). You can find more about ssh-keygen [here](https://www.ssh.com/ssh/keygen/).

Now, you should export the key to server

```bash
ssh-copy-id -i ~/.ssh/mey-new-key username@my-server-ip
```

Once the public key has been configured on the server, the server will allow any connecting user that has the private key to log in, and now We can disable root user and password authentication. After edit the `sshd_config` is necessary to run `sudo systemctl restart ssh` to reload the ssh service. In AWS, We need to perform [additional steps](https://aws.amazon.com/premiumsupport/knowledge-center/new-user-accounts-linux-instance/) to import the ssh key to EC2.

### Firewall

By default, later versions of Ubuntu should come with Uncomplicated Firewall or UFW. You can check to see if UFW is installed with the following

```bash
sudo ufw status
```

That will return a status of active or inactive. It’s a good idea to think through a list of components that will require access to your server. For a while, let's allow only ssh access through ou firewall by

```bash
sudo ufw allow ssh
```

And then enable the firewall with this

```bash
sudo ufw enable
```

This may interrupt the current SSH connection if that is how you are logged in so be sure your information is correct, so you don’t get logged out.

# Going further

## Nginx

[Nginx](https://www.nginx.com/) is one of the most popular web servers in the world and is responsible for hosting some of the largest and highest-traffic sites on the internet. It is a lightweight choice that can be used as either a web server or reverse proxy

### Installing

We can install nginx by

```bash
sudo apt install nginx
```

### Adjusting the Firewall

Before testing `nginx`, the firewall software needs to be adjusted to allow access to the service. `nginx` registers itself as a service with `ufw` upon installation, making it straightforward to allow `nginx` access.

List the application configurations that `ufw` knows how to work with by typing

```bash
sudo ufw app list
```

You should get a listing of the application profiles

```bash
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

As demonstrated by the output, there are three profiles available for Nginx:

- **Nginx Full**: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
- **Nginx HTTP**: This profile opens only port 80 (normal, unencrypted web traffic)
- **Nginx HTTPS**: This profile opens only port 443 (TLS/SSL encrypted traffic)

It is recommended that you enable the most restrictive profile that will still allow the traffic you’ve configured. Right now, we will only need to allow traffic on port 80.

You can enable this by typing

```bash
sudo ufw allow 'Nginx HTTP'
```

You can verify the change by typing

```bash
sudo ufw status
```

The output will indicated which HTTP traffic is allowed

```bash
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

### Checking your Web Server

We can check with the `systemd` init system to make sure the service is running by typing

```bash
systemctl status nginx
```

```bash
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-04-20 16:08:19 UTC; 3 days ago
     Docs: man:nginx(8)
 Main PID: 2369 (nginx)
    Tasks: 2 (limit: 1153)
   Memory: 3.5M
   CGroup: /system.slice/nginx.service
           ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2380 nginx: worker process
```

As confirmed by this out, the service has started successfully. However, the best way to test this is to actually request a page from Nginx.

You can access the default Nginx landing page to confirm that the software is running properly by navigating to your server’s IP address. If you do not know your server’s IP address, you can find it by using the icanhazip.com tool, which will give you your public IP address as received from another location on the internet

```bash
curl -4 icanhazip.com
```

You can also use `nslookp` command to find your server's IP address

```bash
nslookup mydomain.com #for a custom domain
nslookup my-instance-public-dns-name # for a public dns name 
```

When you have your server’s IP address, enter it into your browser’s address bar

```bash
http://your_server_ip
```

You should receive the default Nginx landing page:

![Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled%202.png](Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled%202.png)

If you are on this page, your server is running correctly and is ready to be managed.

### Setting Up Server Blocks

Nginx on Ubuntu 20.04 has one server block enabled by default that is configured to serve documents out of a directory at `/var/www/html`. While this works well for a single site, it can become unwieldy if you are hosting multiple sites. Instead of modifying `/var/www/html`, let’s create a directory structure within `/var/www` for our **your_domain** site, leaving `/var/www/html` in place as the default directory to be served if a client request doesn’t match any other sites.

Create the directory for **your_domain** as follows, using the `p` flag to create any necessary parent directories

```bash
sudo mkdir -p /var/www/your_domain/html
```

Next, assign ownership of the directory with the `$USER` environment variable

```bash
sudo chown -R $USER:$USER /var/www/your_domain/html
```

The permissions of your web roots should be correct if you haven’t modified your umask value, which sets default file permissions. To ensure that your permissions are correct and allow the owner to read, write, and execute the files while granting only read and execute permissions to groups and others, you can input the following command

```bash
sudo chmod -R 755 /var/www/your_domain
```

Next, create a sample index.html page using nano or your favorite editor

```bash
vi /var/www/your_domain/html/index.html
```

Inside, add the following sample HTML

```bash
<html>
    <head>
        <title>Welcome to your_domain!</title>
    </head>
    <body>
        <h1>Success!  The your_domain server block is working!</h1>
    </body>
</html>
```

Save and close the file by typing ESC then :wq when you are finished.

In order for Nginx to serve this content, it’s necessary to create a server block with the correct directives. Instead of modifying the default configuration file directly, let’s make a new one at `/etc/nginx/sites-available/your_domain`

```bash
sudo vi /etc/nginx/sites-available/your_domain
```

Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name

```bash
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Next, let’s enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup

```bash
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled
```

Two server blocks are now enabled and configured to respond to requests based on their `listen` and `server_name` directives (you can read more about how Nginx processes these directives [here](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms)):

- `your_domain`: Will respond to requests for `your_domain` and `www.your_domain`.
- `default`: Will respond to any requests on port 80 that do not match the other two blocks.

Next, test to make sure that there are no syntax errors in any of your Nginx files:

```bash
sudo nginx -t
```

If there aren’t any problems, restart Nginx to enable your changes:

```bash
sudo systemctl restart nginx
```

Nginx should now be serving your domain name. You can test this by navigating to `http://your_domain`, where you should see something like this

![Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled%203.png](Setup%20a%20Fresh%20Server%2010a11ced12b1419ba660063a9dbd158b/Untitled%203.png)

To get more familiar with important Nginx files and directories, please checkout [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04).

Also, checkout [Let's Encrypt tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-20-04) to obtain and install free  TLS/SSL certificates. Summarizing you have

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx
```

## Rstudio Server

You can install R using the following command

```bash
sudo apt-get install r-base
```

To download and install RStudio Server open a terminal window and execute the following commands

```bash
sudo apt-get install gdebi-core
wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.3.1093-amd64.deb
sudo gdebi rstudio-server-1.3.1093-amd64.deb
```

You can check if Rstudio server is working by typing

```bash
sudo systemctl status rstudio-server
```

Once your Rstudio server is up, you may create a route to access your IDE through `your_domain`.

You can do this by adding a location specifying the port where Rstudio server is running. In this case is 8787.

```bash
location /rstudio/ {
                proxy_pass http://127.0.0.1:8787/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }
```

To checkout Rstudio with `nginx`, run `systemctl restart nginx`.

## Shiny Server

While many people primarily turn to the open-source programming language R for statistical and graphics applications, Shiny is an R package that allows you to convert your R code into interactive webpages. And when combined with Shiny Server — available in both a free, open-source and a paid, professional format — you can also host and manage Shiny applications and interactive R markdown documents.

### Installing Shiny

Before installing Shiny Server, you’ll need to install the Shiny R package which provides the framework that Shiny web applications run on.

If you’re familiar with R, you might be tempted to install packages directly from R instead of from the command line. But, using the following command is the safest way to ensure that the package gets installed for all users and not just for the user currently running R.

The `su -` runs the following command as if in the user’s own environment, and the `c` option specifies the command that will be run. That command, in this case, is what follows in double quotes.

The `install.packages` is the R command used to install R packages. So, in this command specifically, the `shiny` package is installed from the specified repository.

```bash
sudo su - -c "R -e \"install.packages('shiny', repos='http://cran.rstudio.com/')\""
```

### Installing Shiny Server

Once you’ve installed R and the Shiny package, execute the following commands in a terminal window to install `gdebi` (which is used to install Shiny Server and all of its dependencies) and Shiny server.

```bash
sudo apt-get install gdebi-core
wget https://download3.rstudio.org/ubuntu-14.04/x86_64/shiny-server-1.5.14.948-amd64.deb
sudo gdebi shiny-server-1.5.14.948-amd64.deb
```

At this point, the output should indicate that a service named `ShinyServer` is both installed and an `active` Systemd service. If the output indicates that there’s a problem, re-trace your previous steps before continuing.

```bash
[Secondary_label Output]
...
● shiny-server.service - ShinyServer
   Loaded: loaded (/etc/systemd/system/shiny-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2017-10-13 14:24:28 UTC; 2 days ago
...
```

Next, verify that Shiny Server is indeed listening on port `3838`.

```bash
sudo netstat -plunt | grep -i shiny
```

If successful, the output will include the following line:

```bash
Output
tcp        0      0 0.0.0.0:3838            0.0.0.0:*               LISTEN      18749/shiny-server
```

If your output doesn’t look like this, double-check your terminal for additional warnings and error messages.

Now, modify the firewall to allow traffic through to Shiny Server.

```bash
sudo ufw allow 3838
```

Finally, point your browser to [`http://www.example.com:3838`](http://www.example.com:3838/) to bring up the default Shiny Server homepage, welcoming you to Shiny Server and congratulating you on your installation.

```bash
location / {
       proxy_pass http://your_server_ip:3838/;
       proxy_redirect http://your_server_ip:3838/ https://$host/;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection $connection_upgrade;
       proxy_read_timeout 20d;
   }
```

Once your Shiny server is up, you may create a route to access your apps through `your_domain`.

You can do this by adding a location specifying the port where Shiny server is running. In this case is 3838.

Now, test your new configuration before activating the changes.

```bash
sudo nginx -t
```

If you run into any problems, follow the instructions in the output to resolve them.

Once your syntax is okay and your test is successful, you’re ready to activate all the changes by reloading Nginx.

```bash
sudo systemctl restart nginx
```

After Nginx has restarted, verify that your Shiny Server is serving requests via HTTPS by pointing your browser to `https://example.com`. You should see the same default Shiny Server homepage.

Then, verify that incoming HTTP requests are redirected to HTTPS by typing `http://example.com` into your browser’s address bar. If it’s working correctly, you should automatically be redirected to `https://example.com`.

Shiny Server is now secured with a reverse proxy and SSL certificate, so you’re ready to configure your setup for interactive R Markdown documents.

### Hosting Interactive R Documents

Shiny Server is useful not only for hosting Shiny applications but also for hosting interactive R Markdown documents.

At this point, you have a working Shiny Server that can host Shiny applications, but it can’t yet host interactive R Markdown documents because the `rmarkdown` R package isn’t installed.

So, using a command that works like the one used for installing the Shiny package, install `rmarkdown`.

```bash
sudo su - -c "R -e \"install.packages('rmarkdown', repos='http://cran.rstudio.com/')\""
```

Then, verify the installation by going to `https://example.com/sample-apps/rmd/`. You should see an interactive R Markdown document in your browser. Additionally, if you return to `https://example.com`, the error message you received earlier should now be replaced with dynamic content.

If you receive an error message, follow the on-screen instructions and review your terminal output for more information.

Your Shiny Server setup is complete, secured, and ready to serve both Shiny applications as well as Interactive R Markdown documents.
