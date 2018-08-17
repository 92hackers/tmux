~~~~
PLEASE NOTE: most display problems are due to incorrect TERM! Before
reporting problems make SURE that TERM settings are correct inside and
outside tmux.

Inside tmux TERM must be "screen", "tmux" or similar (such as "tmux-256color").
Don't bother reporting problems where it isn't!

Outside, it should match your terminal: particularly, use "rxvt" for rxvt
and derivatives.
~~~~
### What is TERM and what does it do?

The environment variable TERM tells applications the name of a terminal
description to read from the terminfo(5) database. Each description consists of
a number of named capabilities which tell applications what to send to control
the terminal. For example, the "cup" capability contains the escape sequence
used to move the cursor up.

It is important that TERM points to the correct description for the terminal an
application is running in - if it doesn't, applications may misbehave.

The infocmp(1) command shows the contents of a terminal description and the
tic(1) command builds and installs a description from a file (the -x flag is
normally required with both).

### I found a bug in tmux! What do I do?

Check the latest version of tmux from Git to see if the problem is still
present.

Please send bug reports by email to nicholas.marriott@gmail.com or
tmux-users@googlegroups.com or by opening a GitHub issue. Please see the
CONTRIBUTING file for information on what to include.

### Why doesn't tmux do $x?

Please send feature requests by email to tmux-users@googlegroups.com.

### Why do you use the screen terminal description inside tmux?

It is already widely available. tmux and tmux-256color entries are provided by
modern ncurses and can be used instead by setting the default-terminal option.

### I don't see any colour in my terminal! Help!

On a few platforms, common terminal descriptions such as xterm do not include
colour. screen ignores this, tmux does not. If the terminal emulator in use
supports colour, use a value for TERM which correctly lists this, such as
xterm-color.

### tmux freezes my terminal when I attach to a session. I have to kill -9 the shell it was started from to recover!

Some consoles don't like attempts to set the window title. Tell tmux not to do
this by turning off the "set-titles" option (you can do this in .tmux.conf):

~~~~
set -g set-titles off
~~~~

If this doesn't fix it, send a bug report.

### Why is C-b the prefix key? How do I change it?

The default key is C-b because the prototype of tmux was originally developed
inside screen and C-b was chosen not to clash with the screen meta key.

To change it, change the "prefix" option, and - if required - move the binding
of the "send-prefix" command from C-b (C-b C-b sends C-b by default) to the new
key. For example:

~~~~
set -g prefix C-a
unbind C-b
bind C-a send-prefix
~~~~

### How do I use UTF-8?

tmux requires a system that supports UTF-8 (that is, where the C library has a
UTF-8 locale) and will not start if support is missing.

tmux will attempt to detect if the terminal it is running in supports UTF-8 by
looking at the LC_ALL, LC_CTYPE and LANG environment variables.

If it believes the terminal is not compatible with UTF-8, any UTF-8 characters
will be replaced with underscores. The -u flag explicitly tells tmux that the
terminal supports UTF-8:

~~~~
$ tmux -u new
~~~~

### How do I use a 256 colour terminal?

Provided the underlying terminal supports 256 colours, it is usually sufficient
to add one of the following to ~/.tmux.conf:

~~~~
set -g default-terminal "screen-256color"
~~~~

Or:

~~~~
set -g default-terminal "tmux-256color"
~~~~

And make sure that TERM outside tmux also shows 256 colours, or use the tmux -2
flag.

### How do I use RGB colour?

tmux must be told that the terminal outside supports RGB colour. This is done by specifying the RGB or Tc terminfo(5) flags. RGB is the official flag, Tc is a tmux extension. The easiest method is with the terminal-overrides option, for example:

~~~~
set -as terminal-overrides ",gnome*:RGB"
~~~~

For tmux itself, colours may be specified in hexadecimal, for example bg=#ff0000.

### Why are tmux pane separators dashed rather than continuous lines?

Some terminals (such as mintty) or certain fonts (particularly some Japanese
fonts) do not correctly handle UTF-8 line drawing characters.

