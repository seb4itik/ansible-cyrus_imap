# Ansible role cyrus_imap

Install and configure Cyrus IMAP server.


## Features

- Idempotent.
- SSL activation.
- Manage Cyrus daemons (trhough `/etc/cyrus.conf`).
- Configure IMAP/POP/NNTP/etc. options (trhough `/etc/imapd.conf`).
- Debian friendly (Ubuntu soon, anyone for Redhat likes and other platforms?).
- A developer/maintainer willing to receive feedback and bug reports.


## Requirements

This role must be run as `root` but will **not** `become` by itself.


## Role Variables

| Name                   | Default      | Description                                                                           |
|------------------------|--------------|---------------------------------------------------------------------------------------|
| `cyrus_imap_user`      | `"cyrus"`    | System user for running daemons.                                                      |
| `cyrus_imap_ssl`       | `false`      | Activate SSL.                                                                         |
| `cyrus_imap_ssl_group` | `"ssl-cert"` | Group `slapd` will be added to if `slapd_ssl` (to access keys in `/etc/ssl/private`). |
| `cyrus_imap_services`  | `{}`         | Configure startup of Cyrus daemons. Will be merged in default values from `cyrus_imap_default_services` (see below). |
| `cyrus_imap_config`    | `{}`         | Configure services options. Will be merged in default values from `cyrus_imap_default_config` (see below). |

### cyrus_imap_ssl

If `slapd_ssl` is `true`:

- Cyrus IMAP system user (`cyrus_imap_user`) will be added to group `slapd_ssl_group`;
- `cyrus-imapd` service will be restarted.

At least, these parameters must be set in `cyrus_imap_config`:

- `tls_server_cert` (name of a file that should be under `/etc/ssl/certs`);
- `tls_server_key` (name of a file that should be under `/etc/ssl/private`, owner `root`, group `ssl-cert`, mode `0640`);

Then some SSL services should be activate in `cyrus.conf` trhough `cyrus_imap_services`. For example:

```
  vars:
    cyrus_imap_services:
      services:
        imaps:
          active: true
        pop3s:
          active: true        
```

### cyrus_imap_services

`cyrus_imap_services` describes the daemons started by the Cyrus master process (`cyrmaster`).
See `cyrus.conf(5)`.

`cyrus_imap_services` is a dictionnary with four keys, each section of the `cyrus.conf` file:
- `start`: This  section lists the processes to run before any SERVICES are spawned.
- `daemon`: This section lists long running daemons to start before any SERVICES are spawned.
- `services`: This section lists the processes that should be spawned to handle client connections made on certain Internet/UNIX sockets.
- `events`: This  section  lists processes that should be run at specific intervals, similar to cron jobs.

It will be merged with default values from `cyrus_imap_default_services` variable.
See [vars/main.yml].

Section `start`:
- `active`: `true` or `false`.
- `cmd`: The command (with options) to spawn as a child process (required).

Section `daemon`:
- `active`: `true` or `false`.
- `cmd`: The command (with options) to spawn as a child process (required).
- `wait`: Switch: whether or not master(8) should wait for this daemon to successfully start  before  continuing to load (default `n`).

Section `services`:
- `active`: `true` or `false`.
- `cmd`: The command (with options) to spawn as a child process (required).
- `listen`: The  UNIX or internet socket to listen on (required).
- `proto`: The protocol used for this service (`tcp`, `tcp4`, `tcp6`, `udp`, `udp4`, `udp6`, default `tcp`).
- `prefork`:  The number of instances of this service to always have running and waiting for a connection (default 0).
- `maxchild`: The  maximum number of instances of this service to spawn (default -1, unlimited).
- `babysit`: If non-zero, will make sure at least one process is pre-forked, and will set the maxforkrate to 10 if it’s zero (default 0).
- `maxfds`: The maximum number of file descriptors to which to limit this process (default 256).
- `maxforkrate`: Maximum number of processes to fork per second (default 0).

