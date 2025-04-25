# Autosetup - Automated system configuration

A system that configures a newly installed linux machine to your specifications.  Automates the bootstrapping of new systems with no dependencies beyond what is provided in a fresh Linux install.

## Overview

This system automates the secondary installation and configuration of multiple packages on Debian-based systems. It allows you to define a list of packages to install, along with any needed repositories, package sources, installation scripts, and cleanup actions. The installer handles both standard APT packages and downloadable .deb files, and can execute custom commands before and after installation. Packages can be tagged with flags to control installation order and behavior, making it especially useful for setting up consistent environments across multiple systems.

The package list is a text file containing package entries, where each entry starts with a package name at the beginning of a line. If the package name is followed by a colon, the subsequent indented lines contain directives that customize its installation; without a colon, the package is installed with default settings. Directives can include download URLs for DEB packages, APT repository information, source list entries, pre/post installation scripts, and flags that control installation behavior. Lines can be continued with a backslash, and comments start with '#'.  Packages cannot appear in the file more than once (and `autosetup` will error if it encounters duplicate packages).

Packages are installed in the order they appear in the package file, however packages that have the 'start' or 'end' flags are installed first and last respectively in the order they appear in the file.  Packages that have neither of these flags are installed after 'start' packages but before 'end' packages.

Each package is installed in it's entirety and `autosetup` will halt on any errors encountered in the process if `--stop_on_errors` is set.  For each package, installation process proceeds as follows: 
1. If the package has a new APT package repository or a new entry in `/etc/apt.d/sources.list.d/`, add it
3. If the package has a preinstall script, run it
4. If the package needs to download any packages directly, do so.
5. If the package has an installable component, install it.
    1. If the installable component is a downloaded .deb, directly install it.
    2. If the installable component is an APT package, install it and any dependencies it might have.
6. If the package has a postinstall script, run it

Any errors at any step in the package installation process will halt `autosetup`

### Example Package List

Here's an example that demonstrates the various package list formats:

    emacs     # Install emacs & all dependencies via APT
    atop      # Install atop

    tcsh:     # Install tcsh, and add it to /etc/shells
      postscript: echo "/bin/tcsh" >> /etc/shells

    code:     # Install code from a downloaded .deb file
      deb: https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64
    
    tree:
      hosts: *.foo.com
    
    # Clean up from installation -- note that the 'no_apt' flag tells autosetup to not try 
    # to 'apt install' this package
    __termination_package:
      flags: end, no_apt
      prescript: apt -y autoremove

This example:
- Installs `emacs` and `atop` via APT with default settings
- Installs `tcsh` via APT and then runs a script to add it to /etc/shells
- Downloads and installs Visual Studio Code from the official .deb package
- Installs `tree` on machines in the `foo.com` domain
- Runs `apt autoremove` after all other packages are installed

### Available Directives
Packages in the Package List can have directives that provide `autosetup` instructions on how to install the package.  The directives are:
- `flags`: Comma-separated list of flags (see below)
- `repo`: APT repository to add
- `source`: APT source list entry (filename and content)
- `prescript`: Commands to run before installation
- `postscript`: Commands to run after installation
- `deb`: URL to download .deb package
- `apt`: Alternative package name for APT installation
- `hosts`: Specify which hosts to install this package on.  The hostnames specified in this directive is a comma-separated list of hostname matches (web, web.example.com), domain wildcards (\*.example.com) or exclusions with ! prefix (!web, !*.example.com)

### Package Flags

Packages can have optional flags that control how `autosetup` manages that specific package.  The current set of flags are:
- `start`: Install this package before all other packages
- `end`: Install this package after all other packages
- `force`: Always reinstall this package
- `skip`: Skip this package
- `force_apt_update`: Force APT database update after installing this package
- `virtual`: This is a virtual package, do not attempt to apt install the package.
- `prescript_run_once`: Run the prescript(s) for this package only once on this system
- `postscript_run_once`: Run the postscript(s) only once on this system
- `script_run_once`: Run the pre- and postscript(s) only once on this system

## Using autosetup

Copy the script to a location in your PATH. Ensure you have Python 3.10 or later installed.

## Usage

The script must be run as root unless using --dryrun mode:

    sudo ./autosetup [options] package-list-file

### Options

- `-n, --dryrun`: Preview changes without installing anything
- `-p, --preserve`: Keep temporary working directory
- `-v, --verbose`: Show subprocess output
- '-s. --stop_on_errors`: Stop if an error is encountered
- `--debug`: Show debugging information
- `--force-all`: Force reinstallation of all packages
- `--only PKGS`: Only install specified packages
- `--only-flags FLAGS`: Only install packages with specified flags
- `--skip PKGS`: Skip specified packages
- `--skip-flags FLAGS`: Skip packages with specified flags
- `--show`: Parse the package list file, print the packages found and exit
- `--version`: Show program version

## Examples

Install all packages:

    sudo ./autosetup packages.list

Preview installation:

    ./autosetup -n packages.list

Install specific packages:

    sudo ./autosetup --only package1 package2 packages.list

Install packages with specific flags:

    sudo ./autosetup --only-flags server packages.list

## Run-once Scripts
To ensure that scripts flagged as `run_once` only do run one time, ``autosetup`` places a semaphore file in `/var/run/autosetup`.  It checks for the existance of that file each time it encounters a pre- or post-script in a package declaration, and if it exists, it will not run the associated script.  

To force `autosetup` to run a script again, delete the associated file in `/var/run/autosetup`.  **Note**: Pacakge scripts may not be written to correctly run more than once; doing so may lead to unknown errors.