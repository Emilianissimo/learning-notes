# Deploy process for Vue3
### Prerequisites
- Domain and its www subdomain alias -> example.com, www.example.com
- "A" record from domain to server's IP
- Server on ubuntu 20.04 (tested well on it)
### Stack
- Vue3 (JS/TS)
- Node.js (to build)
- Nginx
### Installation
  - Create a new user (if needed)
      - ```
        sudo adduser fastapi-user # replace fastapi-user with your preferred name
        sudo gpasswd -a fastapi-user sudo # add to sudoers
        su - fastapi-user # login as fastapi-user 
        ```
      - Install node 18.2.0 or newer (using nvm)
        - ```
          curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh
          curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
          source ~/.bashrc
          nvm install v19.2.0
          sudo apt install npm
          ```
      - Install nginx
        - ```
          sudo apt install nginx -y
          sudo ufw allow 'Nginx Full' # for ssl needs
          ```
      - Set your project into ```/var/www/project_folder```. If folders are not exist, just create them.
      - run ```npm install```
      - Build the project (if there any problem, END_OF_LIFECYCLE, for example, update node version to the latest):
        - ```
          npm run build
          ```
      - You may get response: Killed. That means, system ran out of memory, let's increase swap file:
        - ```
          sudo dd if=/dev/zero of=/swapfile bs=1024 count=1024k
          sudo mkswap /swapfile
          sudo swapon /swapfile
          echo 10 | sudo tee /proc/sys/vm/swappiness echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf
          sudo chown root:root /swapfile
          sudo chmod 0600 /swapfile
          ```
          And run build again.
      - nginx config:
        - Create new file for domain into the sites-available folder -> ```sudo nano /etc/nginx/sites-available/example.com```
        - ```nginx
           server {
            listen      80;
            server_name example.com www.example.com;    
            charset utf-8;
            root    /var/www/project_folder/dist;
            index   index.html;
            #Always serve index.html for any request
            location / {
             root /var/www/project_folder/dist;
             try_files $uri  /index.html;
            }    
            error_log  /var/www/project_folder/vue-app-error.log;
            access_log /var/www/project_folder/vue-app-access.log;
           }
          ```
          - make symbolic link: ```sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/```
          - don't forget to delete ```default``` file from ```sites-available``` in case of it could be a mess with conflicting domain names for root access
          - check and restart ```sudo nginx -t && sudo systemctl restart nginx```
          - if you're getting an error, that you should increase ```server_names_hash_bucket_size```, than just add it at the top of your file, like so:
            - ```
              server_names_hash_bucket_size 128;
      
              upstream app_server {
              ...
              ```
            - It may not work and require http if it is not Debian kind system, so just wrap it into the http {} block. (in Debian you are already in the http {} by nginx.conf)
          
# Setting up SSL
We're gonna use certbot for our needs:
```
  sudo snap install --classic certbot
  sudo ln -s /snap/bin/certbot /usr/bin/certbot
  sudo certbot --nginx -d example.com -d www.example.com
```
