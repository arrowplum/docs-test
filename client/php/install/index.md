---
title: Installation
description: Install the Aerospike PHP client.
---
The Aerospike PHP client works with PHP versions >= 7.0.

The PHP extension was tested to build on 64-bit:

 - Ubuntu 14.04+, Debian 6, 7 and related distros using the **apt** package manager
 - CentOS 6.x, 7.x, RedHat 6.x, 7.x and related distros using the **yum** package manager
 - OS X 10.9 (Mavericks), 10.10 (Yosemite)

Windows is not supported.

### Dependencies

Set the dependencies appropriate for your system.

#### CentOS and RedHat (yum)

```bash
sudo yum groupinstall "Development Tools"
sudo yum install openssl-devel
# You will need PHP7 development headers. If PHP7 was manually installed, these should be available
# by default; Otherwise, you will need to fetch them from a repository, the package name may vary.
sudo yum install php-devel php-pear # unless PHP was manually built
```

#### Ubuntu and Debian (apt)

```bash
sudo apt-get install build-essential autoconf libssl-dev
sudo apt-get install php7.0-dev php-pear # unless PHP was manually built
```

#### OS X

By default OS X is missing command line tools.

On Mavericks (OS X 10.9) and higher, you can [install command line tools without Xcode](http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/).

```bash
xcode-select --install # install the command line tools, if missing
```

Use the OS X package manager [Homebrew](http://brew.sh/) to install dependencies:

```bash
brew update && brew doctor
brew install automake
brew install openssl
```

To switch PHP versions [see this gist](https://gist.github.com/rbotzer/198a04f2315e88c75322).

### Build Instructions

Use [Composer](https://getcomposer.org/) to download and build the PHP extension:

```bash
composer require aerospike/aerospike-client-php ~7.0
find vendor/aerospike/aerospike-client-php/ -name "*.sh" -exec chmod +x {} \;
cd vendor/aerospike/aerospike-client-php/ && sudo composer run-script post-install-cmd
```
For manual builds and configuration instructions, see the [aerospike/aerospike-client-php](https://github.com/aerospike/aerospike-client-php) repository on GitHub.

