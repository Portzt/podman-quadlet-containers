# =============================================================================
# Quadlet .container TEMPLATE — every available [Container] key with examples
# =============================================================================
# Source of truth: man podman-systemd.unit (5)
# Place in: ~/.config/containers/systemd/  (rootless)
#           /etc/containers/systemd/       (rootful)
#
# NOTE: Quadlet is strict — an unknown key makes the generator REJECT the unit.
# Keys marked [5.x+] may not exist on older podman (RHEL 9 ships 4.9.x on
# 9.4/9.5; check `podman --version` and your local man page before using).
#
# Validate before use:
#   rootless: /usr/libexec/podman/quadlet -dryrun -user
#   rootful:  /usr/libexec/podman/quadlet -dryrun
#
# Almost every key below is OPTIONAL. Only Image= (or Rootfs=) is required.
# Delete what you don't need — this file is a reference, not a runnable unit.
# =============================================================================

[Unit]
# --- Standard systemd [Unit] section; nothing Quadlet-specific here ----------
Description=Example application container
Documentation=https://example.com/docs
# Ordering/dependency on network (common for containers that bind ports)
After=network-online.target
Wants=network-online.target
# Depend on another Quadlet unit (note: use the *generated* service name)
# After=postgres.service
# Requires=postgres.service


[Container]
# =============================================================================
# IMAGE / RUNTIME BASICS
# =============================================================================

# REQUIRED (unless Rootfs= is used). Always fully-qualify the image.
# Can also reference a Quadlet .image or .build unit: Image=myapp.image
Image=docker.io/library/nginx:1.27

# Alternative to Image= — run from a directory on disk (conflicts with Image=)
# Rootfs=/var/lib/rootfs/myapp

# Container name (default: systemd-%N, i.e. systemd-<unitname>)
ContainerName=myapp

# Override the systemd service unit name (default: <quadletname>.service)
# ServiceName=my-custom-name        # [5.x+]

# Override image ENTRYPOINT (JSON form for multi-arg)
# Entrypoint=/entrypoint.sh
# Entrypoint=["/bin/sh","-c","exec /app"]

# Arguments appended after the image (same as args after `podman run IMAGE`)
# Exec=--config /config/app.yml --verbose

# Working directory inside the container
# WorkingDir=/app

# Hostname inside the container
# HostName=myapp.internal

# Timezone: local, UTC, or an IANA name (default: system-configured)
Timezone=local

# Image pull policy: always | missing | never | newer
Pull=missing

# Retries for image pull on HTTP error, and delay between them  [5.x+]
# Retry=3
# RetryDelay=5s

# Auto-update label for podman-auto-update: registry | local
# AutoUpdate=registry

# How builtin image VOLUMEs are handled: bind (default) | tmpfs | ignore
# ImageVolume=tmpfs

# Run a minimal init (catatonit) as PID 1 to reap zombies (default: false)
RunInit=true

# Use sdnotify from the app itself instead of conmon: true | false | healthy
# `healthy` = unit is "started" only once the healthcheck passes  [5.x+ for healthy]
# Notify=healthy

# Stop signal and timeout
# StopSignal=SIGTERM
# StopTimeout=30

# systemctl reload support — run a command in the container, or send a signal  [5.2+]
# ReloadCmd=/usr/sbin/nginx -s reload
# ReloadSignal=SIGHUP

# cgroups mode: split (default) | enabled | disabled | no-conmon
# CgroupsMode=split

# Load a containers.conf module
# ContainersConfModule=/etc/nvd.conf

# =============================================================================
# ENVIRONMENT
# =============================================================================

# Repeatable; systemd-style quoting
Environment=APP_MODE=production
Environment="APP_MOTD=hello world"

# Line-delimited env file (absolute, or relative to this unit file)
# EnvironmentFile=/etc/myapp/env

# Pass the entire host environment into the container (rarely wise)
# EnvironmentHost=true

# Pass http_proxy/https_proxy/no_proxy from host into pulls/container (default true)
# HttpProxy=false

# =============================================================================
# SECRETS
# =============================================================================

# Mount a podman secret as a file at /run/secrets/<name> (default), or...
Secret=db_password
# ...inject as an environment variable, or custom target/mode/uid/gid:
# Secret=db_password,type=env,target=DB_PASSWORD
# Secret=tls_key,type=mount,target=/certs/key.pem,mode=0400,uid=0,gid=0

# =============================================================================
# STORAGE: VOLUMES / MOUNTS / TMPFS
# =============================================================================

# Bind mount or named volume. Repeatable.
#   :Z / :z  = SELinux relabel (private/shared)
#   :ro      = read-only
# Can also reference a Quadlet .volume unit: Volume=data.volume:/data
Volume=/srv/myapp/config:/config:Z,ro
Volume=/srv/myapp/data:/data:Z
# Volume=myapp-cache:/cache

# Generic --mount syntax (bind, volume, tmpfs, image, glob, artifact...)
# Mount=type=bind,source=/srv/shared,destination=/shared,ro=true
# Mount=type=volume,source=dbdata,destination=/var/lib/db

# tmpfs mounts; size/mode go as options ON THE MOUNT (there is no TmpfsSize= key!)
Tmpfs=/tmp/cache:size=1g,mode=1777

# Size of /dev/shm
ShmSize=256m

# Read-only root filesystem (default false); ReadOnlyTmpfs (default true)
# mounts rw tmpfs on /dev, /dev/shm, /run, /tmp, /var/tmp when ReadOnly=true
# ReadOnly=true
# ReadOnlyTmpfs=true

