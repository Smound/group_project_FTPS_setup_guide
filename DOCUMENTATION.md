# Setup an FTPS server

## Creating the VM

vagrant init generic/debian10

## Setup the service

Using `vsftpd`.

### vsftpd config

See the [config file](./vsftpd.conf) for all the details, but the gist of it is we are using the default config file with the following edits:

- Permission-related edits. These edits allow logged-in users to write files and dirs, as well as define a list of allowed users that are able to log in:
  
  - `write_enable`changed to `YES`
  - set `userlist_enable` to `YES
  - set a `userlist_file` and `userlist_deny` to `NO`
  
- Chroot and TLS configs. These add on the security of the service.
  - `chroot_local_users` makes it so that local users are dropped in their home directory in a chroot, so their path will be `/`.
  - `allow_writeable_chroot` makes the `chroot` writeable
  - The following options are needed to have working TLS:
    - `rsa_cert_file=/etc/ssl/vsftpd.pem`
    - `rsa_private_key_file=/etc/ssl/vsftpd.pem`
    - `ssl_enable=YES`
    - `require_ssl_reuse=NO`
  
  - I added this one for good measure, not sure if really needed:
    - `utf8_filesystem=YES`

### Self-signed TLS cert
`openssl req -x509 -nodes -days 7300 -newkey rsa:2048 -keyout vsftpd.pem -out vsftpd.pem`

## Deployment

Run `vagrant up`. `rsync` into the VM `./vsftpd.*` (conf, pem and user_list file).

Run `vagrant ssh`. Once in, move `vsftpd.conf` and `vsftpd.user_list` to `/etc/`, as well as `vsftpd.pem` to `/etc/ssl/`.

Add the needed users with `useradd` and set their passwords with `passwd`. Add them to the user_list file in `/etc/`.

Make sure `vsftpd.service` is enabled and started.