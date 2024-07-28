# Ansible role cyrus_imap

FIXME

## Features

- Idempotent.
- SSL activation.
FIXME
- Debian and Ubuntu friendly (anyone for Redhat likes and other platforms?).
- A developer/maintainer willing to receive feedback and bug reports.


## Requirements

This role must be run as `root` but will **not** `become` by itself.


## Role Variables

| Name                    | Default              | Description                                                                           |
|-------------------------|----------------------|---------------------------------------------------------------------------------------|
| `FIXME`                 | `"FIXME"`            | FIXME.                                                                                |


## Dependencies

FIXME


## Notes

FIXME


## Example Playbooks

Minimal playbook:

```
- name: Minimal playbook for role seb4itik.cyrus_imap
  hosts: mail
  vars:
    FIXME
  roles:
    - "seb4itik.cyrus_imap"
```

More complete example:

```
- name: Example playbook for role seb4itik.cyrus_imap
  hosts: mail
  vars:
    FIXME
  roles:
    - "seb4itik.cyrus_imap"
```


## Tips & Tricks

FIXME


## TODO

FIXME
- Write tests (but problem between *Docker* and *systemd*).
- Other platforms (Redhat, ...).


## License

MIT


## Author Information

- [seb4itik](https://github.com/seb4itik)