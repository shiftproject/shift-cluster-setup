# Shift Cluster Setup

This script can be used to setup and install the IPFS daemon and Phoenix cluster daemon (IPFS pin manager) for Shift.

## Prerequisites

*Note: This has only been tested on Ubuntu. Make sure you are using a clean install. If you had previously installed shift-cluster for testing purposes you will have to run `shift-cluster remove` before `shift-cluster install`.*

Before running the script you should make sure your system is up to date and install dependencies as root:

```
apt-get update && apt-get -y upgrade && apt-get -y dist-upgrade
apt-get -y install git nano ufw build-essential jq
```

You can optionally create a shift user to run as

```
useradd -s /bin/bash -m shift
```

Give the shift user sudo access by running `sudo visudo` and adding `shift ALL=(ALL) NOPASSWD:ALL` to the sudoers file.

Then switch to the shift user

```
sudo su - shift
```

And make sure the following ports are open on the firewall:

```
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 4001/tcp
sudo ufw allow 3473/tcp
sudo ufw enable
```

## Installation

To install or update the `shift-cluster` command on linux run the following:

```
curl https://install.shiftproject.com | env=testnet bash
```

This will place it in the bin directory of your home folder and make it executable. Make sure that `$HOME/bin` is included in your $PATH.

### Commands

These are the available commands from `shift-cluster`.

#### Install

```
shift-cluster install
```

This installs the latest IPFS and phoenix-cluster binaries to run a Shift storage node.

#### Update

```
shift-cluster update
```

This will ensure that the latest IPFS and phoenix-cluster binaries are up to date, and if they are not, it will download the latest ones.

Note that it will not immediately start running the new code. For that you will need to run `shift-cluster restart`.

#### Remove

```
shift-cluster remove
```

Removes all of the files installed as part of the installation process. Note that this will also remove any content pinned in your IPFS storage.

#### Start

```
shift-cluster start
```

Starts up the IPFS daemon and cluster instance, connects to the Shift cluster, and starts up a local proxy to forward HTTP requests.

#### Stop

```
shift-cluster stop
```

Stops the proxy and IPFS instance and disconnects from the cluster.

#### Restart

```
shift-cluster restart
```

Stops then starts the running instance.

#### Check

```
shift-cluster check
```

Checks if IPFS is running.

## Help

If you are running into issues and need help, jump into the Shift discord channel: https://discord.gg/fgzxABX
