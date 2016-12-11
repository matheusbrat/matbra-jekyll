---
layout: post
title: Migrate old Wordpress to Heroku, Amazon RDS and S3. 
lang: en
description: Overview from matbra.com blog migration to Heroku using RDS and S3
tags: amazon heroku wordpres s3 rds composer
comments: true
---

After a few good years with my blog out of date, I decided to start to write again and to migrate it to Heroku since his server was with a really old stack. I decided to use Heroku, Amazon RDS as Database service and S3 as file storage (for uploaded files)

### Steps:

1. Disable all Wordpress' extensions
2. Do a full backup (Wordpress, Database, Uploads, etc) 
3. Really, do a backup! 
4. Create a git repository 
5. Add Wordpress code to your git
    1. If you want to update your Wordpress add [latest Wordpress version](https://wordpress.org/download/) 
        1. **Don't add your private configs (wp-config.php)!**
        2. Don't add **uploads** folder
        3. Add your plugins
        4. Add your theme 
    2. If you want to keep your Wordpress version, add your current blog's code
        1. **Don't add your private configs (wp-config.php)!**
        2. Don't add **uploads** folder
6. Atention with your private files!! 
7. Update your wp-config 
    1. All your **private** configs must use getenv, this function will be responsible to fetch the values from env vars.
    
    {% highlight php %}
    <?php
    define('AUTH_KEY',         getenv('WP_AUTH_KEY'));
    define('SECURE_AUTH_KEY',  getenv('WP_SECURE_AUTH_KEY'));
    define('LOGGED_IN_KEY',    getenv('WP_LOGGED_IN_KEY'));
    define('NONCE_KEY',        getenv('WP_NONCE_KEY'));
    
    define('AUTH_SALT',        getenv('WP_AUTH_SALT'));
    define('SECURE_AUTH_SALT', getenv('WP_SECURE_AUTH_SALT'));
    define('LOGGED_IN_SALT',   getenv('WP_LOGGED_IN_SALT'));
    define('NONCE_SALT',       getenv('WP_NONCE_SALT'));
    
    define('S3_UPLOADS_BUCKET', getenv('AWS_S3_BUCKET'));
    define('S3_UPLOADS_KEY', getenv('AWS_S3_KEY'));
    define('S3_UPLOADS_SECRET', getenv('AWS_S3_SECRET'));
    define('S3_UPLOADS_REGION', getenv('AWS_S3_REGION')); 
    {% endhighlight %}

8. Create a composer.json file to define requirements and packages versions
    1. Exemple composer.json
    {% highlight json %}
    {
      "require" : {
          "php": ">=7.0.0"
      },
      "require-dev": {
      }
    }
    {% endhighlight %}

9. Execute `composer update` to generate the composer.lock file 
10. Update your .htaccess file to redirect your uploads to your S3 bucket
    1. Update the url (at the 5th line) on the .htaccess to match your S3 and Bucket

    ```
    <IfModule mod_rewrite.c>
     RewriteEngine On
     RewriteBase /
     RewriteRule ^index\.php$ - [L]
     RewriteRule ^wp-content/uploads/(.*)$ https://s3-us-west-2.amazonaws.com/BUCKET/uploads/$1 [R=301,L]
     RewriteCond %{REQUEST_FILENAME} !-f
     RewriteCond %{REQUEST_FILENAME} !-d
     RewriteRule . /index.php [L]
     </IfModule>    
     ```

11. Do a commit with all this files (don't add your secrets/keys to your git)
12. [Amazon setup](https://aws.amazon.com/getting-started/)
    1. Create a RDS Database
    2. Import your backup into it
    3. Send your S3 files to S3 
        1. Remember to import/change the permissions of your s3 files so guest users can access your uploaded files
13. [Heroku setup](https://devcenter.heroku.com/articles/getting-started-with-php#introduction)
    1. Add your environment vars on heroku with the correct values and names which you used on `wp-config.php`, remember to use the RDS ones for database.
    2. [Update your DNS for Heroku](https://devcenter.heroku.com/articles/custom-domains)
    3. Send your code to Heroku using the repository you have created 
14. Access your website

If you're updating your Wordpress, there are chances to something go wrong or to some plugin to stop working with the new Wordpress version, so don't forget to check and update them. 

Also, if you have any other question or need more information a specific test, let me know. I can try to help.


Matheus