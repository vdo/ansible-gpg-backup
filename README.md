vdo.gpg_backup
==============

A role to backup remote files using [GnuPG](https://gnupg.org/) to encrypt tarballs, importing keys if necessary.

Requirements
------------

* [GnuPG](https://gnupg.org/) encryption tool must be installed in the host running the playbook. It's included in most distributions.

How this role works
-------------------

* Creates one or more tarballs of the paths defined in `remote_backup_paths`.
* Downloads the tarballs locally to a temporary path.
* Encrypts the tarballs with all the recipients defined. Any single recipient can decrypt the files!
* Safely deletes any unencrypted file left behind, using [shred](https://linux.die.net/man/1/shred).

The generated files follow the pattern: `{ hostname }_{ path }_{ random string }.tgz.gpg`, converting any dots, slashes and asterisks, for example: `www_example_net__var_log_dmesg@_bxhlndi5.tgz.gpg` would be the backup of `/var/log/dmesg*` from `www.example.net`.

The recipient strings can be any of the GnuPG accepted user ids, and [they can be many](https://www.gnupg.org/documentation/manuals/gnupg/Specify-a-User-ID.html). The recommended one is the **fingerprint**, to avoid ambiguities.

Note: The imported key(s) will be assumed as valid in any case.

Role Variables
--------------
|Variable|Description|Default Value
|---|---|---|
|`remote_backup_paths`| Paths on the remote host(s) to generate the backups from (required) | []
|`gpg_recipients`| User ID(s) of the public keys used to encrypt (required)| []
|`gpg_keyserver`| PGP keystore server to use, in case keys are imported | `hkps://keys.openpgp.org`
|`gpg_import_keys`| User ID(s) of the public keys to be imported from the keyserver | []
|`remote_backup_temp_path`| Path in the remote host(s) to store the temporal tarballs |`/tmp`
|`local_backup_temp_path`| Path in the localhost to store the temporal tarballs |`/tmp`
|`local_backup_destination`| Destination path of the encrypted backups |`./`

Example Playbook
----------------

```yaml
---
- hosts: all
  become: true
  vars:
    remote_backup_paths:
      - "/var/log/audit/*"
      - "/etc/ssl/certs/"
    gpg_recipients:
      - "DE4EFCA3E1AB9E41CE96CECB18C09E865EC948A1"
      - "someones@email.test"
    gpg_keyserver: 'keyserver.ubuntu.com'
    gpg_import_keys: 
      - "DE4EFCA3E1AB9E41CE96CECB18C09E865EC948A1"
  roles:
    - vdo.gpg_backup
```

License
-------
Apache License 2.0
