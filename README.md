mi-poop-mail
==========

This is a machine image for an e-mail server based on Exim, Dovecot, Clamav and Spamassassin. It is
suitable mostly for low to medium-volume use, such as for personal use or a small business.


Please refer to https://github.com/joyent/mibe for use of this repo.

Use with base-64 image from Joyent; tested with 15.3.0.

Metadata
--------
The following customer_metadata is used:

dovecot:qualify_domain - Domain to use as the authentication realm if the client presents none (default: none)
dovecot:primary_hostname - Hostname to use for TLS (default: system hostname)
exim:qualify_domain: - Domain to append to addresses when only a local_part is given
system:ssh_disabled - Whether or not to disable the ssh daemon (default: false)
system:timezone - What timezone to use

Services
--------
When running, the following services are exposed to the network on both IPv4 and IPv6:

* 25, 587: SMTP (tls required for authentication)
* 143, 993: IMAP (tls required)
* 4190: ManageSieve (tls required)
* 22: SSH (if not disabled)

Data
----
This image requires a delegated dataset. It will be mounted at /srv/mail and will store user
mailboxes. Also on this dataset is the user information:

* /srv/mail/passwd/<domain>: users per domain, formatted as user:passwd where passwd can be had using 'doveadm pw'
* /srv/mail/aliases/<domain>: aliases per domain, formatted as user:destination

TLS certificates live in /srv/mail/ssl/<domain>/current, with the following files being used:

* ca: the certificate chain from the CA
* cert: the certificate
* key: the private key
* chained: the certificate + the chain (cat cert ca > chained)
* req: the certificate request

'current' can be a symlink so upon rollover you can prepare a new directory and switch the symlink to activate it.

Multiple certificate directories can exist and exim will pick between them based on the client's SNI info if available.
For clients that don't send SNI info, the server hostname is used. Dovecot currently only uses the cert given by the
dovecot:primary_hostname setting.

Exim keeps its spool in /srv/mail/exim/spool so it is preserved upon reprovision. Also there are some configuration
lists in /srv/mail/exim/conf:

* global_aliases: aliases that exist in *all* handled domains. Suggested use is for a 'postmaster' alias.
* rewrite_domains: domains to rewrite to another domain, as domain: newdomain
* relay_from_hosts: hosts to relay for without authentication
* relay_to_domains: domains to be a secondary mx for, or otherwise relay to without authentication
* senderverify_exceptions: e-mail addresses to accept (on 'MAIL FROM:') without doing a verification callout

Dovecot allows local configuration in /srv/mail/dovecot/conf/local.conf. This can be used for example to add an
imapc config to migrate mail from another server. There is a place to store global sieve scripts in 
/srv/mail/dovecot/sieve/before and .../after, which will be executed before and after the user's own scripts
respectively. 

Example JSON
------------
{
  "brand": "joyent",
  "image_uuid": "",
  "alias": "poopmail",
  "hostname": "poopmail",
  "dns_domain": "poop.nl",
  "max_physical_memory": 1536,
  "cpu_shares": 100,
  "quota": 100,
  "delegate_dataset": "true",
  "nics": [
    {
      "nic_tag": "admin",
      "ips": ["1.2.3.4/24", "2001::1/64"],
      "gateways": ["1.2.3.1"],
      "primary": "true"
    }
  ],
  "resolvers": [
    "8.8.8.8",
    "8.8.4.4"
  ],
  "customer_metadata": {
    "system:ssh_disabled": "true",
    "system:timezone": "Europe/Amsterdam",
    "dovecot:qualify_domain": "cyberhq.nl",
    "dovecot:primary_hostname": "mail.poop.nl"
  }
}

