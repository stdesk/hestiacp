# Proposals for HestiaCP

## Hestia installation

* Split `hestia` package into `hestia-cli` and `hestia-web`.
* Downloading and installing `hestia-cli` should be enough to have CLI Hestia working (no configuration required).
    * Setup and unattended install can continue once `hestia-cli` is installed.
* Download and install `hestia-web` to have basic web panel (no configuration required).
    * Interactive setup can continue from Hestia web panel.
    * `hestia-web` will depend on `hestia-cli`, `hestia-nginx` and `hestia-php`.

### In short

* Separate install and config.
* Install Hestia packages first with default config.
* Setup everything after install (as usual on Linux).

### Proposed installation process

#### CLI

```bash
apt-get install -y hestia-cli.deb
# start using v-commands
```

With optional steps:

```bash
apt-get install -y hestia-cli.deb
v-set-admin-password admin_email admin_password   # or a random password could be generated and
                                                  # given to the user during package installation
v-module-install web-server --server-apache --backend-nginx
v-module-install mail-server --with-av --with-antispam
v-module-install name-server
...
```

#### Web panel

```bash
apt-get install -y hestia-cli.deb hestia-web.deb hestia-php.deb hestia-nginx.deb
# Continue from web panel
# (with randomly generated password, or use v-set-admin-password)
```

## Module installation

Note: I use the term module to refer to subsystems that can be present or absent, enabled or disabled, installed or uninstalled (mail, database, FTP, etc.).

* There should be one installer per module (separation of tasks, and easier to test and maintain).
* Modules could be installed from the Hestia CLI or web panel.
* Module installation should be idempotent.
* Module removal should be possible.

Term could be module, component, add-on or whatever.

### Idempotent module installation

Installing an already installed module should have no impact on a system with a correct state. It should fix things on a system with an incorrect state (i.e. not configured correctly).

This can, among other things, help minimize the divergence between fresh installs and upgraded systems.

This will also make Hestia developers' lives easier as it allows to debug module installation more easily and on heterogeneous systems.

See 'Rely less on a specific state' below.

# Module uninstallation

It should be possible to remove an installed module.

This will allow users to uninstall and reinstall a module to fix an issue and will also help Hestia developers in testing module installation.

Module removal shoudn't be difficult:

* Remove packages.
* Backup and delete all configuration files (even those not installed by Hestia).
* Clean any var files (logs, var data, etc.).
* Maybe rebuild other modules' config?

# Rely less on a specific state

Operations (like installation, upgrade or removal) should rely as little as possible on a specific system state.

Users modify or tweak teir systems. Sometimes, something fails and users fix things by hand. So not all systems are pristine.

For example, `v-add-web-php` relies on the presence of the folder `/etc/php7.2` as an indicator that proper PHP 7.2 support is in place. This might not be the case: that folder can be there because of a failed installation or uninstallation or any other reason.

Hestia needs to keep track of this changes, instead of searching for (unreliable) clues. For example, _after_ PHP 7.2 has been installed and completely setup, a value must be written to a file to indicate this.

## Specifics

* Rebuild or overwrite config files when possible (config files should have a text at the top "Autogenerated config - Don't edit this file, it will be overwritten').
* `crudini` when rebuilding/overwriting entire config is not possible (`crudini` will effectively set the desired value, overwriting it's current value or creating it if it doesn't exist, on the other hand, `sed` will change the value only if the value exist and matches exactly the given expression).
* Use `.d` configuration directories when possbible to allow users to customize configuration by adding `.conf` files instead of editing existing ones (which will be overwritten).