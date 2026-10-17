# Overlay of www/caddy-custom: caddy built with xcaddy plus a preset list of
# plugins, with drop-in configuration support and example configurations.
# Overlay it over the ports tree (poudriere -O), see ../../README.md.
# The build downloads the plugin sources (xcaddy), so poudriere needs
# ALLOW_NETWORKING_PACKAGES="caddy-custom". Unlike the stock port there is no
# NO_PACKAGE, so "make package" works for private use.

PORTNAME=	caddy-custom
PORTVERSION=	${CADDY_VERSION}.${XCADDY_VERSION}
PORTREVISION=	1
CATEGORIES=	www
DISTFILES=	# none

MAINTAINER=	kad@kad.name
COMMENT=	Caddy (www/caddy) with l4, dynamicdns, ratelimit, cloudflare and wstunnel
WWW=		https://caddyserver.com

LICENSE=	APACHE20

BUILD_DEPENDS=	xcaddy:www/xcaddy

USES=		go:1.27+,no_targets

CONFLICTS_INSTALL=	caddy

USE_RC_SUBR=	caddy
SUB_FILES=	Caddyfile.sample pkg-message
ETCDIR=		${PREFIX}/etc/caddy
USERS=		www
GROUPS=		www

CADDY_VERSION!=	${MAKE} -C ${PORTSDIR:U/usr/ports}/www/caddy  -V '$${DISTVERSION}.$${PORTREVISION}'
CADDY_DISTVERSION!=	${MAKE} -C ${PORTSDIR:U/usr/ports}/www/caddy  -V '$${DISTVERSION}'
XCADDY_VERSION!=${MAKE} -C ${PORTSDIR:U/usr/ports}/www/xcaddy -V '$${DISTVERSION}.$${PORTREVISION}'

# Pinned plugin versions. Override the whole list in make.conf with
# CADDY_CUSTOM_PLUGINS= ... to build something else.
CADDY_CUSTOM_PLUGINS?=	\
	github.com/mholt/caddy-l4@v0.1.2 \
	github.com/mholt/caddy-dynamicdns@v0.0.0-20260805195708-67d107a42c02 \
	github.com/mholt/caddy-ratelimit@v0.1.0 \
	github.com/caddy-dns/cloudflare@v0.2.4 \
	github.com/kad/wstunnel-go/pkg/caddy@v0.0.6

# Modules pinned to a version without being plugins (module@version), e.g. the
# core module of a plugin that lives in the same repository.
CADDY_CUSTOM_DEPS?=	

.for plugin in ${CADDY_CUSTOM_PLUGINS}
PLUGINS+=	--with ${plugin}
.endfor
.for dep in ${CADDY_CUSTOM_DEPS}
PLUGINS+=	--replace ${dep:C/@.*//}=${dep}
.endfor

XCADDY_GO_BUILD_FLAGS?=	${GO_BUILDFLAGS}

XCADDY_ENV+=	XCADDY_WHICH_GO="${GO_CMD}" \
		XCADDY_GO_BUILD_FLAGS="${XCADDY_GO_BUILD_FLAGS}" \
		GOCACHE=${WRKDIR}/gocache GOPATH=${WRKDIR}/go \
		GOFLAGS=-modcacherw

EXAMPLES_FILES=	README cloudflare.Caddyfile dynamicdns.Caddyfile environment \
		layer4-alt-port.Caddyfile layer4.Caddyfile ratelimit.Caddyfile \
		wstunnel-rules.yaml wstunnel.Caddyfile

PLIST_FILES=	bin/caddy \
		"@sample ${ETCDIR_REL}/Caddyfile.sample" \
		"@dir ${ETCDIR_REL}/Caddyfile.d"
.for f in ${EXAMPLES_FILES}
PLIST_FILES+=	${EXAMPLESDIR_REL}/${f}
.endfor

do-build:
	${MKDIR} ${WRKSRC}
	cd ${WRKSRC} && ${XCADDY_ENV} ${LOCALBASE}/bin/xcaddy build v${CADDY_DISTVERSION} ${PLUGINS}

do-install:
	${INSTALL_PROGRAM} ${WRKSRC}/caddy ${STAGEDIR}${PREFIX}/bin
	${MKDIR} ${STAGEDIR}${ETCDIR}/Caddyfile.d ${STAGEDIR}${EXAMPLESDIR}
	${INSTALL_DATA} ${WRKDIR}/Caddyfile.sample ${STAGEDIR}${ETCDIR}
.for f in ${EXAMPLES_FILES}
	${INSTALL_DATA} ${FILESDIR}/examples/${f} ${STAGEDIR}${EXAMPLESDIR}
.endfor

.include <bsd.port.mk>
