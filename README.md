Mr. Config
==========

Mr. Config is a simple, command-line, Apache [ZooKeeper][] config management tool.
We use it as a devops tool to upload configuration files (YAML files in our case,
but anything goes) to a ZooKeeper quorum.

*Our Java WAR applications hosted on a farm of Tomcat servers maintain watches
on the ZooKeeper `znode` that corresponds with their contextname. Whenever their
configuration node is updated by Mr. Config, they update their internal
configation with the new node content.*

It is possible of course to manage ZooKeeper nodes with other tools, such as
`zkcli`, but we wanted a very simple tool with a few straight forward commands:

* Download a configuration file
* Upload or overwrite a configuration file
* List all configuration files
* Delete a configuration file

All of the heavy lifting is done by the [KaZoo][] library; a Python 
implementation of the ZooKeeper API.

## Usage

Running `mrconfig` without arguments lists all available options:

```
usage: mrconfig [-h] [--version] [-v] [-q]
                [-l | --set NAME FILE | --get NAME | --edit NAME | --move SOURCE TARGET | --copy SOURCE TARGET | --delete NAME | --delete-without-confirmation NAME]
                [--zookeepers ZOOKEEPERS] [--config-znode CONFIG_ZNODE]

Manage configuration files stored in a ZooKeeper quorum.

optional arguments:
  -h, --help            Show this help message and exit.
  --version             Show program's version number and exit.
  -v, --verbose         Be a bit more chatty about what we're doing.
  -q, --quiet           Suppress all output (overrides verbosity).
  -l, --list            List all the configs stored in the ZooKeeper quorum.
  --set NAME FILE       Upload a configuration file and place it under this
                        name.
  --get NAME            Download named config and output it to stdout.
  --edit NAME           Edit named config in an editor.
  --move SOURCE TARGET  Move a config to a new location.
  --copy SOURCE TARGET  Copy an existing config to another location.
  --delete NAME         Delete named config. This action cannot be undone!
  --delete-without-confirmation NAME
                        Delete named config without asking for confirmation.
                        This action cannot be undone!
  --zookeepers ZOOKEEPERS
                        Comma-separated address of the ZooKeeper quorum.
  --config-znode CONFIG_ZNODE
                        Path to the root configuration znode.

To write the contents of stdin to a configuration node, do something like this: 

    cat MyConf.yml | mrconfig --set NAME -
```

You can retrieve a list of all configuration nodes stored under a znode
called `/myconfigs/` like so:

```
# mrconfig --list --zookeepers zk1,zk2,zk3 --config-znode /myconfigs/
Configuration files (/myconfigs/):
 ├ customer_x
 └ customer_y
```

Mr. Config will not create the parent node used to store the configuration files
for you (`/myconfigs/` in this example). You will have to create it yourself
using a ZooKeeper tool such as `zkcli`.

Uploading and downloading the contents of a configuration node is as simple as
calling `mrconfig` with `--get MyConfig` or `--set MyConfig
localconfigfile.yaml`.

### Settings

Mr. Config will attempt to read both the `--zookeepers` and `--config-znode`
options from settings files in either `~/.mrconfig` or `/etc/mrconfig.conf`.
For the above example, this file would look like this:

```
zookeeper_quorum: zk1,zk2,zk3
configuration_znode: /myconfigs/
``` 

Alternatively, instead of configuring the `configuration_znode` separately, the
*chroot* functionality of ZooKeeper can be used by specifying the node to use as
part of the connect string:

```
zookeeper_quorum: zk1,zk2,zk3/myconfigs
```

After configuring Mr. Config in this manner, the basic operations look like this:

```bash
mrconfig --list
mrconfig --get NAME

mrconfig --set NAME FILE
# or,
cat FILE | mrconfig --set NAME -

mrconfig --delete NAME
mrconfig --move NAME_SRC NAME_TARGET
mrconfig --copy NAME_SRC NAME_TARGET
```

## Virtual file hierarchy

In order to manage a growing set of configuration files, it makes sense to
structure these using a directory/file-like hierarchy. Mr. Config supports this
in a special way: files can be stored in a hierarchy of 'directories':

```bash
mrconfig --set base.yaml base.yaml
mrconfig --set includes/defaults.yaml defaults.yaml
mrconfig -l

# Configuration files (/myconfigs/):
#  ├ base.yaml
#  └ includes
#     └ defaults.yaml
```

On the ZooKeeper quorum these *are not* stored as a hierarchy of ZooKeeper
nodes, but are all kept as nodes directly under the configuration parent node
configured. The reason for this is twofold:

* Keeping all configuration nodes as children of a single node allows for
  configuration libraries to set a single watch on the parent node to track new
  and deleted nodes
* ZooKeeper has no notion of directory-only nodes; a node with children can
  still contain its own content, leading to unintuitive situations (i.e. a 
  'directory' can contain data itself)

To store all configuration files under a single node, their virtual path is
converted to a single node name using `--` as a separator. So
`includes/defaults.yaml` is stored as `includes--defaults.yaml` in the ZooKeeper
database.

A benefit of this approach is that virtual 'directories' are automatically
'created' and 'deleted'.

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


[kazoo]: https://github.com/python-zk/kazoo "KaZoo"
[zookeeper]: http://zookeeper.apache.org/ "Apache ZooKeeper"
