### Connect to the server ###
1) On the Windows terminal (or Mac), connect to the Debian server by typing the following
```
ssh -i [ssh-key-location] root@[ip-address]
```
The `ssh-key-location` is the location of your ssh key in your computer's directory and the `ip-address` is the ip address generated from your digital ocean droplet. For example: `ssh -i .ssh/sample-key root@137.184.116.223`

2) Type `yes` when prompted to continue connecting


### Create a new user ###
1) Create a new user by typing the following
```
useradd -ms /bin/bash [username]
```
This will allow you to create a new user account with a home directory and sets the path to the user's log in shell.

2) Give your account a password. Type
```
passwd [username]
```
You will be prompted to enter your password twice. Creating a password for your account will make your account secure.

3) Add your account to the 'sudo' group
```
usermod -aG sudo [username]
```
This will make your account part of the 'sudo' group allowing you to perform tasks with elevated privileges.

4) You can now login to your account using the command
```
su -l [username]
```

### Allow the new user to access the server via SSH

1) Copy the .ssh directory from the root to your account
```
sudo cp -r /root/.ssh /home/[username]
```
Then change the owner of that directory to your account
```
sudo chown -R [username]:[groupname] /home/[username]/.ssh
```
This will make it possible for your new account to connect to server via ssh.

2) Test if you can connect to the server via ssh. After logging out of the server, type in
```
ssh -i [ssh-key-location] [username]@[ip-address]
```

### Prevent the root user from connecting to the server via ssh ###
1) Log in to the server as the root user or your user account

2) Edit the sshd_config file using
```
sudo vim /etc/ssh/sshd_config
```
This file contains the ssh configuration file to edit login permits.

3) In vim, press `i` in your keyboard to go to `insert` mode and scroll down until you find the line that says `PermitRootLogin yes`. Change `yes` to `no`.

4) Save the file by pressing `ESC` in your keyboard to go back to `visual` mode, then press `:wq` to save your changes.

5) Restart the ssh service by typing
```
sudo systemctl restart ssh.service
```
This will apply the changes you made on the ssh configuration file.

6) Test if you can connect to the server as a root user. After logging out of the server, type in
```
ssh -i [ssh-key-location] root@[ip-address]
```
You should get `Permission denied` when you try to log in as the root user.

### Configure nginx to serve a sample website ###
1) Log in to your user account

2) Upgrade all installed packages in your system. Type
```
sudo apt update
```
This command will refresh the list of available packages from your software repositories. Then type
```
sudo apt upgrade
```
This will upgrade all installed packages to their latest versions and ensure all your packages are up-to date.

3) Install nginx.
```
sudo apt install nginx
```
After installing, your web server should be up and running. If you want to double check if the web service is running, type
```
sudo systemctl status nginx.service
```
It should say that the service is running. But if not, type
```
sudo systemctl start nginx.service
```
This will start the service immediately. Then type,
```
sudo systemctl enable nginx.service
```
This will start the service automatically at boot time. Then type,
```
sudo systemctl daemon-reload
```
This will reload the systemd manager configuration and will apply all the changes you've made on service configuration files.

4) Create a new directory in `/var/www`directory and name it however you want. For example, a directory named `my-site` will be 
```
sudo mkdir /var/www/my-site
```
This will create the directory to hold the documents served by nginx.

5) Create a new html file in the `my-site` directory and name it `index.html`
```
sudo touch /var/www/my-site/index.html
```
This will create your html file that contains the code you want to display in your webpage.

6) Insert the html code below in your `index.html`
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```
This is a sample html code that will allow us to confirm that our sample webpage is being served by nginx with no issues.

7) Create an nginx configuration file in `/etc/nginx/sites-available`. Preferrably the same name as the directory you made to hold the documents served by nginx.
```
sudo touch /etc/nginx/sites-available/my-site.conf
```
This file will contain the server configuration that are **NOT** enabled yet.

8) Insert a default server configuration to `my-site.conf` just to see if the server runs without any issues. Type
```
sudo vim /etc/nginx/sites-available/my-site.conf
```
Inside the my-site.conf, insert the text below using vim:
```
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	
	root /var/www/my-site;
	
	index index.html index.htm index.nginx-debian.html;
	
	server_name _;
	
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}
```
In the code above, ensure that the line that says `root /var/www/my-site;` matches the directory where your `index.html` is in.

9) Create a symbolic link to your new nginx config file in `/etc/nginx/sites-enabled`.
```
sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled/my-site.conf@
```
Creating a symbolic link to `sites-enabled` will create a reference to the file in `sites-available` which will enable the file to be served by nginx. 

10) Once you have a symbolic link in sites-enabled, run the following command
```
sudo nginx -t
```
This will ensure if there are no issues with your nginx configurations.

10) Restart the nginx service
```
 sudo systemctl restart nginx
```
This will apply the changes that you made on the configuration file.

11) Confirm that the server is running the html file you created in step #6. Use the ipaddress for your Debian server.
```
curl [ip address]
```

Congratulations! You have successfully configure nginx to serve a sample website.
