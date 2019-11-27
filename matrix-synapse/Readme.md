# How to Install Matrix Synapse Chat on Ubuntu 18.04 LTS

## Step 1 - Update and Upgrade System

Login to your Ubuntu server, update the repository and upgrade all packages using the apt command below.

```sh
sudo apt update
sudo apt upgrade
```
And all ubuntu packages have been upgraded.

## Step 2 - Install Matrix Synapse

In this step, we will install the matrix synapse software using the Debian packages from the official matrix repository.

Add the matrix key and repository by running all commands below.

```sh
wget -qO - https://matrix.org/packages/debian/repo-key.asc | sudo apt-key add -
sudo add-apt-repository https://matrix.org/packages/debian/
```

The command will automatically update the repository.

Install Matrix Synapse

Now install matrix synapse using the apt command as below.

```sh
sudo apt install matrix-synapse -y
```

During the installation, it will ask you about the matrix server name - type the matrix domain name 'matrix.hakase-labs.io'.

Matrix synapse apt installer - part 1

And for the anonymous data report, choose 'No'.

Matrix synapse apt installer - part 1

When the matrix synapse installation is complete, start the service and enable it to launch everytime at system boot.

```sh
sudo systemctl start matrix-synapse
sudo systemctl enable matrix-synapse
```

The matrix synapse is now up and running using the default configuration on port '8008' and '8448'. Check using netstat command.

```sh
netstat -plntu
```

Check open ports



## Step 3 - Configure Matrix Synapse

After the matrix synapse installation, we will configure it to run under the local IP address, disable matrix synapse registration, and enable the registration-shared-secret.

Before editing the home server configuration, we need to generate the shared secret key.

Run the command below.

```sh
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1
```

And you will get the generated key. Copy the result key.

Now we need to edit the home server configuration file 'homeserver.yaml' on the '/etc/matrix-synapse/' directory. Change the current directory to '/etc/matrix-synapse' and edit the configuration file using vim.

```sh
cd /etc/matrix-synapse/
vim homeserver.yaml
Change the HTTP and HTTPS Listener port '8008' and '8448' to the local IP address '127.0.0.1'.

    port: 8448
    bind_addresses:
      - '127.0.0.1'

    - port: 8008
    bind_addresses: ['127.0.0.1']
```

Matrix Synapse configuration

Matrix Synapse port config

Disable the matrix synapse registration, uncomment the 'registration_shared_secret' configuration and paste the secret key generated.

```sh
enable_registration: False
registration_shared_secret: "MtkF9JOkNHsRRISyR5L91KAQlrrPhyWX"
```

Save and exit.


Note:

registration_shared_secret: If set allows registration by anyone who also has the shared secret, even if registration is disabled.

Now restart the matrix synapse services.

```sh
sudo systemctl restart matrix-synapse
```

restart matrix-synapse

Check the homeserver service using the command below.

```sh
netstat -plntu
```

You will get the matrix synapse service is now on the local IP address.

Check Matrix Synapse Ports

And we have completed the matrix synapse installation and configuration.



## Step 4 - Generate SSL Letsencrypt Certificates

In this tutorial, we will enable HTTPS for the Nginx reverse proxy, and we will generate the SSL certificate files from Letsencrypt.

Install the letsencrypt tool using the apt command below.

```sh
sudo apt install letsencrypt -y
```

The Letsencrypt tool is installed on the system, now generate the SSL certificate files for the matrix domain name 'matrix.hakase-labs.io' using the certbot command as shown below.

```sh
certbot certonly --rsa-key-size 2048 --standalone --agree-tos --no-eff-email --email hakaselabs@gmail.com -d matrix.hakase-labs.io
```

The Letsencrypt tool will generate SSL certificate files by running the 'standalone' temporary web server for verification.

And when it's complete, you will get the result as shown below.

Generate SSL Letsencrypt Certificates

SSL certificate files for the matrix synapse domain name 'matrix.hakase-labs.io' is generated inside the '/etc/letsencrypt/live/' directory.



## Step 5 - Install and Configure Nginx as a Reverse Proxy


In this step, we will install the Nginx web server and configure it as a reverse proxy for home server that is running on the port '8008'.

Install the Nginx web server using the apt command below.

```sh
sudo apt install nginx -y
```

After the installation is complete, start the service and enable it to launch everytime at system boot

```sh
sudo systemctl start nginx
sudo systemctl enable nginx
```

Next, we will create a new virtual host configuration for the matrix domain name 'matrix.hakase-labs.io'.

Go to the '/etc/nginx' configuration directory and create a new virtual host file 'matrix'.

```sh
cd /etc/nginx/
vim sites-available/matrix
Paste the following configuration there.

server {
       listen 80;
       server_name matrix.hakase-labs.io;
       return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name matrix.hakase-labs.io;

    ssl_certificate /etc/letsencrypt/live/matrix.hakase-labs.io/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/matrix.hakase-labs.io/privkey.pem;

    # If you don't wanna serve a site, comment this out
    root /var/www/html;
    index index.html index.htm;

    location /_matrix {
      proxy_pass http://127.0.0.1:8008;
      proxy_set_header X-Forwarded-For $remote_addr;
    }
}
```

Save and exit.

Activate the virtual host file and test the configuration.

```sh
ln -s /etc/nginx/sites-available/matrix /etc/nginx/sites-enabled/
nginx -t
```

Make sure there is no error, then restart the Nginx services.

```sh
sudo systemctl restart nginx
```

Nginx installation and configuration as a reverse proxy for the Matrix Synapse homeserver has been completed.





## Step 7 - Create a New Matrix User
At this stage, the matrix synapse homeserver installation and configuration is complete. And in this step, we will show you how to add a new matrix user from the command line server.

To create a new matrix user, run the command below.

```sh
register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml https://127.0.0.1:8448
```

Now you need to input the user name, password, and decide whether the user will have the admin privileges or not.

Below is the result on my system.
