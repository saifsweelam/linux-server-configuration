# Project : Linux Server Configuration

## Description
An Amazon EC3 instance running `Ubuntu` is used as a web server to serve SSH connections via port 2200 and run a website on port 80.

## Instance details
* **IP Adress** : 3.123.4.195
* **URL** : http://ec2-3-123-4-195.eu-central-1.compute.amazonaws.com/ or http://3.123.4.195.xip.io/
* **SSH Port** : 2200

## Logging into SSH instructions
1. Move `grader` & `grader.pub` files to `~/.ssh` directory. (Recommended)
2. Open a terminal program and run the following command:
    ```shell
    $ ssh -i ~/.ssh/grader -p 2200 grader@3.123.4.195
    ```
3. Enter the passphrase _'udacity'_.
4. Now you're logged in as `grader`.

## Software installed
The instance has `Apache2`, `Python3`, `mod_wsgi` library for Apache to support Python3, `PostgreSQL`, `Git`, `ntp` and Python libraries like: (pip, virtualenv, flask, SQLAlchemy, OAuth2Client, passlib)

## Configurations Made
1. Upgrade installed software
    ```shell
    $ sudo apt-get update
    $ sudo apt-get upgrade
    ```
2. Install software mentioned in [the above section](#Software-installed)
    ```shell
    $ sudo apt-get install apache2 ntp git-all ...
    ```
3. Create `catalog` directory inside `/var/www/`
    ```shell
    $ sudo mkdir /var/www/catalog
    $ cd /var/www/catalog
    ```
4. Clone `catalog` repository from `Github` into the newly created directory.
    ```shell
    $ sudo git clone https://github.com/saifsweelam/catalog.git catalog
    ```
5. Change the name of `views.py` file into `__init__.py`.
    ```shell
    $ sudo mv catalog/views.py catalog/__init__.py
    ```
6. Make modifications to `__init__.py` file:
    ```shell
    $ sudo nano catalog/__init__.py
    ```
    Where:
    ```python
    from db_setup import Base, User, Species, Photo
    ```
    Became:
    ```python
    from catalog.db_setup import Base, User, Species, Photo
    ```
    And Declarations for `client_secrets.json` file were modified to `/var/www/catalog/catalog/client_secrets.json`
7. Modify Engines URI in `__init__.py` & `db_setup.py` to use `PostgreSQL` instead of `SQLite`
    ```python
    SQLALCHEMY_DATABASE_URI = "postgresql://animal-photos:udacity@localhost/animalphotos"
    engine = create_engine(SQLALCHEMY_DATABASE_URI)
    ```
8. Modify `password_hash` and `descriprion` fields to support long text in `PostgreSQL` in `db_setup.py` file.
    ```python
    from sqlalchemy import Column, ForeignKey, Integer, String, TEXT
    description = Column(TEXT)
    ```
9. Create a `WSGI` script to run our program.
    ```shell
    $ pwd
    /var/www/catalog
    $ sudo nano catalog.wsgi
    ```
    Content of file:
    ```python
    #!/usr/bin/python3
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'supersecretkey'
    ```
10. Make sure `PostgreSQL` is installed.
    ```shell
    $ sudo apt-get install postgresql postgresql-contrib
    ```
11. Connect to **postgres** user.
    ```shell
    $ sudo -i -u postgres
    ```
12. Create a new role named `animal-photos`
    ```shell
    $ createuser --interactive
    ```
13. Create a database named `animalphotos`
    ```shell
    $ createdb animalphotos
    ```
14. Set password **'udacity'** for the role `animal-photos`
    ```shell
    $ psql
    \password animal-photos
    ```
15. Initialize tables inside database by running `db_setup.py`
    ```shell
    $ cd /var/www/catalog/catalog
    $ python3 db_setup.py
    ```
16. Create a configuration file for Flask App
    ```shell
    $ sudo nano /etc/apache2/sites-available/catalog-app.conf
    ```
    Contents of file:
    ```properties
    <VirtualHost *:80>
        ServerName 3.123.4.195
        ServerAdmin saifsweelam@gmail.com
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
17. Enable the site:
    ```shell
    $ sudo a2ensite catalog
    $ sudo service apache restart
    ```
18. Visit website at [3.123.4.195](http://ec2-3-123-4-195.eu-central-1.compute.amazonaws.com/) and make sure it works.
19. Set timezone to UTC
    ```shell
    $ sudo dpkg-reconfigure tzdata
    ```
20. Sync time using `ntp` which is already installed.
    ```shell
    $ sudo nano /etc/ntp.conf
    ```
21. Create a new user named `grader` and set password: **'udacity'**
    ```shell
    $ sudo adduser grader
    ```
22. Give `grader` sudo premission.
    ```shell
    $ sudo nano /etc/sudoers.d/grader
    ```
    And enter
    ```properties
    grader ALL=(ALL) ALL
    ```
23. Disable root login & disable password login.
    ```shell
    $ sudo nano /etc/ssh/sshd_config
    ```
    Edit `PermitRootLogin prohibt-password` to `PermitRootLogin no`
    And edit `PasswordAuthentication yes` to `PasswordAuthentication no`
24. Change SSH port from the same file:
    *You should first allow port 2200 from aws firewall*
    ```properties
    Port 22 -->> Port 2200
    ```
25. Save file and restart SSH
    ```shell
    $ sudo service ssh restart
    ```
26. Set Up `UFW` firewall
    ```shell
    $ sudo ufw reset
    $ sudo ufw allow http
    $ sudo ufw allow out 53 (To avoid Blocking DNS)
    $ sudo ufw logging on
    $ sudo ufw default deny incoming
    $ sudo ufw default deny outgoing
    $ sudo ufw limit 2200/tcp
    $ sudo ufw enable
    ```


## Resources
* [[RESOLVED] UFW Blocking Apt-Get Update](http://www.tudormateescu.com/ufw-blocking-apt-get-update-resolved/)
* [Linux Server Configuration](https://github.com/AmroYasser/Linux-Server-Configuration)
* [How to run pyhton flask on apache web server](https://www.youtube.com/watch?v=iVGtJOC71Fw)
* [How can I kill all my postgresql connections?](https://stackoverflow.com/questions/5108876/kill-a-postgresql-session-connection)
* [Using PostgreSQL through SQLAlchemy](https://www.compose.com/articles/using-postgresql-through-sqlalchemy/)