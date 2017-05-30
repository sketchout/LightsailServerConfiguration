# LightsailServerConfiguration

  - This project is to setup the Lightsail Hosting Server with Ubuntu OS. 
  - And it will serve the item catalog web service with postgresql.

# Server information

  - IP address : 34.225.110.205
  - SSH port : 2200 
  - complete URL : ec2-34-225-110-205.compute-1.amazonaws.com
  
# Summary of software installed and configuration changed in Ubuntu
  
      1. Update all installed packages
        - sudo apt update
        - sudo apt upgrade

      2. create SSH key pair for ~ubuntu
        - get local id_rsa.pub from .ssh folder
        - touch  ~ubuntu/.ssh/authorized_keys
        - chmod 700 ~ubuntu/.ssh
        - chmod 644 ~ubuntu/.ssh/authorized_keys
        - append id_rsa.pub to ~ubuntu/.ssh/authorized_keys

      2. Change the SSH port from 22 to 2200
        - sudo vi /etc/ssh/sshd_config
        - sudo service ssh restart

      3. Configure UFW(Uncomplicated Firewall)

        - sudo ufw status
        - sudo ufw allow ssh
        - sudo ufw allow 2200/tcp
        - sudo ufw allow www
        - sudo ufw allow ntp
        - sudo ufw enable

        * Remove firewall
          - sudo ufw deny 22
          - sudo delete deny 22

      4. Create a new user account named grader, give grader access
      
        - sudo adduer grader
        - sudo cat /etc/sudoers
        - sudo ls /etc/sudoers.d
        - sudo cp /etc/sudoers.d/ubuntu /etc/sudoers.d/grader
        - sudo vi /etc/sudoers.d/grader 
          :  grader ALL=(ALL) ALL )
          
        * in local machine,  
          - ssh-keygen 
          : save key to /c/Users/admin/.ssh/linuxCourse

        * in server machine, 
          - touch  ~grader/.ssh/authorized_keys
        
        - chmod 700 ~grader/.ssh
        - chmod 644 ~grader/.ssh/authorized_keys
        
        * get public key from local machine, copy to server machine
          : append linuxCourse.pub to ~grader/.ssh/authorized_keys

      5. Configure the local timezone to UTC
      
        - sudo dpkg-reconfigre tzdata 

      6. Install and configure Apache to server a Python mod_wsgi application

        - sudo apt install apache2
        - sudo apt install libapache2-mod-wsgi
        
        - sudo vi /etc/apache2/sites-enabled/000-default.conf
          : WSGIScriptAlias / /var/www/html/myapp.wsgi
          
        - sudo apache2ctl restart

      7. Install Postgresql
      
        - sudo apt install postgresql
        * check the remote connections 
          - sudo vi /etc/postgresql/9.5/main/pg_hba.conf
          
              # Database administrative login by Unix domain socket
              local   all             postgres                                peer

              # TYPE  DATABASE        USER            ADDRESS                 METHOD

              # "local" is for Unix domain socket connections only
              local   all             all                                     peer
              # IPv4 local connections:
              host    all             all             127.0.0.1/32            md5
              # IPv6 local connections:
              host    all             all             ::1/128                 md5

          - sudo service postgresql restart

        - sudo -u postgres psql ( access psql )
        
        * create a new database and a db users named catalogu
        
        - create user catalogu with password 'catalogp'; ( <-> drop user or drop role )
        - alter user catalogu createdb;
        - create database catalogd with owner catalogu; ( <-> drop database )
          : revoke all on schema public from public;  
        - grant all on schema public to catalogu;
                
        - \du ( List of roles of each Role name )
        - \q ( quit psql )
          : psql catalogu -h 127.0.0.1 -d catalogd
          : create table test01 ( a int );
          : \dt ( List of tables )
          
      8. Install python packages 
      
        - sudo apt install python-sqlalchemy
        - sudo apt install python-requests
        - sudo apt install python-oauth2client
        - sudo apt install python-psycopg2
        - sudo apt install python-flask

# Setup Item Catalog

      1. Clone the Item Catalog repository to ~grader
      
        - git clone https://github.com/sketchout/fullstack-nanodegree-vm.git
        - sudo ln -s ~grader/fullstack-nanodegree-vm/vagrant/catalog /var/www/html/catalog
  
      2. Modify engine in /var/www/html/catalog/application.py
      
        : #engine = create_engine('sqlite:///catalog.db')
        : engine = create_engine('postgresql://catalogu:catalogp@localhost/catalogd')

      3. Update json path in /var/www/html/catalog/application.py
      
        : #CLIENT_ID = json.loads(
        : #    open('g_client_secrets.json', 'r').read())['web']['client_id']
        : CLIENT_ID = json.loads(
        :     open('/var/www/html/catalog/g_client_secrets.json', 'r').read())['web']['client_id']
    
      4. Generate a new client_secrets.json download from https://console.developers.google.com
      
        - touch client_secrets.json
        
        * Get facebook "app_id", "app_secret" from https://developers.facebook.com/
        
          - copy fb_client_secrets_example.json to fb_client_secrets.json
          - and update app_id, app_secret to the fb_client_secrets.json
        
        * Update json path in /var/www/html/catalog/application.py
        app_id = json.loads(open('/var/www/html/catalog/fb_client_secrets.json', 'r').read())[
        'web']['app_id']
        app_secret = json.loads(
        open('/var/www/html/catalog/fb_client_secrets.json', 'r').read())['web']['app_secret']

       
      5. Add a new __init__.py to /var/www/html/catalog
      
        : import applicaton import app
      
      6. Create table and add data with below file which should modified with 'create_engine' correctly
      
        - sudo vi application_database_setup.py
        - python application_database_setup.py
        
        - sudo vi application_lotsofitems.py
        - python application_lotsofitems.py
        
      7. Create a WSGI for the catalog project in the /var/www/html/catalog.wsgi
      
        #!/user/bin/python
        import sys
        sys.path.insert(0, "/var/www/html/catalog")

      8. Make a WSGI configuration for the catalog project
      
        - sudo vi /etc/apache2/sites-avaliable/catalog.conf
        
        <VirtualHost *:80>
            ServerName 34.225.110.205
            ServerAdmin sketchout@live.com
            WSGIScriptAlias / /var/www/html/catalog.wsgi
            <Directory /var/www/html/catalog/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/html/catalog/static
            <Directory /var/www/html/catalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

        
      9. Enable catalog site
      
        - sudo a2ensite catalog
      
        * Disable test site 
        
          - sudo a2dissite 000-default
      
      
      10. Restart the web server
      
        - sudo service apache2 restart
