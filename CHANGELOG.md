# v0.9.8

+ Add `assume_present` (default `False`) kwarg to `files.[file|directory|link]` operations
+ Accept lists for time kwargs in `server.crontab`
+ Fix `su` usage and support w/`shell_executable`
+ Fix/cleanup Docker containers on error
+ Move to `.readthedocs.yml`

# v0.9.7

+ Fix `@hook` functions by correctly checking `state.initialised`

# v0.9.6

+ Add `create_remote_dir` to `files.template` operation

# v0.9.5

+ Fix `apt_repos` fact when `/etc/apt/sources.list.d` doesn't exist
+ Fix parsing of apt repos with multiple option values

# v0.9.4

+ **Rename** `shell` global kwarg to `shell_executable`! (`server.user` uses `shell` already)
+ Add `create_remote_dir` arg to `files.put`, `files.file` and `files.link`

# v0.9.3

+ Add `update_submodules` and `recursive_submodules` args to `git.repo` operation (@chrissinclair)
+ Add `name` args to `server.crontab` operation to allow changing of the command
+ Add global `shell_exectuable` kwarg to all operations, defaults to `sh` as before

# v0.9.2

+ Improve parsing of `ifconfig` for `network_devices` fact (@gchazot)
+ Make printing unicode consistent between Python 2/3
+ Fix parsing Ansible inventories with left-padding in ranges

# v0.9.1

+ Fix for Python 3 (use `six` > `unicode`)

# v0.9

+ Add `@docker` connector, to create docker images
    * eg: `pyinfra @docker/ubuntu:bionic deploy.py`
    * this will spawn a container, execute operations on it and save it as an image
+ Add `linux_name` "short" fact
+ Add `allow_downgrades` keyword argument to `apt.packages`
+ [Experimental]: parse Ansible inventory files (ini format)
+ Handle template errors in arguments better
+ Capture/handle template syntax errors
+ Rename `config.TIMEOUT` -> `config.CONNECT_TIMEOUT` (old deprecated)
+ Properly handle logging unicode output
+ Fix execute `git fetch` before changing branch
+ Fix `find_in_file` fact for files with `~` in the name

Internal changes:
+ Remove the `AttrData` and all `Attr*` classes now we have operation ordering


# v0.8

+ Completely new operation ordering:
    * different args *will not* generate imbalanced operations!
    * no more deploy file compile needed

Internal changes:
+ Inline `sshuserclient` package (original no longer maintained)


# v0.7.1

+ Fix `deb_package` fact and don't assume we have a version in `apt.deb` operation

# v0.7

+ Add **mysql** module
    - Operations: `mysql.sql`, `mysql.user`, `mysql.database`, `mysql.privileges`, `mysql.dump`, `mysql.load`
    - Facts: `mysql_databases`, `mysql_users`, `mysql_user_grants`
+ Add **postgresql** module
    - Operations: `postgresql.sql`, `postgresql.role`, `postgresql.database`, `postgresql.dump`, `postgresql.load`
    - Facts: `postgresql_databases`, `postgresql_roles`
+ Add **puppet** module with `puppet.agent` operation (@tobald)
+ Add `server.crontab`, `server.modprobe` and `server.hostname` operations
+ Add `git.config` operation
+ Add `kernel_modules`, `crontab` and `git_config` facts
+ Add global install virtualenv support (like iPython)
+ Massively improved progress bar which highlights remaining hosts and tracks progress per operation or fact
+ Improved SSH config parsing, including proxyjump support (@tobald)
+ Support for CONFIG variables defined in `local.include` files
+ Fix `command` fact now outputs everything not just the first line

Internal changes:
+ **Replace** `--debug-state` with `--debug-operations` and `--debug-facts`
+ pyinfra now compiles the top-level scope of deploy code, meaning if statements no longer generate imbalanced operations
    * This means the recommendations to use `state.when` in place of conditional statements is invalid
    * Updated the warning shown, now once, with a link
    * Included a test `deploy_branches.py` which can be used to verify operations _do_ run in order for each host when compile is disabled
    * Compiling can be disabled by setting `PYINFRA_COMPILE=off` environment variable
+ **Deprecate** `state.limit` and replace with `state.hosts(hosts)` (consistency with global operation kwarg `hosts` not `limit`)
+ Major internal refactor of `AttrData` handling to reduce operation branching:
    * Generate `AttrData` on creation, rather than read
    * Add nesting support for `AttrData` so `host.data.thing['X']` will not create branching operations
    * Turn fact data into `AttrData`
    * Make `host.name` an `AttrDataStr`
    * Hash `True`, `False` and `None` constants as the same so they can change between hosts without branching operations
    * Update docs and warning on operation branching
+ Better default for pool parallel size
+ Show stdout if stderr is empty on command failure (surprisingly common)


# v0.6.1

+ Fix file "uploading" for the `@local` connector

# v0.6

