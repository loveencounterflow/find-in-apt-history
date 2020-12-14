


# Find in Apt History


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [What it Is](#what-it-is)
- [What it Does](#what-it-does)
- [How to Use it](#how-to-use-it)
- [Thanks](#thanks)
- [To do](#to-do)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# What it Is

Short bash script that searches for installed / deleted /upgraded packages in `/var/log/dpkg*` (including
`*.gz` archives):

```bash
#!/usr/bin/env bash
set -euo pipefail
set +u; pattern="$1"; set -u

find /var/log -name "dpkg.*" -print0 \
  | xargs -0 zgrep -Pi '(install|remove|upgrade)\s+'"$pattern" \
  | sed -E 's/:/ /' \
  | sed -E 's/^(\S+)\s(\S+)\s(\S+)/\2 \3 :: /g' \
  | sort \
  | less -SR#5
```

# What it Does

Output:

```
2020-10-24 10:39:57 ::  upgrade libsnmp-base:all 5.7.3+dfsg-1.8ubuntu3.3 5.7.3+dfsg-1.8ubuntu3.6
2020-10-24 10:39:58 ::  install linux-modules-5.4.0-52-generic:amd64 <none> 5.4.0-52.57~18.04.1
2020-10-24 10:39:58 ::  upgrade libvncserver1:amd64 0.9.11+dfsg-1ubuntu1.2 0.9.11+dfsg-1ubuntu1.3
2020-10-24 10:40:01 ::  install linux-image-5.4.0-52-generic:amd64 <none> 5.4.0-52.57~18.04.1
2020-10-24 10:40:02 ::  install linux-modules-extra-5.4.0-52-generic:amd64 <none> 5.4.0-52.57~18.04.1
2020-10-24 10:40:10 ::  install linux-hwe-5.4-headers-5.4.0-52:all <none> 5.4.0-52.57~18.04.1
2020-10-29 20:33:19 ::  remove libboost-filesystem-dev:amd64 1.65.1.0ubuntu1 <none>
2020-10-29 20:33:19 ::  remove libboost-regex-dev:amd64 1.65.1.0ubuntu1 <none>
2020-10-29 20:33:19 ::  remove libgtk-3-dev:amd64 3.22.30-1ubuntu4 <none>
```

# How to Use it

* Output is sorted oldest to newest.
* Only lines containing one of `install`, `remove`, or `upgrade` are shown.
* Plain-text logs and `*.gz` archived logs are searched.
* System updates as well as manually installed packages are included.
* May use either `find-in-apt-history 'blah'` or `find-in-apt-history | grep 'blah'` to narrow down results.
* Package names are always terminated with a `:` (colon) so that's a good way to restrict matches.

Sample output for `find-in-apt-history 'firefox:'`:

```
2019-12-13 16:28:00 ::  install firefox:amd64 <none> 71.0+linuxmint2+tricia
2020-05-01 20:59:43 ::  upgrade firefox:amd64 71.0+linuxmint2+tricia 75.0+linuxmint2+tricia
2020-05-15 11:48:52 ::  upgrade firefox:amd64 75.0+linuxmint2+tricia 76.0+linuxmint1+tricia
2020-06-20 14:03:11 ::  upgrade firefox:amd64 76.0+linuxmint1+tricia 77.0.1+linuxmint1+tricia
2020-07-08 18:47:55 ::  upgrade firefox:amd64 77.0.1+linuxmint1+tricia 78.0.1+linuxmint1+tricia
2020-07-18 18:33:36 ::  upgrade firefox:amd64 78.0.1+linuxmint1+tricia 78.0.2+linuxmint1+tricia
2020-10-24 10:39:26 ::  upgrade firefox:amd64 78.0.2+linuxmint1+tricia 82.0+linuxmint5+tricia
2020-11-17 06:38:21 ::  upgrade firefox:amd64 82.0+linuxmint5+tricia 82.0.3+linuxmint1+tricia
2020-12-14 21:18:53 ::  upgrade firefox:amd64 82.0.3+linuxmint1+tricia 83.0+linuxmint1+tricia
```

which tells you that Firefox v71.0 was first installed in mid-December 2019 and last upgraded to v82.0
almost exactly one year later.

# Thanks

thx to https://www.linuxuprising.com/2019/01/how-to-show-history-of-installed.html for the original idea.

# To do

* [ ] add option to avoid filtering for install/upgrade/remove
* [ ] surround package name with unique characters to simplify anchored matches




