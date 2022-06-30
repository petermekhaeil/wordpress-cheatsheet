# wordpress-cheatsheet

## Table Of Content

  * [Migrating from cPanel to AWS Lightsail](#migrating-from-cpanel-to-aws-lightsail)
  * [Change site URL](#change-site-url)
  * [Change wp-admin password](#change-wp-admin-password)
  * [Updating Wordpress manually](#updating-wordpress-manually)
  * [Updating Wordpress manually](#updating-wordpress-manually)
  * [Restart server](#restart-server)
  * [Update max file upload limit](#update-max-file-upload-limit)
  * [Upload to upload media - "The uploaded file could not be moved to wp-content/uploads"](#upload-to-upload-media---the-uploaded-file-could-not-be-moved-to-wp-contentuploads)
  * [Renew a Let's Encrypt SSL certificate in a Bitnami stack hosted on a Lightsail instance](https://github.com/petermekhaeil/wordpress-cheatsheet/blob/master/README.md#renew-a-lets-encrypt-ssl-certificate-in-a-bitnami-stack-hosted-on-a-lightsail-instance)
  

## Migrating from cPanel to AWS Lightsail

Replace `ADDRESS` with the IP address of your Lightsail instance.

1. Download a Full Account Backup from cPanel.
  There are two folders we need from the backup: `/public_html` and `/mysql`.
  
2. SSH into Lightsail instance:

```bash
ssh -A -i LightsailDefaultKeyPair-ap-southeast-2.pem bitnami@ADDRESS
```

3. Create a backup folder:

```bash
bitnami:~$ mkdir backup
```

4. Copy across the backup archive:

```bash
scp -i LightsailDefaultKey-ap-southeast-2.pem backup.tar.gz bitnami@ADDRESS:~/backup
```

5. Extract:

```bash
bitnami:~$ cd ~/backup
bitnami:~/backup$ tar -xvf backup.tar.gz
```

6. Copy `public_html` to the wordpress folder:

```bash
sudo rsync --verbose --keep-dirlinks --recursive --progress ~/backup/public_html/ /opt/bitnami/apps/wordpress/htdocs
```

7. Set correct permissions:

```bash
sudo chown bitnami:daemon -R /opt/bitnami/apps/wordpress/htdocs/
sudo chmod g+w -R /opt/bitnami/apps/wordpress/htdocs/
```

8. Get database password:

```bash
bitnami:~$ cat ~/bitnami_credentials
```

9. Log into database:

```bash
bitnami:~$ mysql -u root -p
```

10. Update database:

`DB_NAME`, `DB_USER`, `DB_PASS` are found in `wp-config.php`.

```
CREATE DATABASE DB_NAME;
CREATE USER 'DB_USER'@'localhost' IDENTIFIED BY 'DB_PASS';
GRANT ALL PRIVILEGES ON DB_NAME.* TO 'DB_USER'@'localhost';
USE DB_NAME;
FLUSH PRIVILEGES;
SOURCE /home/bitnami/backup/mysql/database.sql;
EXIT
```

11. Restart

```bash
bitnami:~$ sudo /opt/bitnami/ctlscript.sh restart
```

### Things to verify after a migration:
1. Site address (URL).
2. SMTP Configuration.
3. Administration Email Address.

## Change site URL

wp-config.php:

```php
define('WP_HOME','https://yoursite.com');
define('WP_SITEURL','https://yoursite.com');
```

wp-cli:

```bash
wp option update home 'https://yoursite.com'
wp option update siteurl 'https://yoursite.com'
```

## Change wp-admin password

`DB_NAME` found in wp-config.php. `USER_ID` is a user ID found in the database (most likely `1` for first user created).

```bash
mysql -u root -p DB_NAME -e "SELECT * FROM wp_users;"
mysql -u root -p DB_NAME -e "UPDATE wp_users SET user_pass=MD5('NEWPASSWORD') WHERE ID='USER_ID';"
```

## Updating Wordpress manually

1. Download latest version from [here](https://wordpress.org/download/).
2. Delete the old `wp-includes` and `wp-admin`.
3. Upload new `wp-includes` and `wp-admin`.
4. Upload the individual files from the new `wp-content` folder, overwriting existing files. **Do not** replace the `wp-content` folder.
5. Upload all new loose files from the root directory.

## Restart server

```bash
sudo /opt/bitnami/ctlscript.sh restart
```

## Update max file upload limit

Edit `php.ini`:

```bash
vi /opt/bitnami/php/etc/php.ini
```

And update these values:

```ini
upload_max_filesize = 1000M;
post_max_size = 1000M;
```

Restart server:

```bash
sudo /opt/bitnami/ctlscript.sh restart
```

## Upload to upload media - "The uploaded file could not be moved to wp-content/uploads"

Need to update the file permission for `uploads` directory:

```
sudo chmod g+w -R /opt/bitnami/apps/wordpress/htdocs/wp-content/uploads
```

## Renew a Let's Encrypt SSL certificate in a Bitnami stack hosted on a Lightsail instance

```
sudo /opt/bitnami/ctlscript.sh stop

sudo /opt/bitnami/letsencrypt/lego --tls --email="EMAIL-ADDRESS" --domains="DOMAIN" --path="/opt/bitnami/letsencrypt" renew --days 90

sudo /opt/bitnami/ctlscript.sh start
```

Retrieve previous emails using:

```
sudo ls /opt/bitnami/letsencrypt/accounts/acm*
```

Confirm domain using:

```
sudo /opt/bitnami/letsencrypt/lego --path /opt/bitnami/letsencrypt list
```

https://aws.amazon.com/premiumsupport/knowledge-center/lightsail-bitnami-renew-ssl-certificate/