+ Make `--limit` apply the limit similarly to `state.limit`
    - makes it possible to execute facts on hosts outside the `--limit`
    - `--limit` no longer alters the inventory, instead provides an "initial" state limit
+ Add `when=True` kwarg to `local.include`
+ Make it possible to add `data` to individual hosts in `@vagrant.json` configuration files
+ Add `memory` and `cpus` facts
+ Refactor how we track host state throughout deploy
+ Refactor facts to only gather missing ones (enabling partial gathering)
+ Improve check for valid `/etc/init.d/` services by looking for LSB header
+ Fix boolean constant detection with AST in Python3
+ Fix parsing ls output where `setgid` is set
+ Fix sudo/su file uploads with the `@local` connector


# v0.5.3

+ Fix writing unicode data with `@local`
+ Capture `IOError`s when SFTPing, note where remote disks might be full
+ Properly serialise `Host` objects for `--debug-state`

# v0.5.2

+ Add `exclude_dir` and `add_deploy_dir` kwargs to `files.sync`
+ Add pipfile for dev
+ Fix `files.put` when using `@local`

# v0.5.1

+ Make environment variables stick between multiple commands
+ Fix npm packages fact missing a return(!)

# v0.5

What was originally a module release for pyinfra (see the 0.6 milestone!) has become all about proper conditional branching support (previously resulted in best-effort/guess operation order) and improving 0.4's initial `@deploy` concept:

