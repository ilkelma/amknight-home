---
title: "Systemd and the curious case of 'too many open files'" 
date: 2018-08-12 21:30:07
draft: false
---

If you've worked in linux long enough you've run into that most curious of error in your app code "too many files". This is a fascinating error that can throw the linux newbie and even for the seasoned expert can be maddeningly complex. 

The gist of the issue is that the linux kernel gets to determine how many _file handles_ can be allocated at one time. Now if what you're operating on linux is a web server you may be thinking okay I have like 10 log files max so why do I care? In the *nix world files are used for almost every kind of descriptor you can imagine. Device information? file. Process information? file. Network socket connection? file. It's that last one that is likely your issue in the case of a web server of web application.

Each network socket connection being an open file coupled with the fact that linux sets a hard limit on number of files can lead to terrible effects when a servers is open to the internet and thus excess connections can topple the machine's hard limit. For that reason linux allows you to set max number of files in many locations. It can be set for all processes via `/etc/security/limits.conf` (or files in `/etc/security/limits.d/`) and at the shell/per user level with the `ulimit` command. For many cases those instances are fine, limits.conf would by default affect whole classes of processes and ulimit can prevent users from opening too many files.

But what about cases where the process is launched by a supervising agent - not a normal user or daemon process... such as systemd? Well it turns out that systemd services when not told explicitly what the default is or what the limit for a particular systemd service is can inherit their file limit from any number of places that can be extremely difficult to follow - often a default for a user if the service is launched via a script from a user shell (even a system user). This can cause problems when say, your default bash limit is 4096 or 1024 and you receive above and beyond that number of connections.

The good news is that we can tell systemd to be explicit about these things by setting either `DefaultLimitNOFILE` in the system's `/etc/security/limits.conf` file or `LimitNOFILE` in the systemd unit file. It's generally advisable to do the latter with some specificity - often quite high and then set the former to something reasonably high but not so high it could easily overwhelm the system. This allows a sane default for any processes you or anyone else spins up on the machine but for a specific service like say your web server or proxy server you can set a limit that will be high enough to prevent your site being hugged to death when you pen the great american novel or announce your latest ICO.

For more information refer to this section of the systemd spec: [systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Process%20Properties)