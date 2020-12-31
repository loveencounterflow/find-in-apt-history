


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

sudo find /var/log -name "dpkg.*" -print0 \
  | xargs -0 zgrep -Pi '\b(install|remove|upgrade)\b.*'"$pattern" \
  | sed -E 's/^[^:]+://' \
  | sed -E 's/^(\S+)\s+(\S+)\s+(\S+)\s+([^:]+):(\S+)/\1 \2 \3 :\4: (\5)/g' \
  | sort \
  | grep --color=always -Pi "$pattern" \
  | sed  '1s;^;————— ————— ————— ————— ————— ————— ————— \n;' \
  | sed  '1s;^;Date Time Action Name Arch Previous Current\n;' \
  | column -t -s' ' \
  | less -SR#5 +G
```

# What it Does

`find-in-apt-history` searches the history of all `apt` package installations, upgrades, and removals on
Debian-ish systems (including Ubuntu and Linux Mint). It allows to use `grep` extended patterns to find
matches in the package name and version strings.


# How to Use it

* Output is sorted oldest to newest.
* Only lines containing one of the actions `install`, `remove`, or `upgrade` are shown.
* Plain-text logs and `*.gz` archived logs are searched.
* System updates as well as manually installed packages are included.
* May use either `find-in-apt-history 'blah'` or `find-in-apt-history | grep 'blah'` to narrow down results.
* Package names are always surrounded with `:` (colons) so that's a good way to restrict matches.
* Pattern matching starts *after* with the package name (after the action), so cannot match date, time, or
  action.
* Pattern matching uses `(z)grep` extended, case-insensitive patterns, so `foo` matches the same as `FOO`.
* Uses `less` to page output.
* The tool now uses `sudo` to avoid permission errors. Technically this isn't strictly necessary and may be
  removed in a future version.

Sample output for `find-in-apt-history 'gimp'`:

```
Date        Time      Action   Name                    Arch     Previous           Current
—————       —————     —————    —————                   —————    —————              —————
2020-07-19  18:03:57  install  :gimp-data:             (all)    <none>             2.8.22-1
2020-07-19  18:03:57  install  :libgimp2.0:            (amd64)  <none>             2.8.22-1
2020-07-19  18:04:03  install  :gimp:                  (amd64)  <none>             2.8.22-1
2020-12-31  16:51:18  install  :gimp-gmic:             (amd64)  <none>             1.7.9+zart-4build3
2020-12-31  16:51:19  install  :gimp-plugin-registry:  (amd64)  <none>             7.20140602ubuntu3
2020-12-31  20:00:45  install  :libgimp2.0-dev:        (amd64)  <none>             2.8.22-1
2020-12-31  20:51:57  upgrade  :gimp-plugin-registry:  (amd64)  7.20140602ubuntu3  9.20180625-1ubu18.04~ppa
2020-12-31  20:51:57  upgrade  :libgimp2.0-dev:        (amd64)  2.8.22-1           2.10.14+om-1ubu18.04.7~ppa
2020-12-31  20:51:58  upgrade  :gimp:                  (amd64)  2.8.22-1           2.10.14+om-1ubu18.04.7~ppa
2020-12-31  20:52:00  upgrade  :gimp-data:             (all)    2.8.22-1           2.10.14+om-1ubu18.04.7~ppa
2020-12-31  20:52:00  upgrade  :libgimp2.0:            (amd64)  2.8.22-1           2.10.14+om-1ubu18.04.7~ppa
```

# Thanks

thx to https://www.linuxuprising.com/2019/01/how-to-show-history-of-installed.html for the original idea.

# To do

* [X] provide tabular outpt
* [X] improve pattern matching
* [X] hilite matches
* [X] jump to end of output to show most recent changes
* [X] use `sudo` to avoid permission errors

* [ ] add option to avoid filtering for install/upgrade/remove
* [ ] surround package name with unique characters to simplify anchored matches
* [ ] avoid hiliting when in pipe
* [ ] do not use pager when in pipe


