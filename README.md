# Shift Cluster Setup

This script can be used to setup and install IPFS and IPFS cluster for Shift.

## Prerequisites

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
sudo ufw allow 443/tcp
sudo ufw allow 4001/tcp
sudo ufw allow 9096/tcp
sudo ufw enable
```

## Running

After the prerequisites are done, you should copy the `shift-cluster` script to somewhere on your system inside your $PATH.

Or you can run it using

```
chmod +x shift-cluster
./shift-cluster [command]
```

### Commands

These are the available commands from `shift-cluster`.

#### Install

```
shift-cluster install
```

This installs the latest binaries to run a Shift IPFS cluster node.

#### Remove

```
shift-cluster remove
```

Removes the installed files from the system.

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

#### Check

```
shift-cluster check
```

Checks if IPFS is running.

## Help

If you are running into issues and need help, jump into the Shift ryver channel: https://shiftnrg.ryver.com/
