# Ansible role: letsencrypt

This role was designed to work with [acromedia.nginx](https://github.com/AcroMedia/ansible-role-nginx) and [acromedia.nginx](https://github.com/AcroMedia/ansible-role-virtual-host), but it is possible to use it only install certbot if you set your vars as in the example bare-bones playbook below.

Assuming the above scenario is being used, the default behaviour of the role is to:
- Install certbot from LetsEncrypt (snapd version),
- Make a `/.well-known/acme-challenge` virtual directory, which is automatically available to all the virtual hosts on the server (including the default site), so all sites can register and renew LE SSL certificates without extra configuration,
- Overwrite the default site config (after backing up the original), so it can be served with a valid LetsEncrypt certificate instead of the default snakeoil certificate.

After this role is installed using its default configuration, you won't need to create any (additional) virtual hosts to register new LetsEncrypt certificates. Since the default site is already a catch-all, as long as DNS points at your server for the name you need a certificate for, you can register a certificate for it using certbot's `webroot` method.

## Requirements

###### In all cases:
- Snapd + core must already be working
- Working DNS: LetsEncrypt uses DNS validation. Any certificate name you're registering must resolve to the machine you're registering the cert from.

###### When `letsencrypt_webserver` is not `none`:
- (NGINX or Apache 2) on (Ubuntu >= 16.04 or CentOS/RedHat >= 7)

###### When: `letsencrypt_create_default_server_cert` is `true` (the default):
- A working fully qualified host name, OR, that you specify `letsencrypt_default_site_fqdn` as a dns name that does correctly. If `hostname -f` on the machine doesn't correctly resolve to the machine from the outside world, you need to either fix the hostname, or override it with one that does resolve by specifying a valid `default_site_fqdn` from your playbook instead.

## Role Variables

- **letsencrypt_webserver**

  The brand of web server that's installed on the server. Defaults to `nginx`. Can be one of: `nginx`, `apache`, or `none`.

- **http_port**

  Defaults to `80`. Change it to something else if you're installing Varnish in front of your web service.

- **https_port**

  Defaults to `443`. Change it to something else if you're installing Varnish in front of your web service.

- **letsencrypt_renew_cron_minute**, **letsencrypt_renew_cron_hour**, **letsencrypt_renew_cron_day**

    These variable names are a inaccurate now, but they're still used to control what time the server ~~attempts LE certificate renewal~~ reloads your web service. These default to `5`, `7`, and `*`, respectively (ie. 7:05 AM daily, local server time). The snap version of letsencrypt renews certs automatically without the need for a cron job, but your web server will won't pick up the new versions of those certificates without being reloaded.

- **letsencrypt_create_default_server_cert**

  Boolean (`true`) by default. The role will attempt to create a certificate for `$(hostname -f)` unless you set this to `false`.

- **letsencrypt_webroot**

  String, defaults to `/var/www/letsencrypt`. The physical directory for certbot to create acme challenge files in. Each virtual host's `/.well-known/acme-challenge` location maps to `{{ letsencrypt_webroot }}/.well-known/acme-challenge`.

- **letsencrypt_staging**

  Boolean (`false`) by default. If set to `true`, certbot will use the LetsEncrypt staging server instead of the production server. This is useful for testing, as the staging server has much higher rate limits.

## Dependencies

* [acromedia.nginx](https://github.com/AcroMedia/ansible-role-nginx) when `letsencrypt_webserver: nginx` (the default), or
* none, when `letsencrypt_webserver: none`.

## Example Playbooks

#### Normal scenario
```yaml
- hosts: simple_web_servers
  roles:
    - role: acromedia.nginx
    - role: acromedia.letsencrypt
      vars:
        letsencrypt_default_site_fqdn: host-id.example.com
        letsencrypt_webserver: nginx
        http_port: 80
        https_port: 443
    - role: acromedia.virtual-host
      # ... etc etc etc

```

#### **Only** install certbot

```yaml
- hosts: some_other_specialized_group
  roles:
    - role: acromedia.letsencrypt
      vars:
        letsencrypt_webserver: none
        letsencrypt_create_default_server_cert: false
```



## License

GPLv3

## Author Information

Acro Media Inc.
https://www.acromedia.com/
