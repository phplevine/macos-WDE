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
