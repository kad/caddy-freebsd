# caddy-freebsd

Ports overlay with `www/caddy-custom`: caddy (same version as `www/caddy`)
built with xcaddy and the modules

- github.com/mholt/caddy-l4
- github.com/mholt/caddy-dynamicdns
- github.com/mholt/caddy-ratelimit
- github.com/caddy-dns/cloudflare
- github.com/kad/wstunnel-go/pkg/caddy

It mirrors the openSUSE package: drop-ins from `etc/caddy/Caddyfile.d/*.caddyfile`,
an optional secrets file `etc/caddy/environment` (read by the rc script) and
examples in `share/examples/caddy-custom/` (start with `README`).

Differences to the FreeBSD `www/caddy-custom` port: preset and pinned plugin
list, config directory and sample Caddyfile with the drop-in import, rc
variable `caddy_envfile`, examples, `www` user/group.

Status: written on Linux, the plugin set was cross-compiled for
FreeBSD/amd64 there, but the port itself has not been built yet.

## Build with poudriere

The port downloads sources at build time (xcaddy), so poudriere needs
network access for it. In `/usr/local/etc/poudriere.conf`:

    ALLOW_NETWORKING_PACKAGES="caddy-custom"

Use the overlay on top of your ports tree (poudriere >= 3.3 overlays):

    git clone https://github.com/kad/caddy-freebsd /usr/local/poudriere/overlays/caddy-freebsd
    poudriere ports -c -p default -m git+https       # if not present yet
    poudriere bulk -j 15amd64 -p default -O caddy-freebsd www/caddy-custom

Optional, different plugins in `make.conf` for the jail:

    .if ${.CURDIR:M*/www/caddy-custom*}
    CADDY_CUSTOM_PLUGINS= github.com/mholt/caddy-l4@v0.1.2 github.com/caddy-dns/cloudflare@v0.2.4
    .endif

Install from the resulting repository (replaces `caddy`, they conflict):

    pkg install caddy-custom

## Build a package as a normal user (no poudriere)

Needs root only once, to install the build dependencies:

    # pkg install xcaddy go127 git

Then, as a normal user, with a ports tree in your home directory (no need to
touch /usr/ports):

    git clone --depth 1 https://git.FreeBSD.org/ports.git ~/ports
    git clone https://github.com/kad/caddy-freebsd ~/caddy-freebsd
    cp -R ~/caddy-freebsd/www/caddy-custom ~/ports/www/    # replaces the stock one
    cd ~/ports/www/caddy-custom
    make PORTSDIR=$HOME/ports WRKDIRPREFIX=$HOME/pbuild PACKAGES=$HOME/packages \
        DISTDIR=$HOME/distfiles BATCH=yes package

The package is written to `~/packages/All/`. Install it (root):

    # pkg install ~/packages/All/caddy-custom-*.pkg

The port can also be built straight from the clone, without copying it into
the ports tree: `cd ~/caddy-freebsd/www/caddy-custom` and run the same `make`
command (the caddy and xcaddy versions are looked up through `PORTSDIR`).

Dependencies are not installed automatically as a normal user, so a missing
one stops the build with a message.

## Updating

When `www/caddy` is updated the port version follows it. Plugin versions are
pinned in `www/caddy-custom/Makefile`; update them there. wstunnel-go
`pkg/caddy` needs caddy >= 2.11.7 and Go >= 1.27.2.

## Verifying

    caddy version
    caddy list-modules | grep -E 'layer4|dynamic_dns|rate_limit|cloudflare|wstunnel'
    service caddy configtest
