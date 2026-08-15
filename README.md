# dtach-sk

dtach is a program written in C that emulates the detach feature of screen, which allows a program to be executed in an environment that is protected from the controlling terminal. For instance, the program under the control of dtach would not be affected by the terminal being disconnected for some reason.

dtach was written because screen did not adequately meet my needs; I did not need screen's extra features, such as support for multiple terminals or terminal emulation support. screen was also too big, bulky, and had source code that was difficult to understand.

screen also interfered with my use of full-screen applications such as emacs and ircII, due to its excessive interpretation of the stream between the program and the attached terminals. dtach does not have a terminal emulation layer, and passes the raw output stream of the program to the attached terminals. The only input processing that dtach does perform is scanning for the detach character (which signals dtach to detach from the program) and processing the suspend key (which tells dtach to temporarily suspend itself without affecting the running program), and both of these can both be disabled if desired.

Contrary to screen, dtach has minimal features, and is extremely tiny. This allows dtach to be more easily audited for bugs and security holes, and makes it accessible in environments where space is limited, such as on rescue disks.

dtach has only been tested on the Linux/x86 platform, however it should be easily portable to other variants of Unix. It currently assumes that the host system uses POSIX termios, and has a working forkpty function available.

dtach may need access to various devices in the filesystem depending on what forkpty does. For example, dtach on Linux usually needs access to /dev/ptmx and /dev/pts.

## Quick start

Compiling dtach should be simple, as it uses autoconf:

```sh
./configure
make
```

If all goes well, a dtach binary should be built for your system. You can then copy it to the appropriate place on your system.

dtach uses Unix-domain sockets to represent sessions; these are network sockets that are stored in the filesystem. You specify the name of the socket that dtach should use when creating or attaching to dtach sessions.

For example, let's create a new session that is running ircII. We will use /tmp/foozle as the session's socket:

```sh
dtach -A /tmp/foozle irc RuneB irc.freenode.net
```

Here, -A tells dtach to either create a new session or attach to the existing session. If the session at /tmp/foozle does not exist yet, the program will be executed. If it does exist, then dtach will attach to the existing session.

dtach has another attach mode, which is specified by using -a. The -a mode attaches to an already existing session, but will not create a new session. Each attaching process can have a separate detach character, suspend behavior, and redraw method, which are explained in the following sections.

dtach is able to attach to the same session multiple times, though you will likely encounter problems if your terminals have different window sizes. Pressing ^L (Ctrl-L) will reset the window size of the program to match the current terminal.

dtach also has a mode that copies the contents of standard input to a session. For example:

```sh
echo -ne 'cd /var/log\nls -l\n' | dtach -p /tmp/foozle
```

The contents are sent verbatim including any embedded control characters (e.g. the newline characters in the above example), and dtach will not scan the input for a detach character.

## Detaching from the session

By default, dtach scans the keyboard input looking for the detach character. When the detach character is pressed, dtach will detach from the current session and exit, leaving the program running in the background. You can then re-attach to the program by running dtach again with -A or -a.

The default detach character is ^\ (Ctrl-\). This can be changed by supplying the -e option to dtach when attaching. For example:

```sh
dtach -a /tmp/foozle -e '^A'
```

That command would attach to the existing session at /tmp/foozle and use ^A (Ctrl-A) as the detach character, instead of the default ^\.

You can disable processing of the detach character by supplying the -E option to dtach when attaching.

## Suspending dtach

By default, dtach also processes the suspend key (^Z or Ctrl-Z) itself, instead of passing it to the program. Thus, pressing suspend only suspends the attaching process, instead of the running program. This can be very useful for applications such as ircII, where you may not necessarily want the program to be suspended.

Processing of the suspend key can be disabled by supplying the -z option to dtach when attaching.

## Redraw method

When attaching, dtach can use one of three methods to redraw the screen (none, ctrl_l, or winch). By default, dtach uses the ctrl_l method, which simply sends a ^L (Ctrl-L) character to the program if the terminal is in character-at-a-time and no-echo mode. The winch method forces a WINCH signal to be sent to the program, and the none method disables redrawing completely.

For example, this command tells dtach to attach to a session at /tmp/foozle and use the winch redraw method:

```sh
dtach -a /tmp/foozle -r winch
```

When creating a new session (with the -c or -A modes), the specified method is used as the default redraw method for the session.

## License

dtach was originally written by Ned T. Crigler, and dtach-sk is a modified version by ShinKouyo.

```text
Copyright (C) 2004-2016 Ned T. Crigler <crigler@gmail.com>
Copyright (C) 2026 ShinKouyo <i@0x0f.dev>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.
```

See [COPYING](COPYING) for the full license text.
