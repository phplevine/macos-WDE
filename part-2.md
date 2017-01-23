## Part 2: macOS 10.12 Sierra Web Development Environment

This is an updated version of our prior OS X development series. The newly released macOS 10.12 Sierra requires significant changes compared to prior releases, necessitating a thorough revamp in the process. The main change is why now use `Homebrew's Apache`, rather than the built-in version, but it should continue to work on prior OS X versions.``

In [Part 1](https://github.com/phplevine/macos-WDE/blob/master/README.md) of this 3-part series, we covered configuring Apache on macOS to work better with your local user account, as well as the installation process for installing multiple versions of PHP.

In this `Part 2`, we will cover installing `MySQL, Virtual Hosts, APC` caching, `YAML`, and `Xdebug`. After finishing this tutorial, be sure to check out [how to enable SSL in Part 3 of the series](https://github.com/phplevine/macos-WDE/blob/master/part-3.md).

This guide is intended for `experienced web developers`. If you are a beginner developer, you will be better served using [MAMP or MAMP Pro](http://www.mamp.info/en/mamp-pro/).

## MySQL

Although not required for development of `Grav`, there are times you definitely need an installation of MySQL. In the original guide, we used the Oracle MySQL installation package. However, we now have switched to [MariaDB](https://mariadb.org/) which is a drop-in replacement for MySQL and is easily installed and updated with Brew. Detailed information on the HomeBrew installation process can be found on the [mariadb.org site](https://mariadb.com/kb/en/mariadb/documentation/getting-started/compiling-mariadb-from-source/building-mariadb-on-mac-os-x-using-homebrew/) but the essentials are as follows:

Install `MariaDB` with Brew:

```
$ brew install mariadb
$ mysql_install_db
```

After a successful installation, you can start the server:

```
$ mysql.server start
```

You should get some positive feedback on that action:

```
Starting MySQL
. SUCCESS!
```

[Download SequelPro](http://www.sequelpro.com/) and install it. (it's awesome and free!). You should be able to automatically create a new connection via the `Socket` option without changing any settings.

![sequelpro](https://github.com/phplevine/macos-WDE/blob/master/sequelpro.png?raw=true)

It is advisable to change the `MySQL server password` and secure your installation. The simplest way to do this is to use the provided script:

```
$ /usr/local/bin/mysql_secure_installation
```

Just answer the questions and fill them in as is appropriate for your environment.

If you need to stop the server, you can use the simple command:

```
$ mysql.server stop
```

## Apache Virtual Hosts

A very handy development option is to have `multiple virtual hosts` set up for you various projects. This means that you can set up names such as ``grav.mydomain.com`` which point to your Grav setup, or `project-x.mydomain.com` for a project-specific URL.

Apache generally performs name-based matching, so you don't need to configure multiple IP addresses. Detailed information can be found on the [apache.org](http://httpd.apache.org/docs/2.4/vhosts/) site.

Apache already comes preconfigured to support this behavior but it is not enabled. First you will need to `uncomment` the following lines in your `/usr/local/etc/apache2/2.4/httpd.conf` file:

```
LoadModule vhost_alias_module libexec/mod_vhost_alias.so
```

and:

```
# Virtual hosts
Include /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
```

Then you can edit this referenced file and configure it to your needs:

```
$ open -e /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
```

This file has some instructions already but the important thing to remember is that these rules are matched in order. When you set up virtual hosts, you will lose your older document root, so you will need to add back support for that first as a virtual host.

```
<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites"
    ServerName localhost
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites/grav-admin"
    ServerName grav-admin.dev
</VirtualHost>
```

As you set up your `.dev` virtual hosts, you may receive a warning such as `Warning: DocumentRoot [/Users/your_user/Sites/grav-admin] does not exist` when restarting Apache. This just lets you know that the source directory listed for your virtual hosts is not present on the drive. It's an issue that can be resolved by editing this file with the corrected `DocumentRoot`.

## Dnsmasq

In the example virtualhost we setup above, we defined a `ServerName` of `grav-admin.dev`. This by default will not resolve to your local machine, but it's often very useful to be able to setup various virtual hosts for development purposes. You can do this by manually adding entries to `/etc/hosts` ever time, or you can install and configure [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) to automatically handle wildcard `*.dev` names and forward all of them to localhost (`127.0.0.1`).

First we install it with brew:

```
$ brew install dnsmasq
```

Then we setup `*.dev` hosts:

```
$ echo 'address=/.dev/127.0.0.1' > /usr/local/etc/dnsmasq.conf
```

Start it and ensure it auto-starts on reboot in the future:

```
$ sudo brew services start dnsmasq
```

And lastly, add it to the resolvers:

```
$ sudo mkdir -v /etc/resolver
$ sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/dev'
```

Now you can test it out by pinging some bogus `.dev` name:

```
ping bogus.dev
PING bogus.dev (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.044 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.118 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.045 ms
```

Voila! we have successfully setup wildcard forwarding of all `*.dev` DNS names to localhost.

## APC Cache

Caching in PHP is a big part of the performance equation. There are two types of caching typically available, and both have a big impact on speed and performance.

The first type of cache is called an `opcode cache`, and this is what takes your PHP script and compiles it for faster execution. This alone can typically result in a `3X speed increase!`.

The second type of cache is a `user cache`, and this is a data-store that PHP can use to quickly store and retrieve data from. These typically run in memory which means they are transient, but very fast.

For PHP 5.5 and newer runs best with `Zend OPcache` by default but you can still use `APCu Cache` as a data store.

## Install OPcache and APCu

Switch to PHP 5.5 mode, then run the following `brew` commands:

```
$ sphp 55
$ brew install php55-opcache
$ brew install php55-apcu
```

If you have installed PHP 5.6, you will need to repeat the process:

```
$ sphp 56
$ brew install php56-opcache
$ brew install php56-apcu
```

If you have installed PHP 7.0, you will need to repeat the process:

```
$ sphp 70
$ brew install php70-opcache
$ brew install php70-apcu
```

And lastly, for PHP 7.1:

```
$ sphp 71
$ brew install php71-opcache
$ brew install php71-apcu
```

Restart Apache with the standard `sudo apachectl -k restart` command to pick up your changes.

Point your browser to `http://localhost/info.php` and ensure you see a reference to `Zend OPcache` in the `Zend Engine` block.
