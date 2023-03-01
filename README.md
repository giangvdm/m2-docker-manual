# Magento 2 on Docker Setup

## Disclaimers

- This is the Manual Version.
- This work should only be used internally within AHT Tech JSC, Vietnam.

## Overview

- Magento Version: 2.4.5-p1.
- Preferred Operating System: Linux Ubuntu (20.04 and up).
- [System Requirements](https://experienceleague.adobe.com/docs/commerce-operations/installation-guide/system-requirements.html)

## Instructions

### Setup Environment

#### 1. Install PHP and required extensions

Execute this command:

```Bash
sudo apt install php8.1 [libapache2-mod-php8.1] php8.1-mysql php8.1-soap php8.1-bcmath php8.1-xml php8.1-mbstring php8.1-gd php8.1-common php8.1-cli php8.1-curl php8.1-intl php8.1-zip zip unzip -y
```

#### 2. Install Composer (globally)

Refer to [this guide](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-composer-on-ubuntu-20-04) specifically for Ubuntu 20.04.

#### 3. Install Docker and Docker Compose

It is recommended to install Docker CLI rather than Docker Desktop on Linux OSes.

[Docker Installation Guide](https://docs.docker.com/engine/install/ubuntu/)

[Docker Post-installtion Guide](https://docs.docker.com/engine/install/linux-postinstall/)

[Docker Compose Installation Guide](https://docs.docker.com/compose/install/linux/#install-using-the-repository)

### Setup Application

#### 1. Clone M2 Docker Manual Project

```Bash
git clone git@github.com:giangvdm/m2-docker-manual.git
```

Rename the cloned directory as desired.

#### 2. Configure the Application

Change the permission of all files in application:

```Bash
chmod 777 -R application_dir_name
```

Change directory to the application root:

```Bash
cd application_dir_name
```

Grant proper permissions to files and folders inside application:

```Bash
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} + && find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} + && chown -R www-data:www-data . && chmod u+x bin/magento
```

Open the `docker-compose.yml` file, and make modifications as suggested below:

##### a. Web service (`web`)

Under `environment` section:

- Change the `WEB_ALIAS_DOMAIN` variable to your custom domain.
- Add other PHP configurations if needed.

Under `port` section:

- If port `80` on your local machine has already been used, change it to another port, for example, `81`, then update it like so: `- 81:80`.
- The same goes with port `443` and `32823`.

##### b. Database service (`mysql`)

Under `environment` section:

- Change the root user password at `MYSQL_ROOT_PASSWORD` variable.

Configure `port` mapping at will.

##### c. Phpmyadmin service (`phpmyadmin`)

Under `environment` section:

- Change the root user password at `MYSQL_ROOT_PASSWORD` variable to match with the password configured for `mysql` service as above.

Configure `port` mapping at will.

##### d. Elasticsearch service (`elasticsearch`)

N/A

#### 3. Bring the Application up

Inside the application root, execute this command:

```Bash
docker-compose up -d --build
```

Wait for the process to complete.

#### 4. Install Magento 2

Inside application root, execute this command to start using Bash inside `web` service container:

```Bash
docker exec -it web bash
```

Change directory to the root of the application inside `web` container (by default it is `app`):

```Bash
cd app
```

Execute this script to start Magento 2 installation:

```Bash
php bin/magento setup:install \
--admin-firstname=Foo \
--admin-lastname=Bar \
--admin-email=webmaster@m2ce.local \
--admin-user=admin \
--admin-password=admin123 \
--base-url=http://docker.m2ce.local \
--base-url-secure=https://docker.m2ce.local \
--backend-frontname=admin \
--db-host=mysql \
--db-name=magento \
--db-user=root \
--db-password=ub9M3FnHb6R3ykbk \
--use-rewrites=1 \
--language=en_US \
--currency=USD \
--timezone=America/Chicago \
--use-secure-admin=0 \
--admin-use-security-key=1 \
--session-save=files \
--elasticsearch-host=elasticsearch
```

Also remember to make any modifications needed to the script before executing.

Wait for the process to complete.

(Optional) Deploy static content

```Bash
php bin/magento sample-data:deploy
```

Deploy the application:

```Bash
php bin/magento setup:upgrade && \
php bin/magento setup:di:compile && \
php bin/magento setup:static-content:deploy -f;
```

### 4. Start using the Application

Add your defined domain to the system `hosts` file:

```Bash
sudo -- sh -c "echo '127.0.0.1 docker.m2ce.local' >> /etc/hosts"
```

Access the Storefront on your browser at: <http://docker.m2.local:80>.

Access the Admin Panel on your browser at: <http://docker.m2.local:80/admin>

Access Phpmyadmin on your browser at: <http://localhost:8080>.

## Troubleshoot

### Resources failed to load in Storefront

After installation, if you access the Storefront and see a broken page, chances are the application URLs are not configured correctly.

It may fall into one of these cases:

- Web service (`web`) port was mapped to another one rather than `80` (by default), but the new port was not included in the base URL.
  - **Solution**:
    - *Before installation*: Remember to include your port in the installation script as well, i.e. when you specify the base URL for the application at `--base-url=http://docker.m2ce.local:<port>` and `--base-url=https://docker.m2ce.local:<port>`.
    - *After installation*: Access the database, in `core_config_data` table, update the value of records with path resembling `secure/base_url` to include your port as well.

## Appendix

### Useful Commands

Use Bash inside `web` container:

```Bash
docker exec -it web bash
```

Use Bash inside `mysql` container:

```Bash
docker exec mysql 'mysql -u root -p'
```

Stop the application:

```Bash
docker-compose down
```

### References

- [How to Install Magento 2.4.3 + elasticsearch using docker – Ubuntu 20.04](https://www.vengalavinay.com/how-to-install-magento-2-4-3-elasticsearch-using-docker-ubuntu-20-04/)

