# LightsailServerConfiguration

This project is to setup the Lightsail with UbuntuOS.

# Server information

  - IP address : 34.225.110.205
  - SSH port : 2200 
  - complete URL : 
  
# Summary of software installed & cnd configuration changed
  
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
        - local machine ssh-keygen 
          : /c/Users/admin/.ssh/linuxCourse

        - touch  ~grader/.ssh/authorized_keys
        - chmod 700 ~grader/.ssh
        - chmod 644 ~grader/.ssh/authorized_keys
        - append linuxCourse.pub to ~grader/.ssh/authorized_keys

      5. Configure the local timezoe to UTC

      6. Install and configure Apache to server a Python mod_wsgi application

        - sudo apt install apache2
        - sudo apt install libapache2-mod-wsgi
        - sudo vi /etc/apache2/sites-enabled/000-default.conf
          : WSGIScriptAlias / /var/www/html/myapp.wsgi
        - sudo apache2ctl restart

# 
  
    Please follow these steps to properly submit this project:

    Create a new GitHub repository and add a file named README.md.
    Your README.md file should include all of the following:
    i. The IP address and SSH port so your server can be accessed by the reviewer.
    ii. The complete URL to your hosted web application.
    
    iii. A summary of software you installed and configuration changes made.
    
    iv. A list of any third-party resources you made use of to complete this project.

    Locate the SSH key you created for the grader user.
    During the submission process, paste the contents 
    of the grader user's SSH key into the "Notes to Reviewer" field.
