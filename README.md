# WordPress Deployment with Ansible

This repository contains Ansible playbooks for automated WordPress deployment with Nginx, PHP-FPM, and MariaDB.

## Prerequisites

### Target Server Requirements
- Ubuntu/Debian-based system
- SSH access with sudo privileges
- MariaDB/MySQL installed and secured (see Manual Setup section)
- Python3 and python3-pymysql installed

### Local Machine Requirements
- Ansible installed (2.9+)
- SSH key configured for server access

## Manual MariaDB Setup (Required)

Before running the playbook, set up MariaDB on the target server:

```bash
# Install MariaDB
sudo apt update
sudo apt install mariadb-server

# Secure the installation
sudo mysql_secure_installation
```

Follow the prompts to:
- Set root password
- Remove anonymous users
- Disallow root login remotely
- Remove test database
- Reload privilege tables

Note: Remember the root password set during this process as it will be needed in the playbook variables.


## Directory Structure

```
.
├── inventory.ini
├── wordpress_install.yml
├── templates/
│   ├── wordpress.conf.j2
│   ├── wp-config.php.j2
│   └── phpadmin.conf.j2
└── README.md
```

### Configuration Files
#### 1. Inventory File
```ini
[wordpress_servers]
staging_server ansible_host=your_server_ip ansible_user=your_user ansible_ssh_private_key_file=path_to_key
```
#### 2. Variables
Edit the variables section in wordpress_install.yml:

```
vars:
    domain_name: your.domain.com
    subdomain: your.subdomain.com
    mariadb_root_password: "your_mariadb_root_password"
    wp_db_name: your_database_name
    wp_db_user: your_database_user
    php_version: "8.1"
    SSL_EMAIL: your.email@domain.com
```

## Features

1. **Web Server Setup**
   - Nginx installation and configuration
   - PHP-FPM installation with necessary extensions
   - Domain-specific configuration
   - www to non-www redirection

2. **WordPress Setup**
   - Automated WordPress download and extraction
   - Domain-specific directory structure
   - Secure file permissions
   - wp-config.php configuration with secure salts

3. **Database Configuration**
   - WordPress database creation
   - Database user creation with appropriate privileges
   - Secure password handling

4. **SSL Configuration**
   - Certbot installation
   - Automatic SSL certificate acquisition
   - HTTPS redirection

## Usage

1. Clone this repository:
```bash
git clone <repository-url>
```

2. Update the inventory file with your server details:

```1:2:inventory.ini
[wordpress_servers]
staging_server ansible_host=your_server_ip ansible_user=your_user ansible_ssh_private_key_file=path_to_key
```


3. Update variables in wordpress_install.yml:

```5:13:wordpress_install.yml
  vars:
    domain_name: prodwp.opscience.org
    subdomain: phpadmin.agi.com.ng
    mariadb_root_password: "TRQNPta0IWM5BpDp"
    wp_db_name: OsiProdDB
    wp_db_user: OsiTestUser
    wp_db_password: "{{ lookup('password', '/dev/null chars=ascii_letters,digits length=16') }}"
    php_version: "8.1"
    SSL_EMAIL: devops@hordanso.com
```


4. Run the playbook:
```bash
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES  # Required for macOS users
ansible-playbook -i inventory.ini wordpress_install.yml
```

## File Permissions

The playbook sets the following permissions:
- WordPress directory: 755
- WordPress files: 640
- wp-config.php: 640
- Owner/Group: www-data:www-data

## Nginx Configuration

The Nginx configuration includes:
- PHP-FPM integration
- Static file caching
- Security headers
- www to non-www redirection
- SSL configuration (when enabled)

See the template at:

```1:46:wordpress.conf.j2
server {
    listen 80;
    server_name {{ domain_name }};
    server_name {{ domain_name }};
    root /var/www/{{ domain_name }};
    index index.php index.html index.htm;

    access_log /var/log/nginx/{{ domain_name }}.access.log;
    error_log /var/log/nginx/{{ domain_name }}.error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php{{ php_version }}-fpm.sock;
    }
        include fastcgi_params;
    location ~ /\.ht {
        deny all;
    }
            deny all;
    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }
        access_log off;
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
        access_log off;
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires max;
        log_not_found off;
    }
}
    }
server {
    listen 80;
    server_name www.{{ domain_name }};
    return 301 http://{{ domain_name }}$request_uri;
}

```


## Troubleshooting

1. **MariaDB Connection Issues**
   - Verify MariaDB is running: `systemctl status mariadb`
   - Check root password is correct
   - Ensure socket file exists: `/var/run/mysqld/mysqld.sock`

2. **PHP-FPM Issues**
   - Verify PHP-FPM is running: `systemctl status php8.1-fpm`
   - Check socket file: `/var/run/php/php8.1-fpm.sock`
   - Review logs: `/var/log/php8.1-fpm.log`

3. **Nginx Issues**
   - Check configuration: `nginx -t`
   - Review logs: `/var/log/nginx/error.log`
   - Verify permissions on web root

## Security Considerations

- Database passwords are handled securely
- File permissions are set restrictively
- SSL certificates are automatically configured
- wp-config.php is protected
- `.htaccess` files are blocked in Nginx

## Maintenance

- SSL certificates will auto-renew via Certbot
- Keep PHP and Nginx updated via system updates
- Regularly backup WordPress files and database
- Monitor logs in `/var/log/nginx/` and `/var/log/php/`

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a new Pull Request

## License

MIT License
