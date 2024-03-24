# Hosting on Amazon AWS
AWS EC2 is one of the popular options to host a React app. In this article we’ll see how to deploy a react app with ngnix on a Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-1022-aws x86_64) hosted as an AWS EC2 Instance.

*Note: This article assumes that you have a basic knowledge of AWS cloud system and ReactJS. This article also assumes that you have already bought a domain and it is already pointed to your server, the EC2 instance and it is working.

## Install and Configure ngnix on EC2 instance
1. Launch an EC2 instance with latest Ubuntu LTS AMI, Connect to the console from preferred terminal through ssh:
```
ssh <username>@<server-ip> -i <key-name>
```

2. Update Apt and Install nginx
```
sudo apt update
sudo apt install nginx -y
```

3. Install Nodejs
In order to install Nodejs we need curl to get Nodejs downloaded to our server. On a EC2 instance curl comes installed by default. So if `curl --version` does not show you result on your server, install it by running:
```
sudo apt-get install curl
```
and next, install node by running:
```
# installs NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```
and next, update source bash, by running:
```
source .bashrc
```
Test the NodeJs version and npm version, by running :
```
node -v
#AND  
npm --version
```

4. Get your source code, here I use [github](github.com), and I will take the code that I have uploaded there using git, by running:
```
git clone <URL To Your Repository>
```
and next, instal your existing dependencies, by running:
```
npm install
```

5. After that I need to make my file static by running:
```
npm run build
```
Note: 3000 port should be opened in the security group of your EC2 instance, as shown below. You can add an inbound rule to the security group attached to your EC2 instance.

The ReactApp is now running but there is a problem. The problem is, if as you exit the ssh console, the ReactApp will stop. In order to fix this, and keep app running even if we closed or exited the ssh console, we need a process manager which would keep app running all the time, unless we ask it to stop it.

PM2 is a daemon process manager that will help you manage and keep your application online 24/7. Let's install it.

6. Install pm2
```
sudo npm install pm2@latest -g
```

Once pm2 is installed you may want to test some of its basic commands, such as
```
pm2 status
pm2 list 
```
To run our app run the following while being at the app folder, i.e. <Name App>
```
pm2 start yarn --name "Your App" -- start
```

Once App is started, running would yeild `pm2 list`
Now our react application will keep running in the background unless stopped anyhow.

Since we have access to our app at port 3000 we would like it to show it at the default port 80 or our Nginx web server. That also means that it has to show at the very root of our our domain name since we already have pointed our domain to this server's public IP address.

In the next step of this tutorial we are going to see how we can use Nginx as a reverse proxy and divert traffic to port 80 i.e. the default landing page of our domain or public IP.

Create a new site config in `/etc/nginx/sites-available`
```
cd /etc/nginx/sites-available
sudo nano <Name-App> (using dash | underscore | camel case)
```

The last command will open a text file to be edited. Paste the following code into it while replacing xxx.xx.. with your IP address, add domain name with space, use any one of two or both.
```
server {
    listen 80;            //if you have an application run in port 80, you can change it
    listen [::]:80;       //if you change the top one, you need to change this    
    
    server_name xxx.xxx.xxx.xxx;
    access_log /var/log/nginx/reat-tutorial.com.access.log;                
    error_log /var/log/nginx/reat-tutorial.com.error.log;       
    location / {
            proxy_pass http://127.0.0.1:3000;        //you just need to change it
            client_max_body_size 50m;
            client_body_buffer_size 16k;
            proxy_http_version 1.1;                                              
            proxy_set_header Upgrade $http_upgrade;                              
            proxy_set_header Connection 'upgrade';                               
            proxy_set_header Host $host;                                         
            proxy_cache_bypass $http_upgrade;   
    }
}
```

Save and exit the file with `Ctrl + x -> Y -> Enter`

Next, we need to activate this new site by creating a symlink to new site configuration
```
sudo ln -s /etc/nginx/sites-available/<Name-App> /etc/nginx/sites-enabled/
```
Make sure that your nginx configuration syntax is error free
```
sudo nginx -t
```
If an error occurs when you run this code make sure all your code is correct, you can see your code by running:
```
cd  /etc/nginx/sites-enabled/
```
Next, see if the file name is correct or not, by running `ls`
if it does not match then you can delete it, and repeat from step 6. delete file by running:
```
rm "Your App"
```
Restart Nginx
```
sudo systemctl restart nginx
```
You may want to restart your app also:
```
pm2 restart "Your App"
```
If everything goes well you should see your app running at the root domain or your Amazon EC2 instance's public IP address.
