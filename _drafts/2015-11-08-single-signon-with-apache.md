---
layout: post
title: Single Sign On with Active Directory and MediaWiki
---


We decided to install and configure a wiki for the entire organization, where to write the various procedures.
In the end we chose MediaWiki.
I won't describe here the installation process, since it's quite straightforward.
What we wanted to achieve is to have a place where any user could write down something easily, so we needed to lower the entry barrier as much as possible, but still maintain the modification history and be able to track who did a particular change.
The biggest issue for users is the need to register to the wiki, and have to remember a new user/password combo.
Since our infrastructure is based on Active Directory, we thought about trying to have MediaWiki use a Single-Sign-On mechanism, so it would not ask any password, and authenticate the user already logged on in Active Directory.

MediaWiki already have a "LDAP" authentication plugin: it can bind to Active Directory and authenticate the user using its AD credentials. Nice! But the user still needs to re-enter the credentials he already typed on login.
Luckily, there is a solution: MediaWiki can use the user passed by the WebServer (Apache, in this case), using the "Auth remoteuser extension" https://www.mediawiki.org/wiki/Extension:Auth_remoteuser , so... let me introduce you to Apache Kerberos Authentication.



