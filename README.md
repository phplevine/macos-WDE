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
