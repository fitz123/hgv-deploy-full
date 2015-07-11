# Mercury Vagrant (HGV) Deployment Playbook (with MariaDB)

## Introduction

This Ansible Playbook is designed to setup a [Mercury-Like](https://github.com/wpengine/hgv/) environment on a Production server without the configuration hassle. This playbook was forked from [WPEngine's Mercury Vagrant](https://github.com/wpengine/hgv/). It includes the ability to install multiple hostnames and installs of WordPress on one server super easily.

*Note: Remember not to run weird scripts on your server as root without reviewing them first. Please review this playbook to ensure I'm not installing malicious software.*

This Playbook will setup:

- **MariaDB 10.0** (Data Base)
- **HHVM** (Default PHP Parser)
- **PHP-FPM** (Backup PHP Parser)
- **Nginx**
- **Varnish** (Running by default)
- **Memcached and APC**
- **Clean WordPress Install** (Latest Version)
- **WP-CLI**

#### This playbook tested only run on Digital Ocean Ubuntu 14.04 LTS

## Installation

1. SSH onto a newly created server 1.5. Add necessary Apt package (if not already installed) with `sudo apt-get install software-properties-common -y`
2. Add Ansible with `sudo add-apt-repository ppa:ansible/ansible`
3. Update Apt with `sudo apt-get update && sudo apt-get upgrade -y`
4. Install Git and Ansible with `sudo apt-get install ansible git -y`
5. Clone this repository with `git clone https://github.com/fitz123/hgv-deploy-full.git`
6. Move into `cd hgv-deploy-full`
7. Edit the `hosts` file and change `my.wordpress.com` to your host name. If you have more than one website that you want to install on this server add each on a new line.
8. Edit the name of `my.wordpress.com` file in the `host_vars` folder to your hostname. If you have more than one website that you want to install on this server copy the current one and name it the hostname of the website.
9. Change your sites specific information **including passwords** inside the hostname file inside the `host_vars` directory
10. Run Ansible with `sudo ansible-playbook -i hosts -u root playbook.yml`. If you have any errors please open a new issue in this repository.
11. Remove the cloned git directory from your server with `rm -rf hgv-deploy-full/`
12. You're good to go! A new WordPress install running HHVM and Varnish should be waiting for you at your hostname/s!

## Installing a New Website/Hostname

*This only works on sites that were installed using the method above. Always backup your server before running code that could break it.*

1. Backup your server
2. Follow steps 1-6 above
3. When you come to your `hosts` file follow the same steps however **do not include any previous installations of WordPress or hostnames, only list the new ones you want.**
4. Likewise with your `host_var` folder
5. Follow steps 9-12 and if you run into any issues or errors post them in this repository!

## Turning off Varnish (Use only Nginx)

If you are having issues making changes or having issues with the backend while using Varnish, you can turn it off and just use Nginx while maintaining good performance. Here's how you can do that:

1. Open each of the Nginx configurations of the sites installed on your server with `sudo nano /etc/nginx/sites-available/your-hostname.com`
2. Change `listen = 8080;` to `listen = 80;` 
3. Make sure you do this to **all** sites installed on your server
4. Stop Varnish and Restart Nginx with `sudo service varnish stop && sudo service nginx restart`
5. You should be good to go! If you do not have a caching plugin installed I would highly recommend one.

## Switching HHVM back to PHP-FPM

Your Nginx configuration should automatically facilitate switching to PHP-FPM if there's an issue with HHVM, however if you want to switch back manually you can do so like this:

1. Open your Nginx configuration with `vim|emacs|nano /etc/nginx/sites-available/( Your Hostname )`
2. Find the following section towards the bottom:

```
    location ~ \.php$ {
        proxy_intercept_errors on;
        error_page 500 501 502 503 = @fallback;
        fastcgi_buffers 8 256k;
        fastcgi_buffer_size 128k;
        fastcgi_intercept_errors on;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass hhvm;
    }
```

3. Change `fastcgi_pass hhvm;` to `fastcgi_pass php;`
4. Restart Nginx with `sudo service nginx restart`
5. You should now be running PHP-FPM! Check to make sure using `phpinfo();`

## Issues

Please report any issues through GitHub and I'll do my best to get back to you!
