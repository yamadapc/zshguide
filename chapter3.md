

Chapter 3: Dealing with basic shell syntax
==========================================

This chapter is a more thorough examination of much of what appeared in
the [chapter 2](zshguide02.html#init); to be more specific, I assume
you're sitting in front of your terminal about to use the features you
just set up in your initialisation files and want to know enough to get
them going. Actually, you will probably spend most of the time editing
command lines and in particular completing commands --- both of these
activities are covered in later chapters. For now I'm going to talk
about commands and the syntax that goes along with using them. This will
let you write shell functions and scripts to do more of your work for
you.

In the following there are often several consecutive paragraphs about
quite minor features. If you find you read this all through the first
time, maybe you need to get out more. Most people will probably find it
better to skim through to find what the subject matter is, then come
back if they later find they want to know more about a particular aspect
of the shell's commands and syntax.

One aspect of the syntax is left to [chapter 5](zshguide05.html#subst):
there's just so much to it, and it can be so useful if you know enough
to get it right, that it can't all be squashed in here. The subject is
expansion, covering a multitude of things such as parameter expansion,
globbing and history expansions. You've already met the basics of these
in [chapter 2](zshguide02.html#init); but if you want to know how to
pick a particular file with a globbing expression with pinpoint
accuracy, or how to make a single parameter expansion reduce a long
expression to the words you need, you should read that chapter; it's
more or less self-contained, so you don't necessarily need to know
everything in this one.

We start with the most basic issue in any command line interpreter,
running commands. As you know, you just type words separated by spaces,
where the first word is a command and the remainder are arguments to it.
It's important to distinguish between the types of command.

3.1: External commands
----------------------

External commands are the easiest, because they have the least
interaction with the shell --- many of the commands provided by the
shell itself, which are described in the next section, are built into
the shell especially to avoid this difficulty.

The only major issue is therefore how to find them. This is done through
the parameters `$path` and `$PATH`, which, as I described in [chapter
2](zshguide02.html#init), are tied together because although the first
one is more useful inside the shell --- being an array, its various
parts can be manipulated separately --- the second is the one that is
used by other commands called by the shell; in the jargon, `$PATH` is
\`exported to the environment', which means exactly that other commands
called by the shell can see its value.

So suppose your `$path` contains

      /home/pws/bin /usr/local/bin /bin /usr/bin

and you try to run \``ls`'. The shell first looks in `/home/pws/bin` for
a command called `ls`, then in `/usr/local/bin`, then in `/bin`, where
it finds it, so it executes `/bin/ls`. Actually, the operating system
itself knows about paths if you execute a command the right way, so the
shell doesn't strictly need to.

There is a subtlety here. The shell tries to remember where the commands
are, so it can find them again the next time. It keeps them in a
so-called \`hash table', and you find the word \`hash' all over the
place in the documentation: all it means is a fast way of finding some
value, given a particular key. In this case, given the name of a
command, the shell can find the path to it quickly. You can see this
table, in the form \`*key*`=`*value*', by typing \``hash`'.

In fact the shell only does this when the option `HASH_CMDS` is set, as
it is by default. As you might expect, it stops searching when it finds
the directory with the command it's looking for. There is an extra
optimisation in the option `HASH_ALL`, also set by default: when the
shell scans a directory to find a command, it will add all the other
commands in that directory to the hash table. This is sensible because
on most UNIX-like operating systems reading a whole lot of files in the
same directory is quite fast.

The way commands are stored has other consequences. In particular, zsh
won't look for a new command if it already knows where to find one. If I
put a new `ls` command in `/usr/local/bin` in the above example, zsh
would continue to use `/bin/ls` (assuming it had already been found). To
fix this, there is the command `rehash`, which actually empties the
command hash table, so that finding commands starts again from scratch.
Users of csh may remember having to type `rehash` quite a lot with new
commands: it's not so bad in zsh, because if no command was already
hashed, or the existing one disappeared, zsh will automatically scan the
path again; furthermore, zsh performs a `rehash` of its own accord if
`$path` is altered. So adding a new duplicate command somewhere towards
the head of `$path` is the main reason for needing `rehash`.

One thing that can happen if zsh hasn't filled its command hash table
and so doesn't know about all external commands is that the `AUTO_CD`
option, mentioned in the previous chapter and again below, can think you
are trying to change to a particular directory with the same name as the
command. This is one of the drawbacks of `AUTO_CD`.

To be a little bit more technical, it's actually not so obvious that
command hashing is needed at all; many modern operating systems can find
commands quickly without it. The clincher in the case of zsh is that the
same hash table is necessary for command completion, a very commonly
used feature. If you type \``compr<TAB>`', the shell completes this to
\``compress`'. It can only do this if it has a list of commands to
complete, and this is the hash table. (In this case it didn't need to
know where to find the command, just its name, but it's only a little
extra work to store that too.) If you were following the previous
paragraphs, you'll realise zsh doesn't necessarily know *all* the
possible commands at the time you hit `TAB`, because it only looks when
it needs to. For this purpose, there is another option, `HASH_LIST_ALL`,
again set by default, which will make sure the command hash table is
full when you try to complete a command. It only needs to do this once
(unless you alter `$path`), but it does mean the first command
completion is slow. If `HASH_LIST_ALL` is not set, command completion is
not available: the shell could be rewritten to search the path
laboriously every single time you try to complete a command name, but it
just doesn't seem worth it.

The fact that `$PATH` is passed on from the shell to commands called
from it (strictly only if the variable is marked for export, as it
usually is --- this is described in more detail with the `typeset`
family of builtin commands below) also has consequences. Some commands
call subcommands of their own using `$PATH`. If you have that set to
something unusual, so that some of the standard commands can't be found,
it could happen that a command which *is* found nonetheless doesn't run
properly because it's searching for something it can't find in the path
passed down to it. That can lead to some strange and confusing error
messages.

One important thing to remember about external commands is that the
shell continues to exist while they are running; it just hangs around
doing nothing, waiting for the job to finish (though you can tell it not
to, as we'll see). The command is given a completely new environment in
which to run; changes in that don't affect the shell, which simply
starts up where it left off after the command has run. So if you need to
do something which changes the state of the shell, an external command
isn't good enough. This brings us to builtin commands.

3.2: Builtin commands
---------------------

Builtin commands, or builtins for short, are commands which are part of
the shell itself. Since builtins are necessary for controlling the
shell's own behaviour, introducing them actually serves as an
introduction to quite a lot of what is going on in the shell. So a fair
fraction of what would otherwise appear later in the chapter has
accumulated here, one way or another. This does make things a little
tricksy in places; count how many times I use the word \``subtle`' and
keep it for your grandchildren to see.

I just described one reason for builtins, but there's a simpler one:
speed. Going through the process of setting up an entirely new
environment for the command at the beginning, swapping between this
command and anything else which is being run on the computer, then
destroying it again at the end is considerable overkill if all you want
to do is, say, print out a message on the screen. So there are builtins
for this sort of thing.

### 3.2.1: Builtins for printing

The commands \``echo`' and \``print`' are shell builtins; they just show
what you typed, after the shell has removed all the quoting. The
difference between the two is really historical: \``echo`' came first,
and only handled a few simple options; ksh provided \``print`', which
had more complex options and so became a different command. The
difference remains between the two commands in zsh; if you want wacky
effects, you should look to `print`. Note that there is usually also an
external command called `echo`, which may not be identical to zsh's;
there is no standard external command called `print`, but if someone has
installed one on your system, the chances are it sends something to the
printer, not the screen.

One special effect is \``print -z`' puts the arguments onto the editing
buffer stack, a list maintained by the shell of things you are about to
edit. Try:

      print -z print -z print This is a line

(it may look as if something needs quoting, but it doesn't) and hit
return three times. The first time caused everything after the first
\``print -z`' to appear for you to edit, and so on.

For something more useful, you can write functions that give you a line
to edit:

      fn() { print -z print The time now is $(date); }

Now when you type \``fn`', the line with the date appears on the command
line for you to edit. The option \``-s`' is a bit similar; the line
appears in the history list, so you will see it if you use up-arrow, but
it doesn't reappear automatically.

A few other useful options, some of which you've already seen, are

**`-r`**

don't interpret special character sequences like \``\n`'

**`-P`**

use \``%`' as in prompts

**`-n`**

don't put a newline at the end in case there's more output to follow

**`-c`**

print the output in columns --- this means that \``print -c *`' has the
effect of a sort of poor person's \``ls`', only faster

**`-l`**

use one line per argument instead of one column, which is sometimes
useful for sticking lists into files, and for working out what part of
an array parameter is in each element.

If you don't use the `-r` option, there are a whole lot of special
character sequences. Many of these may be familiar to you from C.

**`\n`**

newline

**`\t`**

tab

**`\e` or `\E`**

escape character

**`\a`**

ring the bell (alarm), usually a euphemism for a hideous beep

**`\b`**

move back one character.

**`\c`**

don't print a newline --- like the `-n` option, but embedded in the
string. This alternative comes from Berkeley UNIX.

**`\f`**

form feed, the phrase for \`advance to next page' from the days when
terminals were called teletypes, maybe more familiar to you as `^L`

**`\r`**

carriage return --- when printed, the annoying `^M`'s you get in DOS
files, but actually rather useful with \``print`', since it will erase
everything to the start of the line. The combination of the `-n` option
and a `\r` at the start of the print string can give the illusion of a
continuously changing status line.

**`\v`**

vertical tab, which I for one have never used (I just tried it now and
it behaved like a newline, only without assuming a carriage return, but
that's up to your terminal).

In fact, you can get any of the 255 characters possible, although your
terminal may not like some or all of the ones above 127, by specifying a
number after the backslash. Normally this consists of three octal
characters, but you can use two hexadecimal characters after `\x`
instead --- so \``\n`', \``\012`' and \``\x0a`' are all newlines. \``\`'
itself escapes any other character, i.e. they appear as themselves even
if they normally wouldn't.

Two notes: first, don't get confused because \``n`' is the fourteenth
letter of the alphabet; printing \``\016`' (fourteen in octal) won't do
you any good. The remedy, after you discover your text is unreadable
(for VT100-like terminals including xterm), is to print \``\017`'.

Secondly, those backslashes can land you in real quoting difficulties.
Normally a backslash on the command line escapes the next character ---
this is a *different* form of escaping to `print`'s --- so

      print \n

doesn't produce a newline, it just prints out an \``n`'. So you need to
quote that. This means

      print \\

passes a single backslash to quote, and

      print \\n

or

      print '\n'

prints a newline (followed by the extra one that's usually there). To
print a real backslash, you would thus need

      print \\\\

Actually, you can get away with the two if there's nothing else after
--- `print` just shrugs its shoulders and outputs what it's been given
--- but that's not a good habit to get into. There are other ways of
doing this: since single quotes quote anything, including backslashes
(they are the only way of making backslashes behave like normal
characters), and since the \``-r`' option makes print treat characters
normally,

      print -r '\'

has the same effect. But you need to remember the two levels of quoting
for backslashes. Quotes aren't special to `print`, so

      print \'

is good enough for printing a quote.

**`echotc`**\
\

There's an oddity called \``echotc`', which takes as its argument
\`termcap' capabilities. This now lives in its own module,
`zsh/termcap`.

Termcap is a now rather old-fashioned way of giving the commands
necessary for performing various standard operations on terminals:
moving the cursor, clearing to the end of the line, turning on standout
mode, and so on. It has now been replaced almost everywhere by
\`terminfo', a completely different way of specifying capabilities, and
by \`curses', a more advanced system for manipulating objects on a
character terminal. This means that the arguments you need to give to
`echotc` can be rather hard to come by; try the `termcap` manual page;
if there are two, it's probably the one in section five which gives the
codes, i.e. \``man 5 zsh`' or \``man -s 5 zsh`' on Solaris. Otherwise
you'll have to search the web. The reason the `zsh` manual doesn't give
a list is that the shell only uses a few well-known sequences, and there
are very many others which will work with `echotc`, because the
sequences are interpreted by a the terminal, not the shell.

This chunk gives you a flavour:

      zmodload -i zsh/termcap
      echotc md
      echo -n bold
      echotc mr
      echo -n reverse
      echotc me
      echo

First we make sure the module is loaded into the shell; on some older
operating systems, this only works if it was compiled in when zsh was
installed. The option `-i` to `zmodload` stops the shell from
complaining if the module was already loaded. This is a sensible way of
ensuring you have the right facilities available in a shell function,
since loading a module makes it available until it is explicitly
unloaded.

You should see \``bold`' in bold characters, and \``reverse`' in bold
reverse video. The \``md`' capability turns on bold mode; \``mr`' turns
on reverse video; \``me`' turns off both modes. A more typical zsh way
of doing this is:

      print -P '%Bbold%Sreverse%b%s'

which should show the same thing, but using prompt escapes --- prompts
are the most common use of special fonts. The \``%S`' is because zsh
calls reverse \`standout' mode, because it does. (On a colour xterm, you
may find \`bold' is interpreted as \`blue'.)

There's a lot more you can do with `echotc` if you really try. The shell
has just acquired a way of printing terminfo sequences, predictably
called `echoti`, although it's only available on systems where zsh needs
terminfo to compile --- this happens when the termcap code is actually a
part of terminfo. The good news about this is that terminfo tends to be
better documented, so you have a good chance of finding out the
capabilities you want from the `terminfo` manual page. The `echoti`
command lives in another predictably named module, `zsh/terminfo`.

### 3.2.2: Other builtins just for speed

There are only a few other builtins which are there just to make things
go faster. Strictly, tests could go into this category, but as I
explained in the last chapter it's useful to have tests in the form

      if [[ $var1 = $var2 ]]; then
        print doing something
      fi

be treated as a special syntax by the shell, in case `$var1` or `$var2`
expands to nothing which would otherwise confuse it. This example
consists of two features described below: the test itself, between the
double square brackets, which is true if the two substituted values are
the same string, and the \``if`' construct which runs the commands in
the middle (here just the `print`) if that test was true.

The builtins \``true`' and \``false`' do nothing at all, except return a
command status zero or one, respectively. They're just used as
placeholders: to run a loop forever --- `while` will also be explained
in more detail later --- you use

      while true; do
        print doing something over and over
      done

since the test always succeeds.

A synonym for \``true`' is \``:`'; it's often used in this form to give
arguments which have side effects but which shouldn't be used ---
something like

      : ${param:=value}

which is a common idiom in all Bourne shell derivatives. In the
parameter expansion, `$param` is given the value `value` if it was empty
before, and left alone otherwise. Since that was the only reason for the
parameter expansion, you use `:` to ignore the argument. Actually, the
shell blithely builds the command line --- the colon, followed by
whatever the value of `$param` is, whether or not the assignment
happened --- then executes the command; it just so happens that \``:`'
takes no notice of the arguments it was given. If you're switching from
ksh, you may expect certain synonyms like this to be aliases, rather
than builtins themselves, but in zsh they are actually builtins; there
are no aliases predefined by the shell. (You can still get rid of them
using \``disable`', as described below.)

### 3.2.3: Builtins which change the shell's state

A more common use for builtins is that they change something inside the
shell, or report information about what's going on in the shell. There
is one vital thing to remember about external commands. It applies, too,
to other cases we'll meet where the shell \`forks', literally splitting
itself into two parts, where the forked-off part behaves just like an
external command. In both of these cases, the command is in a different
*process*, UNIX's basic unit of things that run. (In fact, even Windows
knows about processes nowadays, although they interact a little bit
differently with one another.)

The vital thing is that no change in a separate process started by the
shell affects the shell itself. The most common case of this is the
current directory --- every process has its own current directory. You
can see this by starting a new zsh:

      % pwd               # show the current directory
      ~
      % zsh               # start a new shell, which 
                          # is a separate process
      % cd tmp
      % pwd               # now I'm in a different
                          # directory...
      ~/tmp
      % exit              # leave the new shell...
      % pwd               # now I'm back where I was...
      ~

Hence the `cd` command must be a shell builtin, or this would happen
every time you ran it.

Here's a more useful example. Putting parentheses around a command asks
the shell to start a different process for it. That's useful when you
specifically *don't* want the effects propagating back:

      (cd some-other-dir; run-some-command)

runs the command, but doesn't change the directory the \`real' shell is
in, only its forked-off \`subshell'. Hence,

      % pwd
      ~
      % (cd /; pwd)
      /
      % pwd
      ~

There's a more subtle case:

      cd some-other-dir | print Hello

Remember, the \``|`' (\`pipe') connects the output of the first command
to the input of the next --- though actually no information is passed
that way in this example. In zsh, all but the last portion of the
\`pipeline' thus created is run in different processes. Hence the `cd`
doesn't affect the main shell. I'll refer to it as the \`parent' shell,
which is the standard UNIX language for processes; when you start
another command or fork off a subshell, you are creating \`children'
(without meaning to be morbid, the children usually die first in this
case). Thus, as you would guess,

      print Hello | cd some-other-dir

*does* have the effect of changing the directory. Note that other shells
do this differently; it is always guaranteed to work this way in zsh,
because many people rely on it for setting parameters, but many shells
have the *left* hand of the pipeline being the bit that runs in the
parent shell. If both sides of the pipe symbol are external commands of
some sort, both will of course run in subprocesses.

There are other ways you change the state of the shell, for example by
declaring parameters of a particular type, or by telling it how to
interpret certain commands, or, of course, by changing options. Here are
the most useful, grouped in a vaguely logical fashion.

### 3.2.4: cd and friends

You will not by now be surprised to learn that the \``cd`' command
changes directory. There is a synonym, \``chdir`', which as far as I
know no-one ever uses. (It's the same name as the system call, so if you
had been programming in C or Perl and forgot that you were now using the
shell, you might use \``chdir`'. But that seems a bit far-fetched.)

There are various extra features built into `cd` and `chdir`. First, if
you miss out the directory to which you want to change, you will be
taken to your home directory, although it's not as if \``cd ~`' is all
that hard to type.

Next, the command \``cd -`' is special: it takes you to the last
directory you were in. If you do a sequence of `cd` commands, only the
immediately preceding directory is remembered; they are not stacked up.

Thirdly, there is a shortcut for changing between similarly named
directories. If you type \``cd <old> <new>`', then the shell will look
for the first occurrence of the string \``<old>`' in the current
directory, and try to replace it with \``<new>`'. For example,

      % pwd
      ~/src/zsh-3.0.8/Src
      % cd 0.8 1.9
      ~/src/zsh-3.1.9/Src

The `cd` command actually reported the new directory, as it usually does
if it's not entirely obvious where it's taken you.

Note that only the *first* match of `<old>` is taken. It's an easy
mistake to think you can change from
`/home/export1/pws/mydir1/something` to
`/home/export1/pws/mydir2/something` with \``cd 1 2`', but that first
\``1`' messes it up. Arguably the shell could be smarter here. Of
course, \``cd r1 r2`' will work in this case.

`cd`'s friend \``pwd`' (print working directory) tells you what the
current working directory is; this information is also available in the
shell parameter `$PWD`, which is special and automatically updated when
the directory changes. Later, when you know all about expansion, you
will find that you can do tricks with this to refer to other
directories. For example, `${PWD/old/new}` uses the parameter
substitution mechanism to refer to a different directory with `old`
replaced by `new` --- and this time `old` can be a pattern, i.e.
something with wildcard matches in it. So if you are in the
`zsh-3.0.8/Src` directory as above and want to copy a file from the
`zsh-3.1.9/Src` directory, you have a shorthand:

      cp ${PWD/0.8/1.9}/myfile.c .

**Symbolic links**\
\

Zsh tries to track directories across symbolic links. If you're not
familiar with these, you can think of them as a filename which behaves
like a pointer to another file (a little like Windows' shortcuts, though
UNIX has had them for much longer and they work better). You create them
like this (`ln` is not a builtin command, but its use to make symbolic
links is very standard these days):

      ln -s existing-file-name name-of-link

for example

      ln -s /usr/bin/ln ln

creates a file called `ln` in the current directory which does nothing
but point to the file `/usr/bin/ln`. Symbolic links are very good at
behaving as much like the original file as you usually want; for
example, you can run the `ln` link you've just created as if it were
`/usr/bin/ln`. They show up differently in a long file listing with
\``ls -l`', the last column showing the file they point to.

You can make them point to any sort of file at all, including
directories, and that is why they are mentioned here. Suppose you create
a symbolic link from your home directory to the root directory and
change into it:

      ln -s / ~/mylink
      cd ~/mylink

If you don't know it's a link, you expect to be able to change to the
parent directory by doing \``cd ..`'. However, the operating system ---
which just has one set of directories starting from `/` and going down,
and ignores symbolic links after it has followed them, they really are
just pointers --- thinks you are in the root directory `/`. This can be
confusing. Hence zsh tries to keep track of where *you* probably think
you are, rather than where the system does. If you type \``pwd`', you
will see \``/home/you/mylink`' (wherever your home directory is), not
\``/`'; if you type \``cd ..`', you will find yourself back in your home
directory.

You can turn all this second-guessing off by setting the option
`CHASE_LINKS`; then \``cd ~/mydir; pwd`' will show you to be in `/`,
where changing to the parent directory has no effect; the parent of the
root directory is the root directory, except on certain slightly
psychedelic networked file systems. This does have advantages: for
example, \``cd ~/mydir; ls ..`' always lists the root directory, not
your home directory, regardless of the option setting, because `ls`
doesn't know about the links you followed, only zsh does, and it treats
the `..` as referring to the root directory. Having `CHASE_LINKS` set
allows \``pwd`' to warn you about where the system thinks you are.

An aside for non-UNIX-experts (over 99.9% of the population of the world
at the last count): I said \`symbolic links' instead of just \`links'
because there are others called \`hard links'. This is what \``ln`'
creates if you don't use the `-s` option. A hard link is not so much a
pointer to a file as an alternative name for a file. If you do

      ln myfile othername
      ls -l

where `myfile` already exists you can't tell which of `myfile` and
`othername` is the original --- and in fact the system doesn't care. You
can remove either, and the other will be perfectly happy as the name for
the file. This is pretty much how renaming files works, except that
creating the hard link is done for you in that case. Hard links have
limitations --- you can't link to directories, or to a file on another
disk partition (and if you don't know what a disk partition is, you'll
see what a limitation that can be). Furthermore, you usually want to
know which is the original and which is the link --- so for most users,
creating symbolic links is more useful. The only drawback is that
following the pointers is a tiny bit slower; if you think you can notice
the difference, you definitely ought to slow down a bit.

The target of a symbolic link, unlike a hard link, doesn't actually have
to exist and no checking is performed until you try to use the link. The
best thing to do is to run \``ls -lL`' when you create the link; the
`-L` part tells `ls` to follow links, and if it worked you should see
that your link is shown as having exactly the same characteristics as
the file it points to. If it is still shown as a link, there was no such
file.

While I'm at it, I should point out one slight oddity with symbolic
links: the name of the file linked to (the first name), if it is not an
absolute path (beginning with `/` after any `~` expansion), is treated
relative to the directory where the link is created --- not the current
directory when you run `ln`. Here:

      ln -s ../mydir ~/links/otherdir

the link `otherdir` will refer to `mydir` in *its own* parent directory,
i.e. `~/links` --- not, as you might think, the parent of the directory
where you were when you ran the command. What makes it worse is that the
second word, if is not an absolute path, *is* interpreted relative to
the directory where you ran the command.

**\$cdpath and AUTO\_CD**\
\

We're nowhere near the end of the magic you can do with directories yet
(and, in fact, I haven't even got to the zsh-specific parts). The next
trick is `$cdpath` and `$CDPATH`. They look a lot like `$path` and
`$PATH` which you met in the last chapter, and I mentioned them briefly
back in the last chapter in that context: `$cdpath` is an array of
directories, while `$CDPATH` is colon-separated list behaving otherwise
like a scalar variable. They give a list of directories whose
subdirectories you may want to change into. If you use a normal cd
command (i.e. in the form \``cd `*dirname*', and *dirname* does not
begin with a `/` or `~`, the shell will look through the directories in
`$cdpath` to find one which contains the subdirectory *dirname*. If
`$cdpath` isn't set, as you'd guess, it just uses the current directory.

Note that `$cdpath` is always searched in order, and you can put a `.`
in it to represent the current directory. If you do, the current
directory will always be searched *at that point*, not necessarily
first, which may not be what you expect. For example, let's set up some
directories:

      mkdir ~/crick ~/crick/dna
      mkdir ~/watson ~/watson/dna
      cdpath=(~/crick .)
      cd ~/watson
      cd dna

So I've moved to the directory `~/watson`, which contains the
subdirectory `dna`, and done \``cd dna`'. But because of `$cdpath`, the
shell will look first in `~/crick`, and find the `dna` there, and take
you to that copy of the self-reproducing directory, not the one in
`~/watson`. Most people have `.` at the start of their `cdpath` for that
reason. However, at least `cd` warns you --- if you tried it, you will
see that it prints the name of the directory it's picked in cases like
this.

In fact, if you don't have `.` in your directory at all, the shell will
always look there first; there's no way of making `cd` never change to a
subdirectory of the current one, short of turning `cd` into a function.
Some shells don't do this; they use the directories in `$cdpath`, and
only those.

There's yet another shorthand, this time specific to zsh: the option
`AUTO_CD` which I mentioned in the last chapter. That way a command
without any arguments which is really a directory will take you to that
directory. Normally that's perfect --- you would just get a \`command
not found' message otherwise, and you might as well make use of the
option. Just occasionally, however, the name of a directory clashes with
the name of a command, builtin or external, or a shell function, and
then there can be some confusion: zsh will always pick the command as
long as it knows about it, but there are cases where it doesn't, as I
described above.

What I didn't say in the last chapter is that `AUTO_CD` respects
`$cdpath`; in fact, it really is implemented so that \`*dirname*' on its
own behaves as much like \``cd` *dirname*' as is possible without tying
the shell's insides into knots.

**The directory stack**\
\

One very useful facility that zsh inherited from the C-shell family
(traditional Korn shell doesn't have it) is the directory stack. This is
a list of directories you have recently been in. If you use the command
\``pushd`' instead of \``cd`', e.g. \``pushd` *dirname*', then the
directory you are in is saved in this list, and you are taken to
*dirname*, using `$CDPATH` just as `cd` does. Then when you type
\``popd`', you are taken back to where you were. The list can be as long
as you like; you can `pushd` any number of directories, and each `popd`
will take you back through the list (this is how a \`stack', or more
precisely a \`last-in-first-out' stack usually operates in computer
jargon, hence the name \`directory stack').

You can see the list --- which always starts with the current directory
--- with the `dirs` command. So, for example:

      cd ~
      pushd ~/src
      pushd ~/zsh
      dirs

displays

      ~/zsh ~/src ~

and the next `popd` will take you back to `~/src`. If you do it, you
will see that `pushd` reports the list given by `dirs` automatically as
it goes along; you can turn this off with the option `PUSHD_SILENT`,
when you will have to rely on typing `dirs` explicitly.

In fact, a lot of the use of this comes not from using simple `pushd`
and `popd` combinations, but from two other features. First, \``pushd`'
on its own swaps the top two directories on the stack. Second, `pushd`
with a numeric argument preceded by a \``+`' or \``-`' can take you to
one of the other directories in the list. The command \``dirs -v`' tells
you the numbers you need; `0` is the current directory. So if you get,

      0       ~/zsh
      1       ~/src
      2       ~

then \``pushd +2`' takes you to `~`. (A little suspension of disbelief
that I didn't just use `AUTO_CD` and type \``..`' is required here.) If
you use a `-`, it counts from the other end of the list; `-0` (with
apologies to the numerate) is the last item, i.e. the same as `~` in
this case. Some people are used to having the \``-`' and \``+`'
arguments behave the other way around; the option `PUSHD_MINUS` exists
for this.

Apart from `PUSHD_SILENT` and `PUSHD_MINUS`, there are a few other
relevant options. Setting `PUSHD_IGNORE_DUPS` means that if you `pushd`
to a directory which is already somewhere in the list, the duplicate
entry will be silently removed. This is useful for most human operations
--- however, if you are using `pushd` in a function or script to
remember previous directories for a future matching `popd`, this can be
dangerous and you probably want to turn it off locally inside the
function.

`AUTO_PUSHD` means that any directory-changing command, including an
auto-cd, is treated as a `pushd` command with the target directory as
argument. Using this can make the directory stack get very long, and
there is a parameter `$DIRSTACKSIZE` which you can set to specify a
maximum length. The oldest entry (the highest number in the \``dirs -v`'
listing) is automatically removed when this length is exceeded. There is
no limit unless this is explicitly set.

The final `pushd` option is `PUSHD_TO_HOME`. This makes `pushd` on its
own behave like `cd` on its own in that it takes you to your home
directory, instead of swapping the top two directories. Normally a
series of \``pushd`' commands works pretty much like a series of
\``cd -`' commands, always taking you the directory you were in before,
with the obvious difference that \``cd -`' doesn't consult the directory
stack, it just remembers the previous directory automatically, and hence
it can confuse `pushd` if you just use \``cd -`' instead.

There's one remaining subtlety with `pushd`, and that is what happens to
the rest of the list when you bring a particular directory to the front
with something like \``pushd +2`'. Normally the list is simply cycled,
so the directories which were +3, and +4 are now right behind the new
head of the list, while the two directories which were ahead of it get
moved to the end. If the list before was:

      dir1  dir2  dir3  dir4

then after `pushd +2` you get

      dir3  dir4  dir1 dir2

That behaviour changed during the lifetime of zsh, and some of us
preferred the old behaviour, where that one directory was yanked to the
front and the rest just closed the gap:

      # Old behaviour
      dir3  dir1  dir2  dir4

so that after a while you get a \`greatest hits' group at the front of
the list. If you like this behaviour too (I feel as if I'd need to have
written papers on group theory to like the new behaviour) there is a
function `pushd` supplied with the source code, although it's short
enough to repeat here --- this is in the form for autoloading in the zsh
fashion:

      # pushd function to emulate the old zsh behaviour.
      # With this, pushd +/-n lifts the selected element
      # to the top of the stack instead of cycling
      # the stack.

      emulate -R zsh
      setopt localoptions

      if [[ ARGC -eq 1 && "$1" == [+-]<-> ]] then
              setopt pushdignoredups
              builtin pushd ~$1
      else
              builtin pushd "$@"
      fi

The \``&&`' is a logical \`and', requiring both tests to be true. The
tests are that there is exactly one argument to the function, and that
it has the form of a \``+`' or a \``-`' followed by any number (\``<->`'
is a special zsh pattern to match any number, an extension of forms like
\``<1-100>`' which matches any number in the range 1 to 100 inclusive).

**Referring to other directories**\
\

Zsh has two ways of allowing you to refer to particular directories.
They have in common that they begin with a `~` (in very old versions of
zsh, the second form actually used an \``=`', but the current way is
much more logical).

You will certainly be aware, because I've made a lot of use of it, that
a \``~`' on its own or followed by a `/` refers to your own home
directory. An extension of this --- again from the C-shell, although the
Korn shell has it too in this case --- is that `~name` can refer to the
home directory of any user on the system. So if your user name is `pws`,
then `~` and `~pws` are the same directory.

Zsh has an extension to this; you can actually name your own
directories. This was described in [chapter 2](zshguide02.html#init), à
propos of prompts, since that is the major use:

      host% PS1='%~? '
      ~? cd zsh/Src
      ~/zsh/Src? zsrc=$PWD
      ~/zsh/Src? echo ~zsrc
      /home/pws/zsh/Src
      ~zsrc?

Consult that chapter for the ways of forcing a parameter to be
recognised as a named directory.

There's a slightly more sophisticated way of doing this directly:

      hash -d zsrc=~/zsh/Src

makes `~zsrc` appear in prompts as before, and in this case there is no
parameter `$zsrc`. This is the purist's way (although very few zsh users
are purists). You can guess what \``unhash -d zsrc`' does; this works
with directories named via parameters, too, but leaves the parameter
itself alone.

It's possible to have a named directory with the same name as a user. In
that case \``~name`' refers to the directory you named explicitly, and
there is no easy way of getting `name`'s home directory without removing
the name you defined.

If you're using named directories with one of the `cd`-like commands or
`AUTO_CD`, you can set the option `CDABLEVARS` which allows you to omit
the leading `~`; \``cd zsrc`' with this option would take you to
`~zsrc`. The name is a historical artifact and now a misnomer; it really
is named directories, not parameters (i.e. variables), which are used.

The second way of referring to directories with `~`'s is to use numbers
instead of names: the numbers refer to directories in the directory
stack. So if `dirs -v` gives you

      0       ~zsf
      1       ~src

then `~+1` and `~-0` (not very mathematical, but quite logical if you
think about it) refer to `~src`. In this case, unlike pushd arguments,
you can omit the `+` and use `~1`. The option `PUSHD_MINUS` is
respected. You'll see this was used in the `pushd` function above: the
trick was that `~+3`, for example, refers to the same element as
`pushd +3`, hence `pushd ~+3` pushed that directory onto the front of
the list. However, we set `PUSHD_IGNORE_DUPS`, so that the value in the
old position was removed as well, giving us the effect we wanted of
simply yanking the directory to the front with no trick cycling.

### 3.2.5: Command control and information commands

Various builtins exist which control how you access commands, and which
show you information about the commands which can be run.

The first two are strictly speaking \`precommand modifiers' rather than
commands: that means that they go before a command line and modify its
behaviour, rather than being commands in their own right. If you put
\``command`' in front of a command line, the command word (the next one
along) will be taken as the name of an external command, however it
would normally be interpreted; likewise, if you put \``builtin`' in
front, the shell will try to run the command as a builtin command.
Normally, shell functions take precedence over builtins which take
precedence over external commands. So, for example, if your printer
control system has the command \``enable`' (as many System V versions
do), which clashes with a builtin I am about to talk about, you can run
\``command enable lp`' to enable a printer; otherwise, the builtin
enable would have been run. Likewise, if you have defined `cd` to be a
function, but this time want to call the normal builtin `cd`, you can
say \``builtin cd mydir`'.

A common use for `command` is inside a shell function of the same name.
Sometimes you want to enhance an ordinary command by sticking some extra
stuff around it, then calling that command, so you write a shell
function of the same name. To call the command itself inside the shell
function, you use \``command`'. The following works, although it's
obviously not all that useful as it stands:

      ls() {
        command ls "$[@]"
      }

so when you run \``ls`', it calls the function, which calls the real
`ls` command, passing on the arguments you gave it.

You can gain longer lasting control over the commands which the shell
will run with the \``disable`' and \``enable`' commands. The first
normally takes builtin arguments; each such builtin will not be
recognised by the shell until you give an \``enable`' command for it. So
if you want to be able to run the external `enable` command and don't
particularly care about the builtin version, \``disable enable`' (sorry
if that's confusing) will do the trick. Ha, you're thinking, you can't
run \``enable enable`'. That's correct: some time in the dim and distant
past, `builtin enable enable`' would have worked, but currently it
doesn't; this may change, if I remember to change it. You can list all
disabled builtins with just \``disable`' on its own --- most of the
builtins that do this sort of manipulation work like that.

You can manipulate other sets of commands with `disable` and `enable` by
giving different options: aliases with the option `-a`, functions with
`-f`, and reserved words with `-r`. The first two you probably know
about, and I'll come to them anyway, but \`reserved words' need
describing. They are essentially builtin commands which have some
special syntactic meaning to the shell, including some symbols such as
\``{`' and \``[[`'. They take precedence over everything else except
aliases --- in fact, since they're syntactically special, the shell
needs to know very early on that it has found a reserved word, it's no
use just waiting until it tries to execute a command. For example, if
the shell finds \``[[`' it needs to know that everything until \``]]`'
must be treated as a test rather than as ordinary command arguments.
Consequently, you wouldn't often want to disable a reserved word, since
the shell wouldn't work properly. The most obvious reason why you might
would be for compatibility with some other shell which didn't have one.
You can get a complete list with:

      whence -wm '*' | grep reserved

which I'll explain below, since I'm coming to \``whence`'.

Furthermore, I tend to find that if I want to get rid of aliases or
functions I use the commands \``unalias`' and \``unfunction`' to get rid
of them permanently, since I always have the original definitions stored
somewhere, so these two options may not be that useful either. Disabling
builtins is definitely the most useful of the four possibilities for
`disable`.

External commands have to be manipulated differently. The types given
above are handled internally by the shell, so all it needs to do is
remember what code to call. With external commands, the issue instead is
how to find them. I mentioned `rehash` above, but didn't tell you that
the `hash` command, which you've already seen with the `-d` option, can
be used to tell the shell how to find an external command:

      hash foo=/path/to/foo

makes `foo` execute the command using the path shown (which doesn't even
have to end in \``foo`'). This is rather like an alias --- most people
would probably do this with an alias, in fact --- although a little
faster, though you're unlikely to notice the difference. You can remove
this with `unhash`. One gotcha here is that if the path is rehashed,
either by calling `rehash` or when you alter `$path`, the entire hash
table is emptied, including anything you put in in this way; so it's not
particularly useful.

In the midst of all this, it's useful to be able to find out what the
shell thinks a particular command name does. The command \``whence`'
tells you this; it also exists, with slightly different options, under
the names `where`, `which` and `type`, largely to provide compatibility
with other shells. I'll just stick to `whence`.

Its standard output isn't actually sparklingly interesting. If it's a
command somehow known to the shell internally, it gets echoed back, with
the alias expanded if it was an alias; if it's an external command it's
printed with the full path, showing where it came from; and if it's not
known the command returns status 1 and prints nothing.

You can make it more useful with the `-v` or `-c` options, which are
more verbose; the first prints out an information message, while the
second prints out the definitions of any functions it was asked about
(this is also the effect of using \``which`' instead of \``whence`). A
very useful option is `-m`, which takes any arguments as patterns using
the usual zsh pattern format, in other words the same one used for
matching files. Thus

      whence -vm "*"

prints out every command the shell knows about, together with what it
thinks of it.

Note the quotes around the \``*`' --- you have to remember these
anywhere where the pattern is not to be used to generate filenames on
the command line, but instead needs to be passed to the command to be
interpreted. If this seems a rather subtle distinction, think about what
would happen if you ran

      # Oops.  Better not try this at home.
      # (Even better, don't do it at work either.)
      whence -vm *

in a directory with the files \``foo`' and (guess what) \``bar`' in it.
The shell hasn't decided what command it's going to run when it first
looks at the command line; it just sees the \``*`' and expands the line
to

      whence -vm foo bar

which isn't what you meant.

There are a couple of other tricks worth mentioning: `-p` makes the
shell search your path for them, even if the name is matched as
something else (say, a shell function). So if you have `ls` defined as a
function,

      which -p ls

will still tell what \``command ls`' would find. Also, the option `-a`
searches for all commands; in the same example, this would show you both
the `ls` command and the `ls` function, whereas `whence` would normally
only show the function because that's the one that would be run. The
`-a` option also shows if it finds more than one external command in
your path.

Finally, the option `-w` is useful because it identifies the type of a
command with a single word: `alias`, `builtin`, `command`, `function`,
`hashed`, `reserved` or `none`. Most of those are obvious, with
`command` being an ordinary external command; `hashed` is an external
command which has been explicitly given a path with the `hash` builtin,
and `none` means it wasn't recognised as a command at all. Now you know
how we extracted the reserved words above.

A close relative of `whence` is `functions`, which applies, of course,
to shell functions; it usually lists the definitions of all functions
given as arguments, but its relatives (of which `autoload` is one)
perform various other tricks, to be described in the section on shell
functions below. Be careful with `function`, without the \`s', which is
completely different and not like `command` or `builtin` --- it is
actually a keyword used to *define* a function.

### 3.2.6: Parameter control

There are various builtins for controlling the shells parameters. You
already know how to set and use parameters, but it's a good deal more
complicated than that when you look at the details.

**Local parameters**\
\

The principal command for manipulating the behaviour of parameters is
\``typeset`'. Its easiest usage is to declare a parameter; you just give
it a list of parameter names, which are created as scalar parameters.
You can create parameters just by assigning to them, but the major point
of \``typeset`' is that if a parameter is created that way inside a
function, the parameter is restored to its original value, or removed if
it didn't previously exist, at the end of the function --- in other
words, it has \`local scope' like the variables which you declare in
most ordinary programming languages. In fact, to use the jargon it has
\`dynamical' rather than \`syntactic' scope, which means that the same
parameter is visible in any function called within the current one; this
is different from, say, C or FORTRAN where any function or subroutine
called wouldn't see any variable declared in the parent function.

The following makes this more concrete.

      var='Original value'
      subfn() {
        print $var
      }
      fn() {
        print $var
        typeset var='Value in function'
        print $var
        subfn
      }
      fn
      print $var

This chunk of code prints out

      Original value
      Value in function
      Value in function
      Original value

The first three chunks of the code just define the parameter `$var`, and
two functions, `subfn` and `fn`. Then we call `fn`. The first thing this
does is print out `$var`, which gives \``Original value`' since we
haven't changed the original definition. However, the `typeset` next
does that; as you see, we can assign to the parameter during the
typeset. Thus when we print `$var` out again, we get
\``Value in function`'. Then `subfn` is called, which prints out the
same value as in `fn`, because we haven't changed it --- this is where C
or FORTRAN would differ, and wouldn't recognise the variable because it
hadn't been declared in that function. Finally, `fn` exits and the
original value is restored, and is printed out by the final \``print`'.

Note the value changes twice: first at the `typeset`, then again at the
end of `fn`. The value of `$var` at any point will be one of those two
values.

Although you can do assignments in a `typeset` statement, you can't
assign to arrays (I already said this in the last chapter):

      typeset var=(Doesn\'t work\!)

because the syntax with the parentheses is special; it only works when
the line consists of nothing but assignments. However, the shell doesn't
complain if you try to assign an array to a scalar, or vice versa; it
just silently converts the type:

      typeset var='scalar value'
      var=(array value)

I put in the assignment in the typeset statement to rub the point in
that it creates scalars, but actually the usual way of setting up an
array in a function is

      typeset var
      var=()

which creates an empty scalar, then converts that to an empty array.
Recent versions of the shell have \``typeset -a var`' to do that in one
go --- but you *still* can't assign to it in the same statement.

There are other catches associated with the fact that `typeset` and its
relatives are just ordinary commands with ordinary sets of arguments.
Consider this:

      % typeset var=`echo two words`
      % print $var
      two

What has happened to the \``words`'? The answer is that backquote
substitution, to be discussed below, splits words when not quoted. So
the `typeset` statement is equivalent to

      % typeset var=two words

There are two ways to get round this; first, use an ordinary assignment:

      % typeset var
      % var=`echo two words`

which can tell a scalar assignment, and hence knows not to split words,
or quote the backquotes,

      % typeset var="`echo two words`"

There are three important types we haven't talked about; both of these
can only be created with `typeset` or one of the similar builtins I'll
list in a moment. They are integer types, floating point types, and
associative array types.

**Numeric parameters**\
\

Integers are created with \``typeset -i`', or \``integer`' which is
another way of saying the same thing. They are used for arithmetic,
which the shell can do as follows:

      integer i
      (( i = 3 * 2 + 1 ))

The double parentheses surround a complete arithmetic expression: it
behaves as if it's quoted. The expression inside can be pretty much
anything you might be used to from arithmetic in other programming
languages. One important point to note is that parameters don't need to
have the `$` in front, even when their value is being taken:

      integer i j=12
      (( i = 3 * ( j + 4 ) ** 2 ))

Here, `j` will be replaced by 12 and `$i` gets the value 768 (sixteen
squared times three). One thing you might not recognise is the `**`,
which is the \`to the power of' operator which occurs in FORTRAN and
Perl. Note that it's fine to have parentheses inside the double
parentheses --- indeed, you can even do

      (( i = (3 * ( j + 4 )) ** 2 ))

and the shell won't get confused because it knows that any parentheses
inside must be in balanced pairs (until you deliberately confuse it with
your buggy code).

You would normally use \``print $i`' to see what value had been given to
`$i`, of course, and as you would expect it gets printed out as a
decimal number. However, `typeset` allows you to specify another base
for printing out. If you do

      typeset -i 16 i
      print $i

after the last calculation, you should see `16#900`, which means 900 in
base 16 (hexadecimal). That's the only effect the option \``-i 16`' has
on `$i` --- you can assign to it and use it in arithmetical expressions
just as normal, but when you print it out it appears in this form. You
can use this base notation for inputting numbers, too:

      (( i = 16#ff * 2#10 ))

which means 255 (`ff` in hexadecimal) times 2 (`10` in binary). The
shell understands C notation too, so \``16#ff`' could have been
expressed \``0xff`'.

Floating point variables are very similar. You can declare them with
\``typeset -F`' or \``typeset -E`'. The only difference between the two
is, again, on output; `-F` uses a fixed point notation, while `-E` uses
scientific (mnemonic: exponential) notation. The builtin \``float`' is
equivalent to \``typeset -E`' (because Korn shell does it, that's why).
Floating point expressions also work the way you are probably used to:

      typeset -E e
      typeset -F f
      (( e = 32/3, f = 32.0/3.0 ))
      print $e $f

prints

      1.000000000e+01 10.6666666667

Various points: the \``,`' can separate different expressions, just like
in C, so the `e` and `f` assignments are performed separately. The `e`
assignment was actually an integer division, because neither 32 nor 3 is
a floating point number, which must contain a dot. That means an integer
division was done, producing 10, which was then converted to a floating
point number only at the end. Again, this is just how grown-up languages
work, so it's no use cursing. The `f` assignment was a full floating
point performance. Floating point parameters weren't available before
version `3.1.7`.

Although this is really a matter for a later chapter, there is a library
of floating point functions you can load (actually it's just a way of
linking in the system mathematical library). The usual incantation is
\``zmodload zsh/mathfunc`'; you may not have \`dynamic loading' of
libraries on your system, which may mean that doesn't work. If it does,
you can do things like

      (( pi = 4.0 * atan(1.0) ))

Broadly, all the functions which appear in most system mathematical
libraries (see the manual page for `math`) are available in zsh.

Like all other parameters created with `typeset` or one of its cousins,
integer and floating point parameters are local to functions. You may
wonder how to create a global parameter (i.e. one which is valid outside
as well as inside the function) which has an integer or floating point
value. There's a recent addition to the shell (in version 3.1.6) which
allows this: use the flag `-g` to typeset along with any others. For
example,

      fn() {
        typeset -Fg f
        (( f = 42.75 ))
      }
      fn
      print $f

If you try it, you will see the value of `$f` has survived beyond the
function. The `g` stands for global, obviously, although it's not quite
that simple:

      fn() {
        typeset -Fg f
      }
      outerfn() {
        typeset f='scalar value'
        fn
        print $f
      }
      outerfn

The function `outerfn` creates a local scalar value for `f`; that's what
`fn` sees. So it was not really operating on a \`global' value, it just
didn't create a new one for the scope of `fn`. The error message comes
because it tried to preserve the value of `$f` while changing its type,
and the value wasn't a proper floating point expression. The error
message,

      fn: bad math expression: operator expected at `value'

comes about because assigning to numeric parameters always does an
arithmetic evaluation. Operating on \``scalar value`' it found
\``scalar`' and assumed this was a parameter, then looked for an
operator like \``+`' to come next; instead it found \``value`'. If you
want to experiment, change the string to \``scalar + value`' and set
\``value=42`', or whatever, then try again. This is a little confusing
(which is a roundabout way of saying it confused me), but consistent
with how zsh usually treats parameters.

Actually, to a certain extent you don't need to use the integer and
floating point parameters. Any time zsh needs a numeric expression it
will force a scalar to the right value, and any time it produces a
numeric expression and assigns it to a scalar, it will convert the
result to a string. So

      typeset num=3            # This is the *string* `3'.
      (( num = num + 1 ))      # But this works anyway
                               # ($num is still a string).

This can be useful if you have a parameter which is sometimes a number,
sometimes a string, since zsh does all the conversion work for you.
However, it can also be confusing if you always want a number, because
zsh can't guess that for you; plus it's a little more efficient not to
have to convert back and forth; plus you lose accuracy when you do,
because if the number is stored as a string rather than in the internal
numeric representation, what you say is what you get (although zsh tends
to give you quite a lot of decimal places when converting implicitly to
strings). Anyway, I'd recommend that if you know a parameter has to be
an integer or floating point value you should declare it as such.

There is a builtin called `let` to handle mathematical expressions, but
since

      let "num = num + 1"

is equivalent to

      (( num = num + 1 ))

and the second form is easier and more memorable, you probably won't
need to use it. If you do, remember that (unlike BASIC) each
mathematical expression should appear as one argument in quotes.

**Associative arrays**\
\

The one remaining major type of parameter is the associative array; if
you use Perl, you may call it a \`hash', but we tend not to since that's
really a description of how it's implemented rather than what it does.
(All right, what it does is hash things. Now shut up.)

These have to be declared by a typeset statement --- there's no getting
round it. There are some quite eclectic builtins that produce a
filled-in associative array for you, but the only way to tell zsh you
want your very own associative array is

      typeset -A assoc

to create `$assoc`. As to what it does, that's best shown by example:

      typeset -A assoc
      assoc=(one eins two zwei three drei)
      print ${assoc[two]}

which prints \``zwei`'. So it works a bit like an ordinary array, but
the numeric *subscript* of an ordinary array which would have appeared
inside the square bracket is replaced by the string *key*, in this case
`two`. The array assignment was a bit deceptive; the \`values' were
actually pairs, with \``one`' being the key for the value \``eins`', and
so on. The shell will complain if there are an odd number of elements in
such a list. This may also be familiar from Perl. You can assign values
one at a time:

      assoc[four]=vier

and also unset one key/value pair:

      unset 'assoc[one]'

where the quotes stop the square brackets from being interpreted as a
pattern on the command line.

Expansion has been held over, but you might like to know about the ways
of getting back what you put in. If you do

      print $assoc

you just see the values --- that's exactly the same as with an ordinary
array, where the subscripts 1, 2, 3, etc. aren't shown. Note they are in
random order --- that's the other main difference from ordinary arrays;
associative arrays have no notion of an order unless you explicitly sort
them.

But here the keys may be just as interesting. So there is:

      print ${(k)assoc}
      print ${(kv)assoc}

giving (if you've followed through all the commands above):

      four two three
      four vier two zwei three drei

which print out the keys instead of the values, and the key and value
pairs much as you entered them. You can see that, although the order of
the pairs isn't obvious, it's the same each time. From this example you
can work out how to copy an associative array into another one:

      typeset -A newass
      newass=(${(kv)assoc})

where the \``(kv)`' is important --- as is the `typeset` just before the
assignment, otherwise `$newass` would be a badass ordinary array. You
can also prove that `${(v)assoc}` does what you would probably expect.
There are lots of other tricks, but they are mostly associated with
clever types of parameter expansion, to be described in [chapter
5](zshguide05.html#subst).

**Other typeset and type tricks**\
\

There are variants of `typeset`, some mentioned sporadically above.
There is nothing you can do with any of them that you can't do with
`typeset` --- that wasn't always the case; we've tried to improve the
orthogonality of the options. They differ in the options which are set
by default, and the additional options which are allowed. Here's a list:
`declare`, `export`, `float`, `integer`, `local`, `readonly`. I won't
confuse you by describing all in detail; see the manual.

If there is an odd one out, it's `export`, which not only marks a
parameter for export but has the `-g` flag turned on by default, so that
that parameter is not local to the function; in other words, it's
equivalent to `typeset -gx`. However, one holdover from the days when
the options weren't quite so logical is that `typeset -x` behaves like
`export`, in other words the `-g` flag is turned on by default. You can
fix this by unsetting the option `GLOBAL_EXPORT` --- the option only
exists for compatibility; logically it should always be unset. This is
partly because in the old days you couldn't export local parameters, so
`typeset -x` either had to turn on `-g` or turn off `-x`; that was fixed
for the 3.1.9 release, and (for example) \``local -x`' creates a local
parameter which is exported to the environment; both the parameter
itself, and the value in the environment, will be restored when the
function exits. The builtin `local` is essentially a form of `typeset`
which renounces the `-g` flag and all its works.

Another old restriction which has gone is that you couldn't make special
parameters, in particular `$PATH`, local to a function; you just
modified the original parameter. Now if you say \``typeset PATH`',
things happen the way you probably expect, with `$PATH` having its usual
effect, and being restored to its old value when the function exits.
Since `$PATH` is still special, though, you should make sure you assign
something to it in the function before calling external commands, else
it will be empty and no commands will be found. It's possible that you
specifically don't want some parameter you make local to have the
special property; 3.1.7 and after allow the typeset flag `-h` to hide
the specialness for that parameter, so in \``typeset -h PATH`', `PATH`
would be an ordinary variable for the duration of the enclosing
function. Internally, the same value as was previously set would
continue to be used for finding commands, but it wouldn't be exported.

The second main use of `typeset` is to set attributes for the
parameters. In this case it can operate on an existing parameter, as
well as creating a new one. For example,

      typeset -r msg='This is an important message.'

sets the readonly flag (-r) for the parameter `msg`. If the parameter
didn't exist, it would be created with the usual scoping rules; but if
it did exist at the current level of scoping, it would be made readonly
with the value assigned to it, meaning you can't set that particular
copy of the parameter. For obvious reasons, it's normal to assign a
value to a readonly parameter when you first declare it. Here's a
reality check on how this affects scoping:

       msg='This is an ordinary parameter'
       fn() {
         typeset msg='This is a local ordinary parameter'
         print $msg
         typeset -r msg='This is a local readonly parameter'
         print $msg
         msg='Watch me cause an error.'
       }
       fn
       print $msg
       msg='This version of the parameter'\ 
       ' can still be overwritten'
       print $msg

outputs

      This is a local ordinary parameter
      This is a local readonly parameter
      fn:5: read-only variable: msg
      This is an ordinary parameter
      This version of the parameter can still be overwritten

Unfortunately there was a bug with this code until recently --- thirty
seconds ago, actually: the second `typeset` in `fn` incorrectly added
the readonly flag to the existing `msg` *before* attempting to set the
new value, which was wrong and inconsistent with what happens if you
create a new local parameter. Maybe it's reassuring that the shell can
get confused about local parameters, too. (I don't find it reassuring in
the slightest, since `typeset` is one of the parts of the code where I
tend to fix the bugs, but maybe you do.)

Anyway, when the bug is fixed, you should get the output shown, because
the first typeset created a local variable which the second typeset made
readonly, so that the final assignment caused an error. Then the `$msg`
in the function went out of scope, and the ordinary parameter, with no
readonly restriction, was visible again.

I mentioned another special typeset option in the previous chapter:

      typeset -T TEXINPUTS texinputs

to tie together the scalar `$TEXINPUTS` and the array `$texinputs` in
the same way that `$PATH` and `$path` work. This is a one-off; it's the
only time `typeset` takes exactly two parameter names on the command
line. All other uses of typeset take a list of parameters to which any
flags given are applied. See the manual for the remaining flags,
although most of the more interesting ones have been discussed.

The other thing you need to know about flags is that you use them with a
\``+`' sign to turn off the corresponding attribute. So

      typeset +r msg

allows you to set `$msg` again. From version `4.1`, you won't be able to
turn off the readonly attribute for a special parameter; that's because
there's too much scope for confusion, including attempting to set
constant strings in the code. For example, \``$ZSH_VERSION`' always
prints a fixed string; attempting to change that is futile.

The final use of typeset is to list parameters. If you type \``typeset`'
on its own, you get a complete list of parameters and their values. From
3.1.7, you can turn on the flag `-H` for a parameter, which means to
hide its value while you're doing this. This can be useful for some of
the more enormous parameters, particularly special parameters which I'll
talk about in the section in [chapter 7](zshguide07.html#ragbag) on
modules, which tend to swamp the display `typeset` produces.

You can also list parameters of a particular type, by listing the flags
you want to know about. For example,

      typeset -r

lists all readonly parameters. You might expect \``typeset +r`' to list
parameters which *don't* have that attribute, but actually it lists the
same parameters but without showing their value. \``typeset +`' lists
all parameters in this way.

Another good way of finding out about parameters is to use the special
expansion \``${(t)`*param*`}`', for example

      print ${(t)PATH}

prints \``scalar-export-special`': `$PATH` is a scalar parameter, with
the `-x` flag set, and has a special meaning to the shell. Actually,
\``special`' means something a bit more than that: it means the internal
code to get and set the parameter behaves in a way which has side
effects, either to the parameter itself or elsewhere in the shell. There
are other parameters, like `$HISTFILE`, which are used by the shell, but
which are get and set in a normal way --- they are only special in that
the value is looked at by the shell; and, after all, any old shell
function can do that, too. Contrast this with `$PATH` which has all that
paraphernalia to do with hashing commands to take care of when it's set,
as I discussed above, and I hope you'll see the difference.

**Reading into parameters**\
\

The \``read`' builtin, as its name suggests, is the opposite to
\``print`' (there's no \``write`' command in the shell, though there is
often an external command of that name to send a message to another
user), but reading, unlike printing, requires something in the shell to
change to take the value, so unlike `print`, `read` is forced to be a
builtin. Inevitably, the values are read into a parameter. Normally they
are taken from standard input, very often the terminal (even if you're
running a script, unless you redirected the input). So the simplest case
is just

      read param

and if you type a line, and hit return, it will be put into `$param`,
without the final newline.

The `read` builtin actually does a bit of processing on the input. It
will usually strip any initial or final whitespace (spaces or tabs) from
the line read in, though any in the middle are kept. You can read a set
of values separated by whitespace just by listing the parameters to
assign them to; the last parameter gets all the remainder of the line
without it being split. Very often it's easiest just to read into an
array:

      % read -A array
            this is a line typed in now, \ 
          by me,    in this   space
      % print ${array[1]} ${array[12]}
      this space

(I'm assuming you're using the native zsh array format, rather than the
one set with `KSH_ARRAYS`, and shall continue to assume this.)

It's useful to be able to print a prompt when you want to read
something. You can do this with \``print -n`', but there's a shorthand:

      % read line'?Please enter a line: '
      Please enter a line: some words
      % print $line
      some words

Note the quotes surround the \``?`' to prevent it being taken as part of
a pattern on the command line. You can quote the whole expression from
the beginning of \``line`', if you like; I just write it like that
because I know parameter names don't need quoting, because they can't
have funny characters in. It's almost logical.

Another useful trick with `read` is to read a single character; the
\``-k`' option does this, and in fact you can stick a number immediately
after the \``k`' which specifies a number to read. Even easier, the
\``-q`' option reads a single character and returns status 0 if it was
`y` or `Y`, and status 1 otherwise; thus you can read the answer to
yes/no questions without using a parameter at all. Note, however, that
if you don't supply a parameter, the reply gets assigned in any case to
`$REPLY` if it's a scalar --- as it is with `-q` --- or `$reply` if it's
an array --- i.e. if you specify `-A`, but no parameter name. These are
more examples of the non-special parameters which the shell uses --- it
sets `$REPLY` or `$reply`, but only in the same way you would set them;
there are no side-effects.

Like `print`, `read` has a `-r` flag for raw mode. However, this just
has one effect for `read`: without it, a `\` at the end of the line
specifies that the next line is a continuation of the current one (you
can do this when you're typing at the terminal). With it, `\` is not
treated specially.

Finally, a more sophisticated note about word-splitting. I said that,
when you are reading to many parameters or an array, the word is split
on whitespace. In fact the shell splits words on any of the characters
found in the (genuinely special, because it affects the shell's guts)
parameter `$IFS`, which stands for \`input field separator'. By default
--- and in the vast majority of uses --- it contains space, tab, newline
and a null character (character zero: if you know that these are usually
used to mark the end of strings, you might be surprised the shell
handles these as ordinary characters, but it does, although printing
them out usually doesn't show anything). However, you can set it to any
string: enter

      fn() {
        local IFS=:
        read -A array
        print -l $array
      }
      fn

and type

    one word:two words:three words:four

The shell will show you what's in the array it's read, one \`word' per
line:

      one word
      two words
      three words
      four

You'll see the bananas, er, words (joke for the over-thirties) have been
treated as separated by a colon, not by whitespace. Making `$IFS` local
didn't work in old versions of zsh, as with other specials; you had to
save it and restore it.

The `read` command in zsh doesn't let you do line editing, which some
shells do. For that, you should use the `vared` command, which runs the
line editor to edit a parameter, with the `-c` option, which allows
`vared` to create a new parameter. It also takes the option `-p` to
specify a prompt, so one of the examples above can be rewritten

      vared -c -p 'Please enter a line: ' line

which works rather like read but with full editing support. If you give
the option `-h` (history), you can even retrieve values from previous
command lines. It doesn't have all the formatting options of read,
however, although when reading an array (use the option `-a` with `-c`
if creating a new array) it will perform splitting.

**Other builtins to control parameters**\
\

The remaining builtins which handle parameters can be dealt with more
swiftly.

The builtin `set` simply sets the special parameter which is passed as
an argument to functions or scripts, and which you access as `$*` or
`$@`, or `$<number>` (Bourne-like format), or via `$argv` (csh-like
format), known however you set them as the \`positional parameters':

      % set a whole load of words
      % print $1
      a
      % print $*
      a whole load of words
      % print $argv[2,-2]
      whole load of

It's exactly as if you were in a function and had called the function
with the arguments \``a whole load of words`'. Actually, set can also be
used to set shell options, either as flags, e.g. \``set -x`', or as
words after \``-o`' , e.g. \``set -o xtrace`' does the same as the
previous example. It's generally easier to use `setopt`, and the upshot
is that you need to be careful when setting arguments this way in case
they begin with a \``-`'. Putting \``-``-`' before the real arguments
fixes this.

One other use of `set` is to set any array, via

      set -A any_array words to assign to any_array

which is equivalent to (and the standard Korn shell version of)

      any_array=(words to assign to any_array)

One case where the `set` version is more useful is if the name of an
array itself comes from a parameter:

      arrname=myarray
      set -A $arrname words to assign

has no easy equivalent in the other form; the left hand side of an
ordinary assignment won't expand a parameter:

      # Doesn't work; syntax error
      $arrname=(words to assign)

This worked in old versions of zsh, but that was on the non-standard
side. The `eval` command, described below, gives another way around
this.

Next comes \``shift`', which simply moves an array up one element,
deleting the original first one. Without an array name, it operates on
the positional parameters. You can also give it a number to shift other
than one, before the array name.

      shift array

is equivalent to

      array=(${array[2,-1]})

(almost --- I'll leave the subtleties here for the chapter on expansion)
which picks the second to last elements of the array and assigns them
back to the original array. Note, yet again, that `shift` operates using
the *name*, not the *value* of the array, so no \``$`' should appear in
front, otherwise you get something similar to the trick I showed for
\``set -A`'.

Finally, `unset` unsets a parameter, and I already showed you could
unset a key/value pair of an associative array. There is one subtlety to
be mentioned here. Normally, `unset` just makes the parameter named
disappear off the face of the earth. However, if you call `unset` in a
function, its ghost lives on in the sense that any parameter you create
in the same name will be scoped as the original parameter was. Hence:

      var='global value'
      fn() {
        typeset var='local value'
        unset var
        var='what about this?'
      }
      fn
      print $var

The final statement prints \``global value`': even though the local copy
of `$var` was unset, the shell remembers that it was local, so the
second `$var` in the function is also local and its value disappears at
the end of the function.

### 3.2.7: History control commands

The easiest way to access the shell's command history is by editing it
directly. The second easiest way is to use the \``!`'-history mechanism.
Other ways of manipulating it are based around the `fc` builtin, which
probably once stood for something (according to Oliver Kiddle, \`fix
command', which is as good as anything). I talked quite a bit about it
in the last chapter, and don't really have anything to add. Just note
that the two other commands based around it are `history` and `r`.

### 3.2.8: Job control and process control

One of the major contributions of the C-shell was job control. You need
to know about foreground and background tasks, and again I introduced
these in the last chapter along with the options that control them. Here
is an introduction to the relevant builtins.

You start a background job in two ways. First, directly, by putting an
\``&`' after it:

      sleep 10 &

and secondly by starting it in the normal way (i.e. in the foreground),
then typing `^Z`, and using the `bg` command to put it in the
background. Between typing `^Z` and `bg`, the job is still there, but is
not running; it is \`suspended' or \`stopped' (systems use different
descriptions for the same thing), waiting for you to decide what to do
with it. In either case, the job then continues without the shell
waiting for it. It will still try and read from or write to the terminal
if that's how you started it; you need to use the shell's redirection
facilities right at the start if you want to change that, there's
nothing you can do after the job has already started.

By the way, \`sleep' isn't a builtin. Oddly enough, you can suspend a
builtin command or sequence of commands (such as shell function) with
`^Z`, although since the shell has to continue executing your commands
as well as being suspended, it does the only thing it can do --- fork,
so that the commands you suspend are put into the background. Probably
you will only rarely do this with builtins. No other shell, so far as I
know, has this feature.

A job will stop if it needs to read from the terminal. You see a message
like:

      [1]  + 1348 suspended (tty input)  jobname and arguments

which means the job is suspended very much like you had just typed `^Z`.
You need to bring the job into the forground, as described below, so
that you can type something to it.

By the way, the key to type to suspend a command may not be `^Z`; it
usually is, but that can be changed. Run \``stty -a`' and look for what
is listed after \``susp =`' --- probably, but not necessarily, `^Z`. So
if you want to use another character --- it must be a single character;
this is handled deep in the terminal interface, not in the shell --- you
can run

      stty susp '^]'

or whatever. You will note from the `stty` output that various other job
control characters can be changed similarly. The `stty` command is
external and its format for both output and input can vary quite a bit
from system to system.

Instead of putting the command into the background, you can bring it
back to the foreground again with `fg`. This is useful for temporarily
stopping what you are doing so you can do something else. These days you
would probably do it in another window; in the old days when people
logged in from simple terminals this was even more useful. A typical
example of this is

      more file                        # look at file
      ^Z                               # suspend
      [1] + 8592 suspended  more file  # message printed
      ...                              # do something else
      fg %1                            # resume the `more'

The \``%`' is the usual way of referring to jobs. The number after it is
what appeared in square brackets with the suspended message; I don't
know why the shell doesn't use the \``%`' notation there, too. You also
see that with the \`continued' message when you put something into the
background, and again at the end with the \`done' message which tells
you a background job is finished. The \``%`' can take other forms; the
most common is to follow it by the name of a command, such as \``%more`'
in this case. The forms `%+` and `%-` refer to the most recent and
second most recent jobs --- the \``+`' in the \`suspended' message is
telling you that the `more` job could be referred to like that.

Most of the job control commands will actually assume you are talking
about \``%+`' if you don't give an argument, so assuming I hadn't
started any other commands in the background, I could just have put
\``fg`' at the end of the sequence of commands above. This actually cuts
both ways: `fg` is the default operation on jobs referred to with the
\``%`' notation, so just typing \``%1`' with no command name would have
worked, too.

You can jog your memory about what's going on with the \``jobs`'
command. It looks like a series of messages of the form beginning with
the number in square brackets; usually the jobs will either be
\`running' or \`suspended'. This will tell you the numbers you need.

One other useful thing you can do with a job is to tell the shell to
forget about it. This is only really useful if it is already running in
the background; then you can run \``disown`' with the job identifier.
It's useful for jobs you want to continue after you've logged out, as
well as jobs that have their own windows which you can therefore control
directly. With disowned jobs, the shell doesn't warn you that they are
still there when you log out. You can actually disown a background job
when you start it by putting \``&|`' or \``&!`' at the end of the line
instead of simply \``&`'. Note that if the job was suspended when you
disowned it, it will stay disowned; this is pretty pointless, so you
probably should run \``bg`' on it first.

The next most likely thing you want to do with a job is kill it, or
maybe suspend it when it's already in the background and you can't just
type `^Z`. This is where the `kill` builtin comes in. There's more to
this than there is to the builtins mentioned above. First, you can use
`kill` with other processes that weren't started from the current shell.
In that case, you would use a number to identify it, with no `%` ---
that's why the `%`'s were there in the other cases. Of course, you need
to find out the number; the usual way is with the `ps` command, which is
not a builtin but which appears on all UNIX-like systems. As a stupid
example, here I start a disowned process which does very little, look
for it, then kill it:

      % sleep 60 &|
      % ps -f
      UID        PID  PPID  C STIME TTY          TIME CMD
      pws        623   614  0 22:12 pts/0    00:00:00 zsh
      pws       8613   623  0 23:12 pts/0    00:00:00 sleep 60
      pws       8615   623  0 23:12 pts/0    00:00:00 ps -f
      % kill 8613
      % ps -f
      UID        PID  PPID  C STIME TTY          TIME CMD
      pws        623   614  0 22:12 pts/0    00:00:00 zsh
      pws       8616   623  0 23:12 pts/0    00:00:00 ps -f

The process has disappeared the second time I look. Notice that in the
usual lugubrious UNIX way the shell didn't bother to tell you the
process had been killed; however, it will report an error if it failed
to send it the signal. Sending it the signal is all the shell cares
about; the shell won't warn if you if the process decided it didn't want
to die when told to, so it's still a good idea to check.

Sometimes you want to wait for a process to exit; the `wait` builtin can
do this, and like `kill` can take a process number as well as a job
number. However, that's a bit deceptive --- you can't actually wait for
a process which wasn't started directly from the shell. Indeed, the
mechanism for waiting is all bound up with the way UNIX handles
processes; unless its parent waits for it, a process becomes a \`zombie'
and hangs around until the system's foster parent, the \`init' process
(always process number 1) waits for it instead. It's all a little bit
baroque, but for the shell user, wait just means you can hang on until
something you started has finished. Indeed, that's how foreground
processes work: the shell in effect uses the internal version of `wait`
to hang around until the job exits. (Well, actually that's a lie; the
system wakes it up from whatever it's doing to tell it a child has
finished, so all it has to do is doze off to wait.)

Furthermore, you can wait for a process even if job control isn't
running. Job control, basically anything involving those `%`'s, is only
useful when you are sitting at a terminal fiddling with commands; it
doesn't operate when you run scripts, say. Then the shell has much less
freedom in how to control its jobs, but it can still wait for a
background process, and it can still use `kill` on a process if it knows
its number. For this purpose, the shell stores the ID of the last
process started in the background in the parameter `$!`; there's
probably a good reason for the \``!`', but I don't know what it is. This
happens regardless of job control.

**Signals**\
\

The `kill` command can do a good deal more than just kill a process.
That is the default action, which is why the command has that name. But
what it's really doing is sending a \`signal' to a process. Signals are
the simplest way of communicating to another process; in fact, they are
about the only simple way if you haven't made special arrangements for
the process to read messages from you. Signal names are written like
`SIGINT`, `SIGTSTP`, `SIGKILL`; to send a particular signal to a
process, you remove the `SIG`, stick a hyphen in front, and use that as
the first argument to `kill`, e.g.:

      kill -KILL 8613

Some of the things you already know about are actually doing just that.
When you type `^C` to stop a process, you are actually sending it a
`SIGINT` for \`interrupt', as if you had done

      kill -INT 8613

The usual signal sent by `kill` is not, as you might have guessed,
`SIGKILL`, but actually `SIGTERM` for \`terminate'; `SIGKILL` is
stronger as the process can't block that signal, as it can with many
(we'll see how the shell can do that in a moment). It's familiar to UNIX
hackers as \``kill -9`', because all the signals also have numbers. You
can see the list of signals in zsh by doing:

      % print $signals
      EXIT HUP INT QUIT ILL TRAP ABRT BUS FPE KILL USR1
      SEGV USR2 PIPE ALRM TERM STKFLT CLD CONT STOP TSTP
      TTIN TTOU URG XCPU XFSZ VTALRM PROF WINCH POLL PWR
      UNUSED ZERR DEBUG

Your list will probably be different from mine; this is for Linux, and
the list is very system-specific, even though the first nine are
generally the same, and many of the others are virtually always present.
Actually, `SIGEXIT` is an invention by the shell for you to allow the
shell to do something when a function exits (see the section on \`traps'
below); you can't actually use \``kill -EXIT`'. Thus `SIGHUP` is the
first real signal, and indeed that's number one, so you have to shift
the contents of `$signals` along one to get the right numbers. `SIGTERM`
and `SIGINT` usually have the same effect, stopping the process, unless
that has decided to handle the signal some other way.

The last two signals are bogus, too: `SIGZERR` is to allow the shell to
do something on an error (non-zero exit status), while with `SIGDEBUG`
you can do it on every command. Again, the \`something' to be executed
is a \`trap', as I'll discuss in a short while.

Typing `^Z` to suspend a process actually sends the process a `SIGTSTP`
(terminal stop, since it usually comes from the terminal), while
`SIGSTOP` is similar but usually doesn't come from a terminal. Even
restarting a process as with `bg` sends it a signal, in this case
`SIGCONT`. It seems a bit odd to signal a process to restart; why can't
the operating system just restart it when you ask? The real answer is
probably that signals provide an easy way for you to talk to the
operating system without grovelling around in the dirt too much.

Before I talk about how you make the shell handle signals it receives,
there is one extra oddment: the `suspend` builtin effectively sends the
shell a signal to suspend it, as if you'd typed `^Z`, though as you've
probably found by now that doesn't suspend the shell itself. It's only
useful to do this if the shell is running under some other programme,
else there's no way of restoring it and suspending is effectively the
same as exiting the shell. For this reason, the shell won't let you call
`suspend` in a login shell, because it assumes that is running as the
top level (though in the previous chapter you learnt there's actually
nothing that special about login shells; you can start one just with
\`zsh -l'). If you're logged in remotely via `rsh` or `ssh`, it's
usually more convenient to use the keystrokes \``~^Z`' which those
define, rather than zsh's mechanism; they have to be at the beginning of
a line, so hit return first if necessary. This returns you to your local
terminal; you can resume the remote login with \``fg`' just like any
other programme.

**Traps**\
\

The way of making the shell handle signals is called \`traps'. There are
actually two mechanisms for this. I'll present the more standard one and
then talk about the advantages and drawbacks of the other one at the
end.

The standard version (shared with other shells) is via the \``trap`'
builtin. The first argument is a chunk of shell code to execute, which
obviously needs to be quoted when you pass it as an argument, and the
remaining arguments are a list of signals to handle, minus the `SIG`
prefix. So:

      trap "echo I\\'m trapped." INT

tells the shell what to do on `SIGINT`, i.e. `^C`. Note the extra layer
of quoting: the double quotes surround the code, so that when they are
stripped `trap` sees the chunk

      echo I\'m trapped

Usually the shell would abort what it was doing and return to the main
prompt when you hit `^C`. Now, however, it will simply print the message
and carry on. You can try this, for example, with

      read line

If you hit `^C` while it's waiting for input, you'll see the message go
up, but the shell will still wait for you to type a line.

A warning about this: `^C` is only trapped within the shell itself. If
you start up an external programme, it will have its own mechanism for
handling signals, and if it usually aborts on `^C` it still will. But
there's a sting in the tail: do

      cat

which waits for input to output again (you need to use `^D` to exit
normally). If you type `^C` here, the command will be aborted, as I said
--- but you still get the message \``I'm trapped`'. That's because the
shell is able to tell that the command got that particular signal, and
calls the trap when the `cat` exits. Not all shells do this;
furthermore, some commands which handle signals themselves won't give
the shell enough information to know that a signal arrived, and in that
case the trap won't be called. Such commands are usually the more
sophisticated things like editors or screen managers or whatever; you
just have to find out by trial and error.

You can also make the shell ignore the signal completely. To do this,
the first argument should be an empty string:

      trap '' INT

Now `^C` will have no effect, and *this* time the effect *is* passed on
directly to commands called from the shell --- try the `cat` example and
you won't be able to interrupt it; type `^D` or use the lesser known but
more powerful `^\` (control with backslash), which sends `SIGQUIT`. If
it hasn't been disabled, this will also produce a file `core`, which
contains debugging information about what the programme was doing when
it exited --- never call your own files `core`. You can trap `SIGQUIT`
too, if you want. (The shell itself usually ignores `SIGQUIT`; it's only
useful for external commands.)

Now the other sort of trap. I could have written for the first example:

      TRAPINT() {
        print I\'m trapped.
      }

As you can see, this is just a function: functions beginning `TRAP` are
special. However, it's a real function too; you can call it by hand with
the command \`TRAPINT', and it will run perfectly happily with no funny
side effects.

There is a difference between the way the two types work. In the
\``trap`' sort of trap, the code is just evaluated just as if it
appeared as instructions to the shell at the point where the trap
happened. So if you were in a function, you would see the environment of
that function with its local variables; if you set a local variable with
`typeset`, it would be visible in the function just as if it were
created there.

However, in the function type of trap, the code is provided with its own
function environment. Now if you use `typeset` the parameter created is
local only to the trap. In most cases, that's all the difference there
is; it's up to you to decide which is more convenient. As you can see,
the function type of trap doesn't require the extra layer of quoting, so
looks a little smarter. Conveniently, the \``trap`' command on its own
lists all traps in the form of the shell code you'd need to recreate
them, and you can see which sort is which.

There are two cases where the difference sticks out. One is that the
function type has some extra wiring to allow you both to trap a signal,
and pretend to anyone watching that the shell didn't handle it. An
example will show this:

      TRAPINT() {
        print "Signal caught, stopping anyway."
        return $(( 128 + $1 ))
      }

That second line may look as rococo as the Amalienburg, but it's meaning
is this: `$1`, the first argument to the function, is set to the number
of the signal. In this case it will be 2 because that's the standard
number for `SIGINT`. That means the arithmetic substitution `$((...))`
returns 130, the command \``return 130`' is executed, and the function
returns with status 130. Returning with non-zero status is special in
function traps: it tells the shell you want to abort the surrounding
command even though the trap was handled, and that you want the status
associated with that to be 130. It so happens that this is how UNIX
handles returns from normal traps. Without setting a trap, do

      % cat
      ^C
      % print $?

and you'll see that this, too, has given the status 130, 128 plus the
value of `SIGINT`. So if you *do* have the trap set, you'll see the
message, but the command will abort --- even if it was running inside
the shell.

Try

      % read line
      ^C

to see that happening. If you look at the status in `$?` you'll find
it's actually 1, not 130; that's because the `read` command, when it
aborted, overrode the return value from the trap. But it does that with
an untrapped `^C`, too, so that's not really an exception to what I've
just said.

If you've been paying attention, you'll realise that traps set with the
`trap` builtin can't do it in quite this way, because the function they
return from would be whatever function you were in. You can see that:

      trap 'echo Returning...; return;' INT
      fn() {
        print In fn...
        read param
        print Leaving fn..
      }

If you run `fn` and hit `^C`, the signal is trapped and the message
printed, but because of the `return`, the shell quits `fn` immediately
and you don't see the final message. If you missed out the \``return;`'
(try it), the shell would carry on with the rest of `fn` after you typed
something to `read`. Of course you can use this mechanism to leave
functions after trapping a signal; it just so happens that in this case
the mechanism with `TRAPINT` is a little closer to what untrapped
signals do and hence a little neater.

One final flourish of late Baroque splendour: the trap for `SIGEXIT`,
the one called when a function (or the shell itself, in fact) exits is a
bit special because in the case of exiting a function it will be called
in the environment of the calling function. So if you need to do
something like set a local variable for an enclosing function you can
have

      trap 'typeset param_in_enclosing_func=value' EXIT

do it for you; you couldn't do that with `TRAPEXIT` because the code
would have its own function, so that even though it would be called
after the first function exited, it wouldn't run directly in the
enclosing one but in a separate `TRAPEXIT` function. You can even set an
EXIT trap for the enclosing function by defining a nested
\``trap .. EXIT`' inside that trap itself.

I lied, because there is one more special thing about `TRAPEXIT`: it's
always reset after you exit a function and the trap itself has been
called. Most traps just hang around until you explicitly unset them.
There is an option, `LOCAL_TRAPS`, which makes traps set inside
functions as well insulated as possible from those outside, or inside
deeper functions. In other words, the old trap is saved and then
restored when you exit the function; the scoping works pretty much like
that for `typeset`, and in the same way traps for the enclosing scope,
apart from any for `EXIT`, remain in effect inside a function unless you
explicitly override them; and, again in the same way, if you unset it
inside the function it will still be restored on exit.

`LOCAL_TRAPS` is the fixed behaviour of some other shells. In zsh,
without the option set:

      trap 'echo Hi.' INT
      fn() {
         trap 'echo Bye.' INT
      }

Calling `fn` simply replaces the trap defined outside the function with
the one defined inside while:

      trap 'echo Hi.' INT
      fn() {
         setopt localtraps
         trap 'echo Bye.' INT
      }

puts the original \`Hi' trap back after the function exits.

I haven't told you how to unset a trap for good: the answer is

     trap - INT

As you would guess, you can use `unfunction` with function-type traps;
that will correctly remove the trap as well as deleting the function.
However, \``trap -`' works with both, so that's the recommended way.

**Limits on processes**\
\

One other way that jobs started by the shell can be controlled is by
using limits. These are actually limits set by the operating system, but
the shell gives you a way of controlling them: the `limit` and `unlimit`
commands. Type \``limit`' on its own to see a summary. I get:

      cputime         unlimited
      filesize        unlimited
      datasize        unlimited
      stacksize       8MB
      coredumpsize    0kB
      memoryuse       unlimited
      maxproc         2048
      descriptors     1024
      memorylocked    unlimited
      addressspace    unlimited

where the item on the left of each line is what is being limited, and on
the right is the value. The manual page to look at, at least on Linux is
for the function `getrusage`; that's the function the shell is calling
when you run `limit` or `unlimit`.

In this case, the items are:

**`cputime`**

the total CPU time used by a process

**`filesize`**

maximum size of a file

**`datasize`**

the maximum size of data in use by a programme

**`stacksize`**

the maximum size of the stack, which is the area of memory used to store
information during function calls

**`coredumpsize`**

the maximum size of a `core` file, which is an image of memory left by a
programme that crashes, allowing you to debug it with `gdb`, `dbx`,
`ddd` or some other debugger

**`memoryuse`**

the maximum main memory, i.e. programme memory which is in active use
and hasn't been \`swapped out' to disk

**`maxproc`**

the maximum number of simultaneous processes

**`descriptors`**

the maximum number of simultaneously open files (\`descriptors' are the
internal mechanism for referring to an open file on UNIX-like systems)

**`memorylocked`**

the maximum amount of memory locked in (I don't know what that is,
either)

**`addressspace`**

the total amount of virtual memory, i.e. any memory whether it is main
memory, or refers to somewhere on a disk, or indeed anything else.

You may well see other names; the shell decides when it is compiled what
limits are supported by the system.

Of those, the one I use most commonly is `coredumpsize`: sometimes when
I'm debugging a file I want a crashed programme to produce a \``core`'
files so I can run `gdb` or `dbx` on it (\``unlimit coredumpsize`'),
while other times they are just untidy (\``limit coredumpsize 0`').
Probably you would only alter any of the others if you knew there was a
problem, for example a number-crunching programme used so much memory
that the rest of the system was badly affected and you wanted to limit
`datasize` to 64 megabyte or whatever. You could write this as:

      limit datasize 64m

There is a distinction made between \`hard' and \`soft' limits. Both
have the same effect on programmes, but you can remove or reduce \`soft'
limits, while only the superuser (the system administrator's login,
root) can do that to \`hard' limits. Usually, therefore, `limit` and
`unlimit` manipulate soft limits; to show or set hard limits, give the
option `-h`. If I do \``limit -h`', I get the same list of limits as
above, but with `stacksize` and `coredumpsize` unlimited --- that means
I can reduce or remove the limits on those if I want, they're just set
for my own convenience.

Why is `stacksize` set in this way? As I said, it refers to the memory
in which the functions in programmes store variables and any other local
information. If one function calls another, it uses more memory. You can
get into a situation where functions call themselves recursively and
there is no way out until the machine runs out of memory; limiting
`stacksize` prevents this. You can actually see this with zsh itself
(probably better not to try this if you'd rather the shell you're
running didn't crash):

      % fn() { fn; }
      % fn

defines a function which keeps calling itself. To do this, all the
functions *inside* zsh are calling themselves as well, using more and
more stack memory. Actually, zsh uses other forms of memory inside each
function and my version of zsh crashes due to exhaustion of that memory
instead. However, it depends on the system how this works out.

**Times**\
\

One way of returning information on process resources is with the
\``times`' command. It simply shows the total CPU time used by the shell
and by the programmes called for it --- in that order, and without
description, so you need to remember. On each line, the first number is
the time spent in user space and the second is the time spent in system
space. If you're not concerned about the details of programmes the
difference is pretty irrelevant, but if you are, then the difference is
very roughly that between the time spent in the code you actually see
before you compile a programme, and the time spent in \`hidden' code
where the system is doing something for you. It's not such an obvious
distinction, because many library routines, such as mathematical
functions, are run in user mode as no privileged access to internal bits
of the system is required. Typically, system time is concerned with the
details of input and output --- though even there it's not so simple,
because the C output routines `printf`, `puts`, `fread` and others have
user mode code which then calls the system routines `read`, `write` and
so on.

You can measure the time taken by a particular external command by
putting \``time`', in the singular this time, in front of it; this is
essentially another precommand modifier, and is a shell reserved word
rather than a builtin. This gives fairly obvious information. You can
specify the information using the `$TIMEFMT` parameter, which has its
own percent escapes, different from the ones used in prompts. It exists
partly because the shell allowed you to access all sorts of other
information about the process which ran, such as \`page faults' ---
occasions when the system had to fetch a part of the programme or data
from disk because it wasn't in the main memory. However, that
disappeared because it was too much work to convert the feature to
configure itself automatically for different operating systems. It may
be time to resurrect it.

You can also force the time to be shown automatically by setting the
parameter `$REPORTTIME`; if a command runs for more than this many
seconds, the `$TIMEFMT` output will be shown automatically.

### 3.2.9: Terminals, users, etc.

**Watching for other users**\
\

Although this is more associated with parameters than builtins, the
\``log`' command will tell you whether any of a group of people you want
to watch out for have logged in or out. To use this, you set the
`$watch` array parameter to a list of user names, or \``all`' for
everyone, or \``notme`' for everyone except yourself. Even if you don't
use `log`, any changes will be reported just before the shell prints a
prompt. It will be printed using the `$WATCHFMT` parameter: once again,
this takes its own set of percent escapes, listed in the `zshparam`
manual.

**`ttyctl`**\
\

There is a command `ttyctl` which is designed to keep badly behaved
external commands from messing up the terminal settings. Most programmes
are careful to restore any settings they change, but there are
exceptions. After \``ttyctl -f`', the terminal is frozen; zsh will
restore the settings, no matter what an external programme does with it.
This includes deliberate attempts to change the terminal settings with
the \``stty`' command, so the default is unfrozen, \``ttyctl -u`'.

### 3.2.10: Syntactic oddments

This section collects together a few builtins which, rather than
controlling the behaviour of some feature of the shell, have some other
special effect.

**Controlling programme flow**\
\

The four functions here are `exit`, `return`, `break`, `continue` and
`source` or `.`: they determine what the shell does next. You've met
`exit` --- leave the shell altogether --- and `return` --- leave the
current function. Be very careful not to confuse them. Calling `exit` in
a shell function is usually bad:

      % fn() { exit; }
      % fn

This makes you entire shell session go away, not just the function. If
you write C programmes, you should be very familiar with both, although
there is one difference in this case: `return` at the top level in an
interactive shell actually does nothing, rather than leaving the shell
as you might expect. However, in a script, return outside a function
*does* cause the entire script to stop. The reason for this is that zsh
allows you to write autoloaded functions in the same form as scripts, so
that they can be used as either; this wouldn't work if `return` did
nothing when the file was run as a script. Other shells don't do this:
`return` does nothing at the top level of a script, as well as
interactively. However, other shells don't have the feature that
function definition files can be run as scripts, either.

The next two commands, `break` and `continue`, are to do with constructs
like \``if`'-blocks and loops, and it will be much easier if I introduce
them when I talk about those below. They will also already be familiar
to C programmers. (If you are a FORTRAN programmer, however, `continue`
is *not* the statement you are familiar with; it is instead equivalent
to `CYCLE` in FORTRAN90.)

The final pair of commands are `.` and `source`. They are similar to one
another and cause another file to be read as a stream of commands in the
current shell --- not as a script, for which a new shell would be
started which would finish at the end of the script. The two are
intended for running a series of commands which have some effect on the
current shell, exactly like the startup files. Indeed, it's a very
common use to have a call to one or other in a startup file; I have in
my `~/.zshrc`

      [[ -f ~/.aliasrc ]] && . ~/.aliasrc

which tests if the file `~/.aliasrc` exists, and if so runs the commands
in it; they are treated exactly as if they had appeared directly at that
point in `.zshrc`.

Note that your `$path` is used to find the file to read from; this is a
little surprising if you think of this as like a script being run, since
zsh doesn't search for a script, it uses the name exactly as you gave
it. In particular, if you don't have \``.`' in your `$path` and you use
the form \``.`' rather than \``source`' you will need to say explicitly
when you want to source a file in the current directory:

      . ./file

otherwise it won't be found.

It's a little bit like running a function, with the file as the function
body. Indeed, the shell will set the positional parameters `$*` in just
the same way. However, there's a crucial difference: there is no local
parameter scope. Any variables in a sourced file, as in one of the
startup files, are in the same scope as the point from which it was
started. You can, therefore, source a file from inside a function and
have the parameters in the sourced file local, but normally the only way
of having parameters only for use in a sourced file is to unset them
when you are finished.

The fact that both `.` and `source` exist is historical: the former
comes from the Bourne shell, and the latter from the C shell, which
seems deliberately to have done everything differently. The point noted
above, that source always searches the current directory (and searches
it first), is the only difference.

**Re-evaluating an expression**\
\

Sometimes it's very useful to take a string and run it as if it were a
set of shell commands. This is what `eval` does. More precisely, it
sticks the arguments together with spaces and calls them. In the case of
something like

      eval print Hello.

this isn't very useful; that's no different from a simple

      print Hello.

The difference comes when what's on the command line has something to be
expanded, like a parameter:

      param='print Hello.'
      eval $param

Here, the `$param` is expanded just as it would be for a normal command.
Then `eval` gets the string \``print Hello.`' and executes it as shell
command line. Everything --- really everything --- that the shell would
normally do to execute a command line is done again; in effect, it's run
as a little function, except that no local context for parameters is
created. If this sounds familiar, that's because it's exactly the way
traps defined in the form

      trap 'print Hello.' EXIT

are called. This is one simple way out of the hole you can sometimes get
yourself into when you have a parameter which contains the name of
another parameter, instead of some data, and you want to get your hands
on the data:

      # somewhere above...
      origdata='I am data.'
      # but all you know about is
      paramname=origdata
      # so to extract the data you can do...
      eval data=\$$paramname

Now `$data` contains the value you want. Make sure you understand the
series of expansions going on: this sort of thing can get very
confusing. First the command line is expanded just as normal. This turns
the argument to `eval` into \``data=$origdata`'. The \``$`' that's still
there was quoted by a backslash; the backslash was stripped and the
\``$`' left; the `$paramname` was evaluated completely separately ---
quoted characters like the `\$` don't have any effect on expansions ---
to give `origdata`. Eval calls the new line \``data=$origdata`' as a
command in its own right, with the now obvious effect. If you're even
slightly confused, the best thing to do is simply to quote everything
you don't want to be immediately expanded:

      eval 'data=$'$paramname

or even

      eval 'data=${'$paramname'}'

may perhaps make your intentions more obvious.

It's possible when you're starting out to confuse \``eval`' with the
`` `...` `` and `$(...)` commands, which also take the command in the
middle \``...`' and evaluate it as a command line. However, these two
(they're identical except for the syntax) then insert the output of that
command back into the command line, while `eval` does no such thing; it
has no effect at all on where input and output go. Conversely, the two
forms of command substitution don't do an extra level of expansion.
Compare:

      % foo='print bar'
      % eval $foo
      bar

with

      % foo='print bar'
      % echo $($foo)
      zsh: command not found: print bar

The `$`(*...*) substitution took `$foo` as the command line. As you are
now painfully aware, zsh doesn't split scalar parameters, so this was
turned into the single word \``print bar`', which isn't a command. The
blank line is \``echo`' printing the empty result of the failed
substitution.

### 3.2.11: More precommand modifiers: `exec`, `noglob`

Sometimes you want to run a command *instead* of the shell. This
sometimes happens when you write a shell script to process the arguments
to an external command, or set parameters for it, then call that
command. For example:

      export MOZILLA_HOME=/usr/local/netscape
      netscape "$@"

Run as a script, this sets an environment variable, then starts
`netscape`. However, as always the shell waits for the command to
finish. That's rather wasteful here, since there's nothing more for the
shell to do; you'd rather it simply magically turned into the `netscape`
command. You can actually do this:

      export MOZILLA_HOME=/usr/local/netscape
      exec netscape "$@"

\``exec`' tells the shell that it doesn't need to wait; it can just make
the command to run replace the shell. So this only uses a single
process.

Normally, you should be careful not to use `exec` interactively, since
normally you don't want the shell to go away. One legitimate use is to
replace the current zsh with a brand new one if (say) you've set a whole
load of options you don't like and want to restore the ones you usually
have on startup:

      exec zsh

Or you may have the bad taste to start a completely different shell
altogether. Conversely, a good piece of news about `exec` is that it is
common to all shells, so you can use it from another shell to start zsh
in the way I've just shown.

Like \``command`' and \``builtin`', \``exec`' is a \`precommand
modifier' in that it alters the way a command line is interpreted.
Here's one more:

      noglob print *

If you've remembered what \`glob' means, this is all fairly obvious. It
instructs the shell not to turn the \``*`' into a list of all the files
in the directory, but instead to let well alone. You can do this by
quoting the \``*`', of course; often `noglob` is used as part of an
alias to set up commands where you never need filename generation and
don't want to have to bother quoting everything. However, note that
`noglob` has no effect on any other type of expansion: parameter
expansion and backquote (`` `....` ``) expansion, for example, happen as
normal; the only thing that doesn't is turning patterns into a list of
matching files. So it doesn't take away the necessity of knowing the
rules of shell expansion. If you need that, the best thing to do is to
use `read` or `vared` (see below) to read a line into a parameter, which
you pass to your function:

      read -r param
      print $param

The `-r` makes sure `$param` is the unadulterated input.

### 3.2.12: Testing things

I told you in the last chapter that the right way to write tests in zsh
was using the \``[[ ... ]]`' form, and why. So you can ignore the two
builtins \``test`' and \``[`', even though they're the ones that
resemble the Bourne shell. You can safely write

      if [[ $foo = '' ]]; then
        print The parameter foo is empty.  O, misery me.
      fi

or

      if [[ -z $foo ]]; then
        print Alack and alas, foo still has nothing in it.
      fi

instead of monstrosities like

      if test x$foo != x; then
        echo The emptiness of foo.  Yet are we not all empty\?
      fi

because even if `$foo` does expand to an empty string, which is what is
implied if the tests are true, \``[[ ... ]]`' remembers there was
something there and gets the syntax right. Rather than a builtin, this
is actually a reserved word --- in fact it has to be, to be
syntactically special --- but you probably aren't too bothered about the
difference.

There are two sorts of tests, both shown above: those with three
arguments, and those with two. The three-argument forms all have some
comparison in the middle; in addition to \``=`' (or \``==`', which means
the same here, and which according to the manual page we should be
using, though none of us does), there are \``!=`' (not equal), \``<`',
\``>`', \``<=`' and \``>=`'. All these do *string* comparisons, i.e.
they compare the sort order of the strings.

Since there are better ways of sorting things in zsh, the \``=`' and
\``!=`' forms are by far the most common. Actually, they do something a
bit more than string comparison: the expression on the right can be a
pattern. The patterns understood are just the same as for matching
filenames, except that \``/`' isn't special, so it can be matched by a
\``*`'. Note that, because \``=`' and \``!=`' are treated specially by
the shell, you shouldn't quote the patterns: you might think that unless
you do, they'll be turned into file names, but they won't. So

      if [[ biryani = b* ]]; then
        print Word begins with a b.
      fi

works. If you'd written `'b*'`, including the quotes, it wouldn't have
been treated as a pattern; it would have tested for a string which was
exactly the two letters \``b*`' and nothing else. Pattern matching like
this can be very powerful. If you've done any Bourne shell programming,
you may remember the only way to use patterns there was via the
\``case`' construction: that's still in zsh (see below), and uses the
same sort of patterns, but the test form shown above is often more
useful.

Then there are other three-argument tests which do numeric comparison.
Rather oddly, these use letters rather than mathematical symbols:
\``-eq`', \``-lt`' and \``-le`' compare if two numbers are equal, less
than, or less than or equal, to one another. You can guess what \``-gt`'
and \``-ge`' do. Note this is the other way round to Perl, which much
more logically uses \``==`' to test for equality of numbers (not \``=`',
since that's always an assignment operator in Perl) and \``eq`' (minus
the minus) to test for equality of strings. Unfortunately we're now
stuck with it this way round. If you are only comparing numbers, it's
better to use the \``(( ... ))`' expression, because that has a proper
understanding of arithmetic. However,

      if [[ $number -gt 3 ]]; then
        print Wow, that\'s big
      fi

and

      if (( $number > 3 )); then
        print Wow, that\'s STILL big
      fi

are essentially equivalent. In the second case, the status is zero
(true) if the number in the expression was non-zero (sorry if I'm
confusing you again) and vice versa. This means that

      if (( 3 )); then
        print It seems that 3 is non-zero, Watson.
      fi

is a perfectly valid test. As in C, the test operators in arithmetic
return 1 for true and 0 for false, i.e. \``$number > 3`' is 1 if
`$number` is greater than 3 and 0 otherwise; the inversion to shell
logic, zero for true, only occurs at the final step when the expression
has been completely evaluated and the \``(( ... ))`' command returns. At
least with \``[[ ... ]]`' you don't need to worry about the extra
negation; you can simply think in logical terms (although that's hard
enough for a lot of people).

Finally, there are a few other odd comparisons in the three-argument
form:

      if [[ file1 -nt file2 ]]; then
        print file1 is newer than file2
      fi

does the test implied by the example; there is also \``-ot`' to test for
an older file, and there is also the little-used \``-ef`' which tests
for an \`equivalent file', meaning that they refer to the same file ---
in other words, are linked; this can be a hard or a symbolic link, and
in the second case it doesn't matter which of the two is the symbolic
link. (If you were paying attention above, you'll know it can't possibly
matter in the first case.)

In addition to these tests, which are pretty recognisable from most
programming languages --- although you'll just have to remember that the
\``=`' family compares strings and not numbers --- there are another set
which are largely peculiar to UNIXy scripting languages. These are all
in the form of a hyphen followed by a letter as the test, which always
takes a single argument. I showed one: \`-z \$var' tests whether
\``$var`' has zero length. It's opposite is \`-n \$var' which tests for
non-zero length. Perhaps this is as good a time as any to point out that
the arguments to these commands can be any single word expression, not
just variables or filenames. You are quite at liberty to test

      if [[ -z "$var is sqrt(`print bibble`)" ]]; then
        print Flying pig detected.
      fi

if you like. In fact, the tests are so eager to make sure that they only
have a one word argument that they will treat things like arrays, which
usually return a whole set of words, as if they were in double quotes,
joining the bits with spaces:

      array=(two words)
      if [[ $array = 'two words' ]]; then
        print "The array \$array is OK.  O, joy."
      fi

Apart from \``-z`' and \``-n`', most of the two-argument tests are to do
with files: \``-e`' tests that the file named next exists, whatever type
of file it is (it might be a directory or something weirder); \``-f`'
tests if it exists and is a regular file (so it isn't a directory or
anything weird this time); \``-x`' tests whether you can execute it.
There are all sorts of others which are listed in the manual page for
various properties of files. Then there are a couple of others:
\``-o <option>`' you've met and tests whether the option is set, and
\``-t <fd>`' tests whether the file descriptor is attached to a
terminal. A file descriptor is a number which for the shell must be
between 0 and 9 inclusive (others may exist, but you can't access them
directly); 0 is the standard input, 1 the standard output, and 2 the
channel on which errors are usually printed. Hence \``[[ -t 0 ]]`' tests
whether the input is coming from a terminal.

There are only four other things making up tests. \``&&`' and \``||`'
mean logical \`and' and \`or', \``!`' negates the effect of a test, and
parentheses \``( ... )`' can be used to surround a set of tests which
are to be treated as one. These are all essentially the same as in C. So

      if [[ 3 -gt 2 && ( me > you || ! -z bah ) ]]; then
        print will I, won\'t I...
      fi

will, because 3 is numerically greater than 2; the expression in
parentheses is evaluated and though \`me' actually comes before \`you'
in the alphabet, so the first test fails, \``-z bah`' is false because
you gave it a non-empty string, and hence \``! -z bah`' is true. So both
sides of the \``&&`' are true and the test succeeds.

### 3.2.13: Handling options to functions and scripts

It's often convenient to have your own functions and scripts process
single-letter options in the way a lot of builtin commands (as well as a
great many other UNIX-style commands) do. The shell provides a builtin
for this called \``getopts`'. This should always be called in some kind
of loop, usually a \``while`' loop. It's easiest to explain by example.

      testopts() {
        # $opt will hold the current option
        local opt
        while getopts ab: opt; do
          # loop continues till options finished
          # see which pattern $opt matches...
          case $opt in
            (a)
               print Option a set
               ;;
            (b)
               print Option b set to $OPTARG
               ;;
        # matches a question mark
        # (and nothing else, see text)
            (\?)
               print Bad option, aborting.
               return 1
               ;;
          esac
        done
        (( OPTIND > 1 )) && shift $(( OPTIND - 1 ))
        print Remaining arguments are: $*
      }

There's quite a lot here if you're new to shell programming. You might
want to read the stuff on structures like `while` and `case` below and
then come back and look at this. First let's see what it does.

      % testopts -b foo -a -- args
      Option b set to foo
      Option a set
      Remaining arguments are: args

Here's what's happening. \``getopts ab: opt`' is the argument to the
\``while`'. That means that the `getopts` gets run; if it succeeds
(returns status zero), then the loop after the \``do`' is run. When
that's finished, the `getopts` command is run again, and so on until it
fails (returns a non-zero status). It will do that when there are no
more options left on the command line. So the loop processes the options
one by one. Each time through, the number of the next argument to look
at is left in the parameter `$OPTIND`, so this gradually increases;
that's how `getopts` knows how far it has got.

The first argument to the `getopts` is \``ab:`'. That means \``a`' is an
option which doesn't take an argument, while \``b`' is an argument which
takes a single argument, signified by the colon after it. You can have
any number of single-letter (or even digit) arguments, which are
case-sensitive; for example \``ab:c:ABC:`' are six different options,
three with arguments. If the option found has an argument, that is
stored in the parameter `$OPTARG`; `getopts` then increments `$OPTIND`
by however much is necessary, which may be 2 or just 1 since \``-b foo`'
and \``-bfoo`' are both valid ways of giving the argument.

If an option is found, we use the \``case`' mechanism to find out what
it was. The idea of this is simple, even if the syntax has the look of
an 18th-century French chateau: the argument \``$opt`' is tested against
all of the patterns in the \``pattern`)' lines until one matches, and
the commands are executed until the next \``;;`'. It's the shell's
equivalent of C's \``switch`'. In this example, we just print out what
the `getopts` brought in. Note the last line, which is called if `$opt`
is a question mark --- it's quoted because \``?`' on its own can stand
for any single character. This is how `getopts` signals an unknown
option. If you try it, you'll see that `getopts` prints its own error
message, so ours was unnecessary: you can turn the former off by putting
a colon right at the start of the list of options, making it \``:ab:`'
here.

Actually, having this last pattern as an *un*quoted \``?`' isn't such a
bad idea. Suppose you add a letter to the list that `getopts` should
handle and forget to add a corresponding item in the `case` list for it.
If the last item matches any character, you will get the behaviour for
an unhandled option, which is probably the best thing to do. Otherwise
nothing in the `case` list will match, the shell will sail blithely on
to the next call to `getopts`, and when you try to use the function with
the new option you will be left wondering quite what happened to it.

The last piece of the `getopts` jigsaw is the next line, which tests if
`$OPTIND` is larger than 1, i.e. an option was found and `$OPTIND` was
advanced --- it is automatically set to 1 at the start of every function
or script. If it was, the \``shift`' builtin with a numeric argument,
but no array name, moves the positional parameters, i.e. the function's
arguments, to shift away the options that have been processed. The
`print` in the next line shows you that only the remaining non-option
arguments are left. You don't need to do that --- you can just start
using the remaining arguments from `$argv[$OPTIND]` on --- but it's a
pretty good way of doing it.

In the call, I showed a line with \``-``-`' in it: that's the standard
way of telling `getopts` that the options are finished; even if later
words start with a `-`, they are not treated as options. However,
`getopts` stops anyway when it reaches a word not beginning with \``-`',
so that wasn't necessary here. But it works anyway.

You can do all of what `getopts` does without *that* much difficulty
with a few extra lines of shell programming, of course. The best
argument for using `getopts` is probably that it allows you to group
single-letter options, e.g. \``-abc`' is equivalent to \``-a -b -c`' if
none of them was defined to have an argument. In this case, `getopts`
has to remember the position *inside* the word on the command line for
you to call it next, since the \``a`' \``b`' and \``c`' still appear on
different calls. Rather unsatisfactorily, this is hidden inside the
shell (as it is in other shells --- we haven't fixed *all* of everybody
else's bad design decisions); you can't get at it or reset it without
altering `$OPTIND`. But if you read the small print at the top of the
guide, you'll find I carefully avoided saying everything was
satisfactory.

While we're at it, why do blocks starting with \``if`' and \``then`' end
with \``fi`', and blocks starting with \``case`' end with \``esac`',
while those starting with \``while`' and \``do`' end with \``done`', not
\``elihw`' (perfectly pronounceable in Welsh, after all) or \``od`'?
Don't ask me.

### 3.2.14: Random file control things

We're now down into the odds and ends. If you know UNIX at all, you will
already be familiar with the `umask` command and its effect on file
creation, but as it is a builtin I will describe it here. Create a file
and look at it:

      % touch tmpfile
      % ls -l tmpfile
      -rw-r--r--    1 pws   pws    0 Jul 19 21:19 tmpfile

(I've shortened the output line for the TeX version of this document.)
You'll see that the permissions are read for everyone, write-only for
the owner. How did the command (`touch`, not a builtin, creates an empty
file if there was none, or simply updates the modification time of an
existing file) know what permissions to set?

      % umask
      022
      % umask 077
      % rm tmpfile; touch tmpfile
      % ls -l tmpfile
      -rw-------    1 pws   pws    0 Jul 19 21:22 tmpfile

`umask` was how it knew. It gives an octal number corresponding to the
permissions which should *not* be given to a newly created file (only
newly created files are affected; operations on existing files don't
involve `umask`). Each digit is made up of a 4 for read, 2 for write, 1
for executed, in the same order that `ls` shows for permissions: user,
then group, then everyone else. (On this Linux/GNU-based system, like
many others, users have their own groups, so both are called \``pws`'.)
So my original \`022' specified that everything should be allowed except
writing for group and other, while \`077' disallowed any operation by
group and other. These are the two most common settings, although here
\`002' or \`007' would be useful because of the way groups are specific
to users, making it easier to grant permission to specific other users
to write in my directories. (Except there aren't any other users.)

You can also use `chmod`-like permission changing expressions in
`umask`. So

      % umask go+rx

would restore group and other permissions for reading and executing,
hence returning the mask to 022. Note that because it is *adding*
permissions, just like `chmod` does, it is *removing* numbers from the
umask.

You might have wondered about execute permissions, since \``touch`'
didn't give any, even where it was allowed by `umask`. That's because
only operations which create executable programmes, such as running a
compiler and linker, set that bit; the normal way of opening a new file
--- internally, the UNIX `open` function, with the `O_CREAT` flag set
--- doesn't touch it. For the same reason, if you create shell scripts
which you want to be able to execute by typing the name, you have to
make them executable yourself:

      % chmod +x myscript

and, indeed, you can think of `chmod` as `umask`'s companion for files
which already exist. It doesn't need to be a builtin, because the files
you are operating on are external to `zsh`; `umask`, on the other hand,
operates when you create a file from within `zsh` or any child process,
so needs to be a builtin. The fact that it's inherited means you can set
`umask` before you start an editor, and files created by that editor
will reflect the permissions.

Note that the value set by `umask` is also inherited and used by
`chmod`. In the example of `chmod` I gave, I didn't see *which* type of
execute permission to add; `chmod` looks at my `umask` and decides based
on that --- in other words, with 022, everybody would be allowed to
execute `myscript`, while with 077, only I would, because of the 1's in
the number: (0+0+0)+(4+2+1)+(4+2+1). Of course, you can be explicit with
chmod and say \``chmod u+x myscript`' and so on.

Something else that may or may not be obvious: if you run a script by
passing it as an argument to the shell,

      % zsh myscript

what matters is *read* permission. That's what the shell's doing to the
script to find the commands, after all. Execute permission applies when
the system (or, in some cases, including zsh, the parent shell where you
typed \``myscript`') has to decide whether to find a shell to run the
script by itself.

### 3.2.15: Don't watch this space, watch some other

Finally for builtins, some things which really belong elsewhere. There
are three commands you use to control the shell's editor. These will be
described in [chapter 4](zshguide04.html#zle), where I talk all about
the editor.

The `bindkey` command allows you to attach a set of keystrokes to a
command. It understands an abbreviated notation for the keystrokes.

      % bindkey '^Xc' copy-prev-word

This binds the keystrokes consisting of `Ctrl` held down with `x`, then
`c`, to the command which copies the previous word on the line to the
current position. The commands are listed in the `zshzle` manual page.
`bindkey` can also do things with keymaps, which are a complete set of
mappings between keys and commands like the one I showed.

The `vared` command is an extremely useful builtin for editing a shell
variable. Usually much the easiest way to change `$path` (or `$PS1`, or
whatever) is to run \``vared path`': note the lack of a \``$`', since
otherwise you would be editing whatever `$path` was expanded to. This is
because very often you want to leave most of what's there and just
change the odd character or word. Otherwise, you would end up doing this
with ordinary parameter substitutions, which are a lot more complicated
and error prone. Editing a parameter is exactly like editing a command
line, except without the prompt at the start.

Finally, there is the `zle` command. This is the most mysterious, as it
offers a fairly low-level interface to the line editor; you use it to
define your own editing commands. So I'll leave this alone for now.

### 3.2.16: And also

There is one more standard builtin that I haven't covered: `zmodload`,
which allows you to manipulate add-on packages for zsh. Many extras are
supplied with the shell which aren't normally loaded to keep down the
use of memory and to avoid having too many rarely used builtins, etc.,
getting in the way. In the last chapter I will talk about some of these.
To be more honest, a lot of the stuff in between actually uses these
addons, generically referred to as modules --- the line editor, zle, is
itself a separate module, though heavily dependent on the main shell ---
and you've probably forgotten I mentioned above using
\``zmodload zsh/mathfunc`' to load mathematical functions.

3.3: Functions
--------------

Now it's time to look at functions in more detail. The various issues to
be discussed are: loading functions, handling parameters, compiling
functions, and repairing bike tyres when the rubber solution won't stick
to the surface. Unfortunately I've already used so much space that I'll
have to skip the last issue, however topical it might be for me at the
moment.

### 3.3.1: Loading functions

Well, you know what happens now. You can define functions on the command
line:

      fn() {
        print I am a function
      }

which you call under the name \``fn`'. As you type, the shell knows that
it's in the middle of a function definition, and prompts you until you
get to the closing brace.

Alternatively, and much more normally, you put a file called `fn`
somewhere in a directory listed in the `$fpath` array. At this point,
you need to be familiar with the `KSH_AUTOLOAD` option described in the
last chapter. From now on, I'm just going to assume your autoloadable
function files contain just the body of the function, i.e.
`KSH_AUTOLOAD` is not set. Then the file `fn` would contain:

      print I am a function

and nothing else.

Recent versions of zsh, since `3.1.6`, set up `$fpath` for you. It
contains two parts, although the second may have multiple directories.
The first is, typically, `/usr/local/share/zsh/site-functions`, although
the prefix may vary. This is empty unless your system administrator has
put something in it, which is what it's there for.

The remaining part may be either a single directory such as
`/usr/local/share/zsh/3.1.9/functions`, or a whole set of directories
starting with that path. It simply depends whether the person installing
zsh wanted to keep all the functions in the same directory, or have them
sorted according to what they do. These directories are full of
functions. However, none of the functions is autoloaded automatically,
so unless you specifically put \``autoload ...`' in a startup file, the
shell won't actually take any notice of them. As you'll see, part of the
path is the shell version. This makes it very easy to keep multiple
versions of zsh with functions which use features that may be different
between the two versions. By the way, if these directories don't exist,
you should check `$fpath` to see if they are in some other location, and
if you can't find any correspondence between what's in `$fpath` and
what's on the disk even when you start the shell with `zsh -f` to
suppress loading of startup files, complain to the system administrator:
he or she has either not installed them properly, or has made
`/etc/zshenv` stomp on `$fpath`, both of which are thoroughly evil
things to do. (`/etc/zshrc`, `/etc/zprofile` and `/etc/zlogin` shouldn't
stomp on `$fpath` either, of course. In fact, they shouldn't do very
much; that's up to the user.)

One point about `autoload` is the \``-U`' option. This turns off the use
of any aliases you have defined when the function is actually loaded ---
the flag is remembered with the name of the function for future
reference, rather than being interpreted immediately by the `autoload`
command. Since aliases can pretty much redefine any command into any
other, and are usually interpreted while a function is being defined or
loaded, you can see that without this flag there is fair scope for
complete havoc.

       alias ls='echo Your ls command has been requisitioned.'
       lsroot() {
         ls -l /
       }
       lsroot

That's not what the function writer intended. (Yes, I know it actually
*is*, because I wrote it to show the problem, but that's not what I
*meant*.) So `-U` is recommended for all standard functions, where you
have no easy way of telling quite what could be run inside.

Recently, the functions for the new completion system (described in
[chapter 6](zshguide06.html#comp)) have been changing the fastest. They
either begin with `comp` or an underscore, \``_`'. If the `functions`
directory is subdivided, most of the subdirectories refer to this. There
are various other classes of functions distributed with the shell:

Functions beginning `zf` are associated with zftp, a builtin system for
FTP transfers. Traditional FTP clients, ones which don't use a graphical
interface, tend to be based around a set of commands on a command line
--- exactly what zsh is good at. This also makes it very easy to write
macros for FTP jobs --- they're just shell functions. This is described
in the final chapter along with other modules. It's based around a
single builtin, `zftp`, which is loaded from the module `zsh/zftp`.

Functions beginning `prompt`, which may be in the `Prompts`
subdirectory, are part of a \`prompt themes' system which makes it easy
for you to switch between preexisting prompts. You load it with
\``autoload -U promptinit; promptinit`'. Then \``prompt -h`' will tell
you what to do next. If you have new completion loaded (with
\``autoload -U compinit; compinit`', what else) the arguments to
\``prompt`' can be listed with `^D` and completed with a TAB; they are
various sorts of prompt which you may or may not like.

Functions with long names and hyphens, like `predict-on` and
`incremental-complete-word`. These are editor functions; you use them
with

      zle -N predict-on
      bindkey <keystroke> predict-on

Here, the `predict-on` function automatically looks back in the history
list for matching lines as you type. You should also bind `predict-off`,
which is loaded when `predict-on` is first called.
`incremental-complete-word` is a fairly simple attempt at showing
possible completions for the current word as you type; it could do with
improving.

Everything else; these may be in the `Misc` subdirectory. These are a
very mixed bag which you should read to see if you like any. One of the
most useful is `zed`, which allows you to edit a small file (it's really
just a simple front-end to `vared`). The `run-help` file shows you the
sort of thing you might want to define for use with the `\eh`
(`run-help`) keystroke. `is-at-least` is a function for finding out if
the version of the shell running is recent enough, assuming you know
what version you need for a given feature. Several of the other
functions refer to the old completion system --- which you won't need,
since you will be reading [chapter 6](zshguide06.html#comp) and using
the new completion system, of course.

If you have your own functions --- and if you use zsh a lot, you almost
certainly will eventually --- it's a good idea to add your own personal
directory to the front of `$fpath`, so that everything there takes
precedence over the standard functions. That allows you to override a
completion function very easily, just by copying it and editing it. I
tend to do something like this in my `.zshenv`:

      [[ $fpath = *pws* ]] || fpath=(~pws/bin/fns $fpath)

to protect against the possibility that the directory I want to add is
already there, in case I source that startup file again, and there are
other similar ways. (You may well find your own account isn't called
`pws`, however.)

Chances are you will always want your own functions to be autoloaded.
There is an easy way of doing this: put it just after the line I showed
above:

      autoload ${fpath[1]}/*(:t)

The `${fpath[1]}/*` expands to all the files in the directory at the
head of the `$fpath` array. The `(:t)` is a \`glob modifier': applied to
a filename generation pattern, it takes the tail (basename) of all the
files in the list. These are exactly the names of the functions you want
to autoload. It's up to you whether you want the `-U` argument here.

### 3.3.2: Function parameters

I covered local parameters in some detail when I talked about `typeset`,
so I won't talk about that here. I didn't mention the other parameters
which are effectively local to a function, the ones that pass down the
arguments to the function, so here is more detail. They work pretty much
identically in scripts.

There are basically two forms. There is the form inherited from Bourne
shell via Korn shell, with the typically uninformative names: `$#`,
`$*`, `$@` and the numerical parameters `$1` etc. --- as high a number
as the shell will parse is allowed, not just single digits. Then there
is the form inherited from the C shell: `$ARGC` and `$argv`. I'll mainly
use the Bourne shell versions, which are far more commonly used, and
come back to some oddities of the C shell variety at the end.

`$#` tells you how many arguments were passed to the function, while
`$*` gives those arguments as an array. This was the only array
available in the Bourne shell, otherwise there would probably have been
a more obvious way of doing it. To get the size and the number of
elements of the array you don't use `${#*}` and `${*[1]}` etc. (well,
you usually don't --- zsh is typically permissive here), you use `$1`,
`$2`. Despite the syntax, these are rather like ordinary array elements;
if you refer to one off the end, you will get an empty string, but no
error, unless you have the option `NO_UNSET` set. It is this not-quite
array which gets shifted if you use the `shift` builtin without an
argument: the old `$1` disappears, the old `$2` becomes `$1`, and so on,
while `$#` is reduced by one. If there were no arguments (`$#` was
zero), nothing happens.

The form `$@` is very similar to `$*`, and you can use it in place of
that in most contexts. There is one place where they differ. Let's
define a function which prints the number of its arguments, then the
arguments.

      args() {
        print $# $*
      }

Now some arguments. We'll do this for the current shell --- it's a
slightly odd idea, that you can set the arguments for what's already
running, or that an interactive shell has arguments at all, but
nonetheless it's true:

      set arguments to the shell
      print $*

sets `$*` and hence prints the message \``arguments to the shell`'. We
now pass *these* arguments on to the function in two different ways:

      args $*
      args $@

This outputs

      4 arguments to the shell
      4 arguments to the shell

-- no surprises so far. Remember, too, that zsh doesn't split words on
spaces unless you ask it too. So:

      % set 'one word'
      % args $*
      1 one word
      % args $@
      1 one word

Now here's the difference:

      % set two words
      % args "$*"
      1 two words
      % args "$@"
      2 two words

In quotes, `"$*"` behaves as a normal array, joining the words with
spaces. However, `"$@"` doesn't --- it still behaves as if it was
unquoted. You can't see from the arguments themselves in this case, but
you can from the digit giving the number of arguments the function has.

This probably seems pretty silly. Why quote something to have it behave
like an unquoted array? The original answer lies back in Bourne shell
syntax, and relates to the vexed question of word splitting. Suppose we
turn on Bourne shell behaviour, and try the example of a word with
spaces again:

      % setopt shwordsplit
      % set 'one word'
      % args $*
      2 one word
      % args $@
      2 one word
      % args "$*"
      1 one word
      % args "$@"
      1 one word

Aha! *This* time `"$@"` kept the single word with the space intact. In
other words, `"$@"` was a slightly primitive mechanism for suppressing
splitting of words, while allowing the splitting of arrays into
elements. In zsh, you would probably quite often use `$*`, not `"$@"`,
safe in the knowledge that nothing was split until you asked it to be;
and if you wanted it split, you would use the special form of
substitution `${=*}` which does that:

      % unsetopt shwordsplit
      % args $*
      1 one word
      % args ${=*}
      2 one word

(I can't tell you why the \``=`' was chosen for this purpose, except
that it consists of two split lines, or in an assignment it splits two
things, or something.) This works with any parameter, whether scalar or
array, quoted or unquoted.

However, that's actually not quite the whole story. There are times when
the shell removes arguments, because there's nothing there:

      % set hello '' there
      % args $*
      2 hello there

The second element of the array was empty, as if you'd typed

      2=

--- yes, you can assign to the individual positional parameters
directly, instead of using `set`. When the array was expanded on the
command line, the empty element was simply missed out altogether. The
same happens with all empty variables, including scalars:

      % empty=
      % args $empty
      0

But there are times when you don't want that, any more than you want
word splitting --- you want *all* arguments passed just as you gave
them. This is another side effect of the `"$@"` form.

      % args "$@"
      3 hello there

Here, the empty element was passed in as well. That's why you often find
`"$@"` being used in zsh when wordsplitting is already turned off.

Another note: why does the following not work like the example with
`$*`?

      % args hello '' there
      3 hello there

The quotes were kept here. Why? The reason is that the shell doesn't
elide an argument if there were quotes, even if the result was empty:
instead, it provides an empty string. So this empty string was passed as
the second argument. Look back at:

      set hello '' there

Although you probably didn't think about it at the time, the same thing
was happening here. Only with the `'``'` did we get an empty string
assigned to `$2`; later, this was missed out when passing `$*` to the
function. The same difference occurs with scalars:

      % args $empty
      0
      % args "$empty"
      1

The `$empty` expanded to an empty string in each case. In the first case
it was unquoted and was removed; this is like passing an empty part of
`$*` to a command. In the second case, the quotes stopped that from
being removed completely; this is similar to setting part of `$*` to an
empty string using `''`.

That's all thoroughly confusing the first time round. Here's a table to
try and make it a bit clearer.

                           |   Number of arguments
                           |     if $* contains...
                           |  (two words)
    Expression   Word      |       'one word'
    on line   splitting?   |             empty string
    --------------------------------------------------
    $*             n       |     2     1     0
    $@             n       |     2     1     0
    "$*"           n       |     1     1     1
    "$@"           n       |     2     1     1
                           |                    
    $*             y       |     2     2     0
    $@             y       |     2     2     0
    "$*"           y       |     1     1     1
    "$@"           y       |     2     1     1
                           |                    
    ${=*}          n       |     2     2     0
    ${=@}          n       |     2     2     0
    "${=*}"        n       |     2     2     1
    "${=@}"        n       |     2     2     1

On the left is shown the expression to be passed to the function, and in
the three right hand columns the number of arguments the function will
get if the positional parameters are set to an array of two words, a
single word with a space in the middle, or a single word which is an
empty string (the effect of \``set -- '``'`') respectively. The second
column shows whether word splitting is in effect, i.e. whether the
`SH_WORD_SPLIT` option is set. The first four lines show the normal zsh
behaviour; the second four show the normal sh/ksh behaviour, with word
splitting turned on --- only the case where a word has a space in it
changes, and then only when no quotes are supplied. The final four show
what happens when you use the \``${=..}`' method to turn on word
splitting, for convenience: that's particularly simple, since it always
forces words to be split, even inside quotation marks.

I would recommend that anyone not wedded to the Bourne shell behaviour
use the top set as standard: in particular, \``$*`' for normal array
behaviour with removal of empty items, \``"$@"`' for normal array
behaviour with empty items left as empty items, and \``"$*"`' for
turning arrays into single strings. If you need word-splitting, you
should use \``${=*}`' or \``"${=@}"`' for splitting with/without removal
of empty items (obviously there's no counterpart to the quoted-array
behaviour here). Then keep `SH_WORD_SPLIT` turned off. If you are wedded
to the Bourne shell behaviour, you're on your own.

**It's a bug**\
\

There's a bug in handling of the the form `${1+"$@"}`. This looks rather
an arcane and unlikely combination, but actually it is commonly used to
get around a bug in some versions of the Bourne shell (which is not in
zsh): that `"$@"` generates a single empty argument if there are no
arguments. The form shown tests whether there is a first argument, and
if so substitutes `"$@"`, else it doesn't substitute anything, avoiding
the bug.

Unfortunately, in zsh, when `shwordsplit` is set --- which is the time
you usually run across attempts like this to standardise the way
different shells work --- this will actually cause too much
word-splitting. The way the shell is written at the moment, the embedded
`"$@"` will force extra splitting on spaces inside the arguments. So if
the first argument is \``one word`', and `shwordsplit` is set,
`${1+"$@"}` produces *two* words \``one`' and \``word`'.

Oliver Kiddle spotted a way of getting round this which has been adapted
for use in the GNU autoconf package: in your initialisation code, have

      [ x$ZSH_VERSION != x ] && alias -g '${1+"$@"}'='"$@"'

This uses a global alias to turn `${1+"$@"}` wherever it occurs as a
single word into `"$@"` which doesn't have the problem. Aliasing occurs
so early in processing that the fact that most of the characters have a
special meaning to the shell is irrelevant; the shell behaves as if it
read in `"$@"`. The only catch is that for this to work the script or
function must use *exactly* the character string `${1+"$@"}`, with no
leading or trailing word characters (whitespace, obviously, or
characters which terminate parsing such as \``;`' are all right). Some
day, we may fix the underlying bug, but it's not very easy with the way
the parameter substitution code is written at the moment.

**Parameters inherited from csh**\
\

The final matter is the C-shell syntax. There are two extra variables
but, luckily, there is not much extra in the way of complexity. `$ARGC`
is essentially identical to `$#`, and `$argv` corresponds to `$*`, but
is a real array this time, so instead of `$1` you have `${argv[1]}` and
so on. They use the convention that scalars used by the shell are all
uppercase, while arrays are all lowercase. This feature is probably the
only reason anyone would need these variants. For example,
`${argv[2,-1]}` means all arguments from the second to the last,
inclusive: negative indices count from the end, and a comma indicates a
slice of an array, so that `${argv[1,-1]}` is always the same as the
full array. Otherwise, my advice would be to stick with the Bourne shell
variants, however cryptic they may look at first sight, for the usual
reason that zsh isn't really like the C shell and if you pretend it is,
you will come a cropper sooner or later.

It looks like you're missing `"$@"`, but actually you can do that with
`"${argv[@]}"`. This, like negative indices and slices, works with all
arrays.

There's one slight oddity with `$ARGC` and `$argv`, which isn't really a
deliberate feature of the shell at all, but just in case you run into
it: although the values in them are of course local to functions, the
variables `$ARGC` and `$argv` *themselves* are actually treated like
global variables. That means if you apply a `typeset -g` command to
them, it will affect the behaviour of `$ARGC` and `$argv` in all
functions, even though they have different values. It's probably not a
good idea to rely on this behaviour.

nusubsect(Arguments to all commands work the same)

I've been a little tricky here, because I've been talking about two
levels of functions at once: `$*` and friends as set in the current
function, or even at the top level, as well as how they are passed down
to commands such as my `args` function. Of course, in the second case
the same behaviour applies to all commands, not just functions. What I
mean is, in

      fn() {
        cat $*
        cat "$*"
      }

the \``cat`' command will see the differences in behaviour between the
two calls just as `args` would. That should be obvious.

**It's not a bug**\
\

Let me finally mention again a feature I noted in passing:

      1='first argument'

sets the first command argument for the current shell or function,
independently of any others. People sometimes complain that

      1000000='millionth argument'

suddenly makes the shell use a lot more memory. That's not a bug at all:
you've asked the shell to set the millionth element of an array, but not
any others, so the shell creates an array a million elements long with
the first 999,999 empty, except for any arguments which were already
set. It's not surprising this takes up a lot of memory.

### 3.3.3: Compiling functions

Since version 3.1.7, it has been possible to compile functions to their
internal format. It doesn't make the functions run any faster, it just
reduces their loading time; the shell just has to bring the function
into memory, then it \`runs it as it does any other function. On many
modern computers, therefore, you don't gain a great deal from this. I
have to admit I don't use it, but there are other definite advantages.

Note that when I say \`compiled' I don't mean the way a C compiler, say,
would take a file and turn it into the executable code which the
processor understands; here, it's simply the format that the shell
happens to use internally --- it's useless without a suitable version of
zsh to run it. Also, it's no use thinking you can hide your code from
prying eyes this way, like you can to some extent with an ordinary
compiler (disassembling anything non-trivial from scratch being a
time-consuming job): first of all, ordinary command lines appear inside
the compiled files, except in slightly processed form, and secondly
running \``functions`' on a compiled function which has been loaded will
show you just as much as it would if the function had been loaded
normally.

One other advantage is that you can create \`digest' files, which are
sets of functions stored in a single file. If you often use a large
fraction of those files, or they are small, or you like the function
itself to appear when you run \`functions' rather than a message saying
it hasn't been loaded, then this works well. In fact, you can compile
all the functions in a single directory in one go. You might think this
uses a lot of memory, but often zsh will simply \`memory map' the file,
which means rather than reserving extra main memory for it and reading
it in --- the obvious way of reading files --- it will tell the
operating system to make the file available as if it were memory, and
the system will bring it into memory piece by piece, \`paging' the file
as it is needed. This is a very efficient way of doing it. Actually, zsh
supports both this method and the obvious method of simply reading in
the file (as long as your operating system does); this is described
later on.

A little extra, in case you're interested: if you read in a file
normally, the system will usually reserve space on a disk for it, the
\`swap', and do paging from there. So in this case you still get the
saving of main memory --- this is standard in all modern operating
systems. However, it's not *as* efficient: first of all, you had to read
the file in in the first place. Secondly it eats up swap space, which is
usually a fixed amount of disk, although if you've got enough main
memory, the system probably won't bother allocating swap. Thirdly ---
this is probably the clincher for standard zsh functions on a large
system --- if the file is directly mapped read-only, as it is in this
case, the system only needs one place in main memory, plus the single
original file on disk, to keep the function, which is very much more
efficient. With the other method, you would get multiple copies in both
main memory and (where necessary) swap. This is how the system treats
directly executable programmes like the shell itself --- the data is
specific to each process, but the programme itself can be shared because
it doesn't need to be altered when it's running.

Here's a simple example.

      % echo 'echo hello, world' >hw
      % zcompile hw
      % ls
      hw    hw.zwc
      % rm hw
      % fpath=(. $fpath)
      % autoload hw
      % hw
      hello, world

We created a simple \`hello, world' function, and compiled it. This
produces a file called \``hw.zwc`'. The extension stands for \`Z-shell
Word Code', because it's based on the format of words (integers longer
than a single byte) used internally by the shell. Then we made sure the
current directory was in our `$fpath`, and autoloaded the function,
which ran as expected. We deleted the original file for demonstration
purposes, but as long as the \``.zwc`' file is newer, that will be used,
so you don't need to remove the originals in normal use. In fact, you
shouldn't, because you will lose any comments and formatting information
in it; you can regenerate the function itself with the \``functions`'
command (try it here), but the shell only remembers the information
actually needed to run the commands. Note that the function was in the
zsh autoload format, not the ksh one, in this case (but see below).

**And there's more**\
\

Now some bells and whistles. Remember the `KSH_AUTOLOAD` thing? When you
compile a function, you can specify which format --- native zsh or ksh
emulation --- will be used for loading it next time, by using the option
`-k` or `-z`, instead of the default, which is to examine the option (as
would happen if you were autoloading directly from the file). Then you
don't need to worry about that option. So, for example, you could
compile all the standard zsh functions using \``zcompile -z`' and save
people the trouble of making sure they are autoloaded correctly.

You can also specify that aliases shouldn't be expanded when the files
are compiled by using `-U`: this has roughly the same effect as saying
`autoload -U`, since when the shell comes to load a compiled file, it
will never expand aliases, because the internal format assumes that all
processing of that kind has already been done. The difference in this
case is if you *don't* specify `-U`: then the aliases found when you
compile the file, not when you load the function from it, will be used.

Now digest files. Here's one convenient way of doing it.

      % ls ~/tmp/fns
      hw1   hw2
      % fpath=(~/tmp/fns $fpath)
      % cd ~/tmp
      % zcompile fns fns/*
      % ls
      fns   fns.zwc

We've made a directory to put functions in, `~/tmp/fns`, and stuck some
random files in it. The `zcompile` command, this time, was given several
arguments: a filename to use for the compiled functions, and then a list
of functions to compile into it. The new file, `fns.zwc`, sits in the
same directory where the directory `fns`, found in `$fpath`, is. The
shell will actually search the digest file instead of the directory.
More precisely, it will search both, and see which is the more recent,
and use that as the function. So now

      % autoload hw1
      % hw1
      echo hello, first world

You can test what's in the digest file with:

      % zcompile -t fns
      zwc file (read) for zsh-3.1.9-dev-3
      fns/hw1
      fns/hw2

Note that the names appear as you gave them on the command line, i.e.
with `fns/` in front. Only the basenames are important for autoloading
functions. The note \``(read)`' in the first line means that zsh has
marked the functions to be read into the shell, rather than memory
mapped as discussed above; this is easier for small functions,
particularly if you are liable to remove or alter a file which is
mapped, which will confuse the shell. It usually decides which method to
use based on size; you can force memory mapping by giving the `-M`
option. Memory mapping doesn't work on all systems (currently including
Cygwin).

I showed this for compiling files, but you can actually tell the shell
to output compiled functions --- in other words, it will look along
`$fpath` and compile the functions you specify. I find compiling files
easier, when I do it at all, since then I can use patterns to find them
as I did above. But if you want to do it the other way, you should note
two other options: `-a` will compile files by looking along `$fpath`,
while `-c` will output any functions already loaded by the shell (you
can combine the two to use either). The former is recommended, because
then you don't lose any information which was present in the autoload
file, but not in the function stored in memory ---- this is what would
happen if the file defined some extra widgets (in the non-technical
sense) which weren't part of the function called subsequently.

If you're perfectly happy with the shell *only* searching a digest file,
and not comparing the datestamp with files in the directory, you can put
that directly into your `$fpath`, i.e. `~/tmp/fns.zwc` in this case.
Then you can get rid of the original directory, or archive it somewhere
for reuse.

You can compile scripts, too. Since these are in the same format as a
zsh autoload file, you don't need to do anything different from
compiling a single function. You then run (say) `script.zwc` by typing
\``zsh script`' --- note that you should omit the `.zwc`, as zsh decides
if there's a compiled version of a script by explicitly appending the
suffix. What's more, you can run it using \``.`' or \``source`' in just
the same way (\``. script`') --- this means you can compile your startup
files if you find they take too long to run through; the shell will spot
a `~/.zshrc.zwc` as it would any other sourceable file. It doesn't make
much sense to use the memory mapping method in this case, since once
you've sourced the files you never want to run them again, so you might
as well specify \``zcompile -R`' to use the reading (non-memory-mapping)
method explicitly.

If you ever look inside a `.zwc` file, you will see that the information
is actually included twice. That's because systems differ about the
order in which numbers are stored: some have the least significant byte
first (notably Intel and some versions of Mips) and some the most
significant (notably SPARC and Cambridge Consultants' XAP processor,
which is notable here mainly because I spend my working hours
programming for it --- you can't run zsh on it). Since zsh uses integers
a great deal in the compiled code, it saves them in both possible orders
for ease of use. Why not just save it for the machine where you compiled
it? Then you wouldn't be able to share the files across a heterogeneous
network --- or even worse, if you made a distribution of compiled files,
they would work on some machines, and not on others. Think how Emacs
users would complain if the `.elc` files that arrived weren't the right
ones. (Worse, think how the vi users would laugh.) The shell never reads
or maps in the version it doesn't use, however; only extra disk space is
used.

**A little -Xtra help**\
\

There are two final autoloading issues you might want to know about. In
versions of zsh since 3.1.7, you will see that when you run `functions`
on a function which is marked for autoload but hasn't yet been loaded,
you get:

    afunctionmarkedforautoloadwhichhasntbeenloaded () {
            # undefined
            builtin autoload -XU
    }

The \``# undefined`' is just printed to alert you that this was a
function marked as autoloadable by the `autoload` command: you can tell,
because it's the only time `functions` will emit a comment (though there
might be other \``#`' characters around). What's interesting is the
`autoload` command with the `-X` option. That option means \`Mark me for
autoloading and run me straight away'. You can actually put it in a
function yourself, and it will have the same effect as running
\``autoload`' on a not-yet-existent function. Obviously, the `autoload`
command will disappear as soon as you do run it, to be replaced by the
real contents. If you put this inside a file to be autoloaded, the shell
will complain --- the alternative is rather more unpalatable.

Note also the `-U` option was set in that example: that simply means
that I used `autoload` with the `-U` option when I originally told the
shell to autoload the function.

There's another option, `+X`, the complete opposite of `-X`. This one
can *only* be used with autoload outside the function you're loading,
just as `-X` was only meaningful inside. It means \`load the file
immediately, but don't run it', so it's a more active (or, as they say
nowadays, since they like unnecessarily long words, proactive) form of
`autoload`. It's useful if you want to be able to run the `functions`
command to see the function, but don't want to run the function itself.

**Special functions**\
\

I'm in danger of simply quoting the manual, but there are various
functions with a special meaning to the shell (apart from `TRAP...`
functions, which I've already covered). That is, the functions
themselves are perfectly normal, but the shell will run them
automatically on certain occasions if they happen to exist, and silently
skip them if they don't.

The two most frequently used are `chpwd` and `precmd`. The former is
called whenever the directory changes, either via `cd`, or `pushd`, or
an `AUTO_CD` --- you could turn the first two into functions, and avoid
needing `chpwd` but not the last. Here's how to force an xterm, or a
similar windowing terminal, to put the current directory into the title
bar.

      chpwd() {
        [[ -t 1 ]] || return
        case $TERM in
          (sun-cmd) print -Pn "\e]l%~\e\\"
            ;;
          (*xterm*|rxvt|(dt|k|E)term) print -Pn "\e]2;%~\a"
            ;;
        esac
      }

The first line tests that standard output is really a terminal --- you
don't want to print the string in the middle of a script which is
directing its output to a file. Then we look to see if we have a
`sun-cmd` terminal, which has its own *sui generis* sequence for putting
a string into the title bar, or something which recognises xterm escape
sequences. In either case, the special sequences (a bit like termcap
sequences as discussed for `echotc`) are interpreted by the terminal,
and instead of being printed out cause it to put the string in the
middle into the title bar. The string here is \``%~`': I added the `-P`
option to `print` so it would expand prompt escapes. I could just have
used `$PWD`, but this way has the useful effect of shortening your home
directory, or any other named directory, into `~`-notation, which is a
bit more readable. Of course, you can put other stuff there if you like,
or, if you're really sophisticated, put in a parameter `$HEADER` and
define that elsewhere.

If programmes other than the shell alter what appears in the xterm title
bar, you might consider changing that `chwpd` function to `precmd`. The
function `precmd` is called just before every prompt; in this case it
will restore the title line after every command has run. Some people
make the mistake of using it to set up a prompt, but there are enough
ways of getting varying information into a fixed prompt string that you
shouldn't do that unless you have *very* odd things in your prompt. It's
a big nuisance having to redefine `precmd` to alter your prompt ---
especially if you don't know it's there, since then your prompt
apparently magically returns to the same format when you change it.
There are some good reasons for using `precmd`, too, but most of them
are fairly specialised. For example, on one system I use it to check if
there is new input from a programme which is sending data to the shell
asynchronously, and if so printing it out onto the terminal. This is
pretty much what happens with job control notification if you don't have
the `NOTIFY` option set.

The name `precmd` is a bit of a misnomer: `preprompt` would have been
better. It usurps the name more logically applied to the function
actually called `preexec`, which is run after you finished editing a
command line, but just before the line is executed. `preexec` has one
additional feature: the line about to be executed is passed down as an
argument. You can't alter what's going to be executed by editing the
parameter, however: that has been suggested as an upgrade, but it would
make it rather easy to get the shell into a state where you can't
execute any commands because `preexec` always messes them up. It's
better, where possible, to write function front-ends to specific
commands you want to handle specially. For example, here's my `ls`
function:

      local ls
      if [[ -n $LS_COLORS ]]; then
        ls=(ls --color=auto)
      else
        ls=(ls -F)
      fi
      command $ls $*

This handles GNU and non-GNU versions of ls. If `$LS_COLORS` is set, it
assumes we are using GNU ls, and hence colouring (or colorizing, in
geekspeak) is available. Otherwise, it uses the standard option `-F` to
show directories and links with a special symbol. Then it uses `command`
to run the real `ls` --- this is a key thing to remember any time you
use a function front-end to a command. I could have done this another
way: test in my initialisation files which version of `ls` I was using,
then alias `ls` to one of the two forms. But I didn't.

Apart from the trap functions, there is one remaining special function.
It is `periodic`, which is executed before a prompt, like `precmd`, but
only every now and then, in fact every `$PERIOD` seconds; it's up to you
to set `$PERIOD` when you defined `periodic`. If `$PERIOD` isn't set, or
is zero, nothing happens. Don't get `$PERIOD` confused with `$SECONDS`,
which just counts up from 0 when the shell starts.

3.4: Aliases
------------

Aliases are much simpler than functions. In the C shell and its
derivatives, there are no functions, so aliases take their place and can
have arguments, which involve expressions rather like those which
extract elements of previous history lines with \``!`'. Zsh's aliases,
like ksh's, don't take arguments; you have to use functions for that.
However, there are things aliases can do which functions can't, so
sometimes you end up using both, for example

      zfget() {
        # function to retrieve a file by FTP,
        # using globbing on the remote host
      }
      alias zfget='noglob zfget'

The function here does the hard work; this is a function from the zftp
function suite, supplied with the shell, which retrieves a file or set
of files from another machine. The function allows patterns, so you can
retrieve an entire directory with \``zfget *`'. However, you need to
avoid the \``*`' being expanded into the set of files in the current
directory on the machine you're logged into; this is where the alias
comes in, supplying the \``noglob`' in front of the function. There's no
way of doing this with the function alone; by the time the function is
called, the \``*`' would already have been expanded. Of course you could
quote it, but that's what we're trying to avoid. This is a common reason
for using the alias/function combination.

Remember to include the \``=`' in alias definition, necessary in zsh,
unlike csh and friends. If you do:

      alias zfget noglob zfget

they are treated as a list of aliases. Since none has the \``=`' and a
definition, the shell thinks you want to list the definitions of the
listed words; I get the output

      zfget='noglob zfget'
      zfget='noglob zfget'

since `zfget` was aliased as before, but `noglob` wasn't aliased and was
skipped, although the failed alias lookup caused status 1 to be
returned. Remember that the `alias` command takes as many arguments as
you like; any with \``=`' is a definition, any without is a request to
print the current definition.

Aliases can in fact be allowed to expand to almost anything the shell
understands, not just sets of words. That's because the text retrieved
from the alias is put back into the input, and reread more or less as if
you'd typed it. That means you can get away with strange combinations
like

      alias tripe="echo foo | sed 's/foo/bar/' |"
      tripe cat

which is interpreted exactly the same way as

      echo foo | sed 's/foo/bar/' | cat

where the word \``foo`' is sent to the stream editor, which alters it to
\``bar`' (\``s/old/new/`' is `sed`'s syntax for a substitution), and
passes it on to \``cat`', which simply dumps the output. It's useless,
of course, but it does show what can lurk behind an apparently simple
command if it happens to be an alias. It is usually not a good idea to
do this, due to the potential confusion.

As the manual entry explains, you can prevent an alias from being
expanded by quoting it. This isn't like quoting any other expansion,
though; there's no particular important character which has to be
interpreted literally to stop the expansion. The point is that because
aliases are expanded early on in processing of the command line, looking
up an alias is done on a string without quotes removed. So if you have
an alias \``drivel`', none of the strings \``\drivel`', \``'d'rivel`',
or \``drivel""`' will be expanded as the alias: they all would have the
same effect as proper commands, after the quotes are removed, but as
aliases they appear different. The manual entry also notes that you can
actually make aliases for any of these special forms, e.g.
\``alias '\drivel'=...`' (note the quotes, since you need the backslash
to be passed down to the alias command). You would need a pretty good
reason to do so.

Although my \``tripe`' example was silly, you know from the existence of
\`precommand modifiers' that it's sometimes useful to have a special
command which precedes a command line, like `noglob` or the non-shell
command `nice`. Since they have commands following, you would probably
expect aliases to be expanded there, too. But this doesn't work:

      % alias foo='echo an alias for foo'
      % noglob foo
      zsh: command not found: foo

because the `foo` wasn't in command position. The way round this is to
use a special feature: aliases whose definitions end in a space force
the next word along to be looked up as a possible alias, too:

      % alias noglob='noglob '
      % noglob foo
      an alias for foo

which is useful for any command which can take a command line after it.
This also shows another feature of aliases: unlike functions, they
remember that you have already called an alias of a particular name, and
don't look it up again. So the \``noglob`' which comes from expanding
the alias is not treated as an alias, but as the ordinary precommand
modifier.

You may be a little mystified about this difference. A simple answer is
that it's useful that way. It's sometimes useful for functions to call
themselves; for example if you are handling a directory hierarchy in one
go you might get a function to examine a directory, do something for
every ordinary file, and for every directory file call itself with the
new directory name tacked on. Aliases are too simple for this to be a
useful feature. Another answer is that it's particularly easy to mark
aliases as being \`in use' while they are being expanded, because it
happens while the strings inside them are being examined, before any
commands are called, where things start to get complicated.

Lastly, there are \`global aliases'. If aliases can get you into a lot
of trouble, global aliases can get you into a lot of a lot of trouble.
They are defined with the option `-g` and are expanded not just in
command position, but anywhere on the command line.

      alias -g L='| less'
      echo foo L

This turns into \``echo foo | less`'. It's a neat trick if you don't
mind your command lines having only a minimal amount to do with what is
actually executed.

I already pointed out that alias lookups are done so early that aliases
are expanded when you define functions:

      % alias hello='echo I have been expanded'
      % fn() {
      function>  hello
      function> }
      % which fn
      fn () {
              echo I have been expanded
      }

You can't stop this when typing in functions directly, except by quoting
part of the name you type. When autoloading, the `-U` option is
available, and recommended for use with any non-trivial function.

A brief word about that \``function>`' which appears to prompt you while
you are editing a function; I mentioned this in the previous chapter but
here I want to be clearer about what's going on. While you are being
prompted like that, the shell is not actually executing the commands you
are typing in. Only when it is satisfied that it has a complete set of
commands will it go away and execute them (in this case, defining the
function). That means that it won't always spot errors until right at
the end. Luckily, zsh has multi-line editing, so if you got it wrong you
should just be able to hit up-arrow and edit what you typed; hitting
return will execute the whole thing in one go. If you have redefined
`$PS2` (or `$PROMPT2`), or you have an old version of the shell, you may
not see the full prompt, but you will usually see something ending in
\``>`' which means the same.

3.5: Command summary
--------------------

As a reminder, the shell looks up commands in this order:

aliases, which will immediately be interpreted again as texts for
commands, possible even other aliases; they can be deleted with
\``unalias`',

reserved words, those special to the shell which often need to be
interpreted differently from ordinary commands due to the syntax,
although they can be disabled if you really need to,

functions; these can also be disabled, although it's usually easier to
\``unfunction`' them,

builtin commands, which can be disabled, or called as a builtin by
putting \``builtin`' in front,

external commands, which can be called as such, even if the name clashes
with one of the above types, by putting \``command`' in front.

3.6: Expansions and quotes
--------------------------

As I keep advertising, there will be a whole chapter dedicated to the
subject of shell expansions and what to do with them. However, it's a
rather basic subject, which definitely comes under the heading of basic
shell syntax, so I shall here list all the forms of expansion. As given
in the manual, there are five stages.

### 3.6.1: History expansion

This is the earliest, and is only done on an interactive command line,
and only if you have not set `NO_BANG_HIST`. It was described in the
section \`*The history mechanism; types of history*' in the previous
chapter. It is almost independent of the shell's processing of the
command line; it takes place as the command line is read in, not when
the commands are interpreted. However, in zsh it is done late enough
that the \``!`'s can be quoted by putting them in single quotes:

      echo 'Hello!!'

doesn't insert the previous line at that point, but

      echo "Hello!!"

does. You can always quote active \``!`'s with a backslash, so

      echo "Hello\!\!"

works, with or without the double quotes. Amusingly, since single quotes
aren't special in double quotes, if you set the `HIST_VERIFY` option,
which puts the expanded history line back on the command line for
possible further editing, and try the first two of the three
possibilities above in order, then keep hitting return, you will find
ever increasing command lines:

      % echo 'Hello!!'
      Hello!!
      % echo "Hello!!"
      % echo "Helloecho 'Hello!!'"
      % echo "Helloecho 'Helloecho 'Hello!!''"
      % echo "Helloecho 'Helloecho 'Helloecho 'Hello!!'''"

and if you understand why, you have a good grasp of how quotes work.

There's another way of quoting exclamation marks in a line: put a
\``!"`' in it. It can appear anywhere (as long as it's not in single
quotes) and will be removed from the line, but it has the effect of
disabling any subsequent exclamation marks till the end of the line.
This is the only time quote marks which are significant to the shell
(i.e. are not themselves quoted) don't have to occur in a matching pair.

Note that as exclamation marks aren't active in any text read
non-interactively --- and this includes autoloaded functions and sourced
files, such as startup files, read inside interactive shells --- it is
an error to quote any \``!`'s in double quotes in files. This will
simply pass on the backslashes to the next level of parsing. Other forms
of quoting are all right: \``\!`', because any character quoted with a
backslash is treated as itself, and `'!'` because single quotes can
quote anything anyway.

### 3.6.2: Alias expansion

As discussed above, alias expansion also goes on as the command line is
read, so is to a certain extent similar to history expansion. However,
while a history expansion may produce an alias for expansion, \``!`'s in
the text resulting from alias expansions are normal characters, so it
can be thought of as a later phase (and indeed it's implemented that
way).

### 3.6.3: Process, parameter, command, arithmetic and brace expansion

There are a whole group of expansions which are done together, just by
looking at the line constructed from the input after history and alias
expansion and reading it from left to right, picking up any active
expansions as the line is examined. Whenever a complete piece of
expandable text is found, it is expanded; the text is not re-examined,
except in the case of brace expansion, so none of these types of
expansion is performed on any resulting text. Whether later forms of
expansion --- in other words, filename generation and filename expansion
are performed --- is another matter, depending largely on the
`GLOB_SUBST` option as discussed in the previous chapter. Here's a brief
summary of the different types.

**Process substitution**\
\

There are three forms that result in a command line argument which
refers to a file from or to which input or output is taken:
\``<`(*process*)' runs the process which is expected to generate output
which can be used as input by a command; \``>`(*process*)' runs the
process which will take input to it; and \``=`(*process*)' acts like the
first one, but it is guaranteed that the file is a plain file.

This probably sounds like gobbledygook. Here are some simple examples.

      cat < <(echo This is output)

(There are people in the world with nothing better to do than compile
lists of dummy uses of the \``cat`' command, as in that example, and
pour scorn on them, but I'll just have to brave it out.) What happens is
that the command \``echo This is output`' is run, with the obvious
result. That output is *not* put straight into the command line, as it
would be with command substitution, to be described shortly. Instead,
the command line is given a filename which, when read, gets that output.
So it's more like:

      echo This is output >tmpfile
      cat < tmpfile
      rm tmpfile

(note that the temporary file is cleaned up automatically), except that
it's more compact. In this example I could have missed out the remaining
\``<`', since `cat` does the right thing with a filename, but I put it
there to emphasise the fact that if you want to redirect input from the
process substitution you need an *extra* \``<`', over and above the one
in the substitution syntax.

Here's an example for the corresponding output substitution:

      echo This is output > \ 
      >(sed 's/output/rubbish/' >outfile)

which is a perfectly foul example, but works essentially like:

      echo This is output >tmpfile
      sed 's/output/rubbish/' <tmpfile >outfile

There's an obvious relationship to pipes here, and in fact this example
could be better written,

      echo This is output | sed 's/output/rubbish/' >outfile

A good example of an occasion where the output process substitution
can't be replaced by a pipe is when it's on the error output, and
standard output is being piped:

      ./myscript 2> >(grep -v idiot >error.log) |
          process-output >output.log

a little abstract, but here the main point of the script \`myscript' is
to produce some output which undergoes further processing on the
right-hand side of the pipe. However, we want to process the error
output here, by filtering out occurrences of lines which use the word
\`idiot', before dumping those errors into a file `error.log`. So we get
an effect similar to having two pipelines at once, one for output and
one for error. Note again the *two* \``>`' signs present next to one
another to get that effect.

Finally, the \``=`(*process*)' form. Why do we need this as well as the
one with \``<`'? To understand that, you need to know a little of how
zsh tries to implement the latter type efficiently. Most modern
UNIX-like systems have \`named pipes', which are essentially files that
behave like the \``|`' on the command line: one process writes to the
file, another reads from it, and the effect is essentially that data
goes straight through. If your system has them, you will usually find
the following demonstration works:

      % mknod tmpfile p
      % echo This is output >tmpfile &
      [2] 1507
      % read line <tmpfile
      %
      [2]  + 1507 done       echo This is output >> tmpfile
      % print -- $line
      This is output
      %

The syntax to create a named pipe is that rather strange \``mknod`'
command, with \``p`' for pipe. We stick this in the background, because
it won't do anything yet: you can't write to the pipe when there's
no-one to read it (a fundamental rule of pipes which isn't *quite* as
obvious as it may seem, since it *is* possible for data to lurk in the
pipe, buffered, before the process reading from it extracts it), so we
put that in the background to wait for action. This comes in the next
line, where we read from the pipe: that allows the `echo` to complete
and exit. Then we print out the line we've read.

The problem with pipes is that they are just temporary storage spaces
for data on the way through. In particular, you can't go back to the
beginning (in C-speak, \`you can't seek backwards on a pipe') and
re-read what was there. Sometimes this doesn't matter, but some
commands, such as editors, need that facility. As the \``<`' process
substitution is implemented with named pipes (well, maybe), there is
also the \``=`' form, which produces a real, live temporary file,
probably in the \``/tmp`' directory, containing the output from the
file, and then puts the name of that file on the command line. The
manual notes, unusually helpfully, that this is useful with the
\``diff`' command for comparing the output of two processes:

      diff =(./myscript1) =(./myscript2)

where, presumably, the two scripts produce similar, but not identical,
output which you want to compare.

I said \`well, maybe' in that paragraph because there's another way zsh
can do \``<`' process substitutions. Many modern systems allow you to
access a file with a name like \``/dev/fd/0`' which corresponds to file
descriptor 0, in this case standard input: to anticipate the section on
redirection, a \`file descriptor' is a number assigned to a particular
input or output stream. This method allows you to access it as a file;
and if this facility is available, zsh will use it to pass the name of
the file in process substitution instead of using a named pipe, since in
this case it doesn't have to create a temporary file; the system does
everything. Now, if you are really on the ball, you will realise that
this doesn't get around the problem of pipes --- where is data on this
file descriptor going to come from? The answer is that it will either
have to come from a real temporary file --- which is pointless, because
that's what we wanted to avoid --- or from a pipe opened from some
process --- which is equivalent to the named pipe method, except with
just a file descriptor instead of a name. So even if zsh does it this
way, you still need the \``=`' form for programmes which need to go
backwards in what they're reading.

**Parameter substitution**\
\

You've seen enough of this already. This comes from a \``$`' followed
either by something in braces, or by alphanumeric characters forming the
name of the parameter: \``$foo`' or \``${foo}`', where the second form
protects the expansion from any other strings at the ends and also
allows a veritable host of extra things to appear inside the braces to
modify the substitution. More detail will be held over to till [chapter
5](zshguide05.html#subst); there's a lot of it.

**Command substitution**\
\

This has two forms, `$`(*process*) and `` ` ``*process*`` ` ``. They
function identically; the first form has two advantages: substitutions
can be nested, since the end character is different from the start
character, and (because it uses a \``$`') it reminds you that, like
parameter substitutions, command substitutions can take place inside
double-quoted strings. In that case, like most other things in quotes,
the result will be a single word; otherwise, the result is split into
words on any field separators you have defined, usually whitespace or
the null character. I'll use the `args` function again:

      % args() { print $# $*; }
      % args $(echo two words)
      2 two words
      % args "$(echo one word)"
      1 one word

The first form will split on newlines, not just spaces, so an equivalent
is

      % args $(echo two; echo words)
      2 two words

Thus entire screens of text will be flattened out into a single line of
single-word command arguments. By contrast, with the double quotes no
processing is done whatsoever; the entire output is put verbatim into
one command argument, with newlines intact. This means that the quite
common case of wanting a single complete line from a file per command
argument has to be handled by trickery; zsh has such trickery, but
that's the stuff of [chapter 5](zshguide05.html#subst).

Note the difference from process substitution: no intermediate file name
is involved, the output itself goes straight onto the command line. This
form of substitution is considerably more common, and, unlike the other,
is available in all UNIX shells, though not in all shells with the more
modern form \``$`(`...`)'.

The rule that the command line is evaluated only once, left to right, is
adhered to here, but it's a little more complicated in this case since
the expression being substituted is scanned *as a complete command
line*, so can include anything a command usually can, with all the rules
of quoting and expansion being applied. So if you get confused about
what a command substitution is actually up to, you should extract the
commands from it and think of them as a command line in their own right.
When you've worked out what that's doing, decide what it's output will
be, and that's the result of the substitution. You can ignore any error
output; that isn't captured, so will go straight to the terminal. If you
want to ignore it, use the standard trick (see below) \``2>/dev/null`'
*inside* the command substitution --- not on the main command line,
where it won't work because substitutions are performed before
redirection of the main command line, and in any case that will have the
obvious side effect of changing the error output from the command line
itself.

The only real catch with command substitution is that, as it is run as a
separate process --- even if it only involves shell builtins --- no
effects other than the output will percolate back to the main shell:

      % print $(bar=value; print bar is $bar)
      bar is value
      % print bar is $bar
      bar is

There is maybe room for a form of substitution that runs inside the
shell, instead; however, with modern computers the overhead in starting
the extra process is pretty small --- and in any case we seem to have
run out of new forms of syntax.

Once you know and are comfortable with command substitution, you will
probably start using it all the time, so there is one good habit to get
into straight away. A particularly common use is simply to put the
contents of a file onto the command line.

      # Don't do this, do the other.
      process_cmd `cat file_arguments`

But there's a shortcut.

      # Do do this, don't do the other
      process_cmd $(<file_arguments)

It's not only less writing, it's more efficient: zsh spots the special
syntax, with the `<` immediately inside the parentheses, reads the file
directly without bothering to start \``cat`', and inserts its contents:
no external process is involved. You shouldn't confuse this with \`null
redirections' as described below: the syntax is awfully similar,
unfortunately, but the feature shown here is not dependent on that other
feature being enabled or set up in a particular way. In fact, this
feature works in ksh, which doesn't have zsh's null redirections.

You can quote the file-reading form too, of course: in that case, the
contents of the file \``cmd_arguments`' would be passed as just one
argument, with newlines and spaces intact.

Sometimes, the rule about splitting the result of a command substitution
can get you into trouble:

      % typeset foo=`echo words words`
      % print $foo
      words

You probably expected the command substitution *not* to be split here.
but it was, and the shell executed typeset with the arguments
\``foo=words`' and \`words'. That's because in zsh arguments to
`typeset` are treated pretty much normally, except for some jiggery
pokery with tildes described below. Other shells do this differently,
and zsh (from 4.0.2 and 4.1.1) provides a compatibility option,
`KSH_TYPESET`. In earlier versions you need to use quotes:

      % typeset foo="`echo words words`"
      % print $foo
      words words

A really rather technical afterword: using \``$(cat file_arguments)`',
you might have counted two extra processes to be started, one being the
usual one for a command substitution, and another the \``cat`' process,
since that's an external command itself. That would indeed be the
obvious way of doing it, but in fact zsh has an optimisation in cases
like this: if it knows the shell is about to exit --- in this case, the
forked process which is just interpreting the command line for the
substitution --- it will not bother to start a new process for the last
command, and here just replaces itself with the `cat`. So actually
there's only one extra process here. Obviously, an interactive shell is
never replaced in this way, since clairvoyance is not yet a feature of
the shell.

**Arithmetic substitution**\
\

Arithmetic substitution is easy to explain: everything I told you about
the `(( ... ))` command under numerical parameters, above, applies to
arithmetic substitution. You simply bang a \``$`' in front, and it
becomes an expansion.

      % print $(( 32 + 2 * 5 ))
      42

You can perform everything inside arithmetic substitution that you can
inside the builtin, including assignments; the only difference is that
the status is not set, instead the value is put directly onto the
command line in place of the original expression. As in C, the value of
an assignment is the value being assigned, \``$(( param = 3 + 2))`'
substitutes the value 5 as well as assigning it to `$param`.

By the way, there's an extra level of substitution involved in all
arithmetic expansions, since scalar parameters are subject to arithmetic
expansion when they're read in. This is simple if they only contain
numbers, but less obvious if they contain complete expressions:

      % foo=3+5
      % print $(( foo + 2))
      10

The foo was evaluated into 8 before it was substituted in. Note this
means there were two evaluations: this doesn't work:

      % foo=3+
      % print $(( foo 2 ))
      zsh: bad math expression: operand expected at `'

--- the complaint here is about the missing operand after the \``+`' in
the `$foo`. However the following *does* work:

      % foo=3+
      % print $(( $foo 2 ))
      5

That's because the scalar `$foo` is turned into `3+` first. This is more
logical than you might think: with the rule about left to right
evaluation, the `$foo` is picked up inside the `$((...))` and expanded
as an ordinary parameter substitution while the argument of `$((...))`
is being scanned. Then the complete argument \``3+ 2`' is expanded as an
arithmetical expression. (Unfortunately, zsh isn't always this logical;
there could easily be cases where we haven't thought it through --- you
should feel free to bring these to our attention.)

There's an older form with single square brackets instead of double
parentheses; there is now no reason to use it, as it's non-standard, but
you may sometimes still meet it.

**Brace expansion**\
\

Brace expansion is a feature acquired from the C shell and it's
relatives, although some versions of ksh have it, as it's a compile time
option there. It's a useful way of saving you from typing the same thing
twice on a single command line:

      % print -l {foo,bar}' is used far too often in examples'
      foo is used far too often in examples
      bar is used far too often in examples

\``print`' is given two arguments which it is told to print out one per
line. The text in quotes is common to both, but one has \``foo`' in
front, while the other has \``bar`' in front. The brace expression can
equally be in the middle of an argument: for example, a common use of
this among programmers is for similarly named source files:

      % print zle_{tricky,vi,word}.c
      zle_tricky.c zle_vi.c zle_word.c

As you see, you're not limited to two; you can have any number. You can
quote a comma if you need a real one:

      % print -l \`{\,,.}\'' is a punctuation character'
      `,' is a punctuation character
      `.' is a punctuation character

The quotes needed quoting with a backslash to get them into the output.
The second comma is the active one for the braces.

You can nest braces. Once again, this is done left to right. In

      print {now,th{en,ere{,abouts}}}

the first argument of the outer brace is \``now`', and the second is
\``th{en,ere{,abouts}}`'. This brace expands to \``then`' and then the
expansion of \``there{,abouts}`', which is \``there thereabouts`' ---
there's nothing to stop you having an empty argument. Putting this all
together, we have

      print now then there thereabouts

There's more to know about brace expansion, which will appear in
[chapter 5](zshguide05.html#subst) on clever expansions.

### 3.6.4: Filename Expansion

It's a shame the names \`filename expansion' and \`filename generation'
sound so similar, but most people just refer to \``~` and `=` expansion'
and \`globbing' respectively, which is all that is meant by the two. The
first is by far the simpler. The rule is: unquoted \``~`'s at the
beginning of words perform expansion of named directories, which may be
your home directory:

      % print ~
      /home/pws

some user's home directory:

      % print ~root
      /root

(that may turn up \``/`' on your system), a directory named directly by
you:

      % t=/tmp
      % print ~t
      /tmp

a directory you've recently visited:

      % pwd
      /home/pws/zsh/projects/zshguide
      % print ~+
      /home/pws/zsh/projects/zshguide
      % cd /tmp
      % print ~-
      /home/pws/zsh/projects/zshguide

or a directory in your directory stack:

      % pushd /tmp
      % pushd ~
      % pushd /var/tmp
      % print ~2
      /tmp

These forms were discussed above. There are various extra rules. You can
add a \``/`' after any of them, and the expansions still take place, so
you can use them to specify just the first part of a longer expression
(as you almost certainly have done with a simple \``~`'). If you quote
the \``~`' in any of the ways quoting normally takes place, the
expansion doesn't happen.

A `~` in the middle of the word means something completely different, if
you have the `EXTENDED_GLOB` option set; if you don't, it doesn't mean
anything. There are a few exceptions here; assignments are a fairly
natural one:

      % foo=~pws
      % print $foo
      /home/pws

(note that the \``~pws`', being unquoted, was expanded straight away at
the assignment, not at the print statement). But the following works
too:

      % PATH=$PATH:~pws/bin

because colons are special in assignments. Note that this happens even
if the variable isn't a colon-separated path; the shell doesn't know
what use you're going to make of all the different variables.

The companion of \``~`' is \``=`', which again has to occur at the start
of a word or assignment to be special. The remainder of the word (here
the *entire* remainder, because directory paths aren't useful) is taken
as the name of an external command, and the word is expanded to the
complete path to that command, using `$PATH` just as if the command were
to be executed:

      % print =ls
      /bin/ls

and, slightly confusingly,

      % foo==ls
      % print $foo
      /bin/ls

where the two \``=`'s have two different meanings. This form is useful
in a number of cases. For example, you might want to look at or edit a
script which you know is in your path; the form

      % vi =scriptname

is more convenient than the more traditional

      % vi `whence -p ls`

where I put the \``-p`' in to force `whence` to follow the path,
ignoring builtins, functions, etc. This brings us to another use for
\``=`' expansion,

      % =ls

is a neat and extremely short way of referring to an external command
when `ls` is usually a function. It has some of the same effect as
\``command ls`', but is easier to type.

In versions up to and including `4.0`, this syntax will also expand
aliases, so you need to be a bit careful if you really want a path to an
external command:

      % alias foo='ls -F'
      % print =foo
      ls -F

(Path expansion is done in preference, so you are safe if you use `ls`,
unless your `$PATH` is strange.) Putting \``=foo`' at the start of the
command line doesn't work, and the reason why bears examination:
`=`-expansion occurs quite late on, after ordinary alias expansion and
word splitting, so that the result is the single word \``ls -F`', where
the space is part of the word, which probably doesn't mean anything (and
if it does, don't lend me your computer when I need something done in a
hurry). It's probably already obvious that alias expansion here is more
trouble than it's worth. A less-than-exhaustive search failed to find
anyone who liked this feature, and it has been removed from the shell
from 4.1, so that \``=`'-expansion now only expands paths to external
commands.

If you don't like `=`-expansion, you can turn it off by setting the
option `NO_EQUALS`. One catch, which might make you want to do that, is
that the commands `mmv`, `mcp` and `mln`, which are a commonly used
though non-standard piece of free software, use \``=`' followed by a
number to replace a pattern, for example

      mmv '*.c' '=1.old.c'

renames all files ending with `.c` to end with `.old.c`. If you were not
alert, you might forget to quote the second word. Otherwise, however,
`=`' isn't very common at the start of a word, so you're probably fairly
safe. For a way to do that with zsh patterns, see the discussion of the
function `zmv` below (the answer is \``zmv '(*).c' '$1.old.c'`').

Note that zsh is smart enough to complete the names of commands after an
\``=`' of the expandable sort when you hit TAB.

### 3.6.5: Filename Generation

Filename generation is exactly the same as \`globbing': the expanding of
any unquoted wildcards to match files. This is only done in one
directory at a time. So for example

      print *.c

won't match files in a subdirectory ending in \``.c`'. However, it *is*
done on all parts of a path, so

      print */*.c

will match all \``.c`' files in all immediate subdirectories of the
current directory. Furthermore, zsh has an extension --- one of its most
commonly used special features --- to match files in any subdirectory at
any depth, including the current directory: use two \``*`'s as part of
the path:

      print **/*.c

will match \``prog.c`', \``version1/prog.c`', \``version2/test/prog.c`',
\``oldversion/working/saved/prog.c`', and so on. I will talk about
filename generation and other uses of zsh's extremely powerful patterns
at much greater length in [chapter 5](zshguide05.html#subst). My main
thrust here is to fit it into other forms of expansion; the main thing
to remember is that it comes last, after everything has already been
done.

So although you would certainly expect this to work,

      print ~/*

generating all files in your home directory, you now know why: it is
first expanded to \``/home/pws/*`' (or wherever), then the shell scans
down the path until it finds a pattern, and looks in the directory it
has reached (`/home/pws`) for matching files. Furthermore,

      foo=~/
      print $foo*

works. However, as I explained in the last chapter, you need to be
careful with

      foo=*
      print ~/$foo

This just prints \``/home/pws/*`'. To get the \``*`' from the parameter
to be a wildcard, you need to tell the shell explicitly that's what you
want:

      foo=*
      print ~/${~foo}

As also noted, other shells do expand the `*` as a wildcard anyway. The
zsh attitude here, as with word splitting, is that parameters should do
exactly what they're told rather than waltz off generating extra words
or expansions.

Be even more careful with arrays:

      foo=(*)

will expand the `*` immediately, in the current directory --- the
elements of the array assignment are expanded exactly like a normal
command line glob. This is often very useful, but note the difference
from scalar assignments, which do other forms of expansion, but not
globbing.

I'll mention a few possible traps for the unwary, which might confuse
you until you are a zsh globbing guru. Firstly, parentheses actually
have two uses. Consider:

      print (foo|bar)(.)

The first set of parentheses means \`match either `foo` or `bar`'. If
you've used `egrep`, you will probably be familiar with this. The
second, however, simply means \`match only regular files'. The \``(.)`'
is called a \`globbing qualifier', because it limits the scope of any
matches so far found. For example, if either or both of `foo` and `bar`
were found, but were directories, they would not now be matched. There
are many other possibilities for globbing qualifiers. For now, the
easiest way to tell if something at the end is *not* a globbing
qualifier is if it contains a \``|`'.

The second point is about forms like this:

      print file-<1-10>.dat

The \``<`' and \``>`' smell of redirection, as described next, but
actually the form \``<`', optional start number, \``-`', optional finish
number, \``>`' means match any positive integer in the range between the
two numbers, inclusive; if either is omitted, there is no limit on that
end, hence the cryptic but common \``<->`' to match any positive integer
--- in other words, any group of decimal digits (bases other than ten
are not handled by this notation). Older versions of the shell allowed
the form \``<>`' as a shorthand to match any number, but the overlap
with redirection was too great, as you'll see, so this doesn't work any
more.

Another two cryptic symbols are the two that do negation. These only
work with the option \``EXTENDED_GLOB`' set: this is necessary to get
the most out of zsh's patterns, but it can be a trap for the unwary by
turning otherwise innocuous characters into patterns:

      print ^foo

This means any file in the current directory *except* the file `foo`.
One way of coming unstuck with \``^`' is something like

      stty kill ^u

where you would hope \``^u`' means control with \``u`', i.e. ASCII
character 21. But it doesn't, if `EXTENDED_GLOB` is set: it means \`any
file in the current directory except one called \``u`' ', which is
definitely a different thing. The other negation operator isn't usually
so fraught, but it can look confusing:

      print *.c~f*

is a pattern of two halves; the shell tries to match \``*.c`', but
rejects any matches which also match \``f*`'. Luckily, a \``~`' right at
the end isn't special, so

     rm *.c~

removes all files ending in \``.c~`' --- it wouldn't be very nice if it
matched all files ending in \``.c`' and treated the final \``~`' as an
instruction not to reject any, so it doesn't. The most likely case I can
think of where you might have problems is with Emacs' numeric backup
files, which can have a \``~`' in the middle which you should quote.
There is no confusion with the directory use of \``~`', however: that
only occurs at the beginning of a word, and this use only occurs in the
middle.

The final oddments that don't fit into normal shell globbing are forms
with \``#`'. These also require that `EXTENDED_GLOB` be set. In the
simplest use, a \``#`' after a pattern says \`match this zero or more
times'. So \``(foo|bar)#.c`' matches `foo.c`, `bar.c`, `foofoo.c`,
`barbar.c`, `foobarfoo.c`, ... With an extra `#`, the pattern before (or
single character, if it has no special meaning) must match at least
once. The other use of \``#`' is in a facility called \`globbing flags',
which look like \``(#X)`' where \``X`' is some letter, possibly followed
by digits. These turn on special features from that point in the pattern
and are one of the newest features of zsh patterns; they will receive
much more space in [chapter 5](zshguide05.html#subst).

3.7: Redirection: greater-thans and less-thans
----------------------------------------------

Redirection means retrieving input from some other file than the usual
one, or sending output to some other file than the usual one. The
simplest examples of these are \``<`' and \``>`', respectively.

      % echo 'This is an announcement' >tempfile
      % cat <tempfile >newfile
      % cat newfile
      This is an announcement

Here, `echo` sends its output to the file `tempfile`; `cat` took its
input from that file and sent its output --- the same as its input ---
to the file `newfile`; the second `cat` takes its input from `newfile`
and, since its output wasn't redirected, it appeared on the terminal.

The other basic form of redirection is a pipe, using \``|`'. Some people
loosely refer to all redirections as pipes, but that's rather confusing.
The input and output of a pipe are *both* programmes, unlike the case
above where one end was a file. You've seen lots of examples already:

      echo foo | sed 's/foo/bar/'

Here, `echo` sends its output to the programme `sed`, which substitutes
foo by bar, and sends its own output to standard output. You can chain
together as many pipes as you like; once you've grasped the basic
behaviour of a single pipe, it should be obvious how that works:

      echo foo is a word | 
        sed 's/foo/bar/' | 
        sed 's/a word/an unword/'

runs another `sed` on the output of the first one. (You can actually
type it like that, by the way; the shell knows a pipe symbol can't be at
the end of a command.) In fact, a single `sed` will suffice:

      echo foo is a word |
        sed -e 's/foo/bar/' -e 's/a word/an unword/'

has the same effect in this case.

Obviously, all three forms of redirection only work if the programme in
question expects input from standard input, and sends output to standard
output. You can't do:

      echo 'edit me' | vi

to edit input, since `vi` doesn't use the input sent to it; it always
deals with files. Most simple UNIX commands can be made to deal with
standard input and output, however. This is a big difference from other
operating systems, where getting programmes to talk to each other in an
automated fashion can be a major headache.

### 3.7.1: Clobber

The word \`clobber', as in the option `NO_CLOBBER` which I mentioned in
the previous chapter, may be unfamiliar to people who don't use English
as their first language. Its basic meaning is \`hit' or \`defeat' or
\`destroy', as in \`Itchy and Scratchy clobbered each other with
mallets'. If you do:

      % echo first go >file
      % echo second go >file

then `file` will contain only the words \`second go'. The first thing
you put into the file, \`first go', has been clobbered. Hence the
`NO_CLOBBER` option: if this is set, the shell will complain when you
try to overwrite the file. You can use \``>|file`' or \``>! file`' to
override this. You usually can't use \``>!file`' because history
expansion will try to expand \``!file`' before the shell parses the
line; hence the form with the vertical bar tends to be more useful.

### 3.7.2: File descriptors

UNIX-like systems refer to different channels such as input, output and
error by \`file descriptors', which are small integers. Usually three
are special: 0, standard input; 1, standard output; and 2, standard
error. Bourne-like shells (but not csh-like shells) allow you to refer
to a particular file descriptor, instead of standard input or output, by
putting the integer immediately before the \``<`' or \``>`' (no space is
allowed). What's more, if the \``<`' or \``>`' is followed immediately
by \``&`', a file descriptor can follow the redirection (the one before
is optional as usual). A common use is:

      % echo This message will go to standard error >&2

The command sends its message to standard output, file descriptor 1. As
usual, \``>`' redirects standard output. This time, however, it is
redirected not to a file, but to file descriptor 2, which is standard
error. Normally this is the same device as standard output, but it can
be redirected completely separately. So:

      % { echo A message
      cursh> echo An error >&2 } >file
      An error
      % cat file
      A message

Apologies for the slightly unclear use of the continuation prompt
\``cursh>`': this guide goes into a lot of different formats, and some
are a bit finicky about long lines in preformatted text. As pointed out
above, the \``>file`' here will redirect all output from the stuff in
braces, just as if it were a single command. However, the \``>&2`'
inside redirects the output of the second `echo` to standard error.
Since this wasn't redirected, it goes straight to the terminal.

Note the form in braces in the previous example --- I'm going to use
that in a few more examples. It simply sends something to standard
output, and something else to standard error; that's its only use. Apart
from that, you can treat the bit in braces as a black box --- anything
which can produce both sorts of output.

Sometimes you want to redirect both at once. The standard Bourne-like
way of doing this is:

      % { echo A message
      cursh> echo An error >&2 } >file 2>&1

The \``>file`' redirects standard output from the `{`*...*`}` to the
file; the following `2>&1` redirects standard error to wherever standard
output happens to be at that point, which is the same file. This allows
you to copy two file descriptors to the same place. Note that the order
is important; if you swapped the two around, \``2>&1`' would copy
standard error to the initial destination of standard output, which is
the terminal, before it got around to redirecting standard output.

Zsh has a shorthand for this borrowed from csh-like shells:

      % { echo A message
      cursh> echo An error >&2 } >&file

is exactly equivalent to the form in the previous paragraph, copying
standard output and standard error to the same file. There is obviously
a clash of syntax with the descriptor-copying mechanism, but if you
don't have files whose names are numbers you won't run into it. Note
that csh-like shells don't have the descriptor-copying mechanism: the
simple \``>&`' and the same thing with pipes are the only uses of \``&`'
for redirections, and it's not possible there to refer to particular
file descriptors.

To copy standard error to a pipe, there are also two forms:

      % { echo A message
      cursh> echo An error >&2 } 2>&1 | sed -e 's/A/I/'
      I message
      In error
      % { echo A message
      cursh> echo An error >&2 } |& sed -e 's/A/I/'
      I message
      In error

In the first case, note that the pipe is opened before the other
redirection, so that \``2>&1`' copies standard error to the pipe, not
the original standard output; you couldn't put that after the pipe in
any case, since it would refer to the \``sed`' command's output. The
second way is like csh; unfortunately, \``|&`' has a different meaning
in ksh (start a coprocess), so zsh is incompatible with ksh in this
respect.

You can also close a file descriptor you don't need: the form \``2<&-`'
will close standard error for the command where it appears.

One thing not always appreciated about redirections is that they can
occur anywhere on the command line, not just at the end.

      % >file echo foo
      % cat file
      foo

### 3.7.3: Appending, here documents, here strings, read write

There are various other forms which use multiple \``>`'s and \``<`'s.
First,

      % echo foo >file
      % echo bar >>file
      % cat file
      foo
      bar

The \``>``>`' appends to the file instead of overwriting it. Note,
however, that if you use this a lot you may find there are neater ways
of doing the same thing. In this example,

      % { echo foo
      cursh> echo bar } >file
      % cat file
      foo
      bar

Here, \``cursh>`' is a prompt from the shell that it is waiting for you
to close the \``{`' construct which executes a set of commands in the
current shell. This construct can have a redirection applied to the
entire sequence of commands: \``>file`' after the closing brace
therefore redirects the output from both `echo`s.

In the case of input, doubling the sign has a totally different effect.
The word after the `<``<` is not a file, but a string which will be used
to mark in the end of input. Input is read until a line with only this
string is found:

      % sed -e 's/foo/bar/' <<HERE
      heredoc> This line has foo in it.
      heredoc> There is another foo in this one.
      heredoc> HERE
      This line has a bar in it.
      There is another bar in this one.

The shell prompts you with \``heredoc>`' to tell you it is reading a
\`here document', which is how this feature is referred to. When it
finds the final string, in this case \``HERE`', it passes everything you
have typed as input to the command as if it came from a file. The
command in this case is the stream editor, which has been told to
replace the first \``foo`' on each line with a \``bar`'. (Replacing
things with a bar is a familiar experience from the city centre of my
home town, Newcastle upon Tyne.)

So far, the features are standard in Bourne-like shells, but zsh has an
extension to here documents, sometimes referred to as \`here strings'.

      % sed -e 's/string/nonsense/' \ 
      > <<<'This string is the entire document.'
      This nonsense is the entire document.

Note that \``>`' on the second line is a continuation prompt, not part
of the command line; it was just too long for the TeX version of this
document if I didn't split it. This is a shorthand form of \`here'
document if you just want to pass a single string to standard input.

The final form uses both symbols: \``<>file`' opens the file for reading
and writing --- but only on standard input. In other words, a programme
can now both read from and write to standard input. This isn't used all
that often, and when you do use it you should remember that you need to
open standard output explicitly to the same file:

      % echo test >/tmp/redirtest
      % sed 's/e/Z/g' <>/tmp/redirtest 1>&0
      % cat /tmp/redirtest
      tZtst

As standard input (the 0) was opened for writing, you can perform the
unusual trick of copying standard output (the 1) into it. This is
generally not a particularly safe way of doing in-place editing,
however, though it seems to work fine with sed. Note that in older
versions of zsh, \``<>`' was equivalent to \``<->`', which is a pattern
that matches any number; this was changed quite some time ago.

### 3.7.4: Clever tricks: exec and other file descriptors

All Bourne-like shells have two other features. First, the \`command'
`exec`, which I described above as being used to replace the shell with
the command you give after it, can be used with only redirections after
it. These redirections then apply permanently to the shell itself,
rather than temporarily to a single command. So

      exec >file

makes `file` the destination for standard output from that point on.
This is most useful in scripts, where it's quite common to want to
change the destination of all output.

The second feature is that you can use file descriptors which haven't
even been opened yet, as long as they are single digits --- in other
words, you can use numbers 3 to 9 for your own purposes. This can be
combined with the previous feature for some quite clever effects:

      exec 3>&1               
      # 3 refers to stdout
      exec >file
      # stdout goes to `file', 3 untouched
          # random commands output to `file'
      exec 1>&3               
      # stdout is now back where it was
      exec 3>&-
      # file descriptor 3 closed to tidy up

Here, file descriptor 3 has been used simply as a placeholder to
remember where standard output was while we temporarily divert it. This
is an alternative to the \``{`*...*`} >file`' trick. Note that you can
put more than one redirection on the `exec` line: \``exec 3>&1 >file`'
also works, as long as you keep the order the same.

### 3.7.5: Multios

Multios allow you to do an implicit \``cat`' (concatenate files) on
input and \``tee`' (send the same data to different files) on output.
They depend on the option `MULTIOS` being set, which it is by default. I
described this in the last chapter in discussing whether or not you
should have the option set, so you can look at the examples there.

Here's one fact I didn't mention. You use output multios like this:

      command-generating-output >file1 >file2

where the command's output is copied to both files. This is done by a
process forked off by the shell: it simply sits waiting for input, then
copies it to all the files in its list. There's a problem in all
versions of the shell to date (currently 4.0.6): this process is
asynchronous, so you can't rely on it having finished when the shell
starts executing the next command. In other words, if you look at
`file1` or `file2` immediately after the command has finished, they may
not yet contain all the output because the forked process hasn't
finished writing to it.

This is really a bug, but for the time being you will have to live with
it as it's quite complicated to fix in all cases. Multios are most
useful as a shorthand in interactive use, like so much of zsh; in a
script or function it is safer to use `tee`,

      command-generating-output | tee file1 file2

which does the same thing, but as `tee` is handled as a synchronous
process `file1` and `file2` are guaranteed to be complete when the
pipeline exits.

3.8: Shell syntax: loops, (sub)shells and so on
-----------------------------------------------

### 3.8.1: Logical command connectors

I have been rather cavalier in using a couple of elements of syntax
without explaining them:

      true  &&  print Previous command returned true
      false  ||  print Previous command returned false

The relationship between \``&&`' and \``||`' and tests is fairly
obvious, but in this case they connect complete commands, not test
arguments. The \``&&`' executes the following command if the one before
succeeded, and the \``||`' executes the following command if the one
before failed. In other words, the first is equivalent to

      if true; then
        print Previous command returned true
      fi

but is more compact.

There is a perennial argument about whether to use these or not. In the
comp.unix.shell newsgroup on Usenet, you see people arguing that the
\``&&`' syntax is unreadable, and only an idiot would use it, while
other people argue that the full \``if`' syntax is slower and clumsier,
and only an idiot would use that for a simple test; but Usenet is like
that, and both answers are a bit simplistic. On the one hand, the
difference in speed between the two forms is minute, probably measurable
in microseconds rather than milliseconds on a modern computer; the
scheduling of the shell process running the script by the operating
system is likely to make more difference if these are embedded inside a
much longer script or function, as they will be. And on the other hand,
the connection between \``&&`' and a logical \`and' is so strong in the
minds of many programmers that to anyone with moderate shell experience
they are perfectly readable. So it's up to you. I find I use the \``&&`'
and \``||`' forms for a pair of simple commands, but use \``if`' for
anything more complicated.

I would certainly advise you to avoid chains like:

      true || print foo && print bar || false

If you try that, you will see \``bar`' but not \``foo`', which is not
what a C programmer might expect. Using the usual rules of precedence,
you would parse it as: either `true` must be true; or both the `print`
statements must be true; or the false must be true. However, the shell
parses it differently, using these rules:

If you encounter an \``&&`',

if the command before it (really the complete pipeline) succeeded,
execute the command immediately after, and execute what follows normally

else if the command failed, skip the next command and any others until
an \``||`' is encountered, or until the group of commands is ended by a
newline, a semicolon, or the end of an enclosing group. Then execute
whatever follows in the normal way.

If you encounter an \``||`',

if the command before it succeeded, skip the next command and any others
until an \``&&`' is encountered, or until the end of the group, and
execute what follows normally

else if the command failed, execute the command immediately after the
\``||`'.

If that's hard to follow, just note that the rule is completely
symmetric; a simple summary is that the logical connectors don't
remember their past state. So in the example shown, the \``true`'
succeeds, we skip \``print foo`' but execute \``print bar`' and then
skip `false`. The expression returns status zero because the last thing
it executed did so. Oddly enough, this is completely standard behaviour
for shells. This is a roundabout way of saying \`don't use combined
chains of \``&&`'s and \``||`'s unless you think Gödel's theorem is for
sissies'.

Strictly speaking, the and's and or's come in a hierarchy of things
which connect commands. They are above pipelines, which explains my
remark above --- an expression like
\``echo $ZSH_VERSION | sed '/dev//'`' is treated as a single command
between any logical connectors --- and they are below newlines and
semicolons --- an expression like
\``true && print yes; false || print no`' is parsed as two distinct sets
of logically connected command sequences. In the manual, a list is a
complete set of commands executed in one go:

      echo foo; echo bar

      echo small furry animals

--- a shell function is basically a glorified list with arguments and a
name. A sublist is a set of commands up to a newline or semicolon, in
other words a complete expression possibly involving the logical
connectors:

      show -nomoreproc | 
        grep -q foo && 
        print The word '`foo'\' occurs.

A pipeline is a chain of one or more commands connected by \``|`', for
example both individual parts of the previous sublist,

      show -nomoreproc | grep -q foo

and

      print The word '`foo'\' occurs.

count as pipelines. A simple command is one single unit of execution
with a command name, so to use the same example that includes all three
of the following,

      show -nomoreproc
      grep -q foo
      print The word '`foo'\' occurs.

This means that in something like

      print foo

where the command is terminated by a newline and then executed in one
go, the expression is all of the above --- list, sublist, pipeline and
simple command. Mostly I won't need to make the formal distinction; it
sometimes helps when you need to break down a complicated set of
commands. It's a good idea, and usually possible, to write in such a way
that it's obvious how the commands break down. It's not too important to
know the details, as long as you've got a feel for how the shell finds
the next command.

### 3.8.2: Structures

I've shown plenty of examples of one sort of shell structure already,
the `if` statement:

      if [[ black = white ]]; then
        print Yellow is no colour.
      fi

The main points are: the \``if`' itself is followed by some command
whose return status is tested; a \``then`' follows as a new command; any
number of commands may follow, as complex as you like; the whole
sequence is ended by a \``fi`' as a command on its own. You can write
the \``then`' on a new line if you like, I just happen to find it neater
to stick it where it is. If you follow the form here, remember the
semicolon before it; the `then` must start a separate command. (You can
put another command immediately after the `then` without a newline or
semicolon, though, although people tend not to.)

The double-bracketed test is by far the most common thing to put here in
zsh, as in ksh, but any command will do; only the status is important.

      if true; then
        print This always gets executed
      fi
      if false; then
        print This never gets executed
      fi

Here, `true` always returns true (status 0), while `false` always
returns false (status 1 in zsh, although some versions return status 255
--- anything nonzero will do). So the statements following the `print`s
are correct.

The `if` construct can be extended by \``elif`' and \``else`':

      read var
      if [[ $var = yes ]]; then
        print Read yes
      elif [[ $var = no ]]; then
        print Read no
      else
        print Read something else
      fi

The extension is pretty straightforward. You can have as many \``elif`'s
with different tests as you like; the code following the first test to
succeed is executed. If no test succeeded, and there is an \``else`'
(there doesn't need to be), the code following that is executed. Note
that the form of the \``elif`' is identical to that of \``if`',
including the \``then`', while the else just appears on its own.

The `while`-loop is quite similar to `if`. There are two differences:
the syntax uses `while`, `do` and `done` instead of `if`, `then` and
`fi`, and after the loop body is executed (if it is), the test is
evaluated again. The process stops as soon as the test is false. So

      i=0
      while (( i++ < 3 )); do
        print $i
      done

prints 1, then 2, then 3. As with `if`, the commands in the middle can
be any set of zsh commands, so

      i=0
      while (( i++ < 3 )); do
        if (( i & 1 )); then
          print $i is odd
        else
          print $i is even
        fi
      done

tells you that 1 and 3 are odd while 2 is even. Remember that the
indentation is irrelevant; it is purely there to make the structures
more easy to understand. You can write the code on a single line by
replacing all the newlines with semicolons.

There is also an `until` loop, which is identical to the `while` loop
except that the loop is executed until the test is true.
\``until [[`*...*' is equivalent to \``while ! [[`*...*'.

Next comes the `for` loop. The normal case can best be demonstrated by
another example:

      for f in one two three; do
        print $f
      done

which prints out \``one`' on the first iteration, then \``two`', then
\``three`'. The `f` is set to each of the three words in turn, and the
body of the loop executed for each. It is very useful that the words
after the \``in`' may be anything you would normally have on a shell
command line. So \``for f in *; do`' will execute the body of the loop
once for each file in the current directory, with the file available as
`$f`, and you can use arrays or command substitutions or any other kind
of substitution to generate the words to loop over.

The `for` loop is so useful that the shell allows a shorthand that you
can use on the command line: try

      for f in *; print $f

and you will see the files in the current directory printed out, one per
line. This form, without the `do` and the `done`, involves less typing,
but is also less clear, so it is recommended that you only use it
interactively, not in scripts or functions. You can turn the feature off
with `NO_SHORT_LOOPS`.

The `case` statement is used to test a pattern against a series of
possibilities until one succeeds. It is really a short way of doing a
series of `if` and `elif` tests on the same pattern:

      read var
      case $var in
        (yes) print Read yes
              ;;
        (no) print Read no
             ;;
        (*) print Read something else
            ;;
       esac

is identical to the `if`/`elif`/`else` example above. The `$var` is
compared against each pattern in turn; if one matches, the code
following that is executed --- then the statement is exited; no further
matches are looked for. Hence the \``*`' at the end, which can match
anything, acts like the \``else`' of an `if` statement.

Note the quirks of the syntax: the pattern to test must appear in
parentheses. For historical reasons, you can miss out the left
parenthesis before the pattern. I haven't done that mainly because
unbalanced parentheses confuse the system I am using for writing this
guide. Also, note the double semicolon: this is the only use of double
semicolons in the shell. That explains the fact that if you type \``;;`'
on its own the shell will report a \`parse error'; it couldn't find a
`case` to associate it with.

You can also use alternative patterns by separating them with a vertical
bar. Zsh allows alternatives with extended globbing anyway; but this is
actually a separate feature, which is present in other shells which
don't have zsh's extended globbing feature; it doesn't depend on the
`EXTENDED_GLOB` option:

      read var
      case $var in
        (yes|true|1) print Reply was affirmative
                     ;;
        (no|false|0) print Reply was negative
                     ;;
        (*) print Reply was cobblers
                  ;;
      esac

The first \``print`' is used if the value of `$var` read in was
\``yes`', \``true`' or \``1`', and so on. Each of the separate items can
be a pattern, with any of the special characters allowed by zsh, this
time depending on the setting of the option `EXTENDED_GLOB`.

The `select` loop is not used all that often, in my experience. It is
only useful with interactive input (though the code may certainly appear
in a script or function):

      select var in earth air fire water; do
        print You selected $var
      done

This prints a menu; you must type 1, 2, 3 or 4 to select the
corresponding item; then the body of the loop is executed with `$var`
set to the value in the list corresponding to the number. To exit the
loop hit the break key (usually `^G`) or end of file (usually `^D`: the
feature is so infrequently used that currently there is a bug in the
shell that this tells you to use \``exit`' to exit, which is nonsense).
If the user entered a bogus value, then the loop is executed with `$var`
set to the empty string, though the actual input can be retrieved from
`$REPLY`. Note that the prompt printed for the user input is `$PROMPT3`,
the only use of this parameter in the shell: all normal prompt
substitutions are available.

There is one final type of loop which is special to zsh, unlike the
others above. This is \``repeat`'. It can be used two ways:

      % repeat 3 print Hip Hip Hooray
      Hip Hip Hooray
      Hip Hip Hooray
      Hip Hip Hooray

Here, the first word after `repeat` is a count, which could be a
variable as normal substitutions are performed. The rest of the line (or
until the first semicolon) is a command to repeat; it is executed
identically each time.

The second form is a fully fledged loop, just like `while`:

      % repeat 3; do
      repeat> print Hip Hip Hooray
      repeat> done
      Hip Hip Hooray
      Hip Hip Hooray
      Hip Hip Hooray

which has the identical effect to the previous one. The \``repeat>`' is
the shell's prompt to show you that it is parsing the contents of a
\``repeat`' loop.

### 3.8.3: Subshells and current shell constructs

More catching up with stuff you've already seen. The expression in
parentheses here:

      % (cd ~; ls)
      <all the files in my home directory>
      % pwd
      <where I was before, not necessarily ~>

is run in a subshell, as if it were a script. The main difference is
that the shell inherits almost everything from the main shell in which
you are typing, including options settings, functions and parameters.
The most important thing it doesn't inherit is probably information
about jobs: if you run `jobs` in a subshell, you will get no output; you
can't use `fg` to resume a job in a subshell; you can't use
\``kill %`*n*' to kill a job (though you can still use the process ID);
and so on. By now you should have some feel for the effect of running in
a separate process. Running a command, or set of commands, in a
different directory, as in this example, is one quite common use for
this construct. (In zsh 4.1, you can use `jobs` in a subshell; it lists
the jobs running in the parent shell; this is because it is very useful
to be able to pipe the output of jobs into some processing loop.)

On the other hand, the expression in braces here:

      % {cd ~; ls}
      <all the files in my home directory>
      % pwd
      /home/pws

is run in the current shell. This is what I was blathering on about in
the section on redirection. Indeed, unless you need some special effect
like redirecting a whole set of commands, you won't use the
current-shell construct. The example here would behave just the same way
if the braces were missing.

As you might expect, the syntax of the subshell and current-shell forms
is very similar. You can use redirection with both, just as with simple
commands, and they can appear in most places where a simple command can
appear:

      [[ $test = true ]] && {
        print Hello.
        print Well, this is exciting.
      }

That would be much clearer using an \``if`', but it works. For some
reason, you often find expressions of this form in system start-up files
located in the directory `/etc/rc.d` or, on older systems, in files
whose names begin with \``/etc/rc.`'. You can even do:

      if { foo=bar; [[ $foo = bar ]] }; then
        print yes
      fi

but that's also pretty gross.

One use for `{`*...*`}` is to make sure a whole set of commands is
executed at once. For example, if you copy a set of commands from a
script in one window and want them to be run in one go in a shell in
another window, you can do:

       % {
       cursh>            # now paste your commands in here...
        ...
       cursh> }

and the commands will only be executed when you hit return after the
final \``}`'. This is also a workaround for some systems where cut and
paste has slightly odd effects due to the way different states of the
terminal are handled. The current-shell construct is a little bit like
an anonymous function, although it doesn't have any of the usual
features of functions --- you can't pass it arguments, and variables
declared inside aren't local to that section of code.

### 3.8.4: Subshells and current shells

In case you're confused about what happens in the current shell and what
happens in a subshell, here's a summary.

The following are run in the current shell.

1.  All shell builtins and anything which looks like one, such as a
    precommand modifier and tests with \``[[`'.
2.  All complex statements and loops such as `if` and `while`. Tests and
    code inside the block must both be considered separately.
3.  All shell functions.
4.  All files run by \``source`' or \``.`' as well as startup files.
5.  The code inside a \``{`*...*`}`'.
6.  The right hand side of a pipeline: this is guaranteed in zsh, but
    don't rely on it for other shells.
7.  All forms of substitution except `` ` ``*...*`` ` ``, `$`(*...*),
    `=`(*...*), `<`(*...*) and `>`(*...*).

The following are run in a subshell.

1.  All external commands.
2.  Anything on the left of a pipe, i.e. all sections of a pipeline but
    the last.
3.  The code inside a \```(*...*)'.
4.  Substitutions involving execution of code, i.e. `` ` ``*...*`` ` ``,
    `$`(*...*), `=`(*...*), `<`(*...*) and `>`(*...*). (TCL fans note
    that this is different from the \``[`*...*`]`' command substitution
    in that language.)
5.  Anything started in the background with \``&`' at the end.
6.  Anything which has ever been suspended. This is a little subtle:
    suppose you execute a set of commands in the current shell and
    suspend it with `^Z`. Since the shell needs to return you to the
    prompt, it forks a subshell to remember the commands it was
    executing when you interrupted it. If you use `fg` or `bg` to
    restart, the commands will stay in the subshell. This is a special
    feature of zsh; most shells won't let you interrupt anything in the
    current shell like that, though you can still abort it with `^C`.

With an alias, you can't tell where it will be executed --- you need to
find out what it expands too first. The expansion naturally takes place
in the current shell.

Of course, if for some reason the current set of commands is already
running in a subshell, it doesn't get magically returned to the current
shell --- so a shell builtin on the left hand side of a pipeline is
running in a subshell. However, it doesn't get an extra subshell, as an
external command would. What I mean is:

      { print Hello; cat file } |
        while read line; print $line; done

The shell forks, producing a subshell, to execute the left hand side of
the pipeline, and that subshell forks to execute the `cat` external
command, but nothing else in that set of commands will cause a new
subshell to be created.

(For the curious only: actually, that's not quite true, and I already
pointed this out when I talked about command substitutions: the shell
keeps track of occasions when it is in a subshell and has no more
commands to execute. In this case it will not bother forking to create a
new process for the `cat`, it will simply replace the subshell which is
not needed any more. This can only happen in simple cases where the
shell has no clearing up to do.)

3.9: Emulation and portability
------------------------------

I described the options you need to set for compatibility with ksh in
the previous chapter. Here I'm more interested in the best way of
running ksh scripts and functions.

First, you should remember that because of all zsh's options you can't
assume that a piece of zsh code will simply run a piece of sh or ksh
code without any extra changes. Our old friend `SH_WORD_SPLIT` is the
most common problem, but there are plenty of others. In addition to
options, there are other differences which simply need to be worked
around. I will list some of them a bit later. Generally speaking, Bourne
shell is simple enough that zsh emulates it pretty well --- although
beware in case you are using bash extensions, since to many Linux users
bash is the nearest approximation to the Bourne shell they ever come
across. Zsh makes no attempt to emulate bash, even though some of bash's
features have been incorporated.

To make zsh emulate ksh or sh as closely as it knows how, there are
various things you can do.

1.  Invoke zsh under the name sh or ksh, as appropriate. You can do this
    by creating a symbolic link from zsh to sh or ksh. Then when zsh
    starts up all the options will be set appropriately. If you are
    starting that shell from another zsh, you can use the feature of zsh
    that tricks a programme into thinking it has a different name:
    \``ARGV0=sh zsh`' runs zsh under the name sh, just like the symbolic
    link method.
2.  Use \``emulate ksh`' at the top of the script or function you want
    to run. In the case of a function, it is better to run
    \``emulate -L ksh`' since this makes sure the normal options will be
    restored when the function exits; this is irrelevant for a script as
    the options cannot be propagated to the process which ran the
    script. You can also use the option \``-R`' after `emulate`, which
    forces more options to be like ksh; these extra options are
    generally for user convenience and not relevant to basic syntax, but
    in some cases you may want the extra cover provided.

    If it's possible the script may already be running under ksh, you
    can instead use

          [[ -z $ZSH_VERSION ]] && emulate ksh

    or for sh, using the simpler test command there,

          [ x$ZSH_VERSION = x ] && emulate sh

Both these methods have drawbacks, and if you plan to be a heavy zsh
user there's no substitute for simply getting used to zsh's own basic
syntax. If you think there is some useful element of emulation we
missed, however, you should certainly tell the zsh-workers mailing list
about it.

Emulation of ksh88 is much better than emulation of ksh93. Support for
the latter is gradually being added, but only patchily.

There is no easy way of converting code written for any csh-like shell;
you will just have to convert it by hand. See the FAQ for some hints on
converting aliases to functions.

### 3.9.1: Differences in detail

Here are some differences from ksh88 which might prove significant for
ksh programmers. This is lifted straight from the corresponding section
of the FAQ; it is not complete, and indeed some of the \`differences'
could be interpreted as bugs. Those marked \`\*' perform in a ksh-like
manner if the shell is invoked with the name \`ksh', or if \`emulate
ksh' is in effect.

Syntax:

\* Shell word splitting.

\* Arrays are (by default) more csh-like than ksh-like: subscripts start
at 1, not 0; `array[0]` refers to `array[1]`; `$array` refers to the
whole array, not `$array[0]`; braces are unnecessary:
`$a[1] == ${a[1]}`, etc. The `KSH_ARRAYS` option is now available.

Coprocesses are established by `coproc`; `|&` behaves like csh. Handling
of coprocess file descriptors is also different.

In `cmd1 && cmd2 &`, only `cmd2` instead of the whole expression is run
in the background in zsh. The manual implies this is a bug. Use
`{ cmd1 && cmd2 } &` as a workaround.

Command line substitutions, globbing etc.:

\* Failure to match a globbing pattern causes an error (use
`NO_NOMATCH`).

\* The results of parameter substitutions are treated as plain text:
`foo="*"; print $foo` prints all files in ksh but `*` in zsh (unset
`GLOB_SUBST`).

\* `$PSn` do not do parameter substitution by default (use
`PROMPT_SUBST`).

\* Standard globbing does not allow ksh-style \`pattern-lists'. See
[chapter 5](zshguide05.html#subst) for a list of equivalent zsh forms.
The `^`, `~` and `#` (but not `|`) forms require `EXTENDED_GLOB`. From
version 3.1.3, the ksh forms are fully supported when the option
`KSH_GLOB` is in effect.

[1] Note that `~` is the only globbing operator to have a lower
precedence than `/`. For example, `**/foo~*bar*` matches any file in a
subdirectory called `foo`, except where `bar` occurred somewhere in the
path (e.g. `users/barstaff/foo` will be excluded by the `~` operator).
As the `**` operator cannot be grouped (inside parentheses it is treated
as `*`), this is the way to exclude some subdirectories from matching a
`**`.

Unquoted assignments do file expansion after colons (intended for
PATHs).

`integer` does not allow `-i`.

`typeset` and `integer` have special behaviour for assignments in ksh,
but not in zsh. For example, this doesn't work in zsh:

            integer k=$(wc -l ~/.zshrc)
        

because the return value from `wc` includes leading whitespace which
causes wordsplitting. Ksh handles the assignment specially as a single
word.

Command execution:

\* There is no `$ENV` variable (use `/etc/zshrc`, `~/.zshrc`; note also
`$ZDOTDIR`).

`$PATH` is not searched for commands specified at invocation without -c.

Aliases and functions:

The order in which aliases and functions are defined is significant:
function definitions with () expand aliases.

Aliases and functions cannot be exported.

There are no tracked aliases: command hashing replaces these.

The use of aliases for key bindings is replaced by \`bindkey'.

\* Options are not local to functions (use LOCAL\_OPTIONS; note this may
always be unset locally to propagate options settings from a function to
the calling level).

Traps and signals:

\* Traps are not local to functions. The option LOCAL\_TRAPS is
available from 3.1.6.

TRAPERR has become TRAPZERR (this was forced by UNICOS which has
SIGERR).

Editing:

The options `emacs`, `gmacs`, `viraw` are not supported. Use bindkey to
change the editing behaviour: `set -o {emacs,vi}` becomes
`bindkey -{e,v}`; for gmacs, go to emacs mode and use
`bindkey \^t gosmacs-transpose-characters`.

The `keyword` option does not exist and `-k` is instead
interactivecomments. (`keyword` will not be in the next ksh release
either.)

Management of histories in multiple shells is different: the history
list is not saved and restored after each command. The option
`SHARE_HISTORY` appeared in 3.1.6 and is set in ksh compatibility mode
to remedy this.

`\` does not escape editing chars (use `^V`).

Not all ksh bindings are set (e.g. `<ESC>#`; try `<ESC>q`).

\* `#` in an interactive shell is not treated as a comment by default.

Built-in commands:

Some built-ins (`r`, `autoload`, `history`, `integer` ...) were aliases
in ksh.

There is no built-in command newgrp: use e.g.
`alias       newgrp="exec newgrp"`

`jobs` has no `-n` flag.

`read` has no `-s` flag.

Other idiosyncrasies:

`select` always redisplays the list of selections on each loop.

### 3.9.2: Making your own scripts and functions portable

There are also problems in making your own scripts and functions
available to other people, who may have different options set.

In the case of functions, it is always best to put \``emulate -L zsh`'
at the top of the function, which will reset the options to the default
zsh values, and then set any other necessary options. It doesn't take
the shell a great deal of time to process these commands, so try and get
into the habit of putting them any function you think may be used by
other people. (Completion functions are a special case as the
environment is already standardised --- see [chapter
6](zshguide06.html#comp) for this.)

The same applies to scripts, since if you run the script without using
the option \``-f`' to zsh the user's non-interactive startup files will
be run, and in any case the file `/etc/zshenv` will be run. We urge
system administrators not to set options unconditionally in that file
unless absolutely necessary; but they don't always listen. Hence an
`emulate` can still save a lot of grief.

3.10: Running scripts
---------------------

Here are some final comments on running scripts: they apply regardless
of the problems of portability, but you should certainly also be aware
of what I was saying in the previous section.

You may be aware that you can force the operating system to run a script
using a particular interpreter by putting \``#!`' and the path to the
interpreter at the top of the script. For example, a zsh script could
start with

       #!/usr/local/bin/zsh
       print The arguments are $*

assuming that zsh lives in the directory `/usr/local/bin`. Then you can
run the script under its name as if it were an ordinary command. Suppose
the script were called \``scriptfile`' and in the current directory, and
you want to run it with the arguments \``one two forty-three`'. First
you must make sure the script is executable:

      % chmod +x scriptfile

and then you can run it with the arguments:

      % ./scriptfile one two forty-three
      The arguments are one two forty-three

The shell treats the first line as a comment, since it begins with a
\``#`', but note it still gets evaluated by the shell; the system simply
looks inside the file to see if what's there, it doesn't change it just
because the first line tells it to execute the shell.

I put the \``./`' in front to refer to the current directory because I
don't usually have that in my path --- this is for safety, to avoid
running things which happen to have names like commands simply because
they were in the current directory. But many people aren't so paranoid,
and if \``.`' is in your path, you can omit the \``./`'. Hence,
obviously, it can be anywhere else in your path: it is searched for as
an ordinary executable.

The shell actually provides this mechanism even on operating systems
(now few and far between in the UNIX world) that don't have the feature
built into them. The way this works is that if the shell found the file,
and it was executable, but running it didn't work, then it will look for
the `#!`, extract the name following and run (in this example)
\``/usr/local/bin/zsh` *\<path\>*/scriptfile `one two forty-three`',
where *\<path\>* is the path where the file was found. This is, in fact,
pretty much what the system does if it handles it itself.

Some shells search for scripts using the path when they are given as
filenames at invocation, but zsh happens not to. In other words,
\``zsh scriptfile`' only runs `scriptfile` in the current directory.

There are two other features you may want to be aware of. Both are down
to the operating system, if that is what is responsible for the \``#!`'
trick (true of all the most common UNIX-like systems at the moment).
First, you are usually allowed to supply one, but only one, argument or
option in the \``#!`' line, thus:

      #!/usr/local/bin/zsh -f
      print other stuff here

which stops startup files other than `/etc/zshenv` from being run, but
otherwise works the same as before. If you need more options, you should
combine them in the same word. However, it's usually clearer, for
anything apart from `-f`, `-i` (which forces the shell into interactive
mode) and a few other options which need to take effect immediately, to
put a \``setopt`' line at the start of the body of the script. In a few
versions of zsh, there was an unexpected consequence of the fact that
the line would only be split once: if you accidentally left some spaces
at the end of the line (e.g. \``#!/usr/local/bin/zsh -f `') they would
be passed down to the shell, which would report an error, which was hard
to interpret. The spaces will still usually be passed down, but the
shell is now smart enough to ignore spaces in an option list.

The second point is that the length of the \``#!`' line which will be
evaluated is limited. Often the limit is 32 characters, in total, That
means if your path to zsh is long, e.g.
\``/home/users/psychology/research/dreams/freud/solaris_2.5/bin/zsh`'
the system won't be able to find the shell. Your only recourse is to
find a shorter path, or execute the shell directly, or some sneakier
trick such as running the script under `/bin/sh` and making that start
zsh when it detects that zsh isn't running yet. That's a fairly nasty
way of doing it, but just in case you find it necessary, here's an
example:

      #!/bin/sh

      if [ x$ZSH_VERSION = x ]; then
        # Put the right path in here ---
        # or just rely on finding zsh in
        # $path, since `exec' handles that.
        exec /usr/local/bin/zsh $0 "$@"
      fi

      print $ZSH_VERSION
      print Hello, this is $0
      print with arguments $*.

Note that first \``$0`', which passes down the name of the script that
was originally executed. Running this as \``testexec foo bar`' gives me

      3.1.9-dev-8
      Hello, this is /home/pws/tmp/testexec
      with arguments foo bar.

I hope you won't have to resort to that. By the way, really,
excruciatingly old versions of zsh didn't have `$ZSH_VERSION`. Rather
than fix the script, I suggest you upgrade the shell. Also, on some old
Bourne shells you might need to replace `"$@"` with `${1+"$@"}`, which
is more careful about only putting in arguments if there were any (this
is the sort of thing we'll see in [chapter 5](zshguide05.html#subst)).
Usually this isn't necessary.

You can use the same trick on ancient versions of UNIX which didn't
handle \``#!`'. On some such systems, anything with a \``:`' as the
first character is run with the Bourne shell, so this serves as an
alternative to \``#!/bin/sh`', while on some Berkeley systems, a plain
\``#`' caused csh to be used. In the second case, you will need to
change the syntax of the first test to be understood by both zsh and
csh. I'll leave that as an exercise for the reader. If you have perl
(very probable these days) you can look at the `perlrun` manual page,
which discusses the corresponding problem of starting perl scripts from
a shell, for some ideas.

There's one other glitch you may come across. Sometimes if you type the
name of a script which you know is in your path and is executable, the
shell may tell you \``file not found`', or some equivalent message. What
this usually means is that the *interpreter* wasn't found, because you
mistyped the line after the \``#!`'. This confusing message isn't the
shell's fault: a lot of operating systems return the same system error
in this case as if the script were really not found. It's not worth the
shell searching the path to see if the script is there, because in the
vast majority of cases the error refers to the programme in the
execution path. If the operating system returned the more natural error,
\``exec format error`', then the shell would know that there was
something wrong with the file, and could investigate; but unfortunately
life's not that simple.

