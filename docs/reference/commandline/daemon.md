<!--[metadata]>
+++
title = "daemon"
description = "The daemon command description and usage"
keywords = ["container, daemon, runtime"]
[menu.main]
parent = "smn_cli"
weight = -1
+++
<![end-metadata]-->

# daemon

    Usage: docker daemon [OPTIONS]

    A self-sufficient runtime for linux containers.

    Options:
      --api-cors-header=""                   Set CORS headers in the remote API
      --authorization-plugin=[]              Set authorization plugins to load
      -b, --bridge=""                        Attach containers to a network bridge
      --bip=""                               Specify network bridge IP
      --cgroup-parent=                       Set parent cgroup for all containers
      -D, --debug                            Enable debug mode
      --default-gateway=""                   Container default gateway IPv4 address
      --default-gateway-v6=""                Container default gateway IPv6 address
      --cluster-store=""                     URL of the distributed storage backend
      --cluster-advertise=""                 Address of the daemon instance on the cluster
      --cluster-store-opt=map[]              Set cluster options
      --config-file=/etc/docker/daemon.json  Daemon configuration file
      --dns=[]                               DNS server to use
      --dns-opt=[]                           DNS options to use
      --dns-search=[]                        DNS search domains to use
      --default-ulimit=[]                    Set default ulimit settings for containers
      --exec-opt=[]                          Set exec driver options
      --exec-root="/var/run/docker"          Root of the Docker execdriver
      --fixed-cidr=""                        IPv4 subnet for fixed IPs
      --fixed-cidr-v6=""                     IPv6 subnet for fixed IPs
      -G, --group="docker"                   Group for the unix socket
      -g, --graph="/var/lib/docker"          Root of the Docker runtime
      -H, --host=[]                          Daemon socket(s) to connect to
      --help                                 Print usage
      --icc=true                             Enable inter-container communication
      --insecure-registry=[]                 Enable insecure registry communication
      --ip=0.0.0.0                           Default IP when binding container ports
      --ip-forward=true                      Enable net.ipv4.ip_forward
      --ip-masq=true                         Enable IP masquerading
      --iptables=true                        Enable addition of iptables rules
      --ipv6                                 Enable IPv6 networking
      -l, --log-level="info"                 Set the logging level
      --label=[]                             Set key=value labels to the daemon
      --log-driver="json-file"               Default driver for container logs
      --log-opt=[]                           Log driver specific options
      --mtu=0                                Set the containers network MTU
      --disable-legacy-registry              Do not contact legacy registries
      -p, --pidfile="/var/run/docker.pid"    Path to use for daemon PID file
      --raw-logs                             Full timestamps without ANSI coloring
      --registry-mirror=[]                   Preferred Docker registry mirror
      -s, --storage-driver=""                Storage driver to use
      --selinux-enabled                      Enable selinux support
      --storage-opt=[]                       Set storage driver options
      --tls                                  Use TLS; implied by --tlsverify
      --tlscacert="~/.docker/ca.pem"         Trust certs signed only by this CA
      --tlscert="~/.docker/cert.pem"         Path to TLS certificate file
      --tlskey="~/.docker/key.pem"           Path to TLS key file
      --tlsverify                            Use TLS and verify the remote
      --userns-remap="default"               Enable user namespace remapping
      --userland-proxy=true                  Use userland proxy for loopback traffic

Options with [] may be specified multiple times.

The Docker daemon is the persistent process that manages containers. Docker
uses the same binary for both the daemon and client. To run the daemon you
type `docker daemon`.

To run the daemon with debug output, use `docker daemon -D`.

## Daemon socket option

The Docker daemon can listen for [Docker Remote API](../api/docker_remote_api.md)
requests via three different types of Socket: `unix`, `tcp`, and `fd`.

By default, a `unix` domain socket (or IPC socket) is created at
`/var/run/docker.sock`, requiring either `root` permission, or `docker` group
membership.

If you need to access the Docker daemon remotely, you need to enable the `tcp`
Socket. Beware that the default setup provides un-encrypted and
un-authenticated direct access to the Docker daemon - and should be secured
either using the [built in HTTPS encrypted socket](../../security/https/), or by
putting a secure web proxy in front of it. You can listen on port `2375` on all
network interfaces with `-H tcp://0.0.0.0:2375`, or on a particular network
interface using its IP address: `-H tcp://192.168.59.103:2375`. It is
conventional to use port `2375` for un-encrypted, and port `2376` for encrypted
communication with the daemon.

