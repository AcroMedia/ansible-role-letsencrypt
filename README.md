# Ansible role: letsencrypt

![.github/workflows/molecule.yml](https://github.com/AcroMedia/ansible-role-letsencrypt/workflows/.github/workflows/molecule.yml/badge.svg)

For use on shared hosting servers. The role:
- Installs LetsEncrypt,
- Makes a `/.well-known/acme-challenge` virtual directory available to all virtual hosts on the server (including the default site), so all sites can regsiter and renew LE SSL certificates,
- Overwrites the default site config (after backing up the original), so it can be served with a valid LetsEncrypt certificate instead of the default snakeoil certificate.

As an added bonus, after this role is installed, you won't need to create new virtual hosts to register new LetsEncrypt certificates. Since the default site acts as a catch-all, as long as DNS points at your server, you can register a certificate for that name.

## Requirements

- (NGINX or Apache 2) on (Ubuntu >= 14.04 or CentOS/RedHat >= 6)
- Working DNS: The cert name you're registering must resolve to the machine you're registering the cert from
- A working fully qualified host name: If `hostname -f` on the machine doesn't correctly resolve to the machine from the outside world, you need to either fix it, or override it with one that does resolve with `default_site_fqdn` from your playbook instead.

## Role Variables

- **letsencrypt_webserver**

  The brand of web server that's installed on the server. Defaults to `nginx`. Can be either `nginx` or `apache`.

- **http_port**

  Defaults to `80`. Change it to something else if you're installing Varnish in front of your web service.

- **https_port**

  Defaults to `443`. Change it to something else if you're installing Varnish in front of your web service.

- **letsencrypt_renew_cron_minute**, **letsencrypt_renew_cron_hour**, **letsencrypt_renew_cron_day**

    Control what time the server attempts LE certificate renewal. These default to `5`, `7`, and `*`, respectively (ie. 7:05 AM daily, local server time).

- **letsencrypt_install_certbot_from_ppa**

  Defaults to `false`, is only relevant to Debian/Ubuntu servers, and only exists here for legacy/compatibility purposes. Normally, installation of the self-updating script (the default behaviour) is the best choice in all scenarios.


## Dependencies

None

## Example Playbook

    - hosts: servers
      roles:
        - role: acromedia.letsencrypt
          vars:
            letsencrypt_webserver: nginx
            http_port: 80
            https_port: 443


## License

GPLv3

## Author Information

Acro Media Inc.
https://www.acromedia.com/