Section `events`:
- `active`: `true` or `false`
- `cmd`: The command (with options) to spawn as a child process (required).
- `period`: The interval (in minutes) at which to run the command (default 0).
- `at`: The  time  (24-hour format) at which to run the command each day (default "").

The default services are:

```
cyrus_imap_default_services:
  start:
    recover:
      active: true
      cmd: "/usr/sbin/cyrus ctl_cyrusdb -r"
    idled:
      active: false
      cmd: "idled"
    mupdatepush:
      active: false
      cmd: "/usr/sbin/cyrus ctl_mboxlist -m"
    delprune:
      active: true
      cmd: "/usr/sbin/cyrus expire -E 3"
    tlsprune:
      active: true
      cmd: "/usr/sbin/cyrus tls_prune"
    statscleanup:
      active: false
      cmd: "/usr/sbin/cyrus promstatsd -c"
  services:
    imap:
      active: true
      cmd: "imapd -U 30"
      proto: "tcp4"
      listen: "imap"
      prefork: 0
      maxchild: 100
    imaps:
      active: false
      cmd: "imapd -s -U 30"
      proto: "tcp4"
      listen: "imaps"
      prefork: 0
      maxchild: 100
    pop3:
      active: true
      cmd: "pop3d -U 30"
      proto: "tcp4"
      listen: "pop3"
      prefork: 0
      maxchild: 50
    pop3s:
      active: false
      cmd: "pop3d -s -U 30"
      proto: "tcp4"
      listen: "pop3s"
      prefork: 0
      maxchild: 50
    nntp:
      active: true
      cmd: "nntpd -U 30"
      proto: "tcp4"
      listen: "nntp"
      prefork: 0
      maxchild: 100
    nntps:
      active: false
      cmd: "nntpd -s -U 30"
      proto: "tcp4"
      listen: "nntps"
      prefork: 0
      maxchild: 100
    http:
      active: true
      cmd: "httpd -U 30"
      proto: "tcp4"
      listen: "8008"
      prefork: 0
      maxchild: 100
    https:
      active: false
      cmd: "httpd -s -U 30"
      proto: "tcp4"
      listen: "8443"
      prefork: 0
      maxchild: 100
    lmtp:
      active: false
      cmd: "lmtpd"
      listen: "localhost:lmtp"
      prefork: 0
      maxchild: 20
    lmtpunix:
      active: true
      cmd: "lmtpd"
      listen: "/run/cyrus/socket/lmtp"
      prefork: 0
      maxchild: 20
    sieve:
      active: true
      cmd: "timsieved"
      proto: "tcp4"
      listen: "localhost:sieve"
      prefork: 0
      maxchild: 100
    notify:
      active: true
      cmd: "notifyd"
      listen: "/run/cyrus/socket/notify"
      proto: "udp"
      prefork: 1
    mupdateslave:
      active: false
      cmd: "mupdate"
      listen: "3905"
      prefork: 1
    mupdatemaster:
      active: false
      cmd: "mupdate -m"
      listen: "3905"
      prefork: 1
    imapproxy:
      active: false
      cmd: "proxyd"
      listen: "imap"
      prefork: 0
      maxchild: 100
    imapsproxy:
      active: false
      cmd: "proxyd -s"
      listen: "imaps"
      prefork: 0
      maxchild: 100
    pop3proxy:
      active: false
      cmd: "pop3proxyd"
      listen: "pop3"
      prefork: 0
      maxchild: 50
    pop3sproxy:
      active: false
      cmd: "pop3proxyd -s"
      listen: "pop3s"
      prefork: 0
      maxchild: 50
    lmtpproxy:
      active: false
      cmd: "lmtpproxyd"
      listen: "lmtp"
      prefork: 1
      maxchild: 20
  daemon:
    promstatsd:
      active: false
      cmd: "promstatsd"
  events:
    checkpoint:
      active: true
      cmd: "/usr/sbin/cyrus ctl_cyrusdb -c"
      period: 30
    delprune:
      active: true
      cmd: "/usr/sbin/cyrus expire -E 3"
      at: "0401"
    tlsprune:
      active: true
      cmd: "/usr/sbin/cyrus tls_prune"
      at: "0401"
    deleteprune:
      active: true
      cmd: "/usr/sbin/cyrus expire -E 4 -D 28"
      at: "0430"
    expungeprune:
      active: true
      cmd: "/usr/sbin/cyrus expire -E 4 -X 28"
      at: "0445"
    squatter_1:
      active: false
      cmd: "/usr/bin/nice -n 19 /usr/sbin/cyrus squatter -s"
      period: 120
    squatter_a:
      active: false
      cmd: "/usr/sbin/cyrus squatter"
      at: "0517"
```

