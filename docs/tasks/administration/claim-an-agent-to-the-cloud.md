<!--
title: "Claim an Agent to the Cloud"
sidebar_label: "Claim an Agent to the Cloud"
custom_edit_url: "https://github.com/netdata/netdata/blob/master/docs/tasks/general-configuration/claim-an-agent-to-the-cloud.md"
learn_status: "Published"
sidebar_position: 10
learn_topic_type: "Tasks"
learn_rel_path: "administration"
learn_docs_purpose: "Instructions on how to claim a node on a Space in the Cloud."
-->

You can securely connect a Netdata Agent, running on a distributed node, to Netdata Cloud.  
A Space's administrator creates a **claiming token**, which is used to add an Agent to their Space via
the [ACLK](https://github.com/netdata/netdata/blob/master/aclk/README.md).

When connecting an Agent (also referred to as a node) to the Cloud, you must complete a verification process that proves
you have some level of authorization to manage the node itself. This verification is a security feature that helps
prevent unauthorized users from seeing the data on your node.

:::info
While the data does flow through the Netdata Cloud servers on its way from Agents to the browser, we do not store or log
it in any way.
:::

Only the administrators of a Space in Netdata Cloud can view the claiming token and accompanying script that is
generated.

The connection process ensures no third party can add your node, and then view your node's metrics, in a Netdata Cloud
account, Space, or War Room that you did not authorize.

By connecting a node, you opt in to sending data from your Agent to the Cloud via
the [ACLK](https://github.com/netdata/netdata/blob/master/aclk/README.md). This data
is encrypted by TLS while it is in transit. We use the RSA keypair created during the connection process to authenticate
the identity of the Agent when it connects to the Cloud.

:::note

- _You can only connect any given node in a single Space_. You can, however, add that connected node to multiple War
  Rooms within that one Space.
- You must repeat the connection process on every node you want to add to Netdata Cloud.

:::

## Prerequisites

- A node with a Netdata Agent installed
- A Netdata Cloud account with a Space created

## Steps

There will be four main flows from where you can connect a node to Netdata Cloud:

- [When you are on an Empty War Room and you want to connect your first node](#empty-war-room)
- [When you are on the Admin interface's Manage Space tab](#admin-interface--manage-space-tab)
- [When you are on the Admin interface's War Room settings tab](#admin-interface--war-room-settings-tab)
- [When you are on the Nodes view page and want to connect a node](#nodes-view)

If there are extra steps needed for claiming on your environment, they will be stated on the claiming tab of the UI, and
you can keep reading for detailed guidance.

### Empty War Room

Either at your first sign in or following ones, when you enter Netdata Cloud and are at a War Room that doesn't have any
node added to it, you will be able to:

- Connect a new node to Netdata Cloud and add it to the War Room
- Add a previously connected node to the War Room

If your case is the former, you will need to select on which environment the node is running on (Linux, Docker, macOS,
Kubernetes) and then you will be provided with a script to initiate the connection process.

### Admin Interface / Manage Space tab

To connect a node:

1. Navigate to the **Nodes tab** from the sidebar, and click the **+** button
2. From the dropdown, select which War Rooms you want to connect this node into
3. Select the environment that the node is running on
4. Copy and paste the script on your terminal

### Admin Interface / War Room settings tab

This tab will have predefined the War Room which settings button you clicked.  
To connect a node:

1. Navigate from the top bar **War Room** view to the **Nodes** view
2. Click the **+** button
3. Select the environment that the node is running on
4. Copy and paste the script on your terminal

### Nodes View

From a War Room's Nodes View you can select to connect an Agent to this War Room.  
To do so:

1. Click the **+ Add nodes** button
2. Select the environment that the node is running on
3. Copy and paste the script on your terminal

## A more in depth look

### Claim a Linux Agent

If you want to connect a node that is running on a Linux environment, the script that will be provided to you by Netdata
Cloud is the `kickstart.sh` script which will install the Netdata Agent on your node, if it isn't already installed,
and connect the node to Netdata Cloud. It should be similar to:

```bash
wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh --claim-token TOKEN --claim-rooms ROOM1,ROOM2 --claim-url https://app.netdata.cloud
```

The script should return `Agent was successfully claimed.`. If the connecting to Netdata Cloud process returns errors,
or if you don't see the node in your Space after 60 seconds, see the [troubleshooting information](#troubleshooting).

:::note
Please note that to run the script you will either need to have root privileges or run it with the user that is running
the Agent, for more details, check the next section.
:::

For more details on what are the extra parameters `claim-token`, `claim-rooms` and `claim-url` please refer
to
the [kickstart script reference](https://github.com/netdata/netdata/blob/master/packaging/installer/methods/kickstart.md/packaging/installer/methods/kickstart.md)
.

#### Claim an Agent without root privileges

If you don't want to run the installation script to connect your nodes to Netdata Cloud with root privileges, you can
discover which user is running the Agent, switch to that user, and run the script.

Use `grep` to search your `netdata.conf` file, which is typically located at `/etc/netdata/netdata.conf`, for
the `run as user` setting.

```bash
grep "run as user" /etc/netdata/netdata.conf 
    # run as user = netdata
```

The default user is `netdata`. Yours may be different, so pay attention to the output from `grep`.

:::tip
Some operating systems will use `/opt/netdata/etc/netdata/` as the config directory. If you're not sure where yours is,
you can alternatively navigate to `http://NODE:19999/netdata.conf` in your browser, replacing NODE with the IP address
or hostname of your node, and find the `run as user =` setting this way.
:::

For example, to connect a node:

1. From the flows above, reach the claiming script tab
2. Switch to the user that is running the Agent
3. Then copy and paste the script given by Netdata Cloud into your node's terminal

#### Connect through a proxy

A Space's administrator can connect a node through HTTP(S) proxy.

You should first configure the proxy in the `[cloud]` section of `netdata.conf`. The proxy settings you specify here
will also be used to tunnel the ACLK. The default `proxy` setting is `none`.

```conf
[cloud]
    proxy = none
```

The `proxy` setting can take one of the following values:

- `none`: Do not use a proxy, even if the system configured otherwise.
- `env`: Try to read proxy settings from set environment variables `http_proxy`.
- `http://[user:pass@]host:ip`: The ACLK and connection process will use the specified HTTP(S) proxy.

For example, an HTTP proxy setting may look like the following:

```conf
[cloud]
    proxy = http://203.0.113.0:1080       # With an IP address
    proxy = http://proxy.example.com:1080 # With a URL
```

You can now move on to connecting. When you connect with the kickstart script, add the `--claim-proxy=` parameter and
append the same proxy setting you added to `netdata.conf`.

```bash
wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh --claim-token TOKEN --claim-rooms ROOM1,ROOM2 --claim-url https://app.netdata.cloud --claim-proxy http://[user:pass@]host:ip
```

Hit **Enter**. The script should return `Agent was successfully claimed.`. If the connecting to Netdata Cloud process
returns errors, or if you don't see the node in your Space after 60 seconds, see the
[troubleshooting information](#troubleshooting).

### Claim an Agent running in Docker

To connect an instance of a Netdata Agent running inside a Docker container, it is recommended that you follow the
instructions and use the commands provided either in the `Nodes` tab of an [empty War Room](#empty-war-room) on Netdata
Cloud or in the tab that appears when you click **Connect Nodes** and select **Docker**.

However, users can also claim a new node by using environment variables in the container to have it automatically
connected on startup or restart.

For the connection process to work, the contents of `/var/lib/netdata` _must_ be preserved across container restarts
using a persistent volume. You can read more at
our [recommended `docker run` and Docker Compose deployment Task](https://github.com/netdata/netdata/blob/master/docs/tasks/installation/deploy-netdata-in-a-host-with-docker-runtime.md#steps)
.

<details>
<summary><h4 id="known-issues-on-older-hosts-with-seccomp-enabled"> Known issues on older hosts with seccomp enabled </h4></summary>

The nodes running on the following hosts **cannot be claimed**:

- `libseccomp` version less than v2.3.3.
- Docker version less than v18.04.0-ce.
- The kernel is configured with CONFIG_SECCOMP enabled.

To check if your kernel supports `seccomp`:

```cmd
# grep CONFIG_SECCOMP= /boot/config-$(uname -r) 2>/dev/null || zgrep CONFIG_SECCOMP  /proc/config.gz 2>/dev/null
CONFIG_SECCOMP=y
```

To resolve the issue, do one of the following actions:

- Update to a newer version of Docker and `libseccomp` (recommended).
- Create a custom profile and pass it for the container.
- Run [without the default seccomp profile](https://docs.docker.com/engine/security/seccomp/#run-without-the-default-seccomp-profile) (unsafe, not recommended).

<h5><i>Creating a custom profile</i></h5>

1. Download the moby default seccomp profile and change `defaultAction` to `SCMP_ACT_TRACE` on line 2.

    ```cmd
    sudo wget https://raw.githubusercontent.com/moby/moby/master/profiles/seccomp/default.json -O /etc/docker/seccomp.json
    sudo sed -i '2s/SCMP_ACT_ERRNO/SCMP_ACT_TRACE/' /etc/docker/seccomp.json
    ```

2. Specify the new policy for the container explicitly.

    - When using `docker run`:

    ```cmd
    docker run -d --name=netdata \
      --security-opt=seccomp=/etc/docker/seccomp.json \
      ...
    ```

    - When using `docker-compose`:

   > :warning: The security_opt option is ignored when deploying a stack in swarm mode.

    ```yaml
    version: '3'
    services:
      netdata:
        security_opt:
          - seccomp:/etc/docker/seccomp.json
        ...
    ```

    - When using `docker stack deploy`:

      Change the default profile globally by adding `--seccomp-profile=/etc/docker/seccomp.json` to the options passed
      to dockerd on startup.

</details>

#### Using environment variables

The Netdata Docker container looks for the following environment variables on startup:

- `NETDATA_CLAIM_TOKEN`
- `NETDATA_CLAIM_URL`
- `NETDATA_CLAIM_ROOMS`
- `NETDATA_CLAIM_PROXY`

If the token and URL are specified in their corresponding variables _and_ the container is not already connected,
it will use these values to attempt to connect the container, automatically adding the node to the specified War
Rooms. If a proxy is specified, it will be used for the connection process and for connecting to Netdata Cloud.

These variables can be specified using any mechanism supported by your container tooling for setting environment
variables inside containers.

When using the `docker run` command, if you have an Agent container already running, it is important to know that there
will be a short period of downtime. This is due to the process of recreating the new Agent container.

The command to connect a new node to Netdata Cloud is:

```bash
docker run -d --name=netdata \
  -p 19999:19999 \
  -v netdataconfig:/etc/netdata \
  -v netdatalib:/var/lib/netdata \
  -v netdatacache:/var/cache/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /etc/os-release:/host/etc/os-release:ro \
  --restart unless-stopped \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  -e NETDATA_CLAIM_TOKEN=TOKEN \
  -e NETDATA_CLAIM_URL="https://app.netdata.cloud" \
  -e NETDATA_CLAIM_ROOMS=ROOM1,ROOM2 \
  -e NETDATA_CLAIM_PROXY=PROXY \
 netdata/netdata
```

:::note
This command is suggested for connecting a new container. Using this command for an existing container recreates
the container, though data and configuration of the old container may be preserved. If you are claiming an existing
container that can not be recreated, you can add the container by going to Netdata Cloud, clicking the **Nodes** tab,
clicking **Connect Nodes**, selecting **Docker**, and following the instructions and commands provided or by following
the instructions in an [empty War Room](#empty-war-room).
:::

The output that would be seen from the connection process when using other methods will be present in the container
logs.

Using the environment variables like this to handle the connection process is the preferred method of connecting Docker
containers as it works in the widest variety of situations and simplifies configuration management.

#### Using Docker compose

If you use `docker compose`, you can copy the config provided by Netdata Cloud, which should be same as the one below:

```bash
version: '3'
services:
  netdata:
    image: netdata/netdata
    container_name: netdata
  hostname: example.com # set to fqdn of host
  ports:
    - 19999:19999
  restart: unless-stopped
  cap_add:
    - SYS_PTRACE
  security_opt:
    - apparmor:unconfined
  volumes:
    - netdataconfig:/etc/netdata
    - netdatalib:/var/lib/netdata
    - netdatacache:/var/cache/netdata
    - /etc/passwd:/host/etc/passwd:ro
    - /etc/group:/host/etc/group:ro
    - /proc:/host/proc:ro
    - /sys:/host/sys:ro
    - /etc/os-release:/host/etc/os-release:ro
  environment:
    - NETDATA_CLAIM_TOKEN=TOKEN
    - NETDATA_CLAIM_URL="https://app.netdata.cloud"
    - NETDATA_CLAIM_ROOMS=ROOM1,ROOM2

volumes:
  netdataconfig:
  netdatalib:
  netdatacache:
```

Then run the following command in the same directory as the `docker-compose.yml` file to start the container.

```bash
docker-compose up -d
```

#### Using docker exec

Connect a _running Netdata Agent container_, where you don't want to recreate the existing container, append the script
offered by Netdata Cloud to a `docker exec ...` command, replacing
`netdata` with the name of your running container:

```bash
docker exec -it netdata netdata-claim.sh -token=TOKEN -rooms=ROOM1,ROOM2 -url=https://app.netdata.cloud
```

The values for `ROOM1,ROOM2` can be found by going to Netdata Cloud, clicking the **Nodes** tab, clicking **Connect
Nodes**, selecting **Docker**, and copying the `rooms=` value in the command provided.

The script should return `Agent was successfully claimed.`. If the connection process returns errors, or if
you don't see the node in your Space after 60 seconds, see the [troubleshooting information](#troubleshooting).

### Claim a Kubernetes cluster's parent Netdata pod

To start monitoring Kubernetes, you must first connect your Kubernetes cluster
to [Netdata Cloud](https://app.netdata.cloud). The connection process securely connects your Kubernetes cluster to
stream metrics data to Netdata Cloud, enabling Kubernetes-specific visualizations like the health map and time-series
composite charts.

#### New installations

If you have followed the flows listed above, you got a `helm install` command.

1. Prior to running it to the parent you need to add the netdata's helm repo, so you need to run:

    ```bash
    helm repo add netdata https://netdata.github.io/helmchart/
    ```

2. And then execute the claiming command:

    ```bash
    helm install netdata netdata/netdata --set parent.claiming.enabled="true" --set parent.claiming.token="TOKEN" --set parent.claiming.rooms="ROOM" --set child.claiming.enabled=true --set child.claiming.token="TOKEN" --set child.claiming.rooms="ROOM"
    ```

#### Existing installations

On an existing installation, you will need to override the configuration values by running the `helm upgrade` command
and provide a file with the values to override.

1. Start with creating a file called `override.yml`.

    ```bash
    touch override.yml
    ```
2. Paste the following into your `override.yml` file, replacing instances of `ROOM` and `TOKEN` with those from the
   script
   from Netdata Cloud. These settings connect your `parent`/`child` nodes to Netdata Cloud and store more metrics in the
   nodes' time-series databases.

    ```yaml
    parent:
      claiming:
        enabled: true
        token: "TOKEN"
        rooms: "ROOM"
    
    child:
      claiming:
        enabled: true
        token: "TOKEN"
        rooms: "ROOM"
      configs:
        netdata:
          data: |
            [global]
              memory mode = ram
              history = 3600
            [health]
              enabled = no
    ```

   :::caution
   These override settings, along with the Helm chart's defaults, will retain an hour's worth of
   metrics (`history = 3600`,
   or `3600 seconds`) on each child node. Based on your metrics retention needs, and the resources available on your
   cluster, you may want to increase the `history` setting.
   :::

3. Apply these new settings

    ```bash
    helm upgrade -f override.yml netdata netdata/netdata
    ```

The cluster terminates the old pods and creates new ones with the proper persistence and connection configuration.
You'll see your nodes, containers, and pods appear in Netdata Cloud in a few seconds.

## Troubleshooting

If you're having trouble connecting a node, this may be because
the [ACLK](https://github.com/netdata/netdata/blob/master/aclk/README.md) cannot connect to the Cloud.

With the Netdata Agent running, visit `http://NODE:19999/api/v1/info` in your browser, replacing `NODE` with the IP
address or hostname of your Agent. The returned JSON contains four keys that will be helpful to diagnose any issues you
might be having with the ACLK or connection process.

```json
"cloud-enabled"
"cloud-available"
"agent-claimed"
"aclk-available"
```

On Netdata Agent version `1.32` (`netdata -v` to find your version) and newer, the `netdata -W aclk-state` command can
be used to get some diagnostic information about ACLK. Sample output:

```
ACLK Available: Yes
ACLK Implementation: Next Generation
New Cloud Protocol Support: Yes
Claimed: Yes
Claimed Id: 53aa76c2-8af5-448f-849a-b16872cc4ba2
Online: Yes
Used Cloud Protocol: New
```

Use these keys and the information below to troubleshoot the ACLK.

### kickstart: unsupported Netdata installation

If you run the kickstart script and get the following
error `Existing install appears to be handled manually or through the system package manager.` you most probably
installed Netdata using an unsupported package.

If you are using an unsupported package, such as a third-party `.deb`/`.rpm` package provided by your distribution,
please remove that package and reinstall using
our [recommended express installation method](https://github.com/netdata/netdata/blob/master/docs/tasks/installation/express-installation-deploy-netdata-into-a-linux-unix-node-(via-kickstart).md#steps)
.

### kickstart: Failed to write new machine GUID

If you run the kickstart script but don't have privileges required for the actions done on the connecting to Netdata
Cloud process you will get the following error:

```bash
Failed to write new machine GUID. Please make sure you have rights to write to /var/lib/netdata/registry/netdata.public.unique.id.
```

For a successful execution you will need to run the script with root privileges or run it with the user that is running
the Agent.

### bash: netdata-claim.sh: command not found

If you run the claiming script and see a `command not found` error, you either installed Netdata in a non-standard
location or are using an unsupported package. If you installed Netdata in a non-standard path using the `--install`
option, you need to update your `$PATH` or run `netdata-claim.sh` using the full path. For example, if you installed
Netdata to `/opt/netdata`, use `/opt/netdata/bin/netdata-claim.sh` to run the claiming script.

If you are using an unsupported package, such as a third-party `.deb`/`.rpm` package provided by your distribution,
please remove that package and reinstall using
our [recommended express installation method](https://github.com/netdata/netdata/blob/master/docs/tasks/installation/express-installation-deploy-netdata-into-a-linux-unix-node-(via-kickstart).md#steps)
.

### Connecting on older distributions (Ubuntu 14.04, Debian 8, CentOS 6)

If you're running an older Linux distribution or one that has reached EOL, such as Ubuntu 14.04 LTS, Debian 8, or CentOS
6, your Agent may not be able to securely connect to Netdata Cloud due to an outdated version of OpenSSL. These old
versions of OpenSSL cannot perform [hostname validation](https://wiki.openssl.org/index.php/Hostname_validation), which
helps securely encrypt SSL connections.

We recommend you reinstall Netdata with
a [static build](https://github.com/netdata/netdata/blob/master/packaging/installer/methods/kickstart.md#static-builds),
which uses an up-to-date version of OpenSSL with hostname validation enabled.

If you choose to continue using the outdated version of OpenSSL, your node will still connect to Netdata Cloud, albeit
with hostname verification disabled. Without verification, your Netdata Cloud connection could be vulnerable to
man-in-the-middle attacks.

### cloud-enabled is false

If `cloud-enabled` is `false`, you probably ran the installer with `--disable-cloud` option.

Additionally, check that the `enabled` setting in `var/lib/netdata/cloud.d/cloud.conf` is set to `true`:

```conf
[global]
    enabled = true
```

To fix this issue, reinstall Netdata using
your [preferred method](https://github.com/netdata/netdata/blob/master/docs/tasks/installation/express-installation-deploy-netdata-into-a-linux-unix-node-(via-kickstart).md)
and do not add the `--disable-cloud` option.

### cloud-available is false / ACLK Available: No

If `cloud-available` is `false` after you verified Cloud is enabled in the previous step, the most likely issue is that
Cloud features failed to build during installation.

If Cloud features fail to build, the installer continues and finishes the process without Cloud functionality as opposed
to failing the installation altogether. We do this to ensure the Agent will always finish installing.

If you can't see an explicit error in the installer's output, you can run the installer with the `--require-cloud`
option. This option causes the installation to fail if Cloud functionality can't be built and enabled, and the
installer's output should give you more error details.

You may see one of the following error messages during installation:

- `Failed to build libmosquitto. The install process will continue, but you will not be able to connect this node to
  Netdata Cloud.`
- `Unable to fetch sources for libmosquitto. The install process will continue, but you will not be able to connect
  this node to Netdata Cloud.`
- `Failed to build libwebsockets. The install process will continue, but you may not be able to connect this node to
  Netdata Cloud.`
- `Unable to fetch sources for libwebsockets. The install process will continue, but you may not be able to connect
  this node to Netdata Cloud.`
- `Could not find cmake, which is required to build libwebsockets. The install process will continue, but you may not
  be able to connect this node to Netdata Cloud.`
- `Could not find cmake, which is required to build JSON-C. The install process will continue, but Netdata Cloud
  support will be disabled.`
- `Failed to build JSON-C. Netdata Cloud support will be disabled.`
- `Unable to fetch sources for JSON-C. Netdata Cloud support will be disabled.`

One common cause of the installer failing to build Cloud features is not having one of the following dependencies on
your system: `cmake`, `json-c` and `OpenSSL`, including corresponding `devel` packages.

You can also look for error messages in `/var/log/netdata/error.log`. Try one of the following two commands to search
for ACLK-related errors.

```bash
less /var/log/netdata/error.log
grep -i ACLK /var/log/netdata/error.log
```

If the installer's output does not help you enable Cloud features, contact us by [creating an issue on
GitHub](https://github.com/netdata/netdata/issues/new?assignees=&labels=bug%2Cneeds+triage&template=BUG_REPORT.yml&title=The+installer+failed+to+prepare+the+required+dependencies+for+Netdata+Cloud+functionality)
with details about your system and relevant output from `error.log`.

### Agent-claimed is false / Claimed: No

You must [connect your node](#steps).

### aclk-available is false / Online: No

If `aclk-available` is `false` and all other keys are `true`, your Agent is having trouble connecting to the Cloud
through the ACLK. Please check your system's firewall.

If your Agent needs to use a proxy to access the internet, you must [set up a proxy for
connecting](#connect-through-a-proxy).

If you are certain firewall and proxy settings are not the issue, you should consult the Agent's `error.log` at
`/var/log/netdata/error.log` and contact us by [creating an issue on
GitHub](https://github.com/netdata/netdata/issues/new?assignees=&labels=bug%2Cneeds+triage&template=BUG_REPORT.yml&title=ACLK-available-is-false)
with details about your system and relevant output from `error.log`.

## Related topics

1. [Kickstart script reference](https://github.com/netdata/netdata/blob/master/packaging/installer/methods/kickstart.md)
2. [ACLK reference](https://github.com/netdata/netdata/blob/master/aclk/README.md)