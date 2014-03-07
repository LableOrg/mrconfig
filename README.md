Mr. Config
==========

Mr. Config is a simple, command-line, ZooKeeper config management tool. We use
it as a devops tool to upload configuration files (YAML files in our case, but
anything goes) to a ZooKeeper quorum.

*Our Java WAR applications hosted on a farm of Tomcat servers maintain watches
on the ZooKeeper `znode` that corresponds with their contextname. Whenever their
configuration node is updated by Mr. Config, they update their internal
configation with the new node content.*

## Usage

Running `mrconfig` without arguments lists all available options:

```
usage: mrconfig [-h] [--version] [-v]
                [-l | --set NAME FILE | --get NAME | --delete NAME]
                [--zookeepers ZOOKEEPERS] [--config-znode CONFIG_ZNODE]

Manage configuration files stored in a ZooKeeper quorum.

optional arguments:
  -h, --help            Show this help message and exit.
  --version             Show program's version number and exit.
  -v, --verbose         Be a bit more chatty about what we're doing.
  -l, --list            List all the configs stored in the ZooKeeper quorum.
  --set NAME FILE       Upload a configuration file and place it under this
                        name.
  --get NAME            Download named config and output it to stdout.
  --delete NAME         Delete named config. This action cannot be undone!
  --zookeepers ZOOKEEPERS
                        Comma-separated address of the ZooKeeper quorum.
  --config-znode CONFIG_ZNODE
                        Path to the root configuration znode.

To write the contents of stdin to a configuration node, do something like this: 

    cat MyConf.yml | mrconfig --set NAME -

```

You might retrieve a list of all configuration nodes stored under a znode
called `/myconfigs/` like so:

```
# mrconfig --list --zookeepers zk1,zk2,zk3 --config-znode /myconfigs/
/myconfigs/
 ├ customer_x
 └ customer_y
```

Mr. Config will not create znode the node used to store the configuration files
for you (`/myconfigs/` in this example). You will have to create it yourself
using a tool such as `zkcli`.

Uploading and downloading the contents of a configuration node is as simple as
calling `mrconfig` with `--get MyConfig` or `--set MyConfig
localconfigfile.yml`.

### Settings

Mr. Config will attempt to read both the `--zookeepers` and `--config-znode`
options from settings files in either `~/.mrconfig` or `/etc/mrconfig.conf`.
For the above example, this file would look like this:

```
zookeeper_quorum: zk1,zk2,zk3
configuration_znode: /myconfigs/
``` 

## Caveats

This tool was written without any consideration for security. If you have
access to your ZooKeeper quorum, you can mutate znodes. Mr. Config can, in
essence, add, change, or remove any ZooKeeper node. Using a settings file will
prevent accidentally touching znodes you do not want changed, but it won't
prevent anyone with access to the tool from attempting to do so. Caveat emptor.

ZooKeeper does have ways to limit access to only a subset of znodes, but we
didn't have the need to do add support for this to Mr. Config yet.

## Python dependencies

Mr. Config has only a few dependencies. If you have Python installed (this very
simple tool should work in Python 2.6+) installing the dependencies should be
fairly straight forward.

### Debian, Ubuntu, and similar

```
sudo apt-get install python-pip
sudo pip install kazoo
sudo pip install argparse
```

### Mac OS X

```
sudo easy_install pip
sudo pip install kazoo
sudo pip install argparse
```

### Other operating systems

If you've successfully ran Mr. Config on an OS not listed here, please
let us know what you did, and we'll gladly add the instructions to the list.