# =============================================================================
# NETWORKING
# =============================================================================

# Network mode or named network. Repeatable. Can reference a .network unit.
#   host | none | bridge | pasta | slirp4netns | <netname> | mynet.network
# Network=mynet.network
# Network=host

# DNS aliases on the attached network (repeatable)
# NetworkAlias=web

# Publish ports: ip:hostPort:containerPort — bind loopback unless you mean it
PublishPort=127.0.0.1:8080:80
# PublishPort=127.0.0.1:8443:443/tcp
# PublishPort=127.0.0.1:5514:514/udp

# Expose a port (or range) without publishing
# ExposeHostPort=50-59

# Static addresses (requires a user-defined network)
# IP=10.88.64.128
# IP6=fd46:db93:aa76:ac37::10

# /etc/hosts entries (repeatable)
# AddHost=gateway:10.88.0.1

# Container-scoped DNS settings (repeatable)
# DNS=1.1.1.1
# DNSOption=ndots:1
# DNSSearch=example.internal

# =============================================================================
# SECURITY
# =============================================================================

# Capabilities: space-separated, repeatable
# AddCapability=NET_ADMIN NET_RAW
DropCapability=all
AddCapability=NET_BIND_SERVICE

# Block privilege escalation via setuid/file caps (default false)
NoNewPrivileges=true

# Seccomp: path to JSON profile, or `unconfined`
# SeccompProfile=/usr/share/containers/seccomp.json

# AppArmor profile (non-SELinux distros): profile name or `unconfined`
# AppArmor=unconfined

# SELinux label controls
# SecurityLabelDisable=true            # turn off label separation
# SecurityLabelType=my_container.process
# SecurityLabelLevel=s0:c1,c2
# SecurityLabelFileType=usr_t
# SecurityLabelNested=true             # allow labeling inside the container

# Mask / unmask paths inside the container
# Mask=/proc/acpi:/sys/firmware
# Unmask=/sys/fs/cgroup            # or Unmask=ALL

# Host devices: HOST[:CONTAINER][:PERMISSIONS]
# AddDevice=/dev/dri/renderD128
# AddDevice=/dev/bus/usb/001/004:rwm

# sysctls (repeatable, key=value)
# Sysctl=net.ipv4.ip_unprivileged_port_start=80

# ulimits
# Ulimit=nofile=65536:65536

# pids limit
# PidsLimit=2048

# =============================================================================
# USER / USER NAMESPACE
# =============================================================================

# UID/GID to run as INSIDE the container
# User=1000
# Group=1000

# Supplementary groups; supports keep-groups
# GroupAdd=keep-groups

# User namespace mode: keep-id | auto | nomap | host | ...
# UserNS=keep-id:uid=1000,gid=1000

# Manual mappings (advanced; mutually exclusive with UserNS in places)
# UIDMap=0:100000:65536
# GIDMap=0:100000:65536
# SubUIDMap=myuser
# SubGIDMap=myuser

# =============================================================================
# RESOURCES
# =============================================================================

# Memory=512m

# =============================================================================
# HEALTHCHECKS
# =============================================================================

HealthCmd=curl -fsS http://localhost:80/healthz || exit 1
HealthInterval=30s
HealthTimeout=5s
HealthRetries=3
HealthStartPeriod=10s
# Action when unhealthy: none | kill | restart | stop
# `kill` + [Service] Restart=always integrates best with systemd
HealthOnFailure=kill
# Startup healthcheck (runs until it succeeds, then regular check begins)
# HealthStartupCmd=curl -fsS http://localhost:80/ready
# HealthStartupInterval=5s
# HealthStartupRetries=30
# HealthStartupSuccess=1
# HealthStartupTimeout=10s
# Healthcheck log tuning  [5.2+]
# HealthLogDestination=local
# HealthMaxLogCount=5
# HealthMaxLogSize=500

# =============================================================================
# LOGGING / METADATA
# =============================================================================

# Log driver: journald | k8s-file | json-file | passthrough
LogDriver=journald
# LogOpt=tag=myapp
# LogOpt=max-size=10mb

# OCI labels / annotations (repeatable, key=value)
Label=app=myapp
# Annotation=org.example.owner=admin

# =============================================================================
# POD MEMBERSHIP
# =============================================================================

# Join a Quadlet-managed pod (value must be <name>.pod and the unit must exist)
# Pod=mystack.pod
# Start this container when the pod starts (default true)  [5.x+]
# StartWithPod=true

# =============================================================================
# ESCAPE HATCHES (use sparingly — generator can't sanity-check these)
# =============================================================================

# Args inserted between `podman` and `run`:
# GlobalArgs=--log-level=debug

# Args appended to `podman run` right before the image
# (this is how you get --privileged, since there is no Privileged= key):
# PodmanArgs=--privileged
# PodmanArgs=--isolation=chroot


[Service]
# --- Standard systemd [Service]; Quadlet merges these into the generated unit.
# Restart policy lives HERE, not in [Container]:
Restart=always
RestartSec=5
# Give slow image pulls time on first start
TimeoutStartSec=300
# Do NOT set Type= — Quadlet generates Type=notify + NotifyAccess for you.
# Environment=PODMAN_SYSTEMD_UNIT=%n is also injected automatically.


[Install]
# Required if you want `systemctl enable` / start-at-boot behavior.
# Rootless units: default.target starts them at user login (pair with
# `loginctl enable-linger <user>` for start-at-boot without login).
WantedBy=multi-user.target default.target
