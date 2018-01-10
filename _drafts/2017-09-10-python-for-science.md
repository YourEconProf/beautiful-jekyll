---
layout: post
title: Installing Python for Science on a Mac 
tags: [personal, career]
---

My older son, Sam, just went off to Ohio State for his Freshman year. In week two, he found an internship for which he has to learn Python, but not only does he not know Python, he doesn't have it installed. And since his graduation gift was a MacBook Air with a 128GB SSD, it needs to be installed in as judicious of a manner possible.

First, we need to install the Mac OS X Command Line Tools. The easiest way to do this is to open a Terminal window, and type 'gcc'. The Mac OS X installer will pop up to ask if you want to install Mac OS X Command Line tools, choose 'Install'. This will install a number of tools, including the compilers that will be needed to install the other pieces.

    Samson:~ matthewcroberts$ gcc
    
Installing Homebrew is next. Homebrew is a package manager for Mac OS X, and makes the installation and maintance of many different UNIX programs much much easier.

    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
        
Next, we'll need to download MiniConda. MiniConda is a bare-bones install of Anaconda, one of the most popular Data Science python collections and package managers.
