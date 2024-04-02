# Deploy process for FastApi + Alembic migrations
### Prerequisites
- Domain and its www subdomain alias -> example.com, www.example.com
- "A" record from domain to server's IP
- Server on ubuntu 20.04 (tested well on it)
### Stack
- FastApi (Python)
- PostgreSQL
- Nginx, Gunicorn, Uvicorn, Supervisor
### Installation
  - Create a new user (if needed)
    - ```
      sudo adduser fastapi-user # replace fastapi-user with your preferred name
      sudo gpasswd -a fastapi-user sudo # add to sudoers
      su - fastapi-user # login as fastapi-user 
      ```
  - Install python 11 or newer
    - ```
      sudo add-apt-repository ppa:deadsnakes/ppa
      sudo apt update
      sudo apt install python3.11 python3.11-venv -y
      ```
  - Install nginx + supervisor
    - ```
      sudo apt install supervisor nginx -y
      sudo systemctl enable supervisor
      sudo systemctl start supervisor
      sudo ufw allow 'Nginx Full' # for ssl needs
      ```
  - Install PostgreSQL
    - Database should persist and run in other case you'll have ```socket.gaierror: [Errno -3] Temporary failure in name resolution```
    - ```
        sudo apt-get update
        sudo apt install postgresql postgresql-contrib
        sudo systemctl start postgresql.service
      ```
  - Change password for postgres user and create database
    - ```
        sudo -i -u postgres
        psql
        -> \password
        -> \q
        createdb YOUR_DB_NAME
      ```
  - Set your project into ```/var/www/project_folder```. If folders are not exist, just create them.
  - Create venv with your python version 
    - ```python3.11 -m venv venv && source venv/bin/activate```
  - Switch alembic.ini connection to prod actual and the same for .env db link
  - Install gunicorn, uvicorn (requirements.txt)
  - Check the runtime: ```uvicorn app.main:app``` (my projects are stored into the app folder and main inside, that's why to do not break namespaces we need to call it from project root)
  - We have to have gunicorn_start (custom name) file, which is config for server, in the project root, if it is not persist:
    - ```
      #!/bin/bash
    
      NAME=fastapi-app
      DIR=/var/www/project_folder
      USER=fastapi-user
      GROUP=fastapi-user
      WORKERS=3
      WORKER_CLASS=uvicorn.workers.UvicornWorker
      VENV=/var/www/project_folder/venv/bin/activate
      LOG_LEVEL=error
    
      cd $DIR # important to run app.main:app
      source $VENV
    
      exec gunicorn app.main:app \
        --name $NAME \
        --workers $WORKERS \
        --worker-class $WORKER_CLASS \
        --user=$USER \
        --group=$GROUP \
        --bind=unix:/var/www/project_folder/run/gunicorn.sock \
        --log-level=$LOG_LEVEL \
        --log-file=-
      ```
      - For some reason, several internal variables, like $BIND (--bind param hardcoded) aren't work and socket ain't run, so I hardcoded it, you may do whatever you want it is just running gunicorn command, which will handled by supervisor. It runs socket, workers count is CPU_COUNT + 1
  - Then ```chmod u+x gunicorn_start```, we need to give permission to file to be run (executed) by user
  - Project root must have ```run``` and ```logs``` folders
    - If they are not exist: ```mkdir run && mkdir logs```
  - The run folder will have gunicorn.sock for bind param in previous file, and if permissions are right (you made everything according to this text), it will be generated meanwhile runtime
  - Configure supervisor to run this daemon
    - Logs folder should persist
    - Create new file ```sudo nano /etc/supervisor/conf.d/fastapi-app.conf```
    - Add this:
    - ```
      [program:fastapi-app]
      command=sh /var/www/project_folder/gunicorn_start # sh couldn't work for some reason, use bash instead
      user=fastapi-user
      autostart=true
      autorestart=true
      redirect_stderr=true
      startsecs = 0
      stdout_logfile=/var/www/project_folder/logs/gunicorn-error.log
      ```
    - Make ```sudo supervisorctl reread && sudo supervisorctl update``` to parse new file and update runtime
    - Check it ```sudo supervisorctl status fastapi-app```
    - Check this out by ```curl --unix-socket /var/www/project_folder/run/gunicorn.sock localhost```
  - nginx config:
    - Create new file for domain into the sites-available folder -> ```sudo nano /etc/nginx/sites-available/example.com```
    - ```nginx
       upstream app_server {
          # our file prom the project_root
          server unix:/var/www/project_folder/run/gunicorn.sock fail_timeout=0;
       }
       
       server {
           listen 80;
     
           # add here the ip address of your server
           # or a domain pointing to that ip (like example.com or www.example.com)
           server_name example.com www.example.com;
    
           keepalive_timeout 5;
           client_max_body_size 4G;
    
           access_log /var/www/backend/logs/nginx-access.log;
           error_log /var/www/backend/logs/nginx-error.log;
    
           location / {
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header Host $http_host;
               proxy_redirect off;
      
               # WebSocket WSS support (SSL)
               proxy_http_version 1.1;
               proxy_set_header Upgrade $http_upgrade;
               proxy_set_header Connection "upgrade";
                            
               if (!-f $request_filename) {
                   proxy_pass http://app_server;
                   break;
               }
           }
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
Also you would have troubles with Jinja generated endpoints, so you may do this:
```python
    from typing import Any
    from jinja2 import pass_context
    
    
    @pass_context
    def https_url_for(context: dict, name: str, **path_params: Any) -> str:
    
        http_url = context["request"].url_for(name, **path_params)
    
        # Replace 'http' with 'https'
        return http_url.replace("http", "https", 1)

    
    template.env.globals["https_url_for"] = https_url_for
```
# After redeploy
Each redeploy, should rerun socket:
```
  sudo chmod u+x gunicorn_start # to make it executable by user
  sudo supervisorctl restart fastapi-app
```

# Requirements list
```requrements
aiosmtplib==2.0.1
alembic==1.10.4
anyio==3.6.2
asyncpg==0.27.0
blinker==1.5
click==8.1.3
databases==0.5.5
dnspython==2.3.0
email-validator==1.3.1
fastapi==0.110.0
fastapi-mail
greenlet==2.0.2
h11==0.14.0
humanize==4.9.0
idna==3.4
Jinja2==3.1.3
MarkupSafe==2.1.2
ormar==0.12.1
passlib==1.7.4
psycopg2-binary==2.9.5
pydantic==1.10.4
pydantic[dotenv]
PyJWT==2.6.0
python-dateutil==2.8.2
python-multipart==0.0.9
python-slugify==8.0.1
pytz==2023.3
sniffio==1.3.0
SQLAlchemy==1.4.37
# sqlalchemy-migrate=0.13.0
typing_extensions==4.8.0
gunicorn==21.2.0
uvicorn==0.20.0
websockets==10.4
```