The U8 capability forces tmux to use ACS instead of UTF-8 line drawing:

~~~~
set -as terminal-overrides ",*:U8=0"
~~~~

### How do I make Ctrl-PgUp and Ctrl-PgDn work inside tmux?

tmux sends modified function keys using xterm(1)-style escape
sequences. However, many applications don't accept these when TERM is set to
screen or screen-256color inside tmux because these terminal descriptions lack
the capabilities for modified function keys. The tmux and tmux-256color
descriptions do have such capabilities, so using those instead may work.

### What is the proper way to escape characters with #(command)?

When using the #(command) construction to include the output from a command in
the status line, the command will be parsed twice. First, when it's read by the
configuration file or the command-prompt parser, and second when the status
line is being drawn and the command is passed to the shell. For example, to
echo the string "(test)" to the status line, either single or double quotes
could be used:

~~~~
set -g status-right "#(echo \\\\(test\\\\))"
set -g status-right '#(echo \\\(test\\\))'
~~~~

In both cases, the status-right option will be set to the string "#(echo
\\(test\\))" and the command executed will be "echo \(test\)".

### tmux uses too much CPU. What do I do?

Automatic window renaming may use a lot of CPU, particularly on slow computers:
if this is a problem, turn it off with "setw -g automatic-rename off". If this
doesn't fix it, please report the problem.

### What is the best way to display the load average? Why no #L?

It isn't possible to get the load average portably in code and it is preferable
not to add portability goop. The following works on at least Linux, *BSD and OS
X:

~~~~
uptime|awk '{split(substr($0, index($0, "load")), a, ":"); print a[2]}'
~~~~

### How do I attach the same session to multiple clients but with a different current window, like screen -x?

One or more of the windows can be linked into multiple sessions manually with
link-window, or a grouped session with all the windows can be created with
new-session -t.

### I don't see italics! Or italics and reverse are the wrong way round!

GNU screen does not support italics and the "screen" terminal description uses
the italics escape sequence incorrectly.

As of tmux 2.1, if default-terminal is set to "screen" or matches "screen-*",
tmux will behave like screen and italics will be disabled.

To enable italics, make sure you are using the tmux terminal description:

~~~~
set -g default-terminal "tmux"
~~~~

### How do I see the default configuration?

Show the default session options by starting a new tmux server with no
configuration file:

~~~~
$ tmux -Lfoo -f/dev/null start\; show -g
~~~~

Or the default window options:

~~~~
$ tmux -Lfoo -f/dev/null start\; show -gw
~~~~

### How do I copy a selection from tmux to the system's clipboard?

When running in xterm(1), tmux can automatically send copied text to the
clipboard. This is controlled by the set-clipboard option and also needs this X
resource to be set:

~~~~
XTerm*disallowedWindowOps: 20,21,SetXprop
~~~~

For rxvt-unicode (urxvt), there is an unofficial Perl extension
[here](http://anti.teamidiot.de/static/nei/*/Code/urxvt/).

Otherwise a key binding for copy mode using xclip (or xsel) works:

~~~~
bind -Tcopy-mode C-y send -X copy-pipe "xclip -i >/dev/null"
~~~~

Or for inside and outside copy mode with the prefix key:

~~~~
bind C-y run -b "tmux save-buffer - | xclip -i"
~~~~

On OS X, look at the pbcopy(1) and pbpaste(1) commands.

### Why do I see dots around a session when I attach to it?

tmux limits the size of the window to the smallest attached session. If
it didn't do this then it would be impossible to see the entire window.
The dots mark the size of the window tmux can display.

To avoid this, detach all other clients when attaching:

~~~~
$ tmux attach -d
~~~~

Or from inside tmux by detaching individual clients with C-b D or all
using:

~~~~
C-b : attach -d
~~~~

### Why don't XMODEM, YMODEM and ZMODEM work inside tmux?

tmux is not a file transfer program and these protocols are more effort to support than their remaining popularity deserves. Detach tmux before attempting to use them.
