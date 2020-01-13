## tmux control mode

Control mode is a special mode that allows a tmux client to be used to talk to
tmux using a simple text-only protocol. It was designed and written by George
Nachman and allows his [iTerm2](https://www.iterm2.com/) terminal to interface
with tmux and show tmux panes using the iTerm2 UI.

A control mode client is just like a normal tmux client except that instead of
drawing the terminal, tmux communicates using text. Because control mode is
text only, it can easily be parsed and used over *ssh(1)*.

Control mode clients accept standard tmux commands and return their output, and
additionally send control mode only information (mostly asynchronous
notifications) prefixed by `%`. The idea is that users of control mode use tmux
commands ('new-window`, `list-sesssions`, `show-options`, and so on) to control
tmux rather than duplicating a separate command set just for control mode.

### Entering control mode

The `-C` flag to tmux starts a client in control mode. This is only really
useful with the `attach-session` or `new-session` commands that attach the
client.

`-C` has two forms. A single `-C` doesn't change any terminal attributes -
leaving the terminal in canonical mode, so for example echo is still enabled.
This is intended for testing: typing will appear, control characters like
delete and kill still work.

Two `-C` (so `-CC`) disables canonical mode and most other terminal features
and is intended for applications (that, for example, don't need echo).

In either mode, entering an empty line (just pressing `Enter`) will detach the
client.

For example, this shows output from starting a new tmux server on a socket
called `test` with a new session (it runs `new-session`) and attaching a client
in control mode, then killing the server:

~~~~
$ tmux -Ltest -C new
%begin 1578920019 258 0
%end 1578920019 258 0
%window-add @1
%sessions-changed
%session-changed $1 1
%window-renamed @1 tmux
%output %1 nicholas@yelena:~$
%window-renamed @1 ksh
kill-server
%begin 1578920028 265 1
%end 1578920028 265 1
~~~~

In this example, `kill-server` is a command entered by the user and the
remaining lines starting with `%` are sent by the tmux server.

### Commands

tmux commands or command sequences may be sent to the control mode client, for
example creating a new window:

~~~~
new -n mywindow
%begin 1578920529 257 1
%end 1578920529 257 1
~~~~

Every command produces one block of output. This is wrapped in two guard lines:
either `%begin` and `%end` if the command succeeded, or `%begin` and `%error`
if it failed.

Every `%begin`, `%end` or `%error` has three arguments:

1. the time as seconds from epoch;

2. a unique command number;

3. flags, at the moment this is always one.

The time and command number for `%begin` will always match the corresponding
`%end` or `%error`, although tmux will never mix output for different commands
so there is no requirement to use these.

Output from commands is sent as it would be if the command was ran from inside
tmux or from a shell prompt, for example:

~~~~
lsp -a
%begin 1578922740 269 1
0:0.0: [80x24] [history 0/2000, 0 bytes] %0 (active)
1:0.0: [80x24] [history 0/2000, 0 bytes] %1 (active)
1:1.0: [80x24] [history 0/2000, 0 bytes] %2 (active)
%end 1578922740 269 1
~~~~

Or:

~~~~
abcdef
%begin 1578923149 270 1
parse error: unknown command: abcdef
%error 1578923149 270 1
~~~~~

It is recommended the `-F` flag be used where possible for output in a known
format rather than relying on the default.

### Pane output

Like a normal tmux client, a control mode client may be attached to a single
session (which can be changed using commands like `switch-client`,
`attach-session` or `kill-session`). Any output in any pane in any window in
the attached session is sent to the control client. This takes the form of a
`%output` notification with two arguments:

1. The pane ID (*not* the pane number).

2. The output.

The output has any characters less than ASCII 32 and `\` replaced with their
octal equivalent, so `\` becomes `\134`. Otherwise, it is exactly what the
application running in the pane sent to tmux. It may contain escape sequences
which will be as expected by tmux (so for `TERM=screen` or `TERM=tmux`) and may
not be valid UTF-8.

For example, creating a new window and sending the *ls(1)* command:

~~~~
neww
%begin 1578923903 256 1
%end 1578923903 256 1
%output %1 nicholas@yelena:~$
send 'ls /' Enter
%begin 1578923910 261 1
%end 1578923910 261 1
%output %1 ls /\015\015\012
%output %1 altroot/     bsd.booted   dev/         obsd*        sys@\015\012bin/         bsd.rd       etc/         reboot*      tmp/\015\012
%output %1 boot         bsd.sp       home/        root/        usr/\015\012bsd          cvs@         mnt/         sbin/        var/\015\012
%output %1 nicholas@yelena:~$
~~~~~

### Notifications

XXX

### Special commands

tmux provides two special arguments to the `refresh-client` command for control
mode clients to perform actions not needed by normal clients. These are:

- `refresh-client -C` sets the size of a control mode client. If this is not
  used, control mode clients do not affect the size of other clients no matter
  the value of the `window-size` option. If this is used, then they are treated
  as any other client with the given size.

- `refresh-client -F` sets flags for the control mode client. Only one flag is
  currently supported, `no-output` which stops any `%output` notifications
  being sent to the client.

In addition, a few commands like `suspend-client` have no effect when used with
a control mode client.