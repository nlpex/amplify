---
layout: post
title: Giving invalid parameter to pkill on macOS crashes the parent process
---

- `pkill` on macOS kills its parent process if a invalid parameter for a option is passed
- Verified on OS X 10.11.6 (El Capitan) and macOS 10.12.6 (Sierra)
- Seems to be fixed on macOS High Sierra (10.13)

First, let's look through the usage of `pkill`:

```
$ pkill --help
pkill: illegal option -- h
Usage: pkill [-<signal>] [-finvVx] [-g <pgrplist>] [-G <gidlist>] [-P <ppidlist>] [-s <signal>] [-t <termlist>] [-u <euidlist>] [-U <uidlist>] [<pattern>]
```

Create `test-pkill.sh` which passes a invalid option `-g foo` to `pkill`:

```bash
echo 'Executing pkill'
pkill -g foo
echo 'Done'
```

Execute it.

```
$ sh test-pkill.sh
Executing pkill
pkill: unable to parse pid: foo
Terminated: 15

$ echo $?
143
```

Note that `Done` is not printed; `test-pkill.sh` was crashed! `Terminated: 15` is printed by the shell (here, I used bash) and exit code 143 implies `test-pkill.sh` is killed with `SIGTERM` (143 = 128 + 15 = 128 + `SIGTERM`, see [here](https://unix.stackexchange.com/q/99112/231543) for details). The message and exit code vary by shell:

```
$ csh
[wsh5:~/work] wsh% sh test-pkill.sh
Executing pkill
pkill: unable to parse pid: foo
Terminated
[wsh5:~/work] wsh% echo $?
143
[wsh5:~/work] wsh% exit
exit

$ ksh
$ sh test-pkill.sh
Executing pkill
pkill: unable to parse pid: foo
Terminated
$ echo $? # 271 = 256 = 15
271
$ exit

$ zsh
wsh5% sh test-pkill.sh
Executing pkill
pkill: unable to parse pid: foo
zsh: terminated  sh test-pkill.sh
wsh5% echo $?
143
wsh5% exit

$ fish
wsh@wsh5 /U/w/work> sh test-pkill.sh
Executing pkill
pkill: unable to parse pid: foo
fish: Job 1, 'sh test-pkill.sh' terminated by signal SIGTERM (Polite quit request)
wsh@wsh5 /U/w/work> echo $status
143
wsh@wsh5 /U/w/work> exit
```

I confirmed other options `-G`, `-P`, `-t`, `-u`, and `-U` also causes this crash.

This bug was found on Mac OS X 10.11.6 (El Capitan) while investigating a bug in [the extension for Visual Studio Code](https://github.com/rogalmic/vscode-bash-debug), which crashes Visual Studio Code. [This issue](https://github.com/rogalmic/vscode-bash-debug/issues/38) argues that the crash occurs on macOS 10.12.6 (Sierra).

Although there's a room for further investigation (for example, `sudo dtruss pkill -P foo` reveals that it calls `kill(2)` to many unrelated processes), I won't do that since the bug seems to be fixed on macOS 10.12 (High Sierra):

```
$ sh test-pkill.sh
Executing pkill
usage: pkill [-signal] [-ILfilnovx] [-F pidfile] [-G gid]
             [-P ppid] [-U uid] [-g pgrp]
             [-t tty] [-u euid] pattern
Done
```

[日本語](http://nlpex.hatenablog.com/entry/2017/12/17/180856)