+ Add global `when` kwarg to all operations, similar to `hosts` can be used to prevent operations executing on hosts based on a condition
+ Add `state.limit(hosts)` and `state.when(condition)` context managers to use in place of `if` statements within deploys
+ `@deploy`s and the context managers (`state.limit`, `state.when`) can all be nested as much as needed (although if you need to nest a lot, you're probably doing it wrong!)
+ Add `data_defaults` kwarg to `@deploy` functions, meaning third party pyinfra packages can provide sensible defaults that the user can override individually
+ Display a large warning when imbalanced branches are detected, linking the user to the documentation for the above

Note that if statements/etc still work as before but pyinfra will print out a warning explaining the implications and linking to the docs (http://pyinfra.readthedocs.io/page/using_python.html#conditional-branches).

+ **Vagrant connector**:

```sh
# Run a deploy on all Vagrant machines (vagrant status list)
pyinfra @vagrant deploy.py
pyinfra @vagrant/vm_name deploy.py

# Can be used in tandem with other inventory:
pyinfra @vagrant,my-host.net deploy.py
pyinfra @vagrant,@local,my-host.net fact os
```

+ **Hooks broken**: no longer loaded from deploy files, only from `config.py`, due to changes from `0.4` (removal of `FakeState` nonsense)
+ Add `gpgkey` argument to the `yum.repo` operation
+ Add `lsb_release` fact
+ `apt_sources` fact now supports apt repos with options (`[arch=amd64]`)
+ Improved error output when connecting
+ Update testing box from Ubuntu 15 to Ubuntu 16
+ Ensure `~/.ssh` exists keyscanning in `ssh.keyscan`
+ Don't include tests during setup!
+ Fix caching of local SHA1s on files


# v0.4.1

+ Add `vzctl.unmount` operation (missing from 0.4!)
+ Add script to generate empty test files
+ Increase module test coverage significantly
+ Fix incorrect args in `vzctl.restart` operation
+ Fix `save=False` kwarg on `vzctl.set` not affecting command output (always saved)
+ Fix `gem.packages` install command

# v0.4

+ **Major change**: entirely new, streamlined CLI. Legacy support will remain for the next few releases. Usage is now:

```sh
# Run one or more deploys against the inventory
pyinfra INVENTORY deploy_web.py [deploy_db.py]...

# Run a single operation against the inventory
pyinfra INVENTORY server.user pyinfra,home=/home/pyinfra

# Execute an arbitrary command on the inventory
pyinfra INVENTORY exec -- echo "hello world"

# Run one or more facts on the inventory
pyinfra INVENTORY fact linux_distribution [users]...
```

+ **Major addition**: new `connectors` module that means hosts are no longer limited to SSH targets. Hostnames prefixed in `@` define which non-SSH connector to use. There is a new `local` connector for executing directly on the local machine, use hostname `@local`, eg:

```sh
pyinfra @local fact arch
```

+ **Major addition**: add `@deploy` wrapper for pyinfra related modules (eg [pyinfra-openstack](https://github.com/Oxygem/pyinfra-openstack)) to wrap a deploy (collection of operations) under one function, eg:

```py
from pyinfra.api import deploy
from pyinfra.modules import apt


@deploy('Install Openstack controller')
def install_openstack_controller(state, host):
    apt.packages(
        state, host,
        {'Install openstack-client'},
        ['openstack-client'],
    )
    ...
```

+ Add **SSH module** to execute SSH from others hosts: `ssh.keyscan`, `ssh.command`, `ssh.upload`, `ssh.download`
+ Add **vzctl module** to manage OpenVZ containers: `vzctl.create`, `vzctl.stop`, `vzctl.start`, `vzctl.restart`, `vzctl.delete`, `vzctl.set`
+ Add `on_success` and `on_error` callbacks to all operations (args = `(state, host, op_hash)`)
+ Add `server.script_template` operation
+ Add global `hosts` kwarg to all operations, working like `local.include`'s
+ Add `cache_time` kwarg to `apt.update` operation
+ Add `Inventory.get_group` and `Inventory.get_host`
+ Inventory `__len__` now (correctly) looks at active hosts, rather than all
+ Add `Inventory.len_all_hosts` to replace above bug/qwirk
+ Add progress spinner and % indicator to CLI
+ Replace `docopt`/`termcolor` with `click`
+ Moved `pyinfra.cli` to `pyinfra_cli` (internal breaking)
+ Switch to setuptools `entry_points` instead of distutils scripts
+ Expand Travis.ci testing to Python 3.6 and 3.7 nightly
+ Remove unused kwargs (`sudo`, `sudo_user`, `su_user`) from `pyinfra.api.facts.get_facts`

To-be-breaking changes (deprecated):

+ Deprecate `add_limited_op` function, use `hosts` kwarg on `add_op`
+ Deprecate group access via attribute and host access via index on `Inventory`
    * `Inventory.get_group` and `inventory.get_host` replace


# v0.3

+ Add `init.service` operation
+ Add `config.MIN_PYINFRA_VERSION`
+ Add `daemon_reload` to `init.systemd`
+ Add `pip` path to `pip.packages` (@hoh)
+ Add `virtualenv_kwargs` to `pip.packages`
+ Add `socket` fact
+ Display meta and results in groups
+ Fact arguments now parsed with jinja2 like operation args
+ Use full dates in `file`, `directory` and `link` facts
+ Improve `--run` check between operation and/or shell
+ Improve tests with facts that have multiple arguments
+ Fix how `pip.packages` handles pip path
+ Fix `yum.rpm` when downloading already installed rpm's
+ Fix `users` fact with users that have no home directory
+ Fix command overrides with dict objects (git.repo)
+ Removed compatibility for deprecated changes in v0.2


# v0.2.2

+ Fix bug in parsing of network interfaces
+ Fix `--limit` with a group name


# v0.2.1

+ Use wget & pipe when adding apt keys via URL, rather than `apt-key adv` which breaks with HTTPs
+ Fix bug where file-based group names were uppercased incorrectly (ie dev.py made group DEV, rather than dev)


# v0.2

New stuff:

+ Add LXD facts/module
+ Add iptables facts/module
+ Support usernames with non-standard characters (_, capitals, etc)
+ Add global `get_pty` kwarg for all operations to work with certain dodgy programs
+ Add `--fail-percent` CLI arg
+ Add `exclude` kwarg to `files.sync`
+ Enable `--limit` CLI arg to be multiple, comma separated, hostnames
+ Add `no_recommends` kwarg to `apt.packages` operation
+ Make local imports work like calling `python` by adding `.` to `sys.path` in CLI
+ Add key/value release meta to `linux_distribution` fact
+ Improve how the init module handles "unknown" services
+ Add `force` kwarg to `apt.packages` and `apt.deb` and don't `--force-yes` by default

To-be-breaking changes (deprecated):

+ Switch to lowercase inventory names (accessing `inventory.bsd` where the group is defined as `BSD = []` is deprecated)
+ Rename `yum.upgrade` -> `yum.update` (`yum.upgrade` deprecated)
+ Deprecate `pip_virtualenv_packages` fact as `pip_packages` will now accept an argument for the virtualenv
+ Deprecate `npm_local_packages` fact as `npm_packages` will accept an argument for the directory

Internal changes:

+ Operations now `yield`, rather than returning lists of commands


# v0.1.5

+ Fix `--run` arg parsing splutting up `[],`


# v0.1.4

+ Enable passing of multiple, comma separated hosts, as inventory
+ Use `getpass`, not `raw_input` for collecting key passwords in CLI mode


# v0.1.3

+ Fix issue when removing users that don't exist


# v0.1.2

+ Improve private key error handling
+ Ask for encrypted private key passwords in CLI mode


# v0.1.1

+ Don't generate set groups when `groups` is an empty list in `server.user`.


# v0.1

+ First versioned release, start of changelog
+ Full docs @ pyinfra.readthedocs.io
+ Core API with CLI built on top
+ Two-step deploy (diff state, exec commands)
+ Compatibility tested w/Ubuntu/CentOS/Debian/OpenBSD/Fedora
+ Modules/facts implemented:
    * Apt
    * Files
    * Gem
    * Git
    * Init
    * Npm
    * Pip
    * Pkg
    * Python
    * Server
    * Yum
