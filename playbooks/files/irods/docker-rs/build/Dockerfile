FROM ubuntu:24.04

ARG IRODS_HOST_UID
ARG IRODS_SERVER_PORT_RANGE_START
ARG IRODS_SERVER_PORT_RANGE_END
ARG IRODS_VAULT_PATH
ARG IRODS_ZONE_PORT

ENV DEBIAN_FRONTEND=noninteractive

COPY <<'EOF' /etc/timezone
America/Phoenix
EOF

RUN <<'EOF'
	set -o errexit

# Install dependencies (Ubuntu repos only)
	apt-get update
	apt-get install --yes apt-utils ca-certificates wget

# Add iRODS repository
	wget --quiet --output-document=/etc/apt/keyrings/irods-signing-key.asc \
		https://packages.irods.org/irods-signing-key.asc
	printf 'deb [arch=amd64 signed-by=/etc/apt/keyrings/irods-signing-key.asc] https://packages.irods.org/apt/ noble main\n' \
		> /etc/apt/sources.list.d/renci-irods.list
	apt-get update
	apt-get upgrade --yes

# Install iRODS
	apt-get install --yes irods-icommands=4.3.3 irods-runtime=4.3.3 irods-server=4.3.3

	groupadd --non-unique --system --gid="$IRODS_HOST_UID" irods

	useradd --non-unique --system \
		--uid="$IRODS_HOST_UID" \
		--gid="$IRODS_HOST_UID" \
		--comment='iRODS Administrator (host user)' \
		--home-dir=/var/lib/irods \
		--shell=/bin/bash \
		irods

	mkdir --parents /irods_vault /etc/irods /var/lib/irods/.irods
	chown --recursive irods:irods /etc/irods /var/lib/irods

# Clean up packages
	apt-get remove --yes wget
	apt-get autoremove --yes
	apt-get clean
	rm --force --recursive /var/lib/apt/lists/* /tmp/*
EOF

VOLUME ${IRODS_VAULT_PATH}

EXPOSE ${IRODS_ZONE_PORT}/tcp 1248/tcp
EXPOSE ${IRODS_SERVER_PORT_RANGE_START}-${IRODS_SERVER_PORT_RANGE_END}/tcp
EXPOSE ${IRODS_SERVER_PORT_RANGE_START}-${IRODS_SERVER_PORT_RANGE_END}/udp

COPY --chmod=u+x entrypoint.sh .
ENTRYPOINT [ "./entrypoint.sh" ]