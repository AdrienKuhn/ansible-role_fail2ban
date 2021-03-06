fail2ban
========

This role install and configure fail2ban with some custom filters:

- laravelAuth
- wpAuth
- wpXmlrpc
- nginxAuth
- nginxDos
- nginxLogin
- nginxBadbots
- nginxNoscript
- nginxProxy
- openvpn

All jails are **disabled** by default

Requirements
------------

None

Supported distributions
-----------------------

- Debian:
    - 8 (Jessie)
    - 9 (Stretch)

Role Variables
--------------

```yaml
fail2ban_defaults:
  config:
    ignoreip: 127.0.0.1/8
    bantime: 86400 # 24 hours
    findtime: 86400 # 24 hours
    maxretry: 3
    destemail: root@localhost

  jail:
    recidive:
      filter: recidive
      action: iptables-allports[name=recidive]
      maxretry: 5
      bantime: 604800 # 1 week
      findtime: 86400 # 24 hours
      logpath: /var/log/fail2ban.log

    ssh:
      port: ssh
      filter: sshd
      logpath: /var/log/auth.log

    dropbear:
      port: ssh
      filter: sshd
      logpath: /var/log/dropbear

    sshDdos:
      port: ssh
      filter: sshd-ddos
      maxretry: 6
      logpath: /var/log/auth.log

    #
    # HTTP servers
    #
    laravelAuth:
      filter: laravel-auth
      action: iptables-multiport[name=NoAuthFailures, port="http,https"]
      maxretry: 10
      logpath: /var/log/nginx*/*access*.log

    wpAuth:
      filter: wp-auth
      action: iptables-multiport[name=NoAuthFailures, port="http,https"]
      maxretry: 10
      logpath: /var/log/nginx*/*access*.log

    wpXmlrpc:
      filter: wp-xmlrpc
      action: iptables-multiport[name=NoAuthFailures, port="http,https"]
      maxretry: 1
      logpath: /var/log/nginx*/*access*.log

    nginxAuth:
      filter: nginx-auth
      action: iptables-multiport[name=NoAuthFailures, port="http,https"]
      logpath: /var/log/nginx*/*error*.log

    nginxDos:
      port: http,https
      filter: nginx-dos
      # any IP requesting more than 240 pages in 60 seconds
      # or 4p/s average, is suspicious
      findtime: 60
      bantime: 1800
      maxretry: 240
      logpath: /var/log/nginx/*-access.log

    nginxLogin:
      filter: nginx-login
      action: iptables-multiport[name=NoLoginFailures, port="http,https"]
      logpath: /var/log/nginx*/*access*.log
      bantime: 60
      maxretry: 6

    nginxBadbots:
      filter: apache-badbots
      action: iptables-multiport[name=BadBots, port="http,https"]
      logpath: /var/log/nginx*/*access*.log
      maxretry: 1

    nginxNoscript:
      filter: nginx-noscript
      action: iptables-multiport[name=NoScript, port="http,https"]
      logpath: /var/log/nginx*/*access*.log
      maxretry: 6

    nginxProxy:
      filter: nginx-proxy
      action: iptables-multiport[name=NoProxy, port="http,https"]
      logpath: /var/log/nginx*/*access*.log
      maxretry: 0

    apache:
      port: http,https
      filter: apache-auth
      logpath: /var/log/apache*/*error.log
      maxretry: 6

    apacheMultiport:
      port: http,https
      filter: apache-auth
      logpath: /var/log/apache*/*error.log
      maxretry: 6

    apacheNoscript:
      port: http,https
      filter: apache-noscript
      logpath: /var/log/apache*/*error.log
      maxretry: 6

    apacheOverflows:
      port: http,https
      filter: apache-overflows
      logpath: /var/log/apache*/*error.log
      maxretry: 2

    #
    # OpenVPN
    #
    openvpn:
      port: 1194
      protocol: udp
      filter: openvpn
      logpath: /var/log/syslog
      maxretry: 3
    
# Enable jails
fail2ban:
  jail:
    recidive: false
    ssh: false
    sshDdos: false
    dropbear: false
    laravelAuth: false
    wpAuth: false
    wpXmlrpc: false
    nginxAuth: false
    nginxDos: false
    nginxLogin: false
    nginxBadbots: false
    nginxNoscript: false
    nginxProxy: false
    apache: false
    apacheMultiport: false
    apacheNoscript: false
    apacheOverflows: false
    openvpn: false
```

Dependencies
------------

None

Tests
-----

```
$ cd tests
$ vagrant up --provision
```