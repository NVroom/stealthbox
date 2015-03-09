# StealthBox

Share your favorite content remotely without spoiling your home network.

StealthBox is a [Docker](https://www.docker.com/) image, so you can install it
on any linux server without polluting it.

It embeds several applications that are configured together to automate file 
sharing in the cloud and download content to your home: 

- [Deluge](http://deluge-torrent.org/), a BitTorrent client.
- [SickRage](http://sickrage.tv), an automatic shows downloader.
- [CouchPotato](http://sickrage.tv), an automatic movies downloader.
- [HeadPhones](https://github.com/rembo10/headphones), an automated music
downloader.
- [FlexGet](http://flexget.com/), a multipurpose automation tool
- [Pydio](https://pyd.io/), a web application to view, edit and download files.

## Install

```bash
docker run -d \
  -p 8443:443 -p 8022:22 -p 6881:6881 \
  --name stealthbox \
  --restart always \
  toilal/stealthbox
```

## Username & Password

All services are installed with a `box` username and a randomly generated
password. This password is displayed when container is starting. 

To retrieve, you can use `docker logs`:

```
docker logs stealthbox | grep Password:
```

To set your own password when running the container, add `-e PASSWORD=`
followed with the password of your choice to `docker run` command.

You can also change the password using 'boxpasswd' command from shell.

Password must be long and strong enough to be accepted (checked with 
[cracklib-check](http://sourceforge.net/projects/cracklib)).

## Components

- Change password for each component as soon as container is running.
- Replace `<stealthbox>` with IP address of your server and `<8443>`/`<8022>`
with ports used in `docker run` command.

### [Pydio](https://pyd.io/)

[Pydio](https://pyd.io/) (formerly AjaXplorer) is a mature open source
software solution for file sharing and synchronization. With intuitive user
interfaces (web / mobile / desktop), Pydio provides enterprise-grade features
to gain back control and privacy of your data: user directory connectors,
legacy filesystems drivers, comprehensive admin interface, and much more.

```
web: https://<stealthbox>:<8443>
```

### [Deluge](http://deluge-torrent.org/)

[Deluge](http://deluge-torrent.org/) is a lightweight, Free Software,
cross-platform BitTorrent client.

- Full Encryption
- WebUI
- Plugin System
- Much more ...

```
web: https://<stealthbox>:<8443>/deluge
configuration: ~/.config/deluge
service: deluge
```

### [SickRage](https://sickrage.tv/)

[SickRage](https://sickrage.tv/) is a Video File Manager for TV Shows, It
watches for new episodes of your favorite shows and when they are posted it
does its magic.

```
web: https://<stealthbox>:<8443>/sickrage
```

### [CouchPotato](https://couchpota.to/)

[CouchPotato](https://couchpota.to/) Download movies automatically, easily and
in the best quality as soon as they are available. Awesome PVR for usenet and
torrents. Just fill in what you want to see and CouchPotato will add it to
your "want to watch"-list.

```
web: https://<stealthbox>:<8443>/couchpotato
```

### [HeadPhones](https://github.com/rembo10/headphones)

[HeadPhones](https://github.com/rembo10/headphones) is an automated music
downloader for NZB and Torrent, written in Python. It supports SABnzbd,
NZBget, Transmission, µTorrent and Blackhole.

```
web: https://<stealthbox>:<8443>/couchpotato
```

### [FlexGet](http://flexget.com/)

[FlexGet](http://flexget.com/) is a multipurpose automation tool for
content like torrents, nzbs, podcasts, comics, series, movies, etc. It can use
different kinds of sources like RSS-feeds, html pages, csv files, search
engines and there are even plugins for sites that do not provide any kind of
useful feeds.

```
configuration: ~/flexget
service: flexget
```

### Others

#### SSH/SCP/SFTP

Access StealthBox through SSH/SCP/SFTP to bring content back to home and manage
the environment.

```
ssh: ssh -p <8022> box@<stealthbox>
service: sshd
```

#### [nginx](http://nginx.org/en/)

[nginx](http://nginx.org/en/) \[engine x\] is an HTTP and reverse proxy
server.

```
configuration: /etc/nginx
service: nginx
```

## Configuration

### Flexget

Flexget is installed in [Daemon](http://flexget.com/wiki/Daemon) mode.

Configuration file is `/home/box/flexget/config.yml`, and will be
automatically reloaded in Daemon when changed.

### Use your own SSL certificate

Self-signed SSL certificate is generated when running the container for the
first time. But self-signed certificates generates warning in browser when
trying to connect.

To avoid this warning, you can use a certificate from trusted authority, and
load them in StealthBox. [StartSSL](https://www.startssl.com) can provide free
trusted certificate.

- Replacing certificate in container

Your can use your own SSL certificate by replacing `stealthbox.key` and
`stealthbox.crt` in `/home/box/ssl/`, using Pydio or SSH. This will reload
web server automatically.

- Using certificate from host

Instead of replacing certificate files in container, you can also load volumes
pointing to certificate on the host.

```bash
docker run -d \
  -v /home/docker/ssl/stealthbox.key:/home/box/ssl/stealthbox.key:ro \
  -v /home/docker/ssl/stealthbox.crt:/home/box/ssl/stealthbox.crt:ro \
  -p 8443:443 -p 8022:22 -p 6881:6881 \
  --restart always \
  --name stealthbox \
  toilal/stealthbox
```

Make sure correct access rights are defined.

```bash
chown 1000:1000 \
  /home/docker/ssl/stealthbox.key \ 
  /home/docker/ssl/stealthbox.crt

chmod 400 /home/docker/ssl/stealthbox.crt
```

### Disable SSL

You may have good reasons to do this, but nginx also listen on `tcp/80` for raw
connections. So you can map this port instead of `tcp/443`.

```bash
docker run -d \
  -p 8080:80 -p 8022:22 -p 6881:6881 \
  --name stealthbox \
  --restart always \
  toilal/stealthbox
```

Web server will then be available at `http://<stealthbox>:8080`.

### tcp/6881

`tcp/6881` is the port use by Torrent protocol in deluge. Your have to map
this port on the same host port.

If tcp/6881 is not available on host, or if you really need to change this
port, you have to configure deluge for port to match, in web UI, or in 
`/home/box/.config/deluge/core.conf` (`listen_ports` parameter).