> **Note:**
> If you're using an HTTPS encrypted socket, keep in mind that only
> TLS1.0 and greater are supported. Protocols SSLv3 and under are not
> supported anymore for security reasons.

On Systemd based systems, you can communicate with the daemon via
[Systemd socket activation](http://0pointer.de/blog/projects/socket-activation.html),
use `docker daemon -H fd://`. Using `fd://` will work perfectly for most setups but
you can also specify individual sockets: `docker daemon -H fd://3`. If the
specified socket activated files aren't found, then Docker will exit. You can
find examples of using Systemd socket activation with Docker and Systemd in the
[Docker source tree](https://github.com/docker/docker/tree/master/contrib/init/systemd/).

You can configure the Docker daemon to listen to multiple sockets at the same
time using multiple `-H` options:

    # listen using the default unix socket, and on 2 specific IP addresses on this host.
    docker daemon -H unix:///var/run/docker.sock -H tcp://192.168.59.106 -H tcp://10.10.10.2

The Docker client will honor the `DOCKER_HOST` environment variable to set the
`-H` flag for the client.

    $ docker -H tcp://0.0.0.0:2375 ps
    # or
    $ export DOCKER_HOST="tcp://0.0.0.0:2375"
    $ docker ps
    # both are equal

Setting the `DOCKER_TLS_VERIFY` environment variable to any value other than
the empty string is equivalent to setting the `--tlsverify` flag. The following
are equivalent:

    $ docker --tlsverify ps
    # or
    $ export DOCKER_TLS_VERIFY=1
    $ docker ps

The Docker client will honor the `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY`
environment variables (or the lowercase versions thereof). `HTTPS_PROXY` takes
precedence over `HTTP_PROXY`.

### Daemon storage-driver option

The Docker daemon has support for several different image layer storage
drivers: `aufs`, `devicemapper`, `btrfs`, `zfs` and `overlay`.

The `aufs` driver is the oldest, but is based on a Linux kernel patch-set that
is unlikely to be merged into the main kernel. These are also known to cause
some serious kernel crashes. However, `aufs` is also the only storage driver
that allows containers to share executable and shared library memory, so is a
useful choice when running thousands of containers with the same program or
libraries.

The `devicemapper` driver uses thin provisioning and Copy on Write (CoW)
snapshots. For each devicemapper graph location – typically
`/var/lib/docker/devicemapper` – a thin pool is created based on two block
devices, one for data and one for metadata. By default, these block devices
are created automatically by using loopback mounts of automatically created
sparse files. Refer to [Storage driver options](#storage-driver-options) below
for a way how to customize this setup.
[~jpetazzo/Resizing Docker containers with the Device Mapper plugin](http://jpetazzo.github.io/2014/01/29/docker-device-mapper-resize/)
article explains how to tune your existing setup without the use of options.

The `btrfs` driver is very fast for `docker build` - but like `devicemapper`
does not share executable memory between devices. Use
`docker daemon -s btrfs -g /mnt/btrfs_partition`.

The `zfs` driver is probably not as fast as `btrfs` but has a longer track record
on stability. Thanks to `Single Copy ARC` shared blocks between clones will be
cached only once. Use `docker daemon -s zfs`. To select a different zfs filesystem
set `zfs.fsname` option as described in [Storage driver options](#storage-driver-options).

The `overlay` is a very fast union filesystem. It is now merged in the main
Linux kernel as of [3.18.0](https://lkml.org/lkml/2014/10/26/137). Call
`docker daemon -s overlay` to use it.

> **Note:**
> As promising as `overlay` is, the feature is still quite young and should not
> be used in production. Most notably, using `overlay` can cause excessive
> inode consumption (especially as the number of images grows), as well as
> being incompatible with the use of RPMs.

> **Note:**
> It is currently unsupported on `btrfs` or any Copy on Write filesystem
> and should only be used over `ext4` partitions.

### Storage driver options

Particular storage-driver can be configured with options specified with
`--storage-opt` flags. Options for `devicemapper` are prefixed with `dm` and
options for `zfs` start with `zfs`.

*  `dm.thinpooldev`

     Specifies a custom block storage device to use for the thin pool.

     If using a block device for device mapper storage, it is best to use `lvm`
     to create and manage the thin-pool volume. This volume is then handed to Docker
     to exclusively create snapshot volumes needed for images and containers.

     Managing the thin-pool outside of Docker makes for the most feature-rich
     method of having Docker utilize device mapper thin provisioning as the
     backing storage for Docker's containers. The highlights of the lvm-based
     thin-pool management feature include: automatic or interactive thin-pool
     resize support, dynamically changing thin-pool features, automatic thinp
     metadata checking when lvm activates the thin-pool, etc.

     As a fallback if no thin pool is provided, loopback files will be
     created. Loopback is very slow, but can be used without any
     pre-configuration of storage. It is strongly recommended that you do
     not use loopback in production. Ensure your Docker daemon has a
     `--storage-opt dm.thinpooldev` argument provided.

     Example use:

        $ docker daemon \
              --storage-opt dm.thinpooldev=/dev/mapper/thin-pool

*  `dm.basesize`

    Specifies the size to use when creating the base device, which limits the
    size of images and containers. The default value is 10G. Note, thin devices
    are inherently "sparse", so a 10G device which is mostly empty doesn't use
    10 GB of space on the pool. However, the filesystem will use more space for
    the empty case the larger the device is.

    The base device size can be increased at daemon restart which will allow
    all future images and containers (based on those new images) to be of the
    new base device size.

    Example use:

        $ docker daemon --storage-opt dm.basesize=50G

    This will increase the base device size to 50G. The Docker daemon will throw an
    error if existing base device size is larger than 50G. A user can use
    this option to expand the base device size however shrinking is not permitted.

    This value affects the system-wide "base" empty filesystem
    that may already be initialized and inherited by pulled images. Typically,
    a change to this value requires additional steps to take effect:

        $ sudo service docker stop
        $ sudo rm -rf /var/lib/docker
        $ sudo service docker start

    Example use:

        $ docker daemon --storage-opt dm.basesize=20G

*  `dm.loopdatasize`

    > **Note**:
	> This option configures devicemapper loopback, which should not
	> be used in production.

    Specifies the size to use when creating the loopback file for the
    "data" device which is used for the thin pool. The default size is
    100G. The file is sparse, so it will not initially take up this
    much space.

    Example use:

        $ docker daemon --storage-opt dm.loopdatasize=200G

*  `dm.loopmetadatasize`

    > **Note**:
    > This option configures devicemapper loopback, which should not
    > be used in production.

    Specifies the size to use when creating the loopback file for the
    "metadata" device which is used for the thin pool. The default size
    is 2G. The file is sparse, so it will not initially take up
    this much space.

    Example use:

        $ docker daemon --storage-opt dm.loopmetadatasize=4G

*  `dm.fs`

    Specifies the filesystem type to use for the base device. The supported
    options are "ext4" and "xfs". The default is "xfs"

    Example use:

        $ docker daemon --storage-opt dm.fs=ext4

*  `dm.mkfsarg`

    Specifies extra mkfs arguments to be used when creating the base device.

    Example use:

        $ docker daemon --storage-opt "dm.mkfsarg=-O ^has_journal"

*  `dm.mountopt`

    Specifies extra mount options used when mounting the thin devices.

    Example use:

        $ docker daemon --storage-opt dm.mountopt=nodiscard

*  `dm.datadev`

    (Deprecated, use `dm.thinpooldev`)

    Specifies a custom blockdevice to use for data for the thin pool.

    If using a block device for device mapper storage, ideally both datadev and
    metadatadev should be specified to completely avoid using the loopback
    device.

    Example use:

        $ docker daemon \
              --storage-opt dm.datadev=/dev/sdb1 \
              --storage-opt dm.metadatadev=/dev/sdc1

*  `dm.metadatadev`

    (Deprecated, use `dm.thinpooldev`)

    Specifies a custom blockdevice to use for metadata for the thin pool.

    For best performance the metadata should be on a different spindle than the
    data, or even better on an SSD.

    If setting up a new metadata pool it is required to be valid. This can be
    achieved by zeroing the first 4k to indicate empty metadata, like this:

        $ dd if=/dev/zero of=$metadata_dev bs=4096 count=1

    Example use:

        $ docker daemon \
              --storage-opt dm.datadev=/dev/sdb1 \
              --storage-opt dm.metadatadev=/dev/sdc1

*  `dm.blocksize`

    Specifies a custom blocksize to use for the thin pool. The default
    blocksize is 64K.

    Example use:

        $ docker daemon --storage-opt dm.blocksize=512K

*  `dm.blkdiscard`

    Enables or disables the use of blkdiscard when removing devicemapper
    devices. This is enabled by default (only) if using loopback devices and is
    required to resparsify the loopback file on image/container removal.

    Disabling this on loopback can lead to *much* faster container removal
    times, but will make the space used in `/var/lib/docker` directory not be
    returned to the system for other use when containers are removed.

    Example use:

        $ docker daemon --storage-opt dm.blkdiscard=false

*  `dm.override_udev_sync_check`

    Overrides the `udev` synchronization checks between `devicemapper` and `udev`.
    `udev` is the device manager for the Linux kernel.

    To view the `udev` sync support of a Docker daemon that is using the
    `devicemapper` driver, run:

        $ docker info
        [...]
        Udev Sync Supported: true
        [...]

    When `udev` sync support is `true`, then `devicemapper` and udev can
    coordinate the activation and deactivation of devices for containers.

    When `udev` sync support is `false`, a race condition occurs between
    the`devicemapper` and `udev` during create and cleanup. The race condition
    results in errors and failures. (For information on these failures, see
    [docker#4036](https://github.com/docker/docker/issues/4036))

    To allow the `docker` daemon to start, regardless of `udev` sync not being
    supported, set `dm.override_udev_sync_check` to true:

        $ docker daemon --storage-opt dm.override_udev_sync_check=true

    When this value is `true`, the  `devicemapper` continues and simply warns
    you the errors are happening.

    > **Note:**
    > The ideal is to pursue a `docker` daemon and environment that does
    > support synchronizing with `udev`. For further discussion on this
    > topic, see [docker#4036](https://github.com/docker/docker/issues/4036).
    > Otherwise, set this flag for migrating existing Docker daemons to
    > a daemon with a supported environment.

*  `dm.use_deferred_removal`

    Enables use of deferred device removal if `libdm` and the kernel driver
    support the mechanism.

    Deferred device removal means that if device is busy when devices are
    being removed/deactivated, then a deferred removal is scheduled on
    device. And devices automatically go away when last user of the device
    exits.

    For example, when a container exits, its associated thin device is removed.
    If that device has leaked into some other mount namespace and can't be
    removed, the container exit still succeeds and this option causes the
    system to schedule the device for deferred removal. It does not wait in a
    loop trying to remove a busy device.

    Example use:

        $ docker daemon --storage-opt dm.use_deferred_removal=true

*  `dm.use_deferred_deletion`

    Enables use of deferred device deletion for thin pool devices. By default,
    thin pool device deletion is synchronous. Before a container is deleted,
    the Docker daemon removes any associated devices. If the storage driver
    can not remove a device, the container deletion fails and daemon returns.

        Error deleting container: Error response from daemon: Cannot destroy container

    To avoid this failure, enable both deferred device deletion and deferred
    device removal on the daemon.

        $ docker daemon \
              --storage-opt dm.use_deferred_deletion=true \
              --storage-opt dm.use_deferred_removal=true

    With these two options enabled, if a device is busy when the driver is
    deleting a container, the driver marks the device as deleted. Later, when
    the device isn't in use, the driver deletes it.

    In general it should be safe to enable this option by default. It will help
    when unintentional leaking of mount point happens across multiple mount
    namespaces.

Currently supported options of `zfs`:

* `zfs.fsname`

    Set zfs filesystem under which docker will create its own datasets.
    By default docker will pick up the zfs filesystem where docker graph
    (`/var/lib/docker`) is located.

    Example use:

        $ docker daemon -s zfs --storage-opt zfs.fsname=zroot/docker

## Docker execdriver option

The Docker daemon uses a specifically built `libcontainer` execution driver as
its interface to the Linux kernel `namespaces`, `cgroups`, and `SELinux`.

## Options for the native execdriver

You can configure the `native` (libcontainer) execdriver using options specified
with the `--exec-opt` flag. All the flag's options have the `native` prefix. A
single `native.cgroupdriver` option is available.

The `native.cgroupdriver` option specifies the management of the container's
cgroups. You can specify `cgroupfs` or `systemd`. If you specify `systemd` and
it is not available, the system uses `cgroupfs`. If you omit the
`native.cgroupdriver` option,` cgroupfs` is used.
This example sets the `cgroupdriver` to `systemd`:

    $ sudo docker daemon --exec-opt native.cgroupdriver=systemd

Setting this option applies to all containers the daemon launches.

Also Windows Container makes use of `--exec-opt` for special purpose. Docker user
can specify default container isolation technology with this, for example:

    $ docker daemon --exec-opt isolation=hyperv

Will make `hyperv` the default isolation technology on Windows, without specifying
isolation value on daemon start, Windows isolation technology will default to `process`.

## Daemon DNS options

To set the DNS server for all Docker containers, use
`docker daemon --dns 8.8.8.8`.

To set the DNS search domain for all Docker containers, use
`docker daemon --dns-search example.com`.

## Insecure registries

Docker considers a private registry either secure or insecure. In the rest of
this section, *registry* is used for *private registry*, and `myregistry:5000`
is a placeholder example for a private registry.

A secure registry uses TLS and a copy of its CA certificate is placed on the
Docker host at `/etc/docker/certs.d/myregistry:5000/ca.crt`. An insecure
registry is either not using TLS (i.e., listening on plain text HTTP), or is
using TLS with a CA certificate not known by the Docker daemon. The latter can
happen when the certificate was not found under
`/etc/docker/certs.d/myregistry:5000/`, or if the certificate verification
failed (i.e., wrong CA).

By default, Docker assumes all, but local (see local registries below),
registries are secure. Communicating with an insecure registry is not possible
if Docker assumes that registry is secure. In order to communicate with an
insecure registry, the Docker daemon requires `--insecure-registry` in one of
the following two forms:

* `--insecure-registry myregistry:5000` tells the Docker daemon that
  myregistry:5000 should be considered insecure.
* `--insecure-registry 10.1.0.0/16` tells the Docker daemon that all registries
  whose domain resolve to an IP address is part of the subnet described by the
  CIDR syntax, should be considered insecure.

The flag can be used multiple times to allow multiple registries to be marked
as insecure.

If an insecure registry is not marked as insecure, `docker pull`,
`docker push`, and `docker search` will result in an error message prompting
the user to either secure or pass the `--insecure-registry` flag to the Docker
daemon as described above.

Local registries, whose IP address falls in the 127.0.0.0/8 range, are
automatically marked as insecure as of Docker 1.3.2. It is not recommended to
rely on this, as it may change in the future.

Enabling `--insecure-registry`, i.e., allowing un-encrypted and/or untrusted
communication, can be useful when running a local registry.  However,
because its use creates security vulnerabilities it should ONLY be enabled for
testing purposes.  For increased security, users should add their CA to their
system's list of trusted CAs instead of enabling `--insecure-registry`.

## Legacy Registries

Enabling `--disable-legacy-registry` forces a docker daemon to only interact with registries which support the V2 protocol.  Specifically, the daemon will not attempt `push`, `pull` and `login` to v1 registries.  The exception to this is `search` which can still be performed on v1 registries.

## Running a Docker daemon behind a HTTPS_PROXY

When running inside a LAN that uses a `HTTPS` proxy, the Docker Hub
certificates will be replaced by the proxy's certificates. These certificates
need to be added to your Docker host's configuration:

1. Install the `ca-certificates` package for your distribution
2. Ask your network admin for the proxy's CA certificate and append them to
   `/etc/pki/tls/certs/ca-bundle.crt`
3. Then start your Docker daemon with `HTTPS_PROXY=http://username:password@proxy:port/ docker daemon`.
   The `username:` and `password@` are optional - and are only needed if your
   proxy is set up to require authentication.

This will only add the proxy and authentication to the Docker daemon's requests -
your `docker build`s and running containers will need extra configuration to
use the proxy

## Default Ulimits

`--default-ulimit` allows you to set the default `ulimit` options to use for
all containers. It takes the same options as `--ulimit` for `docker run`. If
these defaults are not set, `ulimit` settings will be inherited, if not set on
`docker run`, from the Docker daemon. Any `--ulimit` options passed to
`docker run` will overwrite these defaults.

Be careful setting `nproc` with the `ulimit` flag as `nproc` is designed by Linux to
set the maximum number of processes available to a user, not to a container. For details
please check the [run](run.md) reference.

## Nodes discovery

The `--cluster-advertise` option specifies the `host:port` or `interface:port`
combination that this particular daemon instance should use when advertising
itself to the cluster. The daemon is reached by remote hosts through this value.
If you  specify an interface, make sure it includes the IP address of the actual
Docker host. For Engine installation created through `docker-machine`, the
interface is typically `eth1`.

The daemon uses [libkv](https://github.com/docker/libkv/) to advertise
the node within the cluster. Some key-value backends support mutual
TLS. To configure the client TLS settings used by the daemon can be configured
using the `--cluster-store-opt` flag, specifying the paths to PEM encoded
files. For example:

```bash
docker daemon \
    --cluster-advertise 192.168.1.2:2376 \
    --cluster-store etcd://192.168.1.2:2379 \
    --cluster-store-opt kv.cacertfile=/path/to/ca.pem \
    --cluster-store-opt kv.certfile=/path/to/cert.pem \
    --cluster-store-opt kv.keyfile=/path/to/key.pem
```

The currently supported cluster store options are:

*  `discovery.heartbeat`

    Specifies the heartbeat timer in seconds which is used by the daemon as a
    keepalive mechanism to make sure discovery module treats the node as alive
    in the cluster. If not configured, the default value is 20 seconds.

*  `discovery.ttl`

    Specifies the ttl (time-to-live) in seconds which is used by the discovery
    module to timeout a node if a valid heartbeat is not received within the
    configured ttl value. If not configured, the default value is 60 seconds.

*  `kv.cacertfile`

    Specifies the path to a local file with PEM encoded CA certificates to trust

*  `kv.certfile`

    Specifies the path to a local file with a PEM encoded certificate.  This
    certificate is used as the client cert for communication with the
    Key/Value store.

*  `kv.keyfile`

    Specifies the path to a local file with a PEM encoded private key.  This
    private key is used as the client key for communication with the
    Key/Value store.

*  `kv.path`

    Specifies the path in the Key/Value store. If not configured, the default value is 'docker/nodes'.

## Access authorization

Docker's access authorization can be extended by authorization plugins that your
organization can purchase or build themselves. You can install one or more
authorization plugins when you start the Docker `daemon` using the
`--authorization-plugin=PLUGIN_ID` option.

```bash
docker daemon --authorization-plugin=plugin1 --authorization-plugin=plugin2,...
```

The `PLUGIN_ID` value is either the plugin's name or a path to its specification
file. The plugin's implementation determines whether you can specify a name or
path. Consult with your Docker administrator to get information about the
plugins available to you.

Once a plugin is installed, requests made to the `daemon` through the command
line or Docker's remote API are allowed or denied by the plugin.  If you have
multiple plugins installed, at least one must allow the request for it to
complete.

For information about how to create an authorization plugin, see [authorization
plugin](../../extend/plugins_authorization.md) section in the Docker extend section of this documentation.


## Daemon user namespace options

The Linux kernel [user namespace support](http://man7.org/linux/man-pages/man7/user_namespaces.7.html) provides additional security by enabling
a process, and therefore a container, to have a unique range of user and
group IDs which are outside the traditional user and group range utilized by
the host system. Potentially the most important security improvement is that,
by default, container processes running as the `root` user will have expected
administrative privilege (with some restrictions) inside the container but will
effectively be mapped to an unprivileged `uid` on the host.

When user namespace support is enabled, Docker creates a single daemon-wide mapping
for all containers running on the same engine instance. The mappings will
utilize the existing subordinate user and group ID feature available on all modern
Linux distributions.
The [`/etc/subuid`](http://man7.org/linux/man-pages/man5/subuid.5.html) and
[`/etc/subgid`](http://man7.org/linux/man-pages/man5/subgid.5.html) files will be
read for the user, and optional group, specified to the `--userns-remap`
parameter.  If you do not wish to specify your own user and/or group, you can
provide `default` as the value to this flag, and a user will be created on your behalf
and provided subordinate uid and gid ranges. This default user will be named
`dockremap`, and entries will be created for it in `/etc/passwd` and
`/etc/group` using your distro's standard user and group creation tools.

> **Note**: The single mapping per-daemon restriction is in place for now
> because Docker shares image layers from its local cache across all
> containers running on the engine instance.  Since file ownership must be
> the same for all containers sharing the same layer content, the decision
> was made to map the file ownership on `docker pull` to the daemon's user and
> group mappings so that there is no delay for running containers once the
> content is downloaded. This design preserves the same performance for `docker
> pull`, `docker push`, and container startup as users expect with
> user namespaces disabled.

### Starting the daemon with user namespaces enabled

To enable user namespace support, start the daemon with the
`--userns-remap` flag, which accepts values in the following formats:

 - uid
 - uid:gid
 - username
 - username:groupname

If numeric IDs are provided, translation back to valid user or group names
will occur so that the subordinate uid and gid information can be read, given
these resources are name-based, not id-based.  If the numeric ID information
provided does not exist as entries in `/etc/passwd` or `/etc/group`, daemon
startup will fail with an error message.

> **Note:** On Fedora 22, you have to `touch` the `/etc/subuid` and `/etc/subgid`
> files to have ranges assigned when users are created.  This must be done
> *before* the `--userns-remap` option is enabled. Once these files exist, the
> daemon can be (re)started and range assignment on user creation works properly.

*Example: starting with default Docker user management:*

```bash
$ docker daemon --userns-remap=default
```

When `default` is provided, Docker will create - or find the existing - user and group
named `dockremap`. If the user is created, and the Linux distribution has
appropriate support, the `/etc/subuid` and `/etc/subgid` files will be populated
with a contiguous 65536 length range of subordinate user and group IDs, starting
at an offset based on prior entries in those files.  For example, Ubuntu will
create the following range, based on an existing user named `user1` already owning
the first 65536 range:

```bash
$ cat /etc/subuid
user1:100000:65536
dockremap:165536:65536
```

If you have a preferred/self-managed user with subordinate ID mappings already
configured, you can provide that username or uid to the `--userns-remap` flag.
If you have a group that doesn't match the username, you may provide the `gid`
or group name as well; otherwise the username will be used as the group name
when querying the system for the subordinate group ID range.

### Detailed information on `subuid`/`subgid` ranges

Given potential advanced use of the subordinate ID ranges by power users, the
following paragraphs define how the Docker daemon currently uses the range entries
found within the subordinate range files.

The simplest case is that only one contiguous range is defined for the
provided user or group. In this case, Docker will use that entire contiguous
range for the mapping of host uids and gids to the container process.  This
means that the first ID in the range will be the remapped root user, and the
IDs above that initial ID will map host ID 1 through the end of the range.

From the example `/etc/subuid` content shown above, the remapped root
user would be uid 165536.

If the system administrator has set up multiple ranges for a single user or
group, the Docker daemon will read all the available ranges and use the
following algorithm to create the mapping ranges:

1. The range segments found for the particular user will be sorted by *start ID* ascending.
2. Map segments will be created from each range in increasing value with a length matching the length of each segment. Therefore the range segment with the lowest numeric starting value will be equal to the remapped root, and continue up through host uid/gid equal to the range segment length. As an example, if the lowest segment starts at ID 1000 and has a length of 100, then a map of 1000 -> 0 (the remapped root) up through 1100 -> 100 will be created from this segment. If the next segment starts at ID 10000, then the next map will start with mapping 10000 -> 101 up to the length of this second segment. This will continue until no more segments are found in the subordinate files for this user.
3. If more than five range segments exist for a single user, only the first five will be utilized, matching the kernel's limitation of only five entries in `/proc/self/uid_map` and `proc/self/gid_map`.

### User namespace known restrictions

The following standard Docker features are currently incompatible when
running a Docker daemon with user namespaces enabled:

 - sharing PID or NET namespaces with the host (`--pid=host` or `--net=host`)
 - sharing a network namespace with an existing container (`--net=container:*other*`)
 - sharing an IPC namespace with an existing container (`--ipc=container:*other*`)
 - A `--readonly` container filesystem (this is a Linux kernel restriction against remounting with modified flags of a currently mounted filesystem when inside a user namespace)
 - external (volume or graph) drivers which are unaware/incapable of using daemon user mappings
 - Using `--privileged` mode flag on `docker run`

In general, user namespaces are an advanced feature and will require
coordination with other capabilities. For example, if volumes are mounted from
the host, file ownership will have to be pre-arranged if the user or
administrator wishes the containers to have expected access to the volume
contents.

Finally, while the `root` user inside a user namespaced container process has
many of the expected admin privileges that go along with being the superuser, the
Linux kernel has restrictions based on internal knowledge that this is a user namespaced
process. The most notable restriction that we are aware of at this time is the
inability to use `mknod`. Permission will be denied for device creation even as
container `root` inside a user namespace.

## Miscellaneous options

IP masquerading uses address translation to allow containers without a public
IP to talk to other machines on the Internet. This may interfere with some
network topologies and can be disabled with `--ip-masq=false`.

Docker supports softlinks for the Docker data directory (`/var/lib/docker`) and
for `/var/lib/docker/tmp`. The `DOCKER_TMPDIR` and the data directory can be
set like this:

    DOCKER_TMPDIR=/mnt/disk2/tmp /usr/local/bin/docker daemon -D -g /var/lib/docker -H unix:// > /var/lib/docker-machine/docker.log 2>&1
    # or
    export DOCKER_TMPDIR=/mnt/disk2/tmp
    /usr/local/bin/docker daemon -D -g /var/lib/docker -H unix:// > /var/lib/docker-machine/docker.log 2>&1


## Default cgroup parent

The `--cgroup-parent` option allows you to set the default cgroup parent
to use for containers. If this option is not set, it defaults to `/docker` for
fs cgroup driver and `system.slice` for systemd cgroup driver.

If the cgroup has a leading forward slash (`/`), the cgroup is created
under the root cgroup, otherwise the cgroup is created under the daemon
cgroup.

Assuming the daemon is running in cgroup `daemoncgroup`,
`--cgroup-parent=/foobar` creates a cgroup in
`/sys/fs/cgroup/memory/foobar`, whereas using `--cgroup-parent=foobar`
creates the cgroup in `/sys/fs/cgroup/memory/daemoncgroup/foobar`

The systemd cgroup driver has different rules for `--cgroup-parent`. Systemd
represents hierarchy by slice and the name of the slice encodes the location in
the tree. So `--cgroup-parent` for systemd cgroups should be a slice name. A
name can consist of a dash-separated series of names, which describes the path
to the slice from the root slice. For example, `--cgroup-parent=user-a-b.slice`
means the memory cgroup for the container is created in
`/sys/fs/cgroup/memory/user.slice/user-a.slice/user-a-b.slice/docker-<id>.scope`.

This setting can also be set per container, using the `--cgroup-parent`
option on `docker create` and `docker run`, and takes precedence over
the `--cgroup-parent` option on the daemon.

## Daemon configuration file

The `--config-file` option allows you to set any configuration option
for the daemon in a JSON format. This file uses the same flag names as keys,
except for flags that allow several entries, where it uses the plural
of the flag name, e.g., `labels` for the `label` flag. By default,
docker tries to load a configuration file from `/etc/docker/daemon.json`
on Linux and `%programdata%\docker\config\daemon.json` on Windows.

The options set in the configuration file must not conflict with options set
via flags. The docker daemon fails to start if an option is duplicated between
the file and the flags, regardless their value. We do this to avoid
silently ignore changes introduced in configuration reloads.
For example, the daemon fails to start if you set daemon labels
in the configuration file and also set daemon labels via the `--label` flag.

Options that are not present in the file are ignored when the daemon starts.
This is a full example of the allowed configuration options in the file:

```json
{
	"authorization-plugins": [],
	"dns": [],
	"dns-opts": [],
	"dns-search": [],
	"exec-opts": [],
	"exec-root": "",
	"storage-driver": "",
	"storage-opts": "",
	"labels": [],
	"log-driver": "",
	"log-opts": [],
	"mtu": 0,
	"pidfile": "",
	"graph": "",
	"cluster-store": "",
	"cluster-store-opts": [],
	"cluster-advertise": "",
	"debug": true,
	"hosts": [],
	"log-level": "",
	"tls": true,
	"tlsverify": true,
	"tlscacert": "",
	"tlscert": "",
	"tlskey": "",
	"api-cors-headers": "",
	"selinux-enabled": false,
	"userns-remap": "",
	"group": "",
	"cgroup-parent": "",
	"default-ulimits": {},
	"ipv6": false,
	"iptables": false,
	"ip-forward": false,
	"ip-mask": false,
	"userland-proxy": false,
	"ip": "0.0.0.0",
	"bridge": "",
	"bip": "",
	"fixed-cidr": "",
	"fixed-cidr-v6": "",
	"default-gateway": "",
	"default-gateway-v6": "",
	"icc": false,
	"raw-logs": false
}
```

### Configuration reloading

Some options can be reconfigured when the daemon is running without requiring
to restart the process. We use the `SIGHUP` signal in Linux to reload, and a global event
in Windows with the key `Global\docker-daemon-config-$PID`. The options can
be modified in the configuration file but still will check for conflicts with
the provided flags. The daemon fails to reconfigure itself
if there are conflicts, but it won't stop execution.

The list of currently supported options that can be reconfigured is this:

- `debug`: it changes the daemon to debug mode when set to true.
- `cluster-store`: it reloads the discovery store with the new address.
- `cluster-store-opts`: it uses the new options to reload the discovery store.
- `cluster-advertise`: it modifies the address advertised after reloading.
- `labels`: it replaces the daemon labels with a new set of labels.

Updating and reloading the cluster configurations such as `--cluster-store`,
`--cluster-advertise` and `--cluster-store-opts` will take effect only if
these configurations were not previously configured. If `--cluster-store`
has been provided in flags and `cluster-advertise` not, `cluster-advertise`
can be added in the configuration file without accompanied by `--cluster-store`
Configuration reload will log a warning message if it detects a change in
previously configured cluster configurations.
