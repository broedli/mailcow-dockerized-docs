Before you run **mailcow: dockerized**, there are a few requirements that you should check:

!!! warning
    When you want to run the dockerized version on your Debian 8 (Jessie) box you should [switch to the kernel 4.9 from jessie backports](https://packages.debian.org/jessie-backports/linux-image-amd64) because there is a bug (kernel panic) with the kernel 3.16 when running docker containers with *healthchecks*! For more details read: [github.com/docker/docker/issues/30402](https://github.com/docker/docker/issues/30402) and [forum.mailcow.email/t/solved-mailcow-docker-causes-kernel-panic-edit/448](https://forum.mailcow.email/t/solved-mailcow-docker-causes-kernel-panic-edit/448)

!!! info
    - Mailcow: dockerized requires [some ports](#default-ports) to be open for incomming connections, so make sure that your firewall is not bloking these. Also make sure that no other application is interferring with mailcow's configuration.
    - A correct DNS setup is crucial to every good mailserver setup, so please make sure you got at least the [basics](prerequesite-dns/#the-minimal-dns-configuration) covered bevore you begin!
    - Make sure that your system has a correct date and [time setup](#date-and-time). This is crucial for stuff like two factor TOTP authentication.

## Minimum System Resources

Please make sure that your system has at least the following resources:

| Resource                | mailcow: dockerized |
| ----------------------- | ------------------- |
| CPU                     | 1 GHz               |
| RAM                     | 1 GiB               |
| Disk                    | 5 GiB               |
| System Type             | x86_64              |

## Firewall & Ports

Please check if any of mailcow's standard ports are open and not blocked by other applications:

```
# netstat -tulpn | grep -E -w '25|80|110|143|443|465|587|993|995'
```

If this command returns any results please remove or stop the application running on that port. You may also adjust mailcows ports via the `mailcow.conf` configuration file.

### Default Ports

If you have a firewall already up and running please make sure that these ports are open for incomming connections:

| Service             | Protocol | Port   | Container       | Variable                       |
| --------------------|:--------:|:-------|:----------------|--------------------------------|
| Postfix SMTP        | TCP      | 25     | postfix-mailcow | `${SMTP_PORT}`                 |
| Postfix SMTPS       | TCP      | 465    | postfix-mailcow | `${SMTPS_PORT}`                |
| Postfix Submission  | TCP      | 587    | postfix-mailcow | `${SUBMISSION_PORT}`           |
| Dovecot IMAP        | TCP      | 143    | dovecot-mailcow | `${IMAP_PORT}`                 |
| Dovecot IMAPS       | TCP      | 993    | dovecot-mailcow | `${IMAPS_PORT}`                |
| Dovecot POP3        | TCP      | 110    | dovecot-mailcow | `${POP_PORT}`                  |
| Dovecot POP3S       | TCP      | 995    | dovecot-mailcow | `${POPS_PORT}`                 |
| Dovecot ManageSieve | TCP      | 4190   | dovecot-mailcow | `${SIEVE_PORT}`                |
| HTTP(S)             | TCP      | 80/443 | nginx-mailcow   | `${HTTP_PORT}` / `${HTTPS_PORT}` |

## Date and Time

To ensure that you have the correct date and time setup on your system, please check the output of `timedatectl status`:

```
$ timedatectl status
      Local time: Sat 2017-05-06 02:12:33 CEST
  Universal time: Sat 2017-05-06 00:12:33 UTC
        RTC time: Sat 2017-05-06 00:12:32
       Time zone: Europe/Berlin (CEST, +0200)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: yes
 Last DST change: DST began at
                  Sun 2017-03-26 01:59:59 CET
                  Sun 2017-03-26 03:00:00 CEST
 Next DST change: DST ends (the clock jumps one hour backwards) at
                  Sun 2017-10-29 02:59:59 CEST
                  Sun 2017-10-29 02:00:00 CET
```

The lines `NTP enabled: yes` and `NTP synchronized: yes` indicate wether you have NTP enabled and if it's syncronized.

To enable NTP you need to run the command `timedatectl set-ntp true`. You also need to edit your `/etc/systemd/timesyncd.conf`:

```
# vim /etc/systemd/timesyncd.conf
[Time]
Servers=0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
```