### cyrus_imap_config

`cyrus_imap_config` is a dictionnary containing all parameters that can be found in `imapd.conf` file.
See `imapd.conf(5)`.

It will be merged with default values from `cyrus_imap_default_config` variable.
See [vars/main.yml].

The default parameters are:

```
cyrus_imap_default_config:
  configdirectory: "/var/lib/cyrus"
  proc_path: "/run/cyrus/proc"
  mboxname_lockpath: "/run/cyrus/lock"
  defaultpartition: "default"
  partition-default: "/var/spool/cyrus/mail"
  partition-news: "/var/spool/cyrus/news"
  newsspool: "/var/spool/news"
  altnamespace: "no"
  unixhierarchysep: "no"
  lmtp_downcase_rcpt: "yes"
  allowanonymouslogin: "no"
  popminpoll: 1
  autocreate_quota: 0
  umask: "077"
  sieveusehomedir: "no"
  sievedir: "/var/spool/sieve"
  httpmodules: "caldav carddav"
  hashimapspool: "yes"
  allowplaintext: "yes"
  sasl_pwcheck_method: "auxprop"
  sasl_auto_transition: "no"
  tls_client_ca_dir: "/etc/ssl/certs"
  tls_session_timeout: 1440
  lmtpsocket: "/run/cyrus/socket/lmtp"
  idlesocket: "/run/cyrus/socket/idle"
  notifysocket: "/run/cyrus/socket/notify"
  syslog_prefix: "cyrus"
```

## Dependencies

None


## Example Playbooks

Minimal playbook:

```
- name: Minimal playbook for role seb4itik.cyrus_imap
  hosts: mail
  roles:
    - "seb4itik.cyrus_imap"
```

More complete example:

```
- name: Example playbook for role seb4itik.cyrus_imap
  hosts: mail
  vars:
    cyrus_imap_ssl: true
    cyrus_imap_services:
      services:
        imap:
          active: false
        imaps:
          active: true
          prefork: 30
          maxchild: 2000
        pop3:
          active: false
        nntp:
          active: false
        http:
          active: false
        lmtpunix:
          prefork: 5
        sieve:
          listen: "2000"
      events:
        checkpoint:
          period: 15
    cyrus_imap_config:
      admins: "cyrusadmin"
      altnamespace: "yes"
      delete_mode: "immediate"
      partition-default: "/data/cyrus/mail"
      sasl_mech_list: "PLAIN LOGIN"
      sasl_minimum_layer: 1
      sasl_pwcheck_method: "saslauthd"
      servername: "mail.{{ env_domain_name }}"
      tls_required: "yes"
      tls_server_cert: "/etc/ssl/certs/_.{{ my_domain }}-bundle.crt"
      tls_server_key: "/etc/ssl/private/_.{{ my_domain }}.key"
  roles:
    - "seb4itik.cyrus_imap"
```


## TODO

- Write tests.
- Other platforms (Ubuntu, Redhat, ...).


## License

MIT


## Author Information

- [seb4itik](https://github.com/seb4itik)
