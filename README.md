# Autosetup - Automated system configuration

A flexible system that configures a newly installed linux machinge to your specifications.  Automates the bootstrapping of new systems with no dependencies beyond what is provided in a fresh Linux install.

## Overview

This system automates the secondary installation and configuration of multiple packages on Debian-based systems. It allows you to define a list of packages to install, along with any needed repositories, package sources, installation scripts, and cleanup actions. The installer handles both standard APT packages and downloadable .deb files, and can execute custom commands before and after installation. Packages can be tagged with flags to control installation order and behavior, making it especially useful for setting up consistent environments across multiple systems.

The package list is a text file containing package entries, where each entry starts with a package name at the beginning of a line. If the package name is followed by a colon, the subsequent indented lines contain directives that customize its installation; without a colon, the package is installed with default settings. Directives can include download URLs for DEB packages, APT repository information, source list entries, pre/post installation scripts, and flags that control installation behavior. Lines can be continued with a backslash, and comments start with '#'.

## Installation

Copy the script to a location in your PATH. Ensure you have Python 3.10 or later installed.

## Usage

The script must be run as root unless using --dryrun mode:

    sudo ./autosetup [options] package-list-file

### Options

- `-n, --dryrun`: Preview changes without installing anything
- `-p, --preserve`: Keep temporary working directory
- `-q, --quiet`: Suppress subprocess output
- `-v, --version`: Show program version
- `--force-all`: Force reinstallation of all packages
- `--only PKGS`: Only install specified packages
- `--only-flags FLAGS`: Only install packages with specified flags
- `--skip PKGS`: Skip specified packages
- `--skip-flags FLAGS`: Skip packages with specified flags

### Example Package List

Here's an example that demonstrates the various package list formats:

    # Install emacs
    emacs
    
    # Install atop
    atop
    
    # Install tcsh, add tcsh to /etc/shells
    tcsh:
      postscript: echo "/bin/tcsh" >> /etc/shells
    
    # Install code from a downloaded .deb file
    code:
      deb: https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64
    
    # Clean up from installation -- note that the 'no_apt' flag tells autosetup to not try 
    # to 'apt install' this package
    __termination_package:
      flags: end, no_apt
      prescript: apt -y autoremove

This example:
- Installs emacs and atop via APT with default settings
- Installs tcsh via APT and adds it to /etc/shells
- Downloads and installs Visual Studio Code from the official .deb package
- Runs apt autoremove after all installations complete

### Special Flags

- `start`: Install package before regular packages
- `end`: Install package after regular packages
- `force`: Always reinstall package
- `skip`: Skip this package
- `force_apt_update`: Force APT database update
- `no_apt`: Skip APT installation phase
- `prescript_run_once`: Run the prescript only once
- `postscript_run_once`: Run the postscript only once
- `script_run_once`: Run the pre- and postscript only once

### Available Directives

- `flags`: Comma-separated list of flags
- `repo`: APT repository to add
- `source`: APT source list entry (filename and content)
- `script` or `prescript`: Commands to run before installation
- `postscript`: Commands to run after installation
- `deb`: URL to download .deb package
- `apt`: Alternative package name for APT installation

## Examples

Install all packages:

    sudo ./autosetup packages.list

Preview installation:

    ./autosetup -n packages.list

Install specific packages:

    sudo ./autosetup --only package1 package2 packages.list

Install packages with specific flags:

    sudo ./autosetup --only-flags server packages.list