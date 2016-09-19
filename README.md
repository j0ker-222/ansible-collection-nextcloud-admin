Still a work in progress.
Some features are missing

**Missing functionalities**: PostgreSQL and certbot

## install_nextcloud

This role installs an _all in one_ nextcloud instance. Database, web services will be on the same host.

Strenghtened permissions and ownership following Nextcloud recommandations.
Strenghtened TLS configuration following _Mozilla SSL Configuration Generator_, intermediate profile.

### Requirements

Ansible 2.0

### Role Variables

Role's variables (and their default value):
#### Main configuration

```YAML
nextcloud_version: 10.0.0
```
The nextcloud version you want to install.
```YAML
nextcloud_trusted_domain: {{ ansible_default_ipv4.address }} 
```
The first domain you will use to access the nextcloud server.
```YAML
nextcloud_websrv: "apache"
```
The http server used by nextcloud. Available values are: **apache** or **nginx**.
```YAML
nextcloud_webroot: "/opt/nextcloud"
```
The nextcloud root directory.

**Warning: only the _parent_ directory must exist prior to installation.**

E.G.: for the default : only **/opt** must exist on the host.
```YAML
nextcloud_data_dir: "{{ nextcloud_webroot }}/data"
```
The nextcloud data directory. This directory will contain all the nextcloud files. Choose wisely.
```YAML
nextcloud_admin_name: "admin"
```
Defines the nextcloud admin's login.
```YAML
nextcloud_admin_pwd: "secret"
```
Defines the nextcloud admin's password.

**Commented by default**

If not defined by the user, a random password will be generated.
```YAML
nextcloud_dl_url: "https://download.nextcloud.com/server/releases" # 
```
The nextcloud repository URL where to download the archive.
#### Database configuration
```YAML
nextcloud_db_backend: "mysql"
```
Database backend used by nextcloud.

Supported values are: 
- mysql
- mariadb
- PostgreSQL

```YAML
nextcloud_db_name: "nextcloud"
```
The nextcloud instance's database name.
```YAML
nextcloud_db_admin: "ncadmin"
```
The nextcloud instance's database user's login
```YAML
nextcloud_db_pwd: "secret"
```
The nextcloud instance's database user's password.

**Commented by default.**

If not defined by the user, a random password will be generated.

#### TLS configuration
```YAML
nextcloud_tls_enforce: true
```
Force http to https.
```YAML
nextcloud_tls_cert_method: "self-signed"
```
Defines various method for retrieving a TLS certificate.
- **self-signed**: generate a _one year_ self-signed certificate for the trusted domain on the remote host and store it in _/etc/ssl_.
- **certbot**: Use _cerbot/letsencrypt_ to provide a signed certificate for the trusted domain.
- **signed**: copy provided signed certificate for the trusted domain to the remote host or in /etc/ssl by default.
  Uses:
```YAML
  # Mandatory:
  nextcloud_tls_src_cert: /local/path/to/cert
  # ^local path to the certificate's key.
  nextcloud_tls_src_cert_key: /local/path/to/cert/key
  # ^local path to the certificate.
  
  # Optional:
  nextcloud_tls_cert: "/etc/ssl/{{ nextcloud_trusted_domain }}.crt"
  # ^remote absolute path to the certificate's key.
  nextcloud_tls_cert_key: "/etc/ssl/{{ nextcloud_trusted_domain }}.key"
  # ^remote absolute path to the certificate.
```
- **installed**: if the certificate for the trusted domain is already on the remote host, specify its location.
  Uses:
```YAML
  nextcloud_tls_cert: /path/to/cert
  # ^remote absolute path to the certificate's key. mandatory
  nextcloud_tls_cert_key: /path/to/cert/key
  # ^remote absolute path to the certificate. mandatory
```

#### System configuration
```YAML
websrv_user: "www-data"
```
system user for the http server
```YAML
websrv_group: "www-data"
```
system group for the http server
```YAML
mysql_root_pwd: "secret"
```
root password for the mysql server

**Commented by default**

If not defined by the user, and mysql/mariadb is installed during the run, a random password will be generated.

#### Generated password
If a password is generated by the role, ansible stores it localy in **nextcloud_instances/{{ nextcloud_trusted_domain }}/** (relative to the working directory)

### Dependencies

none

### Example Playbook
#### Case 1: testing nextcloud
In this case you may want to deploy quickly many instances of Nextcloud on multiple hosts for testing purpose and don't want to tune the role's variables for each hosts: Just run the playbook with all variable by default !

```YAML
---
- hosts: server
  roles:
   - role: install_nextcloud
```

- This will install a nextcloud instance in /opt/nextcloud using apache2 and mysql.
- it will be available at **https://{{ ansible default ipv4 }}**  using a self signed certificate.
- Generated passwords are stored in **nextcloud_instances/{{ nextcloud_trusted_domain }}/** from your working directory.


#### Case 2
- An Ansible master want to install a new nextcloud instance at _cloud.example.tld_ on an existing server with already some services running like mysql and nginx.

- He also have installed a valid certificate for the trusted domain in /etc/nginx/certs/

- He can run the role with the following variables to install nextcloud accordingly to its server existing configuration.

```YAML
---
- hosts: server
  roles:
   - role: install_nextcloud
     nextcloud_trusted_domain: "cloud.example.tld"
     nextcloud_websrv: "nginx"
     nextcloud_admin_pwd: "secret007"
     nextcloud_webroot: "/var/www/nextcloud/"
     nextcloud_data_dir: "/var/ncdata"
     nextcloud_db_pwd: "secretagency"
     nextcloud_tls_cert_method: "installed"
     nextcloud_tls_cert: "/etc/nginx/certs/nextcloud.crt"
     nextcloud_tls_cert_key: "/etc/nginx/certs/nextcloud.key"
     mysql_root_pwd: "42h2g2"
```

License
-------

BSD
