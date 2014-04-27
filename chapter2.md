

Chapter 2: What to put in your startup files
============================================

There are probably various changes you want to make to the shell's
behaviour. All shells have \`startup' files, containing commands which
are executed as soon as the shell starts. Like many others, zsh allows
each user to have their own startup files. In this chapter, I discuss
the sorts of things you might want to put there. This will serve as an
introduction to what the shell does; by the end, you should have an
inkling of many of the things which will be discussed in more detail
later on and why they are interesting. Sometimes you will find out more
than you want to know, such as how zsh differs from other shells you're
not going to use. Explaining the differences here saves me having to lie
about how the shell works and correcting it later on: most people will
simply want to know how the shell normally works, and note that there
are other ways of doing it.

2.1: Types of shell: interactive and login shells
-------------------------------------------------

First, you need to know what is meant by an **interactive** and a
**login** shell. Basically, the shell is just there to take a list of
commands and run them; it doesn't really care whether the commands are
in a file, or typed in at the terminal. In the second case, when you are
typing at a prompt and waiting for each command to run, the shell is
**interactive**; in the other case, when the shell is reading commands
from a file, it is, consequently, **non-interactive**. A list of
commands used in this second way --- typically by typing something like
`zsh filename`, although there are shortcuts --- is called a **script**,
as if the shell was acting in a play when it read from it (and shells
can be real hams when it comes to playacting). When you start up a
script from the keyboard, there are actually two zsh's around: the
interactive one you're typing at, which is waiting for another,
non-interactive one to finish running the script. Almost nothing that
happens in the second one affects the first; they are different copies
of zsh.

Remember that when I give examples for you to type, I often show them as
they would appear in a script, without prompts in front. What you
actually see on the screen if you type them in will have a lot more in
front.

When you first log into the computer, the shell you are presented with
is interactive, but it is also a login shell. If you type \``zsh`', it
starts up a new interactive shell: because you didn't give it the name
of a file with commands in, it assumes you are going to type them
interactively. Now you've got two interactive shells at once, one
waiting for the other: it doesn't sound all that useful, but there are
times when you are going to make some radical changes to the shell's
settings temporarily, and the easiest thing to do is to start another
shell, do what you want to do, and exit back to the original, unaltered,
shell --- so it's not as stupid as it sounds.

However, that second shell will not be a login shell. How does zsh know
the difference? Well, the programme that logs you in after you type your
password (called, predictably, **login**), actually sticks a \``-`' in
front of the name of the shell, which zsh recognises. The other way of
making a shell a login shell is to run it yourself with the option `-l`;
typing \``zsh -l`' will start a zsh that also thinks it's a login shell,
and later I'll explain how to turn on options within the shell, which
you can do with the login option too. Otherwise, any zsh you start
yourself will not be a login shell. If you are using X-Windows, and have
a terminal emulator such as xterm running a shell, that is probably not
a login shell. However, it's actually possible to get xterm to start a
login shell by giving it the option `-ls`, so if you type
\``xterm -ls &`', you will get a window running a login shell (the `&`
means the shell in the first window doesn't wait for it to finish).

The first main difference between a login shell and any other
interactive shell is the one to do with startup files, described below.
The other one is what you do when you're finished. With a login shell
you can type \``logout`' to exit the shell; with another you type
\``exit`'. However, \``exit`' works for all shells, interactive,
non-interactive, login, whatever, so a lot of people just use that. In
fact, the only difference is that \``logout`' will tell you
\``not login shell`' if you use it anywhere else and fail to exit. The
command \``bye`' is identical to \``exit`', only shorter and less
standard. So my advice is just to use \``exit`'.

As somebody pointed out to me recently, login shells don't have to be
interactive. You can always start a shell in the two ways that make it a
login shell; the ways that make it an interactive shell or not are
independent. In fact, some start-up scripts for windowing systems run a
non-interactive login shell to incorporate definitions from the
appropriate login scripts before executing the commands to start the
windowing session.

### 2.1.1: What is a login shell? Simple tests

Telling if the shell you are looking at is interactive is usually easy:
if there's a prompt, it's interactive. As you may have gathered, telling
if it's a login shell is more involved because you don't always know how
the shell was started or if the option got changed. If you want to know,
you can type the following (one line at a time if you like, see below),

      if [[ -o login ]]; then
        print yes
      else
        print no
      fi

which will print \`yes' or \`no' according to whether it's a login shell
or not; the syntax will be explained as we go along. There are shorter
ways of doing it, but this illustrates the commonest shell syntax for
testing things, something you probably often want to do in a startup
file. What you're testing goes inside the \``[[ ... ]]`'; in this case,
the `-o` tells the shell to test an option, here `login`. The next line
says what to do if the test succeeded; the line after the \``else`' what
to do if the test failed. This syntax is virtually identical to ksh; in
this guide, I will not give exhaustive details on the tests you can
perform, since there are many of them, but just show some of the most
useful. As always, see the manual --- in this case, \`Conditional
Expressions' in the `zshmisc` manual pages.

Although you usually know when a shell is interactive, in fact you can
test that in exactly the same way, too: just use
\``[[ -o interactive ]]`'. This is one option you can't change within
the shell; if you turn off reading from the keyboard, where is the shell
supposed to read from? But you can at least test it.

Aside for beginners in shell programming: maybe the semicolon looks a
bit funny; that's because the \``then`' is really a separate command.
The semicolon is just instead of putting it on a new line; the two are
interchangeable. In fact, I could have written,

      if [[ -o login ]]; then; print yes; else; print no; fi

which does exactly the same thing. I could even have missed out the
semicolons after \``then`' and \``else`', because the shell knows that a
command must come after each of those --- though the semicolon or
newline *before* the `then` is often important, because the shell does
`not` know a command has to come next, and might mix up the `then` with
the arguments of the command after the \``if`': it may look odd, but the
\``[[` *...* `]]`' is actually a command. So you will see various ways
of dividing up the lines in shell programmes. You might also like to
know that `print` is one of the builtin commands referred to before; in
other words, the whole of that chunk of programme is executed by the
shell itself. If you're using a newish version of the shell, you will
notice that zsh tells you what it's waiting for, i.e. a \``then`' or an
\``else`' clause --- see the explanation of `$PS2` below for more on
this. Finally, the spaces I put before the \``print`' commands were
simply to make it look prettier; any number of spaces can appear before,
after, or between commands and arguments, as long as there's at least
one between ordinary words (the semicolon is recognised as special, so
you don't need one before that, though it's harmless if you do put one
in).

Second aside for users of sh: you may remember that tests in sh used a
single pair of brackets, \``if [ ... ]; then ...`', or equivalently as a
command called **test**, \``if test ...; then ...`'. The Korn shell was
deliberately made to be different, and zsh follows that. The reason is
that \``[[`' is treated specially, which allows the shell to do some
extra checks and allows more natural syntax. For example, you may know
that in sh it's dangerous to test a parameter which may be empty: \`[
\$var = foo ]' will fail if `$var` is empty, because in that case the
word is missed out and the shell never knows it was supposed to be
there; with \``[[` *...* `]]`', this is quite safe because the shell is
aware there's a word before the \``=`', even if it's empty. Also, you
can use \``&&`' and \``||`' to mean logical \`and' and \`or', which
agrees with the usual UNIX/C convention; in sh, they would have been
taken as starting a new command, not as part of the test, and you have
to use the less clear \``-a`' and \``-o`'. Actually, zsh provides the
old form of test for backward compatibility, but things will work a lot
more smoothly if you don't use it.

2.2: All the startup files
--------------------------

Now here's a list of the startup files and when they're run. You'll see
they fall into two classes: those in the `/etc` directory, which are put
there by the system administrator and are run for all users, and those
in your home directory, which zsh, like many shells, allows you to
abbreviate to a \``~`'. It's possible that the latter files are
somewhere else; type \``print $ZDOTDIR`' and if you get something other
than a blank line, or an error message telling you the parameter isn't
set, it's telling you a directory other than \``~`' where your startup
files live. If `$ZDOTDIR` (another parameter) is not already set, you
won't want to set it without a good reason.

**`/etc/zshenv`**

Always run for every zsh.

**`~/.zshenv`**

Usually run for every zsh (see below).

**`/etc/zprofile`**

Run for login shells.

**`~/.zprofile`**

Run for login shells.

**`/etc/zshrc`**

Run for interactive shells.

**`~/.zshrc`**

Run for interactive shells.

**`/etc/zlogin`**

Run for login shells.

**`~/.zlogin`**

Run for login shells.

Now you know what login and interactive shells are, this should be
straightforward. You may wonder why there are both `~/.zprofile` and
`~/.zlogin`, when they are both for login shells: the answer is the
obvious one, that one is run before, one after `~/.zshrc`. This is
historical; Bourne-type shells run `/etc/profile`, and csh-type shells
run `~/.login`, and zsh tries to cover the bases with its own startup
files.

The complication is hinted at by the \`see below'. The file
`/etc/zshenv`, as it says, is always run at the start of any zsh.
However, if the option `NO_RCS` is set (or, equivalently, the `RCS`
option is unset: I'll talk about options shortly, since they are
important in startup files), none of the others are run. The most common
way of setting this option is with a flag on the command line: if you
start the shell as \``zsh -f`', the option becomes set, so only
`/etc/zshenv` is run and the others are skipped. Often, scripts do this
as a way of trying to get a basic shell with no frills, as I'll describe
below; but if something is set in `/etc/zshenv`, there's no way to avoid
it. This leads to the First Law of Zsh Administration: put as little as
possible in the file `/etc/zshenv`, as every single zsh which starts up
has to read it. In particular, if the script assumes that only the basic
options are set and `/etc/zshenv` has altered them, it might well not
work. So, at the absolute least, you should probably surround any option
settings in `/etc/zshenv` with

      if [[ ! -o norcs ]]; then
        ... <commands to run if NO_RCS is not set, 
             such as setting options> ...
      fi

and your users will be eternally grateful. Settings for interactive
shells, such as prompts, have no business in `/etc/zshenv` unless you
*really* insist that all users have them as defaults for every single
shell. Script writers who want to get round problems with options being
changed in `/etc/zshenv` should put \``emulate zsh`' at the top of the
script.

There are two files run at the end: `~/.zlogout` and `/etc/zlogout`, in
that order. As their names suggest, they are counterparts of the
`zlogin` files, and therefore are only run for login shells --- though
you can trick the shell by setting the `login` option. Note that whether
you use `exit`, `bye` or `logout` to leave the shell does not affect
whether these files are run: I wasn't lying (this time) when I said that
the error message was the only difference between `exit` and `logout`.
If you want to run a file at the end of any other type of shell, you can
do it another way:

      TRAPEXIT() {
        # commands to run here, e.g. if you 
        # always want to run .zlogout:
        if [[ ! -o login ]]; then
          # don't do this in a login shell
          # because it happens anyway
          . ~/.zlogout
        fi
      }

If you put that in `.zshrc`, it will force `.zlogout` to be run at the
end of all interactive shells. Traps will be mentioned later, but this
is rather a one-off; it's really just a hack to get commands run at the
end of the shell. I won't talk about logout files, however, since
there's little that's standard to put in them; some people make them
clear the screen to remove sensitive information with the \``clear`'
command. Other than that, you might need to tidy a few files up when you
exit.

2.3: Options
------------

It's time to talk about options, since I've mentioned them several
times. Each option describes one particular shell behaviour; they are
all Boolean, i.e. can either be on or off, with no other state. They
have short names and in the documentation and this guide they are
written in uppercase with underscores separating the bits (except in
actual code, where I'll write them in the short form). However, neither
of those is necessary. In fact, `NO_RCS` and `norcs` and `__N_o_R_c_S__`
mean the same thing and are all accepted by the shell.

The second thing is that an option with \``no`' in front just means the
opposite of the option without. I could also have written the test
\``[[ ! -o norcs ]]`' as \``[[ -o rcs ]]`'; the \``!`' means \`not', as
in C. You can only have one \``no`'; \``nonorcs`' is meaningless.
Unfortunately, there is an option \``NOMATCH`' which has \``no`' as part
of its basic name, so in this case the opposite really is
\``NO_NOMATCH`'; `NOTIFY`, of course, is also a full name in its own
right.

The usual way to set and unset options is with the commands **setopt**
and **unsetopt** which take a string of option names. Some options also
have flags, like the \``-f`' for `NO_RCS`, which these commands also
accept, but it's much clearer to use the full name and the extra time
and space is negligible. The command \``set -o`' is equivalent to
`setopt`; this comes from ksh. Note that `set` with no \``-o`' does
something else --- that sets the positional parameters, which is zsh's
way of passing arguments to scripts and functions.

Almost everybody sets some options in their startup files. Since you
want them in every interactive shell, at the least, the choice is
between putting them in `~/.zshrc` or `~/.zshenv`. The choice really
depends on how you use non-interactive shells. They can be started up in
unexpected places. For example, if you use Emacs and run commands from
inside it, such as **grep**, that will start a non-interactive shell,
and may require some options. My rule of thumb is to put as many options
as possible into `~/.zshrc`, and transfer them to `~/.zshenv` if I find
I need them there. Some purists object to setting options in `~/.zshenv`
at all, since it affects scripts; but, as I've already hinted, you have
to work a bit harder to make sure scripts are unaffected by that sort of
thing anyway. In the following, I just assume they are going to be in
`~/.zshrc`.

2.4: Parameters
---------------

One more thing you'll need to know about in order to write startup files
is parameters, also known as variables. These are mostly like variables
in other programming languages. Simple parameters can be stored like
this (an **assignment**):

      foo='This is a parameter.'

Note two things: first, there are no spaces around the \``=`'. If there
was a space before, zsh would think \``foo`' was the name of a command
to execute; if there was a space after it, it would assign an empty
string to the parameter `foo`. Second, note the use of quotes to stop
the spaces inside the string having the same effect. Single quotes, as
here, are the nuclear option of quotes: everything up to another single
quote is treated as a simple string --- newlines, equal signs,
unprintable characters, the lot, in this example all would be assigned
to the variable; for example,

      foo='This is a parameter.
      This is still the same parameter.'

So they're the best thing to use until you know what you're doing with
double quotes, which have extra effects. Sometimes you don't need them,
for example,

      foo=oneword

because there's nothing in \``oneword`' to confuse the shell; but you
could still put quotes there anyway.

Users of csh should note that you don't use \``set`' to set parameters.
This is important because there is a `set` command, but it works
differently --- if you try \``set var="this wont't work"`', you won't
get an error but you won't set the parameter, either. Type \``print $1`'
to see what you did set instead.

To get back what was stored in a parameter, you use the name somewhere
on the command line with a \``$`' tacked on the front --- this is called
an **expansion**, or to be more precise, since there are other types of
expansion, a **parameter expansion**. For example, after the first
assignment above.

      print -- '$foo is "'$foo'"'

gives

      $foo is "This is a parameter."

so you can see what I meant about the effect of single quotes. Note the
asymmetry --- there is no \``$`' when assigning the parameter, but there
is a \``$`' in front to have it expanded it into the command line. You
may find the word \`substitution' used instead of \`expansion'
sometimes; I'll try and stick with the terminology in the manual.

Two more things while we're at it. First, why did I put \``-``-`' after
the `print`? That's because **print**, like many UNIX commands, can take
options after it which begin with a \``-`'. \``-``-`' says that there
are no more options; so if what you're trying to print begins with a
\``-`', it will still print out. Actually, in this case you can see it
doesn't, so you're safe; but it's a good habit to get into, and I wish I
had. As always in zsh, there are exceptions; for example, if you use the
`-R` option to print before the \``-``-`', it only recognizes BSD-style
options, which means it doesn't understand \``-``-`'. Indeed, zsh
programmers can be quite lax about standards and often use the old, but
now non-standard, single \``-`' to show there are no more options.
Currently, this works even after `-R`.

The next point is that I didn't put spaces between the single quotes and
the `$foo` and it was still expanded --- expansion happens anywhere the
parameter is not quoted; it doesn't have to be on its own, just
separated from anything which might make it look like a different
parameter. This is one of those things that can help make shell scripts
look so barbaric.

As well as defining your own parameters, there are also a number which
the shell sets itself, and some others which have a special effect when
you set them. All the above still applies, though. For the rest of this
guide, I will indicate parameters with the \``$`' stuck in front, to
remind you what they are, but you should remember that the \``$`' is
missing when you set them, or, indeed, any time when you're referring to
the name of the parameter instead of its value.

### 2.4.1: Arrays

There is a special type of parameter called an **array** which zsh
inherited from both ksh and csh. This is a slightly shaky marriage,
since some of the things those two shells do with them are not
compatible, and zsh has elements of both, so you need to be careful if
you've used arrays in either. The option `KSH_ARRAYS` is something you
can set to make them behave more like they do in ksh, but a lot of zsh
users write functions and scripts assuming it isn't set, so it can be
dangerous.

Unlike normal parameters (known as **scalars**), arrays have more than
one word in them. In the examples above, we made the parameter `$foo`
get a string with spaces in, but the spaces weren't significant. If we'd
done

      foo=(This is a parameter.)

(note the absence of quotes), it would have created an array. Again,
there must be no space between the \``=`' and the \`(', though inside
the parentheses spaces separate words just like they do on a command
line. The difference isn't obvious if you try and print it --- it looks
just the same --- but now try this:

      print -- ${foo[4]}

and you get \``parameter.`'. The array stores the words separately, and
you can retrieve them separately by putting the number of the element of
the array in square brackets. Note also the braces \``{...}`' --- zsh
doesn't always require them, but they make things much clearer when
things get complicated, and it's never wrong to put them in: you could
have said \``${foo}`' when you wanted to print out the complete
parameter, and it would be treated identically to \``$foo`'. The braces
simply screen off the expansion from whatever else might be lying around
to confuse the shell. It's useful too in expressions like \``${foo}s`'
to keep the \``s`' from being part of the parameter name; and, finally,
with `KSH_ARRAYS` set, the braces are compulsory, though unfortunately
arrays are indexed from 0 in that case.

You can use quotes when defining arrays; as before, this protects
against the shell thinking the spaces are between different elements of
the array. Try:

      foo=('first element' 'second element')
      print -- ${foo[2]}

Arrays are useful when the shell needs to keep a whole series of
different things together, so we'll meet some you may want to put in a
startup file. Users of ksh will have noticed that things are a bit
different in zsh, but for now I'll just assume you're using the normal
zsh way of doing things.

2.5: What to put in your startup files
--------------------------------------

At the last count there were over 130 options and several dozen
parameters which are special to the shell, and many of them deal with
things I won't talk about till much later. But as a guide to get you
started, and an indication of what's to come, here are some options and
parameters you might want to think about setting in `~/.zshrc`.

### 2.5.1: Compatibility options: `SH_WORD_SPLIT` and others

I've already mentioned that zsh works differently from ksh, its nearest
standard relative, and that some of these differences can be confusing
to new users, for example the use of arrays. Some options like
`KSH_ARRAYS` exist to allow you to have things work the ksh way. Most of
these are fairly finnicky, but one catches out a lot of people. Above, I
said that after

      foo='This is a parameter.'

then `$foo` would be treated as one word. In traditional Bourne-like
shells including sh, ksh and bash, however, the shell will split `$foo`
on any spaces it finds. So if you run a command

      command $foo

then in zsh the command gets a single argument
\``This is a parameter.`', but in the other shells it gets the first
argument \``This`', the second argument \``is`', and so on. If you like
this, or are so used to it it would be confusing to change, you should
set the option `SH_WORD_SPLIT` in your `~/.zshrc`. Most experienced zsh
users use arrays when they want word splitting, since as I explained you
have control over what is split and what is not; that's why
`SH_WORD_SPLIT` is not set by default. Users of other shells just get
used to putting things in double quotes,

      command "$foo"

which, unlike single quotes, allow the \``$`' to remain special, and
have the side effect that whatever is in quotes will remain a single
word (though there's an exception to that, too: the parameter `$@`).

There are a lot of other options doing similar things to keep users of
standard shells happy. Many of them simply turn features off, because
the other shell doesn't have them and hence unexpected things might
happen, or simply tweak a feature which is a little different or doesn't
usually matter. Currently such options include `NO_BANG_HIST`,
`BSD_ECHO` (sh only), `IGNORE_BRACES`, `INTERACTIVE_COMMENTS`,
`KSH_OPTION_PRINT`, `NO_MULTIOS`, `POSIX_BUILTINS`, `PROMPT_BANG`,
`SINGLE_LINE_ZLE` (I've written them how they would appear as an
argument to `setopt` to put the option the way the other shell expects,
so some have \``NO_`' in front). Most people probably won't change those
unless they notice something isn't working how they expect.

Some others have more noticeable effects. Here are a few of the ones
most likely to make you scratch your head if you're changing from
another Bourne-like shell.

**`BARE_GLOB_QUAL`, `GLOB_SUBST`, `SH_FILE_EXPANSION`, `SH_GLOB`,
`KSH_GLOB`**\
\

These are all to do with how pattern matching works. You probably
already know that the pattern \``*.c`' will be expanded into all the
files in the current directory ending in \``.c`'. Simple uses like this
are the same in all shells, and the way filenames are expanded is often
referred to as \`globbing' for historical reasons (apparently it stood
for \`global replacement'), hence the name of some of these options.

However, zsh and ksh differ over more complicated patterns. For example,
to match either file `foo.c` or file `bar.c`, in ksh you would say
`@(foo|bar).c`. The usual zsh way of doing things is `(foo|bar).c`. To
turn on the ksh way of doing things, set the option `KSH_GLOB`; to turn
off the zsh way, set the options `SH_GLOB` and `NO_BARE_GLOB_QUAL`. The
last of those turns off **qualifiers**, a very powerful way of selecting
files by type (for example, directories or executable files) instead of
by name which I'll talk about in [chapter 5](zshguide05.html#subst).

The other two need a bit more explanation. Try this:

      foo='*'
      print $foo

In zsh, you usually get a \``*`' printed, while in ksh the \``*`' is
expanded to all the files in the directory, just as if you had typed
\``print *`'. This is a little like `SH_WORD_SPLIT`, in that ksh is
pretending that the value of `$foo` appears on the command line just as
if you typed it, while zsh is using what you assigned to `foo` without
allowing it to be changed any more. To allow the word to be expanded in
zsh, too, you can set the option `GLOB_SUBST`. As with `SH_WORD_SPLIT`,
the way around the ksh behaviour if you don't want the value changed is
to use double quotes: `"$foo"`.

You are less likely to have to worry about `SH_FILE_EXPANSION`. It
determines when the shell expands things like `~/.zshrc` to the full
path, e.g. `/home/user2/pws/.zshrc`. In the case of zsh, this is usually
done quite late, after most other forms of expansion such as parameter
expansion. That means if you set `GLOB_SUBST` and do

      foo='~/.zshrc'
      print $foo

you would normally see the full path, starting with a \``/`'. If you
*also* set `SH_FILE_EXPANSION`, however, the \``~`' is tested much
earlier, before `$foo` is replaced when there isn't one yet, so that
\``~/.zshrc`' would be printed. This (with both options) is the way ksh
works. It also means I lied when I said ksh treats `$foo` exactly as if
its value had been typed, because if you type `print ~/.zshrc` the
\``~`' does get expanded. So you see how convenient lying is.

**`NOMATCH`, `BAD_PATTERN`**\
\

These also relate to patterns which produce file names, but in this case
they determine what happens when the pattern doesn't match a file for
some reason. There are two possible reasons: either no file happened to
match, or you didn't use a proper pattern. In both cases, zsh, unlike
ksh, prints an error message. For example,

      % print nosuchfile*
      zsh: no matches found: nosuchfile*
      % print [-
      zsh: bad pattern: [-

(Remember the \``%`' lines are what you type, with a prompt in front
which comes from the shell.) You can see there are two different error
messages: you can stop the first by setting `NO_NOMATCH`, and the second
by setting `NO_BAD_PATTERN`. In both cases, that makes the shell print
out what you originally type without any expansion when there are no
matching files.

**`BG_NICE`, `NOTIFY`**\
\

All UNIX shells allow you to start a *background* job by putting \``&`'
at the end of the line; then the shell doesn't wait for the job to
finish, so you can type something else. In zsh, such jobs are usually
run at a lower priority (a \`higher nice value' in UNIX-speak), so that
they don't use so much of the processor's time as foreground jobs (all
the others, without the \``&`') do. This is so that jobs like editing or
using the shell don't get slowed down, which can be highly annoying. You
can turn this feature off by setting `NO_BG_NICE`.

When a background job finishes, zsh usually tells you immediately by
printing a message, which interrupts whatever you're doing. You can stop
this by setting `NO_NOTIFY`. Actually, this is an option in most
versions of ksh, too, but it's a little less annoying in zsh because if
it happens while you're typing something else to the shell, the shell
will reprint the line you were on as far as you've got. For example:

      % sleep 3 &
      [1] 40366
      % print The quick brown
      [1]  + 40366 done       sleep 3
      % print The quick brown

The command sleep simply does nothing for however many seconds you tell
it, but here it did it in the background (zsh printed a message to tell
you). After you typed for three seconds, the job exited, and with
`NOTIFY` set it printed out another message: the \``done`' is the key
thing, as it tells you the job has finished. But zsh was smart enough to
know the display was messed up, so it reprinted the line you were
editing, and you can continue. If you were already running another
programme in the foreground, however, that wouldn't know that zsh had
printed the message, so the display would still be messed up.

**`HUP`**\
\

Signals are the way of persuading a job to do something it doesn't want
to, such as die; when you type `^C`, it sends a signal (called `SIGINT`
in this case) to the job. In zsh, if you have a background job running
when the shell exits, the shell will assume you want that to be killed;
in this case it is sent a particular signal called \``SIGHUP`' which
stands for \`hangup' (as in telephone, not as in Woody Allen) and is the
UNIX equivalent of \`time to go home'. If you often start jobs that
should go on even when the shell has exited, then you can set the option
`NO_HUP`, and background jobs will be left alone.

**`KSH_ARRAYS`**\
\

I've already mentioned this, but here are the details. Suppose you have
defined an array `arr`, for example with

      arr=(foo bar)

although the syntax in ksh, which zsh also allows, is

      set -A arr foo bar

In zsh, `$arr` gives out the whole array; in ksh it just produces the
first element. In zsh, `${arr[1]}` refers to the first element of the
array, i.e. `foo`, while in ksh the first element is referred to as
`${arr[0]}` so that `${arr[1]}` gives you `bar`. Finally, in zsh you can
get away with `$arr[1]` to refer to an element, while ksh insists on the
braces. By setting `KSH_ARRAYS`, zsh will switch to the ksh way of doing
things. This is one option you need to be particularly careful about
when writing functions and scripts.

**`FUNCTION_ARG_ZERO`**\
\

Shell functions are a useful way of specifying a set of commands to be
run by the shell. Here's a simple example:

      % fn() { print My name is $0; }
      % fn
      My name is fn

Note the special syntax: the \``()`' appears after a function name to
say you are defining one, then a set of commands appears between the
\``{ ... }`'. When you type the name of the function, those commands are
executed. If you know the programming language C, the syntax will be
pretty familiar, although note that the \``()`' is a bit of a delusion:
you might think you would put arguments to the function in there, but
you can't, it must always appear simply as \``()`'. If you don't know C,
it doesn't matter; nothing from C really applies in detail, it's just a
superficial resemblance.

In this case, zsh printed the special parameter \``$0`' (\`argument
zero') and, as you see, that turned into the name of the function. Now
`$0` outside a function means the name of the shell, or the name of the
script for a non-interactive shell, so if you type \``print $0`' it will
probably say \``zsh`'. In most versions of ksh, this is `$0`'s only use;
it doesn't change in functions, and \`fn' would print \`ksh'. To get
this behaviour, you can set `NO_FUNCTION_ARG_ZERO`. There's probably no
reason why you would want to, but zsh functions quite often test their
own name, so this is one reason why they might not work.

There's another difference when defining functions, irrespective of
`FUNCTION_ARG_ZERO`: in zsh, you can get away without the final \``;`'
before the end of the definition of `fn`, because it knows the \``}`'
must finish the last command as well as the function; but ksh is not so
forgiving here. Lots of syntactic know-alls will probably be able to
tell you why that's a good thing, but fortunately I can't.

**`KSH_AUTOLOAD`**\
\

There's an easy way of loading functions built into both ksh and zsh.
Instead of putting them all together in a big startup file, you can put
a single line in that,

      autoload fn

and the function \``fn`' will only be loaded when you run it by typing
its name as a command. The shell needs to know where the function is
stored. This is done by a special parameter called `$fpath`, an array
which is a list of directories; it will search all the directories for a
file called `fn`, and use that as the function definition. If you want
to try this you can type \``autoload fn; fpath=(. $fpath)`' and write a
file called `fn` in the current directory.

Unfortunately ksh and zsh disagree a bit about what should be in that
file. The normal zsh way of doing things is just putting the body of the
function there. So if the file `fn` is autoloadable and contains,

      # this is a simple function
      print My name is $0

then typing \``fn`' will have exactly the same effect as the function
`fn` above, printing \``My name is fn`'. Zsh users tend to like this
because the function is written the same way as a script; if instead you
had typed `zsh fn`, to call the file as a script with a new copy of zsh
of its own, it would have worked the same way. The first line is a
comment; it's ignored, and in zsh not even autoloaded when the function
is run, so it's not only much clearer to add explanatory contents, it
also doesn't use any more memory either. It uses more disk space, of
course, but nowadays even home PCs come with the sort of disk size which
allows you a little indulgence with legibility.

However, ksh does things differently, and here the file `fn` needs to
contain

      fn() {
        # this is a simple function
        print My name is $0
      }

in other words, exactly what you would type to define the function. The
advantage of this form is that you can put other things in the file,
which will then be run straight away and forgotten about, such as
defining things that `fn` may need to use but which don't need to be
redefined every single time you run `fn`. The option to force zsh to
work the ksh way here is called `KSH_AUTOLOAD`. (If you wanted to try
the second example, you would need to type
\``unfunction fn; autoload fn`' to remove the function from memory and
mark it for autoloading again.)

Actually, zsh is a little bit cleverer. If the option `KSH_AUTOLOAD` is
not set, but the file contains just a function definition in the ksh
form and nothing else (like the last one above, in fact), then zsh
assumes that it needs to run the function just loaded straight away. The
other possibility would be that you wanted to define a function which
did nothing other than define a function of the same name, which is
assumed to be unlikely --- and if you really want to do that, you will
need to trick zsh by putting a do-nothing command in the same file, such
as a \``:`' on the last line.

A final complication --- sorry, but this one actually happens --- is
that sometimes in zsh you want to define not just the function to be
called, but some others to help it along. Then you need to do this:

      fn() {
        # this is the function after which the file is named
      }
      helper() {
        # goodness knows what this does
      }
      fn "$@"
      # this actually calls the function the first time,
      # with any arguments passed (see the subsection
      # `Function Parameters' in the section `Functions'
      # of the next chapter for the "$@").

That last non-comment line is unnecessary with `KSH_AUTOLOAD`. The
functions supplied with zsh assume that `KSH_AUTOLOAD` is not set,
however, so you shouldn't turn it on unless you need to. You could just
make `fn` into the whole body, as usual, and define `helper` inside
that; the problem is that `helper` would be redefined each time you
executed `fn`, which is inefficient. A better way of avoiding the
problem would be to define helper as a completely separate function,
itself autoloaded: in both zsh and ksh, it makes no difference whether a
function is defined inside another function or outside it, unlike (say)
Pascal or Scheme.

**`LOCAL_OPTIONS`, `LOCAL_TRAPS`**\
\

These two options also refer to functions, and here the ksh way of doing
things is usually preferable, so many people set at least
`LOCAL_OPTIONS` in a lot of their functions. The first versions of zsh
didn't have these, which is why you need to turn them on by hand.

If `LOCAL_OPTIONS` is set in a function (or was already set before the
function, and not unset inside it), then any options which are changed
inside the function will be put back the way they were when the function
finishes. So

      fn() {
        setopt localoptions kshglob
        ...
      }

allows you to use a function with the ksh globbing syntax, but will make
sure that the option `KSH_GLOB` is restored to whatever it was before
when the function exits. This works even if the function was interrupted
by typing `^C`. Note that `LOCAL_OPTIONS` will itself be restored to the
way it was.

The option `LOCAL_TRAPS`, which first appeared in version 3.1.6, is for
a similar reason but refers to (guess what) **traps**, which are a way
of stopping signals sent to the shell, for example by typing `^C` to
cancel something (`SIGINT`, short for \`signal interrupt'), or `^Z` to
suspend it temporarily (`SIGTSTP`, \`signal terminal stop'), or `SIGHUP`
which we've already met, and so on. To do something of your own when the
shell gets a `^C`, you can do

      trap 'print I caught a SIGINT' INT

and the set of commands in quotes will be run when the `^C` arrives (you
can even try it without running anything). If the string is empty (just
`'``'` with nothing inside), the signal will be ignored; typing `^C` has
no effect. To put it back to normal, the command is \``trap - INT`'.

Traps are most useful in functions, where you may temporarily (say) not
want things to stop when you hit `^C`, or you may want to clear up
something before returning from the function. So now you can guess what
`LOCAL_TRAPS` does; with

      fn() {
        setopt localoptions localtraps
        trap '' INT
        ...
      }

the shell will ignore `^C`'s to the end of the function, but then put
back the trap that was there before, or remove it completely if there
was none. Traps are described in more detail in [chapter
3](zshguide03.html#syntax).

There is a very convenient shorthand for making options and traps local,
as well as for setting the others to their standard values: put
\``emulate -L zsh`' at the start of a function. This sets the option
values back to the ones set when zsh starts, but with `LOCAL_OPTIONS`
and `LOCAL_TRAPS` set too, so you now know exactly how things are going
to work for the rest of the function, whatever options are set in the
outside world. In fact, this only changes the options which affect
normal programming; you can set every option which it makes sense to set
to its standard value with \``emulate -RL zsh`' (it doesn't, for
example, make sense to change options like `login` at this point).
Furthermore, you can make the shell behave as much like ksh as it knows
how to by doing \``emulate -L ksh`', with or without the `-R`.

The `-L` option to `emulate` actually only appears in versions from
3.0.6 and 3.1.6. Before that you needed

      emulate zsh
      setopt localoptions

since `localtraps` didn't exist, and indeed doesn't exist in 3.0.6
either.

**`PROMPT_PERCENT`, `PROMPT_SUBST`**\
\

As promised, setting prompts will be discussed later, but for now there
are two ways of getting information into prompts, such as the parameter
`$PS1` which determines the usual prompt at the start of a new command
line. One is by using *percent escapes*, which means a \``%`' followed
by another character, maybe with a number between the two. For example,
the default zsh prompt is \``%m%# `'. The first percent escape turns
into the name of the host computer, the second usually turns into a
\``%`', but a \``#`' for the superuser. However, ksh doesn't have these,
so you can turn them off by setting `NO_PROMPT_PERCENT`.

The usual ksh way of doing things, on the other hand, is by putting
parameters in the prompt to be substituted. To get zsh to do this, you
have to set `PROMPT_SUBST`. Then assigning

      PS1='${PWD}% '

is another way of putting the name of the current directory (\``$PWD`'
is presumably named after the command \`pwd' to \`print working
directory') into the prompt. Note the single quotes, so that this
happens when the prompt is shown, not when it is assigned. If they
weren't there, or were double quotes, then the `$PWD` would be expanded
to the directory when the assignment took place, probably your home
directory, and wouldn't change to reflect the directory you were
actually in. Of course, you need the quotes for the space, too, else it
just gets swallowed up when the assignment is executed.

As there is potentially much more information available in parameters
than the fixed number of predefined percent escapes, you may wish to set
`PROMPT_SUBST` anyway. Furthermore, you can get the output of commands
into prompts since other forms of expansion are done on them, not just
that of parameters; in fact, prompts with `PROMPT_SUBST` are expanded
pretty much the same as a string inside double quotes every time the
prompt is displayed.

**`RM_STAR_SILENT`**\
\

Everybody at some time or another deletes more files than they mean to
(and *that's* a gross understatement); my favourite is:

      rm *>o

That \``>`' should be a \`.', but I still had the shift key pressed.
This removes all files, echoing the output (there isn't any) into a file
\`o'. Delightfully, the empty file \`o' is not removed. (Don't try this
at home.)

There is a protection mechanism built into zsh to stop you deleting all
the files in a directory by accident. If zsh finds that the command is
\``rm`', and there is a \``*`' on the command line (there may be other
stuff as well), then it will ask you if you really want to delete all
those files. You can turn this off by setting `RM_STAR_SILENT`.
Overreliance on this option is a bad idea; it's only a last line of
defence.

**`SH_OPTION_LETTERS`**\
\

Many options also have single letters to stand for them; you can set an
option in this way by, for example, \``set -f`', which sets `NO_RCS`.
However, even where sh, ksh and zsh share options, not all have the same
letters. This option allows the single letter options to be more like
those in sh and ksh. Look them up in the manual if you want to know, but
I already recommended that you use the full names for options anyway.

**`SH_WORD_SPLIT`**\
\

I've already talked about this, see above, but it's mentioned here so
you don't forget it, since it's an important difference.

**Starting zsh as ksh**\
\

Finally on the subject of compatibility, you might like to know that as
well as \``emulate`' there is another way of forcing zsh to behave as
much like sh or ksh as possible. This is by actually calling zsh under
the name ksh. You don't need to rename zsh, you can make a link from the
name zsh to the name ksh, which will be enough to convince it.

There is an easier way when you are doing this from within zsh itself.
The parameter `$ARGV0` is special; it is the value which will be passed
as the first argument of a command which is run by the shell. Normally
this is the name of the command, but it doesn't have to be since the
command only finds out what it is after it has already been run. You can
use it to trick a programme into thinking its name is different. So

      ARGV0=ksh zsh

will start a copy of zsh that tries to make itself like ksh. Note this
doesn't work unless you're already in zsh, as the `$ARGV0` won't be
special.

I haven't mentioned putting a parameter assignment before a command
name, but that simply assigns the parameter (strictly an environment
variable in this case) for the duration of the command; the value
`$ARGV0` won't be set after that command (the ksh-like zsh) finishes, as
you can easily test with `print`. While I'm here, I should mention a few
of its other features. First, the parameter is automatically exported to
the environment, meaning it's available for other programmes started by
zsh (including, in this case, the new zsh) --- see the section on
environment variables below. Second, this doesn't do what you might
expect:

      FOO=bar print $FOO

because of the order of expansion: the command line and its parameters
are expanded before execution, giving whatever value `$FOO` had before,
probably none, then FOO=bar is put into the environment, and then the
command is executed but doesn't use the new value of `$FOO`.

### 2.5.2: Options for csh junkies

As well as old ksh users, there are some options available to make old
csh and tcsh users feel more at home. As you will already have noticed,
the syntax is very different, so you are never going to feel completely
at home and it might be best just to remember the fact. But here is a
brief list. The last, `CSH_NULL_GLOB`, is actually quite useful.

**`CSH_JUNKIE_HISTORY`**\
\

Zsh has the old csh mechanism for referring to words on a previous
command line using a \``!`'; it's less used, now the editor is more
powerful, but is still a convenient shorthand for extracting short bits
from the previous line. This mechanism is sometimes called
**bang-history**, since busy people sometimes like to say \``!`' as
\`bang'. This option affects how a single \``!`' works. For example,

      % print foo bar
      % print open closed
      % print !-2:1 !:2

In the last line, \``!-2`' means two entries ago, i.e. the line
\``print foo bar`'. The \``:1`' chooses the first word after the
command, i.e. \``foo`'. In the second expression, no number is given
after the \``!`'. Usually zsh interprets that to mean that the same item
just selected, in this case -2, should be used. With
`CSH_JUNKIE_HISTORY` set, it refers instead to the last command. Note
that if you hadn't given that -2, it would refer to the last command in
any case, although the explicit way of referring to the last command is
\``!!`' --- you have to use that if there are no \``:`' bits following.
In summary, zsh usually gives you \``print foo bar`'; with
`CSH_JUNKIE_HISTORY` you get \``print foo closed`'.

There's another option controlling this, `BANG_HIST`. If you unset that,
the mechanism won't work at all. There's also a parameter, `$histchars`.
The first character is the main history expansion character, normally
\``!`' of course; the second is for rapid substitutions (normally \``^`'
--- use of this is described below); the third is the character
introducing comments, normally \``#`'. Changing the third character is
definitely not recommended. There's little real reason to change any.

**`CSH_JUNKIE_LOOPS`**\
\

Normal zsh loops look something like this,

      while true; do
        print Never-ending story
      done

which just prints the message over and over (type it line-by-line at the
prompt, if you like, then `^C` to stop it). With `CSH_JUNKIE_LOOPS` set,
you can instead do

      while true
        print Never-ending story
      end

which will, of course, make your zsh code unlike most other people's, so
for most users it's best to learn the proper syntax.

**`CSH_NULL_GLOB`**\
\

This is another of the family of options like `NO_NOMATCH`, already
mentioned. In this case, if you have a command line consisting of a set
of patterns, at least one of them must match at least one file, or an
error is caused; any that don't match are removed from the command line.
The default is that all of them have to match. There is one final member
of this set of options, `NULL_GLOB`: all non-matching patterns are
removed from the command line, no error is caused. As a summary, suppose
you enter the command \``print file1* file2*`' and the directory
contains just the file `file1.c`.

1.  By default, there must be files matching both patterns, so an error
    is reported.
2.  With `NO_NOMATCH` set, any patterns which don't match are left
    alone, so \``file1.c file2*`' is printed.
3.  With `CSH_NULL_GLOB` set, `file1*` matched, so `file2*` is silently
    removed; \``file1.c`' is reported. If that had not been there, an
    error would have been reported.
4.  With `NULL_GLOB` set, any patterns which don't match are removed, so
    again \``file1.c`' is printed, but in this case if that had not been
    there a blank line would have been printed, with no error.

`CSH_NULL_GLOB` is good thing to have set since it can keep you on the
straight and narrow without too many unwanted error messages, so this
time it's not just for csh junkies.

**`CSH_JUNKIE_QUOTES`**\
\

Here just for completeness. Csh and friends don't allow multiline
quotes, as zsh does; if you don't finish a pair of quotes before a new
line, csh will complain. This option makes zsh do the same. But
multi-line quotes are very useful and very common in zsh scripts and
functions; this is only for people whose minds have been really screwed
up by using csh.

### 2.5.3: The history mechanism: types of history

The name \`history mechanism' refers to the fact that zsh keeps a
\`history' of the commands you have typed. There are three ways of
getting these back; all these use the same set of command lines, but the
mechanisms for getting at them are rather different. For some reason,
items in the history list (a complete line of input typed and executed
at once) have become known as \`events'.

**Editing the history directly**\
\

First, you can use the editor; usually hitting up-arrow will take you to
the previous line, and down-arrow takes you back. This is usually the
easiest way, since you can see exactly what you're doing. I will say a
great deal more about the editor in [chapter 4](zshguide04.html#zle);
the first thing to know is that its basic commands work either like
emacs, or like vi, so if you know one of those, you can start editing
lines straight away. The shell tries to guess whether to use emacs or vi
from the environment variables `$VISUAL` or `$EDITOR`, in that order;
these traditionally hold the name of your preferred editor for
programmes which need you to edit text. In the old days, `$VISUAL` was a
full-screen editor and `$EDITOR` a line editor, like `ed` of blessed
memory, but the distinction is now very blurred. If either contains the
string `vi`, the line editor will start in vi mode, else it will start
in emacs mode. If you're in the wrong mode, \``bindkey -e`' in
`~/.zshrc` takes you to emacs mode and \``bindkey -v`' to vi mode. For
vi users, the thing to remember is that you start in insert mode, so
type \``ESC`' to be able to enter vi commands.

**\`Bang'-history**\
\

Second, you can use the csh-style \`bang-history' mechanism (unless you
have set the option `NO_BANG_HIST`); the \`bang' is the exclamation
mark, \`!', also known as \`pling' or \`shriek' (or factorial, but
that's another story). Thus \``!!`' retrieves the last command line and
executes it; \``!-2`' retrieves the second last. You can select words:
\``!!:1`' picks the first word after the command of the last command (if
you were paying attention above, you will note you just need one \``!`'
in that case); `0` after colon would pick the command word itself;
\``*`' picks all arguments after the command; \``$`' picks the last
word. You can even have ranges: \``!!:1-3`' picks those three words, and
things like \``!!:3-$`' work too.

After the word selector, you can have a second set of colons and then
some special commands called **modifiers** --- these can be very useful
to remember, since they can be applied to parameters and file patterns
to, so here's some more details. The \``:t`' (tail) modifier picks the
last part of a filename, everything after the last slash; conversely,
\``:h`' (head) picks everything before that. So with a history entry,

      % print /usr/bin/cat
      /usr/bin/cat
      % print !!:t
      print cat
      cat

Note two things: first, the bang-history mechanism always prints what
it's about to execute. Secondly, you don't need the word selector; the
shell can tell that the \``:t`' is a modifier, and assumes you want it
applied to the entire previous command. (Be careful here, since actually
the `:t` will reduce the expression to everything after the last slash
in *any* word, which is a little unexpected.)

With parameters:

      % foo=/usr/bin/cat
      % print ${foo:h}
      /usr/bin

(you can usually omit the \``{`' and \``}`', but it's clearer and safer
with them). And finally with files --- this won't work if you set
`NO_BARE_GLOB_QUAL` for sh-like behaviour:

      % print /usr/bin/cat(:t)
      cat

where you need the parentheses to tell the shell the \``:t`' isn't just
part of the file name.

For a complete list, see the `zshexpn` manual, or the section
`Modifiers` in the printed or Info versions of the manual, but here are
a few more of the most useful. \``:r`' removes the suffix of a file,
turning `file.c` into `file`; \``:l`' and \``:u`' make the word(s) all
lowercase or all uppercase; \``:s/foo/bar/`' substitutes the first
occurrence of `foo` with `bar` in the word(s); \``:gs/foo/bar`'
substitutes all occurrences (the \``g`' stands for global); \``:&`'
repeats the last such substitution, even if you did it on a previous
line; \``:g&`' also works. So

      % print this is this line
      this is this line
      % !!:s/this/that/
      print that is this line
      that is this line
      % print this is no longer this line
      this is no longer this line
      % !!:g&
      print that is no longer that line
      that is no longer that line

Finally, there is a shortcut: `^old^new^` is exactly equivalent to
`!!:s/old/new/`; you can even put another modifier after it. The \``^`'
is actually the second character of `$histchars` mentioned above. You
can miss out the last \``^`' if there's nothing else to follow it. By
the way, you can put modifiers together, but each one needs the colon
with it: `:t:r` applied to \``dir/file.c`' produces \``file`', and
repeated applications of `:h` get you shorter and shorter paths.

Before we leave bang-history, note the option `HIST_VERIFY`. If that's
set, then after a substitution the line appears again with the changes,
instead of being immediately printed and executed. As you just have to
type `<RET>` to execute it, this is a useful trick to save you executing
the wrong thing, which can easily happen with complicated bang-history
lines; I have this set myself.

And one last tip: the shell's expansion and completion, which I will
enthuse about at length later on, allows you to expand bang-history
references straight away by hitting `TAB` immediately after you've typed
the complete reference, and you can usually type control together with
slash (on some keyboards, you are restricted to `^Xu`) to put it back
the way it was if you don't like the result --- this is part of the
editor's \`undo' feature.

**Ksh-style history commands**\
\

The third form of history uses the `fc` builtin. It's the most
cumbersome: you have to tell the command which complete lines to
execute, and may be given a chance to edit them first (but using an
external editor, not in the shell). You probably won't use it that way,
but there are three things which are actually controlled by `fc` which
you might use: first, the \``r`' command repeats the last command
(ignoring `r`'s), which is a bit like \``!!`'. Secondly, the command
called \``history`' is also really `fc` in disguise. It gives you a list
of recent commands. They have numbers next to them; you can use these
with bang-history instead of using negative numbers to count backward in
the way I originally explained, the advantage being they don't change as
you enter more commands. You can give ranges of numbers to `history`,
the first number for where to start listing, and the second where to
stop: a particular example is \``history 1`', which lists all commands
(even if the first command it still remembers is higher than 1; it just
silently omits all those). The third use of `fc` is for reading and
writing your history so you can keep it between sessions.

### 2.5.4: Setting up history

In fact, the shell is able to read and write history without being told.
You need to tell it where to save the history, however, and for that you
have to set the parameter `$HISTFILE` to the name of the file you want
to use (a common choice is \``~/.history`'). Next, you need to set the
parameter `$SAVEHIST` to the number of lines of your history you want
saved. When these two are set, the shell will read `$HISTSIZE` lines
from `$HISTFILE` at the start of an interactive session, and save the
last `$SAVEHIST` lines you executed at the end of the session. For it to
read or write in the middle, you will either need to set one of the
options described below (`INC_APPEND_HISTORY` and `SHARE_HISTORY`), or
use the `fc` command: `fc -R` and `fc -W` read and write the history
respectively, while `fc -A` appends it to the the file (although pruning
it if it's longer than `$SAVEHIST`); `fc -WI` and `fc -AI` are similar,
but the `I` means only write out events since the last time history was
written.

There is a third parameter `$HISTSIZE`, which determines the number of
lines the shell will keep within one session; except for special reasons
which I won't talk about, you should set `$SAVEHIST` to be no more than
`$HISTSIZE`, though it can be less. The default value for `$HISTSIZE` is
30, which is a bit stingy for the memory and disk space of today's
computers; zsh users often use anything up to 1000. So a simple set of
parameters to set in `.zshrc` is

      HISTSIZE=1000
      SAVEHIST=1000
      HISTFILE=~/.history

and that is enough to get things working. Note that you *must* set
`$SAVEHIST` and `$HISTFILE` for automatic reading and writing of history
lines to work.

### 2.5.5: History options

There are also many options affecting history; these increased
substantially with version 3.1.6, which provided for the first time
`INC_APPEND_HISTORY`, `SHARE_HISTORY`, `HIST_EXPIRE_DUPS_FIRST`,
`HIST_IGNORE_ALL_DUPS`, `HIST_SAVE_NO_DUPS` and `HIST_NO_FUNCTIONS`. I
have already described `BANG_HIST`, `CSH_JUNKIE_HISTORY` and
`HIST_VERIFY` and I won't talk about them again.

**`APPEND_HISTORY`, `INC_APPEND_HISTORY`, `SHARE_HISTORY`**\
\

Normally, when it writes a history file, zsh just overwrites everything
that's there. `APPEND_HISTORY` allows it to append the new history to
the old. The shell will make an effort not to write out lines which
should be there already; this can get complicated if you have lots of
zshs running in different windows at once. This option is a good one for
most people to use. `INC_APPEND_HISTORY` means that instead of doing
this when the shell exits, each line is added to the history in this way
as it is executed; this means, for example, that if you start up a zsh
inside the main shell its history will look like that of the main shell,
which is quite useful. It also means the ordering of commands from
different shells running at the same time is much more logical ---
basically just the order they were executed --- so for 3.1.6 and higher
this option is recommended.

`SHARE_HISTORY` takes this one stage further: as each line is added, the
history file is checked to see if anything was written out by another
shell, and if so it is included in the history of the current shell too.
This means that zsh's running in different windows but on the same host
(or more generally with the same home directory) share the same history.
Note that zsh tries not to confuse you by having unexpected history
entries pop up: if you use `!`-style history, the commands from other
session don't appear in the history list until you explicitly type the
`history` command to display them, so that you can be sure what command
you are actually reexecuting. The Korn shell always behaves as if
`SHARE_HISTORY` is set, presumably because it doesn't store history
internally.

**`EXTENDED_HISTORY`**\
\

This makes the format of the history entry more complicated: in addition
to just the command, it saves the time when the command was started and
how long it ran for. The `history` command takes three options which use
this: `history -d` prints the start time of the command; `history -f`
prints that as well as the date; `history -D` (which you can combine
with `-f` or `-d`) prints the command's elapsed time. The date format
can be changed with `-E` for European (*day*.*month*.*year*) and `-i`
for international (*year*-*month*-*day*) formats. The main reasons why
you *wouldn't* want to set this would be shortage of disk space, or
because you wanted your history file to be read by another shell.

**`HIST_IGNORE_DUPS`, `HIST_IGNORE_ALL_DUPS`, `HIST_EXPIRE_DUPS_FIRST`,
`HIST_SAVE_NO_DUPS`, `HIST_FIND_NO_DUPS`**\
\

These options give ways of dealing with the duplicate lines that often
appear in the history. The simplest is `HIST_IGNORE_DUPS`, which tells
the shell not to store a history line if it's the same as the previous
one, thus collapsing a lot of repeated commands down to one; this is a
very good option to have set. It does nothing when duplicate lines are
not adjacent, so for example alternating pairs of commands will always
be stored. The next two options can help here: `HIST_IGNORE_ALL_DUPS`
simply removes copies of lines still in the history list, keeping the
newly added one, while `HIST_EXPIRE_DUPS_FIRST` is more subtle: it
preferentially removes duplicates when the history fills up, but does
nothing until then. `HIST_SAVE_NO_DUPS` means that whatever options are
set for the current session, the shell is not to save duplicated lines
more than once; and `HIST_FIND_NO_DUPS` means that even if duplicate
lines have been saved, searches backwards with editor commands don't
show them more than once.

**`HIST_ALLOW_CLOBBER`, `HIST_REDUCE_BLANKS`**\
\

These allow the history mechanism to make changes to lines as they are
entered. The first affects output redirections, where you use the symbol
`>` to redirect the output of a command or set of commands to a named
file, or use `>``>` to append the output to that file. If you have the
`NO_CLOBBER` option set, then

      touch newfile
      echo hello >newfile

fails, because the \``touch`' command has created `newfile` and
`NO_CLOBBER` won't let you overwrite (clobber) it in the next line. With
`HIST_ALLOW_CLOBBER`, the second line appears in the history as

      echo hello >|newfile

where the `>|` overrides `NO_CLOBBER`. So to get round the `NO_CLOBBER`
you can just go back to the previous line and execute it without editing
it.

The second option, `HIST_REDUCE_BLANKS`, will tidy up the line when it
is entered into the history by removing any excess blanks that mean
nothing to the shell. This can also mean that the line becomes a
duplicate of a previous one even if it would not have been in its
untidied form. It is smart enough not to remove blanks which are
important, i.e. are quoted.

**`HIST_IGNORE_SPACE`, `HIST_NO_STORE`, `HIST_NO_FUNCTIONS`**\
\

These three options allow you to say that certain lines shouldn't go
into the history at all. `HIST_IGNORE_SPACE` means that lines which
begin with a space don't go into the history; the idea is that you
deliberately type a space, which is not otherwise significant to the
shell, before entering any line you want to be forgotten immediately
afterwards. In zsh 4.0.1 this is implemented so that you can always
recall the immediately preceding line for editing, even if it had a
space; but when the next line is executed and entered into the history,
the line beginning with the space is forgotten.

`HIST_NO_STORE` tells the shell not to store `history` or `fc` commands.
while `HIST_NO_FUNCTIONS` tells it not to store function definitions as
these, though usually infrequent, can be tiresomely long. A function
definition is anything beginning \``function funcname {...`' or
\``funcname () { ...`'.

**`NO_HIST_BEEP`**\
\

Finally, `HIST_BEEP` is used in the editor: if you try to scroll up or
down beyond the end of the history list, the shell will beep. It is on
by default, so use `NO_HIST_BEEP` to turn it off.

### 2.5.6: Prompts

Most people have some definitions in `.zshrc` for altering the prompt
you see at the start of each line. I've already mentioned
`PROMPT_PERCENT` (set by default) and `PROMPT_SUBST` (unset by default);
I'll assume here you haven't changed these settings, and point out some
of the possibilities with **prompt escapes**, sequences that start with
a \``%`'. If you get really sophisticated, you might need to turn on
`PROMPT_SUBST`.

The main prompt is in a parameter called either `$PS1` or `$PROMPT` or
`$prompt`; the reason for having all these names is historical --- they
come from different shells --- so I'll just stick with the shortest.
There is also `$RPS1`, which prints a prompt at the right of the screen.
The point of this is that it automatically disappears if you type so far
along the line that you run into it, so it can help make the best use of
space for showing long things like directories.

`$PS2` is shown when the shell is waiting for some more input, i.e. it
knows that what you have typed so far isn't a complete line: it may
contain the start of a quoted expression, but not the end, or the start
of some syntactic structure which is not yet finished. Usually you will
keep it different from `$PS1`, but all the same escapes are understood
in all five prompts.

`$PS3` is shown within a loop started by the shell's `select` mechanism,
when the shell wants you to input a choice: see the `zshmisc` manual
page as I won't say much about that.

`$PS4` is useful in debugging: there is an option `XTRACE` which causes
the shell to print out lines about to be executed, preceded by `$PS4`.
Only from version 3.1.6 has it started to be substituted in the same way
as the other prompts, though this turns out to be very useful --- see
\`Location in script or function' in the following list.

Here are some of the things you might want to include in your prompts.
Note that you can try this out before you alter the prompt by using
\``print -P`': this expands strings just are as they are in prompts. You
will probably need to put the string in single quotes.

**The time**\
\

Zsh allows you lots of different ways of putting the time into your
prompt with percent escapes. The simplest are `%t` and `%T`, the time in
12 and 24 hour formats, and `%*`, the same as `%T` but with seconds; you
can also have the date as (e.g.) \``Wed 22`' using `%w`, as \``9/22/99`'
(US format) using %W, or as \``99-09-22`' (International format) using
%D. However, there is another way of using %D to get many more
possibilities: a following string in braces, \``%D{...}`' can contain a
completely different set of percent escapes all of which refer to
elements of the time and date. On most systems, the documentation for
the `strftime` function will tell you what these are. zsh has a few of
its own, given in the `zshmisc` manual page in the `PROMPT EXPANSION`
section. For example, I use `%D{%L:%M}` which gives the time in hours
and minutes, with the hours as a single digit for 1 to 9; it looks more
homely to my unsophisticated eyes.

You can have more fun by using the \``%`(*num**X*.*true*.*false*)'
syntax, where *X* is one of `t` or `T`. For `t`, if the time in minutes
is the same as *num* (default zero), then *true* is used as the text for
this section of the prompt, while *false* is used otherwise. `T` does
the same for hours. Hence

      PS1='%(t.Ding!.%D{%L:%M})%# '

prints the message \``Ding!`' at zero minutes past the hour, and a more
conventional time otherwise. The \``%#`' is the standard sequence which
prints a \``#`' if you are the superuser (root), or a \``%`' for
everyone else, which occurs in a lot of people's prompts. Likewise, you
could use \``%`(`30t.Dong!.`...' for a message at half past the hour.

**The current directory**\
\

The sequence \``%~`' prints out the directory, with any home or named
directories (see below) shortened to the form starting with `~`; the
sequence \``%/`' doesn't do that shortening, so usually \``%~`' is
better. Directories can be long, and there are various ways to deal with
it. First, if you are using a windowing system you can put the directory
in the title bar, rather than anywhere inside the window. Second, you
can use `$RPS1` which disappears when you type near it. Third, you can
pick segments out of \``%~`' or \``%/`' by giving them a number after
the \``%`': for example, \``%1~`' just picks out the last segment of the
path to the current directory.

The fourth way gives you the most control. Prompts or parts of prompts,
not just bits showing the directory, can be truncated to any length you
choose. To truncate a path on the left, use something like
\``%10<`*...*`<%~`'. That works like this: the \``%<``<`' is the basic
form for truncation. The 10 after the \``%`' says that anything
following is limited to 10 characters, and the characters \`*...*' are
to be displayed whenever the prompt would otherwise be longer than that
(you can leave this empty). This applies to anything following, so now
the `%~` can't be longer than 10 characters, otherwise it will be
truncated (to 7 characters, once the \``...`' has been printed). You can
turn off truncation with \``%<``<`', i.e. no number after the \``%`';
truncation then applies to the entire region between where it was turned
on and where it was turned off (this has changed from older versions of
zsh, where it just applied to individual \``%`' constructs).

**What are you waiting for?**\
\

The prompt `$PS2` appears when the shell is waiting for you to finish
entering something, and it's useful to know what the shell is waiting
for. The sequence \``%_`' shows this. It's part of the default `$PS2`,
which is \``%_> `'. Hence, if you type \``if true; then`' and `<RET>`,
the prompt will say \``then> `'. You can also use it in the trace
prompt, `$PS4`, to show the same information about what is being
executed in a script or function, though as there is usually enough
information there (as described next) it's not part of the default. In
this case, a number after the \``%`' will limit the depth shown, so with
\``%1_`' only the most recent thing will be mentioned.

**Location in script or function**\
\

The default `$PS4` contains \``%N`' and \``%i`', which tell you the name
of the most recently started function, script, or sourced file, and the
line number being executed inside it; they are not very useful in other
prompts. However, \``%i`' in `$PS1` will tell you the current
interactive line number, which zsh keeps track of, though doesn't
usually show you; the parameter `$LINENO` contains the same information.

Another point to bear about \``%i`' in mind is that the line number
shown applies to the version of a function first read in, not how it
appears with the \``functions`' command, which is tidied up. If you use
autoloaded functions, however, the file containing the function will
usually be what you want to alter, so this shouldn't be a problem when
debugging.

Remember, the `$PS4` display only happens when the `XTRACE` option is
set; as options may be local to functions, and always are to scripts,
you will often need to put an explicit \``setopt xtrace`' at the top of
whatever you are debugging. Alternatively, you can use \``typeset -ft`
*funcname*' to turn on tracing for that function (something I only just
discovered); use \``typeset +ft` *funcname*' to turn it off again.

**Other bits and pieces**\
\

There are many other percent escapes described in the `zshmisc` manual
page, mostly straightforward. For example, \``%h`' shows you the history
entry number, useful if you are using bang-history; \``%m`' shows you
the current host name up to any dot; \``%n`' shows the username.

There are two other features I happen to use myself. First, it's
sometimes convenient to know when the last command failed. Every command
returns a status, which is a number: zero for success, some other number
for some type of failure. You can get this from the parameter \``$?`' or
\``$status`' (again, they refer to the same thing). It's also available
in the prompt as \``%?`', and there's also one of the so-called
\`ternary' expressions with parentheses I described for time, which pick
different strings depending on a test. Here the test is, reasonably
enough, \``%`(`?...`'. Putting these two together, you can get a message
which is only displayed when the exit status is non-zero; I've put an
extra set of parentheses around the number just to make it clearer,
where the \`)' needs to be turned into \``%`)' to stop it marking the
end of the group:

      PS1='%(?..(%?%))%# '

It's also sometimes convenient to know if you're in a subshell, that is
if you've started another shell within the main one by typing \``zsh`'.
You can do this by using another ternary expression:

      PS1='%(2L.+.)%# '

This checks the parameter `SHLVL`, which is incremented every time a new
zsh starts, so if there was already one running (which would have set
`SHLVL` to 1), it will now be 2; and if `SHLVL` is at least 2, an extra
\``+`' is printed in front of the prompt, otherwise nothing. If you're
using a windowing system, you may need to turn the 2 into 3 as there may
be a zsh already running when you first log in, so that the shells in
the windows have `SHLVL` set to 2 already. This depends a good deal on
how your windowing system is set up; finding out more is left as an
exercise for the reader.

**Colours**\
\

Many terminals can now display colours, and it is quite useful to be
able to put these into prompts to distinguish those from the surrounding
text. I often find a programme has just dumped a whole load of output on
my terminal and it's not obvious where it starts. Being able to find the
prompt just before helps a lot.

Colors, like bold or underlined text, use escape sequences which don't
move the cursor. The golden rule for inserting any such escape sequences
into prompts is to surround them with \``%{`' at the start and \``%}`'
at the end. Otherwise, the shell will be confused about the length of
the line. This affects what happens when the line editor needs to redraw
the line, and also changes the position of the right prompt `$RPS1`, if
you use that. You don't need that with the special sequences `%B` and
`%b`, which start and stop bold text, because the shell already knows
what to do with those; it's only random characters which you happen to
know don't move the cursor, though the shell doesn't, that cause the
problem.

In the case of colours, there is a shell function `colors` supplied with
the standard distribution to help you. When loaded and run, it defines
associative array parameters `$fg` and `$bg` which you use to extract
the escape sequences for given colours, for example
`${fg[red]}${bg[yellow]}` produces the sequences for red text on a
yellow background. So for example,

      PS1="%{${bg[white]}${fg[red]}%}%(?..(%?%))\ 
      %{${fg[yellow]}${bg[black]}%}%# "

produces a red-on-white \``(1)`' if the previous programme exited with
status 1, but nothing if it exited with status 0, followed by a
yellow-on-black \``%`' or \``#`' if you are the superuser. Note the use
of the double quotes here to force the parameters to be expanded
straight away --- the escape sequences are fixed, so they don't need to
be re-extracted from the parameters every time the prompt is shown.

Even if your terminal does support colour, there's no guarantee all the
possibilities work, although the basic ANSI colour scheme is fairly
standard. The colours understood are: cyan, white, yellow, magenta,
black, blue, red, grey, green. You can also used \`default', which puts
the terminal back how it was to begin with. In addition, you can use the
basic colours with the parameters `$bg_bold` and `$fg_bold` for bold
varieties of the colours and `$bg_no_bold` and `$fg_no_bold` to switch
explicitly back to non-bold.

**Themes**\
\

There are also a set of themes provided as functions to set up your
prompt to various predefined possibilities. These make use of the
colours set up as described above. See the `zshcontrib` manual page for
how to do this (search for \`prompt themes').

### 2.5.7: Named directories

As already mentioned, \``~/`' at the start of a filename expands to your
home directory. More generally, \``~`*user*`/`' allows you to refer to
the home directory of any other user. Furthermore, zsh lets you define
your own named directories which use this syntax. The basic idea is
simple, since any parameter can be a named directory:

      dir=/tmp/mydir
      print ~dir

prints \`/tmp/mydir'. So far, this isn't any different from using the
parameter as `$dir`. The difference comes if you use the \``%~`'
construct, described above, in your prompt. Then when you change into
that directory, instead of seeing the message \``/tmp/mydir`', you will
see the abbreviation \``~dir`'.

The shell will not register the name of the directory until you force it
to by using \``~dir`' yourself at least once. You can do the following
in your `.zshrc`:

      dir=/tmp/mydir
      bin=~/myprogs/bin
      : ~dir ~bin

where \``:`' is a command that does nothing --- but its arguments are
checked for parameters and so on in the usual way, so that the shell can
put `dir` and `bin` into its list of named directories. A more simple
way of doing this is to set the option `AUTO_NAME_DIRS`; then any
parameter created which refers to a directory will automatically be
turned into a name. The directory must have an absolute path, i.e. its
expanded value, after turning any \``~`'s at the start into full paths,
must begin with a \``/`'. The parameter `$PWD`, which shows the current
directory, is protected from being turned into `~PWD`, since that would
tell you nothing.

### 2.5.8: \`Go faster' options for power users

Here are a few more random options you might want to set in your
`.zshrc`.

**`NO_BEEP`**\
\

Normally zsh will beep if it doesn't like something. This can get
extremely annoying; \``setopt nobeep`' will turn it off. I refer to this
informally as the `OPEN_PLAN_OFFICE_NO_VIGILANTE_ATTACKS` option.

**`AUTO_CD`**\
\

If this option is set, and you type something with no arguments which
isn't a command, zsh will check to see if it's actually a directory. If
it is, the shell will change to that directory. So \``./bin`' on its own
is equivalent to \``cd ./bin`', as long as the directory \``./bin`'
really exists. This is particularly useful in the form \``..`', which
changes to the parent directory.

**`CD_ABLE_VARS`**\
\

This is another way of saving typing when changing directory, though
only one character. If a directory doesn't exist when you try to change
to it, zsh will try and find a parameter of that name and use that
instead. You can also have a \``/`' and other bits after the parameter.
So \`cd `foo/dir`', if there is no directory \``foo`' but there is a
parameter `$foo`, becomes equivalent to \`cd `$foo/dir`'.

**`EXTENDED_GLOB`**\
\

Patterns, to match the name of files and other things, can be very
sophisticated in zsh, but to get the most out of them you need to use
this option, as otherwise certain features are not enabled, so that
people used to simpler patterns (maybe just \``*`', \``?`' and
\``[...]`') are not confused by strange happenings. I'll say much more
about zsh's pattern features, but this is to remind you that you need
this option if you're doing anything clever with \``~`', \``#`', \``^`'
or globbing flags --- and also to remind you that those characters can
have strange effects if you have the option set.

**`MULTIOS`**\
\

I mentioned above that to get zsh to behave like ksh you needed to set
`NO_MULTIOS`, but I didn't say what the `MULTIOS` option did. It has two
different effects for output and input.

First, for output. Here it's an alternative to the `tee` programme. I've
mentioned once, but haven't described in detail, that you could use
`>filename` to tell the shell to send output into a file with a given
name instead of to the terminal. With `MULTIOS` set, you can have more
than one of those redirections on the command line:

      echo foo >file1 >file2

Here, \``foo`' will be written to **both** the named files; zsh copies
the output. The pipe mechanism, which I'll describe better in [chapter
3](zshguide03.html#syntax), is a sort of redirection into another
programme instead of into a file: `MULTIOS` affects this as well:

      echo foo >file1 | sed 's/foo/bar/'

Here, \``foo`' is again written to `file1`, but is also sent into the
pipe to the programme `sed` (\`stream editor') which substitutes
\``foo`' into \``bar`' and (since there is no output redirection in this
part) prints it to the terminal.

Note that the second example above has several times been reported as a
bug, often in a form like:

     some_command 2>&1 >/dev/null | sed 's/foo/bar/'

The intention here is presumably to send standard error to standard
output (the \``2>&1`', a very commonly used shell hieroglyphic), and not
send standard output anywhere (the \``>/dev/null`'). (If you haven't met
the concept of \`standard error', it's just another output channel which
goes to the same place as normal output unless you redirect it; it's
used, for example to send error messages to the terminal even if your
output is going somewhere else.) In this example, too, the `MULTIOS`
feature forces the original standard output to go to the pipe. You can
see this happening if we put in a version of \``some_command`':

     { echo foo error >&2; echo foo not error;  } 2>&1 >/dev/null |
      sed 's/foo/bar/'

where you can consider the stuff inside the \``{...}`' as a black box
that sends the message \`foo error' to standard error, and \`foo not
error' to standard output. With `MULTIOS`, however, the result is

     error bar
      not error bar

because both have been sent into the pipe. Without `MULTIOS` you get the
expected result,

     error bar

as any other Bourne-style shell would produce. There

On input, `MULTIOS` arranges for a series of files to be read in order.
This time it's a bit like using the programme `cat`, which combines all
the files listed after it. In other words,

      cat file1 file2 | myprog

(where `myprog` is some programme that reads all the files sent to it as
input) can be replaced by

      myprog <file1 <file2

which does the same thing. Once again, a pipe counts as a redirection,
and the pipe is read from first, before any files listed after a \``<`':

      echo then this >testfile
      echo this first | cat <testfile

**`CORRECT`, `CORRECT_ALL`**\
\

If you have `CORRECT` set, the shell will check all the commands you
type and if they don't exist, but there is one with a similar name, it
will ask you if you meant that one instead. You can type \``n`' for no,
don't correct, just go ahead; \``y`' for yes, correct it then go ahead;
\``a`' for abort, don't do anything; \``e`' for edit, return to the
editor to edit the same line again. Users of the new completion system
should note this is not the same correction you get there: it's just
simple correction of commands.

`CORRECT_ALL` applies to all the words on the line. It's a little less
useful, because currently the shell has to assume that they are supposed
to be filenames, and will try to correct them if they don't exist as
such, but of course many of the arguments to a command are not
filenames. If particular commands generate too many attempts to correct
their arguments, you can turn this off by putting \``nocorrect`' in
front of the command name. An alias is a very good way of doing this, as
described next.

### 2.5.9: aliases

An alias is used like a command, but it expands into some other text
which is itself used as a command. For example,

      alias foo='print I said foo'
      foo

prints (guess what) \``I said foo`'. Note the syntax for definition ---
you need the \``=`', and you need to make sure the whole alias is
treated by the shell as one word; you can give a whole list of aliases
to the same \``alias`' command. You may be able to think of some aliases
you want to define in your startup files; `.zshrc` is probably the right
place. If you have `CORRECT_ALL` set, the way to avoid the \``mkdir`'
command spell-checking its arguments --- which is useless, because they
*have* to be non-existent for the command to work --- is to define:

      alias mkdir='nocorrect mkdir'

This shows one useful feature about aliases: the alias can contain
something of the same name as itself. When it is encountered in the
expansion text (the right hand side), the shell knows it is not to
expand the alias again, but this time to treat it as a real command.
Note that functions do *not* have this property: functions are more
powerful than aliases and in some cases it is useful for them to call
themselves, It's a common mistake to have functions call themselves over
and over again until the shell complains. I'll describe ways round this
in [chapter 3](zshguide03.html#syntax).

One other way functions are more powerful than aliases is that functions
can take arguments while aliases can't --- in other words, there is no
way of referring inside the alias to what follows it on the command
line, unlike a function, and also unlike aliases in csh (because that
has no functions, that's why). It is just blindly expanded, and the
remainder of the command line stuck on the end. Hence aliases in zsh are
usually kept for quite simple things, and functions are written for
anything more complicated. You couldn't do that trick with
\``nocorrect`' using a function, though, since the function is called
too late: aliases are expanded straight away, so the `nocorrect` is
found in time to be useful. You can almost think of them as just plain
typing abbreviations.

Normal aliases only work when in command position, i.e. at the start of
the command line (more strictly, when zsh is expecting a command). There
are other things called \`global aliases', which you define by the
\``-g`' option to `alias`, which will be expanded at any position on the
command line. You should think seriously before defining these, as they
can have a drastic effect. Note, however, that quoting a word, or even a
single character, will stop an alias being expanded for it.

I only tend to use aliases in interactive shells, so I define them from
`.zshrc`, but you may want to use `.zshenv` if you use aliases more
widely. In fact, to keep my `.zshrc` neat I save all the aliases in a
separate file called `.aliasrc` and in `.zshrc` I have:

      if [[ -r ~/.aliasrc ]]; then
        . ~/.aliasrc
      fi

which checks if there is a readable file `~/.aliasrc`, and if there is,
it runs it in exactly the same way the normal startup files are run. You
can use \``source`' instead of \``.`' if it means more to you; \``.`' is
the traditional Bourne and Korn shell name, however.

### 2.5.10: Environment variables

Often, the manual for a programme will tell you to define certain
environment variables, usually a collection of uppercase letters with
maybe numbers and the odd underscore. These can pass information to the
programme without you needing to use extra arguments. In zsh,
environment variables appear as ordinary shell parameters, although they
have to be defined slightly differently: strictly, the environment is a
special region outside the shell, and zsh has to be told to put a copy
there as well as keeping one of its own. The usual syntax is

      export VARNAME='value'

in other words, like an ordinary assignment, but with \``export`' in
front. Note there is no \``$`' before the name of the environment
variable; all \``export`' and similar statements work the same way. The
easiest place to put these is in `.zshenv` --- hence it's name.
Environment variables will be passed to any programmes run from a shell,
so it may be enough to define them in `.zlogin` or `.zprofile`: however,
any shell started for you non-interactively won't run those, and there
are other possible problems if you use a windowing system which is
started by a shell other than zsh or which doesn't run a shell start-up
file at all --- I had to tweak mine to make it do so. So `.zshenv` is
the safest place; it doesn't take long to define environment variables.
Other people will no doubt give you completely contradictory views, but
that's people for you.

Note that you can't export arrays. If you export a parameter, then
assign an array to it, nothing will appear in the environment; you can
use the external command \``printenv VARNAME`' (again no \``$`' because
the command needs to know the name, not the value) to check. There's a
more subtle problem with arrays, too. The `export` builtin is just a
special case of the builtin **typeset**, which defines a variable
without marking it for export to the environment. You might think you
could do

      typeset array=(this doesn\'t work)

but you can't --- the special array syntax is only understood when the
assignment does not follow a command, not in normal arguments like the
case here, so you have to put the array assignment on the next line.
This is a very easy mistake to make. More uses of `typeset` will be
described in [chapter 3](zshguide03.html#syntax); they include creating
local parameters in functions, and defining special attributes (of which
the \`export' attribute is just one) for parameters.

### 2.5.11: Path

It helps to be able to find external programmes, i.e. anything not part
of the shell, any command other than a builtin, function or alias. The
`$path` array is used for this. Actually, what the system needs is the
environment variable `$PATH`, which contains a list of directories in
which to search for programmes, separated from each other by a colon.
These directories are the individual components of the array `$path`. So
if `$path` contains

      path=(/bin /usr/bin /usr/local/bin .)

then `$PATH` will automatically contain the effect of

      PATH=/bin:/usr/bin:/usr/local/bin:.

without you having to set that. The idea is simply that, while the
system needs `$PATH` because it doesn't understand arrays, it's much
more flexible to be able to use arrays within the shell and hence pretty
much forget about the `$PATH` form.

Changes to the path are similar to changes to environment variables
described above, so all that applies. There's a slight difficulty in
setting `$path` in `.zshenv` however, even though the reasons given
above for doing so still apply. Usually, the path will be set for you,
either by the system, or by the system administrator in one of the
global start up files, and if you change path you will simply want to
add to it. But if your `.zshenv` contains

      path=(~/bin ~/progs/bin $path)

--- which is the right way of adding something to the front of `$path`
--- then every time `.zshenv` is called, `~/bin` and `~/progs/bin` are
stuck in front, so if you start another zsh you will have two sets
there.

You can add tests to see if something's already there, of course. Zsh
conveniently allows you to test for the existence of elements in an
array. By preceding an array index by `(r)` (for reverse), it will try
to find a matching element and return that, else an empty string. Here's
a way of doing that (but don't add this yet, see the next paragraph):

      for dir in ~/bin ~/progs/bin; do
        if [[ -z ${path[(r)$dir]} ]]; then
          path=($dir $path)
        fi 
      done

That `for`... `do` ... `done` is another special shell construct. It
takes each thing after \``in`' and assigns it in turn to the parameter
named before the \``in`' --- `$dir`, but because this is a form of
assignment, the \``$`' is left off --- so the first time round it has
the effect of `dir=~/bin`, and the next time `dir=~/progs/bin`. Then it
executes what's in the loop. The test `-z` checks that what follows is
empty: in this case it will be if the directory `$dir` is not yet in
`$path`, so it goes ahead and adds it in front. Note that the
directories get added in the reverse of the order they appear.

Actually, however, zsh takes all that trouble away from you. The
incantation \``typeset -U path`', where the `-U` stands for unique,
tells the shell that it should not add anything to `$path` if it's there
already. To be precise, it keeps only the left-most occurrence, so if
you added something at the end it will disappear and if you added
something at the beginning, the old one will disappear. Thus the
following works nicely in `.zshenv`:

      typeset -U path
      path=(~/bin ~/progs/bin $path)

and you can put down that \``for`' stuff as a lesson in shell
programming. You can list all the variables which have uniqueness turned
on by typing \``typeset +U`', with \``+`' instead of \``-`', because in
the latter case the shell would show the values of the parameters as
well, which isn't what you need here. The `-U` flag will also work with
colon-separated arrays, like `$PATH`.

### 2.5.12: Mail

Zsh will check for new mail for you. If all you need is to be reminded
of something arriving in your normal folder every now and then, you just
need to set the parameter `$MAIL` to wherever that is: it's typically
one of `/usr/spool/mail`, `/var/spool/mail`, or `/var/mail`.

The array `$mailpath` allows more possibilities. Like `$path`, it has a
colleague in uppercase, `$MAILPATH`, which is a colon-separated array.
The system doesn't need that, this time, so it's mainly there so that
you can export it to another version of zsh; exporting arrays won't
work. As may by now be painfully clear, if you set in `.zshenv` or
`.zshrc`, you don't need to export it, because it's set in each instance
of the shell. The elements of `$mailpath` work like `$MAIL`, so you can
specify different places where mail arrives. That's most useful if you
have a programme like `filter` or `procmail` running to redistribute
arriving mail to different folders. You can specify a different message
for each folder by putting \``?`*message*' at the end. For example, mine
looks like this.

      mailpref=/temp/pws/Mail
      mailpath=($mailpref/newmail
                $mailpref/zsh-new'?New zsh mail' 
                $mailpref/list-new'?New list mail'
                $mailpref/urth-new'?New Urth mail')

Note that zsh knows the array isn't finished until the \`)', even though
the elements are on different lines; this is one very good reason for
setting `$mailpath` rather than `$MAILPATH`, which needs one long chunk.

The other parameter of interest is `$MAILCHECK`, which gives the
frequency in seconds when zsh should check for new mail. The default is
60. Actually, zsh only checks just after a command has finished running
and it is about to print a prompt. Since checking files doesn't take
long, you can usually set this to its minimum value, which is
`MAILCHECK=1`; zero doesn't work because it switches off checking. One
reason why you wouldn't want to do that might be because `$MAIL` and
`$mailpath` can contain directories instead of ordinary files; these
will be checked recursively for any files with something new in them, so
this can be slow.

Finally, there is one associated option, `MAIL_WARNING` (though
`MAIL_WARN` is also accepted for the same thing for reasons of
compatibility with less grammatical shells). The shell remembers when it
found the mail file was checked; next time it checks, it compares the
date. If there is no new mail, but the date of the file changed anyway,
it will print a warning message. This will happen if you read the mail
with your mail reader and put the messages somewhere else. Presumably
you *know* you did that, so the warning may not be all that useful.

### 2.5.13: Other path-like things

There are other pairs like `$path` and `$PATH`. I will keep back talk of
`$cdpath` until I say more about the way zsh handles directories. When I
mentioned `$fpath`, I didn't say there was also `$FPATH`, but there is.
Then there is `$manpath` and `$MANPATH`; these aren't used by the shell
at all, but `$MANPATH`, if exported, is used by the **man** external
command, and `$manpath` gives an easier way to set it.

From 3.1.6 there is a mechanism to define your own such combinations; if
this had been available before, there would have been no need to build
in `$manpath` and `$MANPATH`. In `.zshenv` you would put,

      export -TU TEXINPUTS texinputs

to define such a pair. The `-T` (for tie) is the key to that; I've used
\``export`' even though the basic variable declaration command is
\``typeset`' because you nearly always want to get the colon-separated
version (`$TEXINPUTS` here) visible to the environment, and I've set
`-U` as described above for `$path` because it's a neat feature anyway.
Now you can assign to the array `$texinputs` and let the programme (TeX
or its derivatives) see `$TEXINPUTS`. Another useful variable to do this
with is `$LD_LIBRARY_PATH`, which on most modern versions of UNIX (and
Linux) tells the system where to find the libraries which provide extra
functions when it runs a programme.

### 2.5.14: Version-specific things

Since zsh changes faster than almost any other command interpreter known
to humankind, you will often find you need to find out what version you
are using. This can get a bit verbose; indeed, the parameter you need to
check, which is now `$ZSH_VERSION`, used simply to be called `$VERSION`
before version `3.0`. If you are not using legacy software of that kind,
you can probably get away with tests like this:

      if [[ $ZSH_VERSION == 3.1.<5->* ||
            $ZSH_VERSION == 3.<2->* ||
            $ZSH_VERSION == <4->* ]]; then
        # set feature which appeared first in 3.1.5
      fi

It's like that to be futureproof: it says that if this is a 3.1 release,
it has to be at least 3.1.5, but any 3.2 release (there weren't any), or
any release 4 or later, will also be OK. The \``<5->`' etc. are advanced
pattern matching tests: pattern matching uses the same symbols as
globbing, but to test other things, here what's on the left of the
\``==`'. This one matches any number which is at least 5, for example 6
or 10 or 252, but not 1 or 4. There are also development releases;
nowadays the version numbers look like *X*`.`*Y*`.`*Z*-*tag*-*N* (*tag*
is some short word, the others are numbers) but unless you're keeping up
with development you won't need to look for those, since they aren't
released officially. That \``==`' in the test could also be just \``=`',
but the manual says the former is preferred, so I've used them here,
even though people usually don't bother.

Version 4 of zsh provides a function `is-at-least` to do this for you:
it looks only at the numbers *X*, *Y* and *Z* (and *N* if it exists),
ignoring all letters and punctuation. You give it the minimum version of
the shell you need and it returns true if the current shell is recent
enough. For example, \``is-at-least 3.1.6-pws-9`' will return true if
the current version of zsh is 3.1.6-dev-20 (or 3.1.9, or 4.0.1, and so
on), which is the correct behaviour. As with any other shell function,
you have to arrange for `is-at-least` to be `autoload`ed if you want to
use it.

### 2.5.15: Everything else

There are many other possibilities for things to go in startup files; in
particular, I haven't touched on defining things for the line editor and
setting up completion. There's quite a lot to explain for those, so I'll
come back to those in the appropriate chapters. You just need to
remember that all that stuff should go in `.zshrc`, since you need it
for all interactive shells, and for no others.

