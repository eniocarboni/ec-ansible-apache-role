# Ansible Role: Apache Web Server
[![Build](https://github.com/eniocarboni/ec-ansible-apache-role/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/eniocarboni/ec-ansible-apache-role/actions/workflows/ci.yml) [![GPL License](https://img.shields.io/badge/license-GPL-blue.svg)](https://www.gnu.org/licenses/) [![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.me/EnioCarboni)

An Ansible Role that install, secure Apache Web Server on Ubuntu and EL (RHEL and derived Linux distributions)

## Role Variables

The available variables are listed below, for all references see `defaults/main.yml`

### apache\_mpm

Define the mandatory mpm to use: **event**, **prefork**, **worker**

### apache\_extra\_packages

Extra package for Apache (use **mod_ssl** on RHEL OS to use mod ssl)

### apache\_modules

List of Apache module to enable/disable

```
apache_modules:
  - name: rewrite
    state: present
```

### apache\_vhost\_disable\_default

Boolean value to disable default vhost (es. on Debian/Ubuntu '000-default.conf')

default: **true**

`apache_vhost_disable_default: true`

### apache\_listens

List of apache port to listen on.

Insert any valid value for Apache Listen option (see https://httpd.apache.org/docs/2.4/mod/mpm\_common.html#listen)

default: **80**

```
apache_ssl_listens:
  - 80
```

### apache\_ssl\_listens

List of apache port to listen as SSL on (default 443, for other port use trailing string "https")

Insert any valid value for Apache Listen option (see https://httpd.apache.org/docs/2.4/mod/mpm\_common.html#listen)

default: **443**

```
apache_ssl_listens:
  - 443
```

### apache\_virtualhost

List of vhost where a vhost is a hash with this meanings:

* **state**: if 'present' vhost must be created and enabled while 'absent' the vhost must be disabled and deleted
* **addr**: value after virtualhost (<VirtualHost addr >), default to '\*:80' ;
* **file_suffix**: suffix to add to vhost's filename (ex. "-ssl" or "-port8080")
    to quickly recognize the "virtualhost" from the file name if "ServerName"
    is identical but enabled on a different port (example port 80 and 443)
* **ServerName**: is the 'ServerName';
* **ServerAlias**: is the 'ServerAlias';
* **DocumentRoot**: is the 'DocumentRoot';
* **ServerAdmin**: is the 'ServerAdmin';
* **Directory_DocumentRoot_default**: if true (default) create  a block like this (DocumentRoot: /var/www/html):
    (see below apache_virtualhost_def_directory_options, apache_virtualhost_def_directory_allowoverride)
    <Directory "/var/www/html">
      Options FollowSymLinks
      AllowOverride All
    </Directory>
  use **directory_options** and **directory_allowoverride** to override default value
  For ssl connections the following variables can be entered (**sslengine** must be 'on' to enable the others):
*    **sslengine**, **sslprotocol**, **sslciphersuite**, **sslhonorcipherorder**, **sslsessiontickets**, **ssloptions**, **sslcertificatefile**, **sslcertificatekeyfile**, **sslcertificatechainfile**
*    **sslcertificatefile** and **sslcertificatekeyfile**
     must exists or defalt to value of *apache_virtualhost_def_sslcertificatefile* and *apache_virtualhost_def_sslcertificatekeyfile* defined per OS
*  **ErrorLog**: is the 'ErrorLog'  (def. {{ apache\_log\_dir }}/{ServerName}-errorlog);
*  **CustomLog**: is the 'CustomLog'(def. {{ apache\_log\_dir }}/{ServerName}-access:log);
*  **Extra**: a multiline string of extra options to add to the VirtualHost;
*  **add_rewrite_rule_to_ssl**: if true add the following rules (use **ServerName: host.example.com**)
     # use must enable RewriteEngine in 'apache_modules'
     RewriteCond %{SERVER_NAME} =host.example.com
     RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]

default: **[]**

`apache_virtualhost: []`

### apache\_virtualhost\_def\_directory\_options

Default value for Options in directory

default: `apache_virtualhost_def_directory_options: FollowSymLinks`

### apache\_virtualhost\_def\_directory\_allowoverride

Default *AllowOverride* in directory

default: `apache_virtualhost_def_directory_allowoverride: All`

### apache\_virtualhost\_def\_sslprotocol

Default *sslprotocol* variable if non defined in apache\_virtualhost

default: `apache_virtualhost_def_sslprotocol: 'all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1'`

### apache\_virtualhost\_def\_sslciphersuite

Default *sslciphersuite* variable if non defined in apache\_virtualhost

default: `apache_virtualhost_def_sslciphersuite: 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384'`

### apache\_virtualhost\_def\_sslhonorcipherorder

Default *sslhonorcipherorder* variable if non defined in apache\_virtualhost

default: `apache_virtualhost_def_sslhonorcipherorder: 'off'`

### apache\_virtualhost\_def\_sslsessiontickets

Default *sslsessiontickets* variable if non defined in apache\_virtualhost

default: `apache_virtualhost_def_sslsessiontickets: 'off'`

### apache\_virtualhost\_def\_ssloptions

Default *ssloptions* variable if non defined in apache\_virtualhost

default: `apache_virtualhost_def_ssloptions: '+StrictRequire'`

### apache\_servertokens

See Apache ServerTokens (https://httpd.apache.org/docs/2.4/mod/core.html#servertokens)

default: `apache_servertokens: Prod`

### apache\_serversignature

See Apache ServerSignature (https://httpd.apache.org/docs/2.4/mod/core.html#serversignature)

default: `apache_serversignature: 'Off'`

### firewall

Bool to enable Apache firewall rules only for Ufw or Firewalld

default: `firewall: false`

### firewall\_force\_install

If true install and active **Ufw** or **Firewalld** based on OS

default: `firewall_force_install: false`

### firewall\_ports

List of firewall rules where each rule use the keys: port, proto, from, to, interface, zone

default: `firewall_ports: []`

```
# Example for firewalld:
firewall_ports:
  - port: 80
    proto: tcp
    zone: public
  - port: 443
# Example for Ufw:
firewall_ports:
  - port: "80,443"
    proto: tcp
```

## Example Playbook

Install with:

```
ansible-galaxy install eniocarboni.apache
```

### Example 1: use default variables

```
---
- hosts: all
  become: true

  roles:
    - eniocarboni.apache
```

### Example 2: with use of custom variables

```
---
- hosts: all
  become: true

  roles:
    - role: eniocarboni.apache
      apache_mpm: event
      apache_extra_packages:
        - mod_ssl
      apache_modules:
        - name: rewrite
          state: present
        - name: ssl
          state: present
```

### Example 3: virtualhost

```
- hosts: all
  become: true

  roles:
    - role: eniocarboni.apache
      apache_virtualhost:
        - state: present
          addr: '*:80'
          ServerName: 'host.example.com'
          ServerAlias: 'myhost.example.com'
          DocumentRoot: '/var/www/html'
          ServerAdmin: 'webmaster@host.example.com'
          Directory_DocumentRoot_default: true
          add_rewrite_rule_to_ssl: false
```

## License

GNU General Public License v3.0, see LICENSE file

## Author Information

This role was created in 2023 by Enio Carboni
