# 辨識語料server討論
[一開始的討論](https://github.com/twgo/tw-on-youtube/issues/24)

# Setup a git lfs server

# Open port 22 before 80
ufw allow 22
ufw enable

Command may disrupt existing ssh connections. Proceed with operation (y|n)? y

ufw status
ufw allow 80



## git http

- skip this if you host reop on github/..
```
sudo apt-get update -y
sudo apt-get upgrade -y

sudo apt-get install nginx git fcgiwrap apache2-utils -y

sudo mkdir /var/www/html/git

sudo chown -R www-data:www-data /var/www/html/git

sudo vi /etc/nginx/sites-available/default

# server...
root /var/www/html/git;
location ~ (/.*) {
    client_max_body_size 0; # Git pushes can be massive, just to make sure nginx doesn't suddenly cut the connection add this.
    auth_basic "Git Login"; # Whatever text will do.
    auth_basic_user_file "/var/www/html/git/htpasswd";
    include /etc/nginx/fastcgi_params; # Include the default fastcgi configs
    fastcgi_param SCRIPT_FILENAME /usr/lib/git-core/git-http-backend; # Tells fastcgi to pass the request to the git http backend executable
    fastcgi_param GIT_HTTP_EXPORT_ALL "";
    fastcgi_param GIT_PROJECT_ROOT /var/www/html/git; # /var/www/git is the location of all of your git repositories.
    fastcgi_param REMOTE_USER $remote_user;
    fastcgi_param PATH_INFO $1; # Takes the capture group from our location directive and gives git that.
    fastcgi_pass  unix:/var/run/fcgiwrap.socket; # Pass the request to fastcgi
}
# server...

sudo nginx -t
sudo htpasswd -c /var/www/html/git/htpasswd lfs
sudo systemctl restart nginx


cd /var/www/html/git
sudo mkdir YOUR_REPO_NAME.git
sudo git --bare init
sudo git update-server-info
sudo chown -R www-data.www-data .
sudo chmod -R 777 .
```
https://www.howtoforge.com/tutorial/install-http-git-server-with-nginx-on-ubuntu-1604/


## git lfs server
### user Nexus OSS 3
https://cloud.tencent.com/developer/article/1010590

ALERT: Use firefox to open Nexus is better then use chrome

### install git lfs
```
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
git lfs install --skip-smudge
```
https://github.com/git-lfs/git-lfs/wiki/Installation

### new lfs repo setup
https://github.com/git-lfs/git-lfs/wiki/Tutorial

```
# .lfsconfig
[lfs]
    url = "http://OSS3_USER:OSS3_PASSWORD@SERVER:HOST/repository/gitlfs-hosted/info/lfs"
```
