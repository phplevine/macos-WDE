## macOS 10.12 Sierra Web Development Environment

This is an updated version of our prior OS X development series. The newly released macOS 10.12 Sierra requires significant changes compared to prior releases, necessitating a thorough revamp in the process. The main change is why now use `Homebrew's Apache`, rather than the built-in version, but it should continue to work on prior OS X versions.

Developing web applications on macOS is a real joy. There are plenty of options for setting up your development environments, including the ever-popular MAMP Pro that provides a nice UI on top of `Apache`, `PHP` and `MySQL`. However, there are times when [MAMP Pro](http://www.mamp.info/en/mamp-pro/) has slow downs, or out of date versions, or is simply behaving badly due to its restrictive system of configuration templates and non-standard builds.

It is times like these that people often look for an alternative approach, and luckily there is one, and it is relatively straight-forward to setup.

In this blog post, we will walk you through setting up and configuring `Apache 2.4` and `multiple PHP versions`. In the second blog post in this two-post series, we will cover `MySQL, Apache virtual hosts, APC` caching, and `Xdebug` installation.

## Tip

This guide is intended for `experienced web developers`. If you are a beginner developer, you will be better served using [MAMP or MAMP Pro](http://www.mamp.info/en/mamp-pro/).

## Homebrew Installation

This process relies heavily on the macOS package manager called `Homebrew`. Using the `brew` command you can easily add powerful functionality to your mac, but first we have to install it. This is a simple process, but you need to launch your `Terminal` (/Applications/Utilities/Terminal) application and then enter:

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Just follow the terminal prompts and enter your password where required. This will install Homebrew and also install the required `XCode Command Line Tools` if you don't already have XCode installed. This may take a few minutes, but when complete, a quick way to ensure you have installed `brew` correctly, simply type:

```
$ brew --version
Homebrew 1.0.6
Homebrew/homebrew-core (git revision 1b10; last commit 2017-01-23)
```

You should probably also run the following command to ensure everything is configured correctly:

```
$ brew doctor
```

It will instruct you if you need to correct anything.

## Extra Brew Taps

We are going to use some brews that require some external `taps:`

```
$ brew tap homebrew/dupes
$ brew tap homebrew/versions
$ brew tap homebrew/php
$ brew tap homebrew/apache
```

If you already have brew installed, make sure you have the all the latest available brews:

```
$ brew update
```

Now you are ready to brew!

## Apache Installation

The latest `macOS 10.12 Sierra` comes with Apache 2.4 pre-installed, however, it is no longer a simple task to use this version with Homebrew because Apple has removed some required scripts in this release. However, the solution is to install Apache 2.4 via Homebrew and then configure it to run on the standard ports (80/443).

If you already have the built-in Apache running, it will need to be shutdown first, and any auto-loading scripts removed. It really doesn't hurt to just run all these commands in order - even if it's a fresh installation:

```
$ sudo apachectl stop
$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
$ brew install httpd24 --with-privileged-ports --with-http2
```

This step takes a little while as it builds Apache from source. Upon completion you should see a message like:

```
🍺  /usr/local/Cellar/httpd24/2.4.23_2: 212 files, 4.4M, built in 1 minute 45 seconds
```

This is important because you will need that path in the next step. In this example the path was `/usr/local/Cellar/httpd24/2.4.23_2`. If you get a newer version, simply use that path in the next line:

```
$ sudo cp -v /usr/local/Cellar/httpd24/2.4.23_2/homebrew.mxcl.httpd24.plist /Library/LaunchDaemons
$ sudo chown -v root:wheel /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
$ sudo chmod -v 644 /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
$ sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
```

You now have installed Homebrew's Apache, and configured it to auto-start with a privileged account. It should already be running, so you can try to reach your server in a browser by pointing it at your localhost, you should see a simple header that says `"It works!"`.

![it-works](https://github.com/phplevine/macos-WDE/blob/master/it-works.png?raw=true)

## Troubleshooting Tips

If you get a message that the browser can't connect to the server, first check to ensure the server is up.

```
$ ps -aef | grep httpd
```

You should see a few httpd processes if Apache is up and running.

Try to restart Apache with:

```
$ sudo apachectl -k restart
```

You can watch the Apache error log in a new Terminal tab/window during a restart to see if anything is invalid or causing a problem:

```
$ tail -f /usr/local/var/log/apache2/error_log
```

If that doesn't work, check to ensure you have `Listen: 80` in your `/usr/local/etc/apache2/2.4/httpd.conf` configuration file.

Apache is controlled via the `apachectl` command so some useful commands to use are:

```
$ sudo apachectl start
$ sudo apachectl stop
$ sudo apachectl -k restart
```

  The `-k` will `force a restart immediately` rather than asking politely to restart when apache is good and ready

## Apache Configuration

Now that we have a working web server, we will want to do is make some configuration changes so it works better as a local development server.

Firs we'll configure it to use the to change the document root for Apache. This is the folder where Apache looks to serve file from. By default, the document root is configured as `/Library/WebServer/Documents`. As this is a development machine, let's assume we want to change the document root to point to a folder in our own home directory. To do this, we will need to edit Apache's configuration file.

```
/usr/local/etc/apache2/2.4/httpd.conf
```

For simplicity we'll use the built-in `TextEditor` application to make all our edits. You can launch this from the Terminal by using the `open -e` command followed by the path to the file:

```
$ open -e /usr/local/etc/apache2/2.4/httpd.conf
```

![textedit](https://github.com/phplevine/macos-WDE/blob/master/textedit.png?raw=true)

Search for the term `DocumentRoot`, and you should see the following line:

```
DocumentRoot "/usr/local/var/www/htdocs"
```

Change this to point to your user directory where `your_user` is the name of your user account:

```
DocumentRoot /Users/your_user/Sites
```

You also need to change the `<Directory>` tag reference right below the DocumentRoot line. This should also be changed to point to your new document root also:

```
<Directory /Users/your_user/Sites>
```

We removed the optional `quotes` around the directory paths as TextEdit will probably try to convert those to smart-quotes and that will result in a Syntax error when you try to restart Apache. Even if you edit around the quotes and leave them where they are, saving the document may result in their conversion and cause an error.

In that same `<Directory>` block you will find an `AllowOverride` setting, this should be changed as follows:

```
# AllowOverride controls what directives may be placed in .htaccess files.
# It can be "All", "None", or any combination of the keywords:
#   AllowOverride FileInfo AuthConfig Limit
#
AllowOverride All
```

Also we should now enable `mod_rewrite` which is commented out by default. Search for `mod_rewrite.so` and uncomment the line by removing the leading #:

```
LoadModule rewrite_module libexec/mod_rewrite.so
```

## User & Group

Now we have the Apache configuration pointing to a `Sites` folder in our home directory. One problem still exists, however. By default, apache runs as the user `daemon` and group `daemon`. This will cause permission problems when trying to access files in our home directory. About a third of the way down the `httpd.conf` file there are two settings to set the `User` and `Group` Apache will run under. Change these to match your user account (replace `your_user` with your real username), with a group of `staff`:

```
User your_user
Group staff
```

## Sites Folder

Now, you need to create a `Sites` folder in the root of your home directory. You can do this in your terminal, or in Finder. In this new `Sites` folder create a simple `index.html` and put some dummy content in it like: `<h1>My User Web Root</h1>`.

```
$ mkdir ~/Sites
$ echo "<h1>My User Web Root</h1>" > ~/Sites/index.html
```

Restart apache to ensure your configuration changes have taken effect:

```
$ sudo apachectl -k restart
```

If you receive an error upon restarting Apache, try removing the quotes around the DocumentRoot and Directory designations we set up earlier. Any error involving `apr_sockaddr_info_get()` may be resolved by searching your `httpd.conf` file for the line `ServerName localhost` and removing the `#`. Once you have done this, try resetting Apache again.

Pointing your browser to `http://localhost` should display your new message. If you have that working, we can move on!

![sites-webroot](https://github.com/phplevine/macos-WDE/blob/master/sites-webroot.png?raw=true)

## PHP Installation

We will proceed by installing `PHP 5.5, PHP 5.6, PHP 7.0`, and `PHP 7.1` and using a simple script to switch between them as we need.

You can get a full list of available options to include by typing `brew options php55`, for example. In this example we are just including Apache support via `--with-httpd24` to build the required PHP modules for Apache.

```
$ brew install php55 --with-httpd24
$ brew unlink php55
$ brew install php56 --with-httpd24
$ brew unlink php56
$ brew install php70 --with-httpd24
$ brew unlink php70
$ brew install php71 --with-httpd24
```

This may take some time as your computer is actually compiling PHP from source.


You must `reinstall` each PHP version with `reinstall` command rather than `install` if you have previously installed PHP through Brew.

Also, you may have the need to tweak configuration settings of PHP to your needs. A common thing to change is the memory setting, or the `date.timezone` configuration. The php.ini files for each version of PHP are located in the following directories:

```
/usr/local/etc/php/5.5/php.ini
/usr/local/etc/php/5.6/php.ini
/usr/local/etc/php/7.0/php.ini
/usr/local/etc/php/7.1/php.ini
```

`PHP 7.1` is currently in a Release Candidate state at the time of this writing, but it's expected to be released before the end of 2016. You can install as many or as few PHP versions as you like, it's nice to have options right?

## Apache PHP Setup - Part 1

You have successfully installed your PHP versions, but we need to tell Apache to use them. You will again need to edit the `/usr/local/etc/apache2/2.4/httpd.conf` file and search for `#LoadModule php5_module`.

You will notice that each PHP version added a `LoadModule` entry, however these are all pointing to very specific versions. We can replace these with some more generic paths (exact versions may differ):

```
LoadModule php5_module        /usr/local/Cellar/php55/5.5.38_11/libexec/apache2/libphp5.so
LoadModule php5_module        /usr/local/Cellar/php56/5.6.26_3/libexec/apache2/libphp5.so
LoadModule php7_module        /usr/local/Cellar/php70/7.0.11_5/libexec/apache2/libphp7.so
LoadModule php7_module        /usr/local/Cellar/php71/7.1.0-rc.3_8/libexec/apache2/libphp7.so
```

Modify the paths as follows:

```
LoadModule php5_module    /usr/local/opt/php55/libexec/apache2/libphp5.so
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so
```

We can only have one module processing PHP at a time, so for now, comment out all but the `php56` entry:

```
#LoadModule php5_module    /usr/local/opt/php55/libexec/apache2/libphp5.so
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so
```

This will tell Apache to use PHP 5.6 to handle PHP requests. `(We will add the ability to switch PHP versions later)`.

Also you must set the Directory Indexes for PHP explicitly, so search for this block:

```
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
```

and replace it with this:

```
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

`Save` the file and `restart Apache again`, now that we have installed PHP:

```
$ sudo apachectl -k restart
```

## Validating PHP Installation

The best way to test if PHP is installed and running as expected is to make use of [phpinfo()](http://php.net/manual/en/function.phpinfo.php). This is not something you want to leave on a production machine, but it's invaluable in a development environment.

Simply create a file called `info.php` in your `Sites/` folder you created earlier. In that file, just enter the line:

```
<?php phpinfo();
```

Point your browser to `http://localhost/info.php` and you should see a shiny PHP information page:

![php-running](https://github.com/phplevine/macos-WDE/blob/master/php-running.png?raw=true)

If you see a similar `phpinfo` result, congratulations! You now have Apache and PHP running successfully. You can test the other PHP versions by commenting the `LoadModule ... php56 ...` entry and uncommenting one of the other ones. Then simply restart apache and reload the same page.

## PHP Switcher Script

We hard-coded Apache to use `PHP 5.6`, but we really want to be able to switch between versions. Luckily, some industrious individuals have already done the hard work for us and written a very handy little [PHP switcher script](https://gist.github.com/w00fz/142b6b19750ea6979137b963df959d11).

We will install the `sphp` script into brew's standard `/usr/local/bin`:

```
$ curl -L https://gist.github.com/w00fz/142b6b19750ea6979137b963df959d11/raw > /usr/local/bin/sphp
$ chmod +x /usr/local/bin/sphp
```

## Check Your Path

Homebrew should have added its preferred /usr/local/bin and /usr/local/sbin to your path as part of its installation process. Quickly test this by typing:

```
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

If you don't see this, you might need to add these manually to your path. Depending on your shell your using, you may need to add this line to `~/.profile`, `~/.bash_profile`, or `~/.zshrc`. We will assume you are using the default bash shell, so add this line to a your `.profile` (create it if it doesn't exist) file at the root of your user directory:

```
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```

At this point, I strongly recommend closing `ALL your terminal tabs and windows`. This will mean opening a new terminal to continue with the next step. This is strongly recommended because some really strange path issues can arise with existing terminals (trust me, I have seen it!).

## Apache PHP Setup - Part 2

Although we configured Apache to use PHP earlier, we now need to change this again to point the PHP module to a location used by the `PHP switcher script`. You will again need to edit the `/usr/local/etc/apache2/2.4/httpd.conf` file and search for the `LoadModule php` text that you edited earlier.

You must `replace` the current block of LoadModule entries (including any commented out ones):

```
#LoadModule php5_module    /usr/local/opt/php55/libexec/apache2/libphp5.so
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so
```

with this:

```
# Brew PHP LoadModule for `sphp` switcher
LoadModule php5_module /usr/local/lib/libphp5.so
#LoadModule php7_module /usr/local/lib/libphp7.so
```

The Commented out line for `php7_module` is important and required if you are installing PHP 7.0 or 7.1 as it uses a unique PHP module handler. The script will automatically handle uncommenting and commenting the appropriate PHP module.

## Testing the PHP Switching

After you have completed these steps, you should be able to switch your PHP version by using the command `sphp` followed by a two digit value for the PHP version:

```
$ sphp 55
```

You will probably have to enter your administrator password, and it should give you some feedback:

```
$ sphp 55
PHP version 55 found
Unlinking old binaries...
Linking new binaries...
Linking /usr/local/Cellar/php55/5.5.38_11... 17 symlinks created
Linking new modphp addon...
Fixing LoadModule...
Updating version file...
Restarting homebrew Apache...
Restarting non-root homebrew Apache...
Done.

PHP 5.5.38 (cli) (built: Oct  4 2016 15:48:32)
Copyright (c) 1997-2015 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2015 Zend Technologies
```

Test to see if your Apache is now running PHP 5.5 by again pointing your browser to `http://localhost/info.php`. With a little luck, you should see something like this:

![php-running-55](https://github.com/phplevine/macos-WDE/blob/master/php-running-55.png?raw=true)

## Updating PHP and other Brew Packages

Brew makes it super easy to update PHP and the other packages you install. The first step is to `update` Brew so that it gets a list of available updates:

```
$ brew update
```

This will spit out a list of available updates, and any deleted formulas. To upgrade the packages simply type:

```
$ brew upgrade
```

You will need to switch to each of your installed PHP versions and run `update` again to get updates for each PHP version and ensure you are running the version of PHP you intend.

## Activating Specific/Latest PHP Versions

Due to the way our PHP linking is set up, only one version of PHP is `linked` at a time, only the current `active` version of PHP will be updated to the latest version. You can see the current active version by typing:

```
$ php -v
```

And you can see the specific versions of PHP available by typing:

```
$ brew info php55
homebrew/php/php55: stable 5.5.30 (bottled), HEAD
https://php.net
Conflicts with: php53, php54, php56, php70
/usr/local/Cellar/php55/5.5.28 (498 files, 51M)
  Poured from bottle
/usr/local/Cellar/php55/5.5.29_3 (327 files, 49M)
  Poured from bottle
/usr/local/Cellar/php55/5.5.30 (327 files, 49M)
  Poured from bottle
  ...
```

You will see all the versions of PHP 5.5 brew has available and then you can switch to a specific version by typing:

```
$ brew switch php55 5.5.28
```

OK, that wraps up Part 1 of this 3 part series You now have a fully functional Apache 2.4 installation with a quick-and-easy way to toggle between PHP 5.5, 5.6, 7.0, and 7.1. [Check out Part 2](https://github.com/phplevine/macos-WDE/blob/master/part-2.md) to find out how to setup your environment with `MySQL, Virtual Hosts, APC` caching, `YAML`, and `Xdebug`. Also [take a gander at Part 3](https://github.com/phplevine/macos-WDE/blob/master/part-3.md) to find out how to setup `SSL` for your Apache Virtual Hosts.
