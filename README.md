# Docker container for CrashPlan
[![Docker Automated build](https://img.shields.io/docker/automated/jlesage/crashplan.svg)](https://hub.docker.com/r/jlesage/crashplan/) [![](https://images.microbadger.com/badges/image/jlesage/crashplan.svg)](http://microbadger.com/#/images/jlesage/crashplan "Get your own image badge on microbadger.com") [![Build Status](https://travis-ci.org/jlesage/docker-crashplan.svg?branch=master)](https://travis-ci.org/jlesage/docker-crashplan)

This is a Docker container for CrashPlan.  The GUI of the application is
accessed through a modern web browser (no installation or configuration needed
on client side) or via any VNC client.

---

<a href="https://www.crashplan.com"><img src="https://github.com/jlesage/docker-templates/raw/master/jlesage/images/crashplan-logo.png" alt="CrashPlan logo" width="60%" ></a>

CrashPlan makes it easy to protect your digital life, so you can get back to
what’s important in real life.  Only CrashPlan offers totally free local and
offsite backup. A subscription to the cloud backup service gets you continuous
backup, mobile file access and lots more. For the ultimate in computer backup,
get all three, from the same easy application.

---

## Quick Start
First create the configuration directory for CrashPlan.  In this example,
`/docker/appdata/crashplan` is used.  To backup files located under your home
directory, launch the CrashPlan docker container with the following command:
```
docker run -d --rm \
    --name=crashplan \
    -p 5800:5800 \
    -p 5900:5900 \
    -v /var/docker/crashplan:/config \
    -v $HOME:/storage:ro \
    jlesage/crashplan
```

Browse to `http://your-host-ip:5800` to access the CrashPlan Backup GUI.  Your
home directories and files appear under the `/storage` folder in the container.

## Usage
```
docker run [-d|--rm] \
    --name=crashplan \
    [-e <VARIABLE_NAME>=<VALUE>]... \
    [-v <HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]]... \
    [-p <HOST_PORT>:<CONTAINER_PORT>]... \
    jlesage/crashplan
```
| Parameter | Description |
|-----------|-------------|
| -d        | Run the container in background.  If not set, the container runs in foreground. |
| --rm      | Automatically remove the container when it exits. |
| -e        | Pass an environment variable to the container.  See the [Environment Variables](#environment-variables) section for more details. |
| -v        | Set a volume mapping (allows to share a folder/file between the host and the container).  See the [Data Volumes](#data-volumes) section for more details. |
| -p        | Set a network port mapping (exposes an internal container port to the host).  See the [Ports](#ports) section for more details. |

### Environment Variables

To customize some properties of the container, the following environment
variables can be passed via the `-e` parameter (one for each variable).  Value
of this parameter has the format `<VARIABLE_NAME>=<VALUE>`.

| Variable       | Description                                  | Default |
|----------------|----------------------------------------------|---------|
|`USER_ID`       | ID of the user the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | 1000    |
|`GROUP_ID`      | ID of the group the application runs as.  See [User/Group IDs](#usergroup-ids) to better understand when this should be set. | 1000    |
|`TZ`            | [TimeZone] of the container.  Timezone can also be set by mapping `/etc/localtime` between the host and the container. | Etc/UTC |
|`DISPLAY_WIDTH` | Width (in pixels) of the display.             | 1280    |
|`DISPLAY_HEIGHT`| Height (in pixels) of the display.            | 768     |
|`VNC_PASSWORD`  | Password needed to connect to the application's GUI.  See the [VNC Pasword](#vnc-password) section for more details. | (unset) |
|`KEEP_GUIAPP_RUNNING`| When set to `1`, the application will be automatically restarted if it crashes or if user quits it. | (unset) |
|`APP_NICENESS`  | Priority at which the application should run.  A niceness value of −20 is the highest priority and 19 is the lowest priority.  By default, niceness is not set, meaning that the default niceness of 0 is used.  **NOTE**: A negative niceness (priority increase) requires additional permissions.  In this case, the container should be run with the docker option `--cap-add=SYS_NICE`. | (unset) |

[TimeZone]: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

### Data Volumes

The following table describes data volumes used by the container.  The mappings
are set via the `-v` parameter.  Each mapping is specified with the following
format: `<HOST_DIR>:<CONTAINER_DIR>[:PERMISSIONS]`.

| Container path  | Permissions | Description |
|-----------------|-------------|-------------|
|`/config`        | rw          | This is where the application stores its configuration, log and any files needing persistency. |
|`/storage`       | ro          | This is where files that need to be backup are located. |
|`/backupArchives`| rw          | This is where inbound backups are stored. |

### Ports

Here is the list of ports used by the container.  They can be mapped to the host
via the `-p` parameter (one per port mapping).  Each mapping is defined in the
following format: `<HOST_PORT>:<CONTAINER_PORT>`.  The port number inside the
container cannot be changed, but you are free to use any port on the host side.

| Port | Mapping to host | Description |
|------|-----------------|-------------|
| 5800 | Mandatory       | Port used to access the application's GUI via the web interface. |
| 5900 | Mandatory       | Port used to access the application's GUI via the VNC protocol.  |
| 4242 | Optional        | Port used by CrashPlan for computer-to-computer backups.  No need to expose this port if this feature is not used. |
| 4243 | Mandatory       | Port used to connect to the CrashPlan service. |

## User/Group IDs

When using data volumes (`-v` flags), permissions issues can occur between the
host and the container.  For example, the user within the container may not
exists on the host.  This could prevent the host from properly accessing files
and folders on the shared volume.

To avoid any problem, you can specify the user the application should run as.

This is done by passing the user ID and group ID to the container via the
`USER_ID` and `GROUP_ID` environment variables.

To find the right IDs to use, issue the following command on the host, with the
user owning the data volume on the host:

    id <username>

Which gives an output like this one:
```
uid=1000(myuser) gid=1000(myuser) groups=1000(myuser),4(adm),24(cdrom),27(sudo),46(plugdev),113(lpadmin)
```

The value of `uid` (user ID) and `gid` (group ID) are the ones that you should
be given the container.

## Accessing the GUI

Assuming the host is mapped to the same ports as the container, the graphical
interface of the application can be accessed via:

  * A web browser:
```
http://<HOST IP ADDR>:5800
```

  * Any VNC client:
```
<HOST IP ADDR>:5900
```

If different ports are mapped to the host, make sure they respect the
following formula:

    VNC_PORT = HTTP_PORT + 100

This is to make sure accessing the GUI with a web browser can be done without
specifying the VNC port manually.  If this is not possible, then specify
explicitly the VNC port like this:

    http://<HOST IP ADDR>:5800/?port=<VNC PORT>

## VNC Password
To restrict access to your application, a password can be specified.  This can
be done via two methods:
  * By using the `VNC_PASSWORD` environment variable.
  * By creating a `.vncpass_clear` file at the root of the `/config` volume.
  This file should contains the password (in clear).  During the container
  startup, content of the file is obfuscated and renamed to `.vncpass`.

**NOTE**: This is a very basic way to restrict access to the application and it
should not be considered as secure in any way.

## Troubleshooting

If CrashPlan crashes unexpectedly with large backups, try to increase the
maximum amount of memory CrashPlan is allowed to use. See:

https://support.code42.com/CrashPlan/4/Troubleshooting/Adjusting_CrashPlan_Settings_For_Memory_Usage_With_Large_Backups
