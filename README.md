# Ansible Role: Certbot (for Let's Encrypt)

Installs and configures Certbot (for Let's Encrypt).
Differences to the original Role (geerlingguy.certbot): this role aims to implement standalone, apache and nginx and deploy_hook feature.

## Requirements

If installing from source, Git is required (i.e. using the `geerlingguy.git` role), but its broken right now.


## Role Variables

By default, this role configures a cron job to run under the provided user account at the given hour and minute, every day. The defaults run `certbot renew` (or `certbot-auto renew`) via the user-crontab of the user ansible was using.

It's preferred that you set a custom user/hour/minute so the renewal is during a low-traffic period and done by a non-root user account, see the defaults.yml and change these variables:

- certbot_auto_renew
- certbot_auto_renew_user
- certbot_auto_renew_hour
- certbot_auto_renew_minute
- certbot_auto_renew_options

### Automatic Certificate Generation


**For a complete example**: see the fully functional test playbook in [molecule/default/playbook-standalone-nginx-aws.yml](molecule/default/playbook-standalone-nginx-aws.yml).

    certbot_create_if_missing: false
    certbot_create_method: standalone

Set `certbot_create_if_missing` to `yes` or `True` to let this role generate certs.

    certbot_admin_email: email@example.com

The email address used to agree to Let's Encrypt's TOS and subscribe to cert-related notifications. This should be customized and set to an email address that you or your organization regularly monitors.

    certbot_certs: []
      # - email: janedoe@example.com
      #   domains:
      #     - example1.com
      #     - example2.com
      # - domains:
      #     - example3.com

A list of domains (and other data) for which certs should be generated. You can add an `email` key to any list item to override the `certbot_admin_email`.

#### Standalone Certificate Generation

    certbot_create_standalone_stop_services:
      - nginx

Services that should be restarted after `certbot` runs.

These services will only be stopped the first time a new cert is generated.


### Snap Installation

Beginning in December 2020, the Certbot maintainers decided to recommend installing Certbot from Snap rather than maintain scripts like `certbot-auto`.

Setting `certbot_install_method: snap` configures this role to install Certbot via Snap.

This install method is currently experimental and may or may not work across all Linux distributions.

#### Webroot Certificate Generation

When using the `webroot` creation method, a `webroot` item has to be provided for every `certbot_certs` item, specifying which directory to use for the authentication. Also, make sure your webserver correctly delivers contents from this directory.

### Source Installation from Git

You can install Certbot from it's Git source repository if desired. This might be useful in several cases, but especially when older distributions don't have Certbot packages available (e.g. CentOS < 7, Ubuntu < 16.10 and Debian < 8).

    certbot_install_from_source: false
    certbot_repo: https://github.com/certbot/certbot.git
    certbot_version: master
    certbot_keep_updated: true

Certbot Git repository options. To install from source, set `certbot_install_from_source` to `yes`. This clones the configured `certbot_repo`, respecting the `certbot_version` setting. If `certbot_keep_updated` is set to `yes`, the repository is updated every time this role runs.

    certbot_dir: /opt/certbot

The directory inside which Certbot will be cloned.

### Wildcard Certificates

Let's Encrypt supports [generating wildcard certificates](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579), but the process for generating and using them is slightly more involved. See comments in [this pull request](https://github.com/geerlingguy/ansible-role-certbot/pull/60#issuecomment-423919284) for an example of how to use this role to maintain wildcard certs.

Michael Porter also has a walkthrough of [Creating A Let’s Encrypt Wildcard Cert With Ansible](https://www.michaelpporter.com/2018/09/creating-a-wildcard-cert-with-ansible/), specifically with Cloudflare.

## Dependencies

None.

## Example Playbook

    - hosts: servers
    
      vars:
        certbot_auto_renew_user: your_username_here
        certbot_auto_renew_minute: 20
        certbot_auto_renew_hour: 5
    
      roles:
        - geerlingguy.certbot

See other examples in the `tests/` directory.

### Manually creating certificates with certbot

_Note: You can have this role automatically generate certificates; see the "Automatic Certificate Generation" documentation above._

You can manually create certificates using the `certbot` (or `certbot-auto`) script (use `letsencrypt` on Ubuntu 16.04, or use `/opt/certbot/certbot-auto` if installing from source/Git. Here are some example commands to configure certificates with Certbot:

    # Automatically add certs for all Apache virtualhosts (use with caution!).
    certbot --apache

    # Generate certs, but don't modify Apache configuration (safer).
    certbot --apache certonly

If you want to fully automate the process of adding a new certificate, but don't want to use this role's built in functionality, you can do so using the command line options to register, accept the terms of service, and then generate a cert using the standalone server:

  1. Make sure any services listening on ports 80 and 443 (Apache, Nginx, Varnish, etc.) are stopped.
  2. Register with something like `certbot register --agree-tos --email [your-email@example.com]`
    - Note: You won't need to do this step in the future, when generating additional certs on the same server.
  3. Generate a cert for a domain whose DNS points to this server: `certbot certonly --noninteractive --standalone -d example.com -d www.example.com`
  4. Re-start whatever was listening on ports 80 and 443 before.
  5. Update your webserver's virtualhost TLS configuration to point at the new certificate (`fullchain.pem`) and private key (`privkey.pem`) Certbot just generated for the domain you passed in the `certbot` command.
  6. Reload or restart your webserver so it uses the new HTTPS virtualhost configuration.

### Certbot certificate auto-renewal

By default, this role adds a cron job that will renew all installed certificates once per day at the hour and minute of your choosing.

You can test the auto-renewal (without actually renewing the cert) with the command:

    /opt/certbot/certbot-auto renew --dry-run

See full documentation and options on the [Certbot website](https://certbot.eff.org/).

## License

MIT / BSD

## Author Information

This role was created in 2016 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
Modified by [Stefan Schwarz](https://www.stefanux.de) in 2019ff.
