Chapter 1: A short introduction
===============================

The Z-Shell, \`zsh' for short, is a command interpreter for UNIX
systems, or in UNIX jargon, a \`shell', because it wraps around the
commands you use. More than that, however, zsh is a particularly
powerful shell --- and it's free, and under regular maintenance --- with
lots of interactive features allowing you to do the maximum work with
the minimum fuss. Of course, for that you need to know what the shell
can do and how, and that's what this guide is for.

The most basic basics: I shall assume you have access to a UNIX system,
otherwise the rest of this is not going to be much use. You can also use
zsh under Windows by installing Cygwin, which provides a UNIX-like
environment for programmes --- given the weakness of the standard
Windows command interpreter, this is a good thing to do. There are ports
of older versions of zsh to Windows which run natively, i.e. without a
UNIX environment, although these have a slightly different behaviour in
some respects and I won't talk about them further.

I'll also assume some basic knowledge of UNIX; you should know how the
filesystem works, i.e. what `/home/users/pws/.zshrc` and `../file` mean,
and some basic commands, for example `ls`, and you should have
experience with using `rm` to delete completely the wrong file by
accident, and that sort of thing. In something like \``rm file`', I will
often refer to the \`command' (`rm`, of course) and the \`argument(s)'
(anything else coming after the command which is used by it), and to the
complete thing you typed in one go as the \`command line'.

You're also going to need zsh itself; if you're reading this, you may
well already have it, but if you don't, you or your system administrator
should read [Appendix A](zshguide08.html#appa). For now, we'll suppose
you're sitting in front of a terminal with zsh already running.

Now to the shell. After you log in, you probably see some prompt (a
series of symbols on the screen indicating that you can input a
command), such as \``$`' or \``%`', possibly with some other text in
front --- later, we'll see how you can change that text in interesting
ways. That prompt comes from the shell. Type \``print hello`', then
backspace over \``hello`' and type \``goodbye`'. Now hit the \`Return'
key (or \`Enter' key, I'll just say `<RET>` from now on, likewise
`<TAB>` for the tab key, `<SPC>` for the space key); unless you have a
serious practical-joker problem on your system, you will see
\``goodbye`', and the shell will come back with another prompt. All of
the time up to when you hit `<RET>`, you were interacting with the shell
and its editor, called \`Z-Shell Line Editor' or \`zle' for short; only
then did the shell go away and tell the `print` command to print out a
message. So you can see that the shell is important.

However, if all you're doing is typing simple commands like that, why do
you need anything complicated? In that case, you don't; but real life's
not that simple. In the rest of this guide, I describe how, with zsh's
help, you can:

customise the environment in which you work, by using startup files,

write your own commands to shorten tasks and store things in shell
variables (\`parameters') so you don't have to remember them,

use zle to minimise the amount of typing you have to do --- in zsh, you
can even edit small files that way,

pick the files you want to use for a particular command such as `mv` or
`ls` using zsh's very sophisticated filename generation (known
colloquially as \`globbing') system,

tell the editor what sort of arguments you use with particular commands,
so that you only need to type part of the name and it will complete the
rest, using zsh's unrivalled programmable completion system,

use the extra add-ons (\`modules') supplied with the latest version of
zsh to do other things you usually can't do in a shell at all.

That's only a tiny sample. Since there's so much to say, this guide will
concentrate on the things zsh does best, and in particular the things it
has which other shells don't. The next chapter gives a few of the
basics, by trying to explain how to set the shell up the way you want
it. Like the rest of the guide, it's not intended to be exhaustive, for
which you should look at the shell manual.

Some other things you should probably know straight away. First, the
shell is always running, even when the command you typed is running,
too; the shell simply hangs around waiting for it to finish: you may
know from other shells about putting commands in the **background** by
putting an \``&`' after the command, which means that the shell doesn't
wait for them to finish. The shell is there even if the command's in the
foreground, but in this case doing nothing.

Second, it doesn't just run other people's commands, it has some of its
own, called **builtin commands** or just **builtins**, and you can even
add your own commands as lists of instructions to the shell called
**functions**; builtins and functions always run in the shell itself.
That's important to know, because things which don't run in the shell
itself can't affect it, and hence can't alter parameters, functions,
aliases, and all the other things I shall talk about.

1.1: Other shells and other guides
----------------------------------

If you want a basic grounding in how shells work, what their syntax is
(i.e. how to write commands), and how to write scripts and functions,
you should read one of the many books on the subject. In particular, you
will get most out of a book that describes the Korn shell (ksh), as zsh
is very similar to this --- so similar that it will be worth my while
pointing out differences as we go along, since they can confuse ksh
users. Recent versions of zsh can emulate ksh (strictly, the 1988
version of ksh, although there are increasingly features from the 1993
version) quite closely, although it's not perfect, and less perfect the
more closely you look. However, it's important to realise that if you
just start up any old zsh there is no guarantee that it will be set up
to work like ksh; unless you or your system adminstrator have changed
some settings, it certainly won't be. You might not see that straight
away, but it affects the shell in subtle ways. I will talk about
emulation a bit more later on.

A few other shells are worth mentioning. The grandfather of all UNIX
shells is sh, now known as the Bourne shell but originally just referred
to as \`the shell'. The story is similar to ksh: zsh can emulate sh
quite closely (much more closely than ksh, since sh is considerably
simpler), but in general you need to make sure it's set up to do that
before you can be sure it will emulate sh.

You may also come across the \`Bourne-Again Shell', bash. This is a
freely-available enhancement of sh written by the GNU project --- but it
is not always enhanced along the lines of ksh, and hence in many ways it
is very different from zsh. On some free UNIX-like systems such as
Linux/GNU (which is what people usually mean by Linux), the command sh
is really bash, so there you should be extra careful when trying to
ensure that something which runs under the so-called \`sh' will also run
under zsh. Some Linux systems also have another simpler Bourne shell
clone, ash; as it's simpler, it's more like the original Bourne shell.

Some more modern operating systems talk about \`the POSIX shell'. This
is an attempt to standardize UNIX shells; it's most like the Korn shell,
although, a bit confusingly, it's often just called sh, because the
standard says that it should be. Usually, this just means you get a bit
extra free with your sh and it still does what you expect. Zsh has made
some attempts to fit the standard, but you have to tell it to --- again,
simply starting up \`zsh' will not have the right settings for that.

There is another common family of shells with, unfortunately,
incompatible syntax. The source of this family is the C-Shell, csh, so
called because its syntax looks more like the C programming language.
This became widespread when the only other shell available was sh
because csh had better interactive features, such as job control. It was
then enhanced to make tcsh, which has many of the interactive features
you will also find in zsh, and so became very popular. Despite these
common features, the syntax of zsh is very different, so you should not
try and use csh/tcsh commands beyond the very simplest in zsh; but if
you are a tcsh user, you will find virtually every capability you are
used to in zsh somewhere, plus a lot more.

1.2: Versions of zsh
--------------------

At the time of writing, the most recent version of zsh available for
widespread use was 4.0.6. You will commonly find two sets of older zsh's
around. The 3.0 series, of which the last release was 3.0.9, was a
stable release, with only bug fixes since the first release of zsh 3.
The 3.1 series were beta versions, with lots of new features; the last
of these, 3.1.9, was not so different from 4.0.1; the main change is
that the shell has now been declared stable, so that as with zsh 3 there
will be a set of bug fixes, labelled 4.0, and a set with new functions
in, labelled 4.1. As 4.0 replaces all zsh 3 versions, I will try to keep
things simple and talk about that; but every now and then it will be
helpful to point out where older versions were different.

One notable feature of zsh is the completion of command line arguments.
The system changed in 3.1.6 and 3.1.7 to make it a lot more
configurable, and (provided you keep your wits about you) a little less
obscure. I therefore won't describe the old completion system, which
used the \`compctl' command, in any detail; a very brief introduction is
given in the zsh FAQ. The old system remains available, however we
strongly recommend new users to start with the new one. See [chapter
6](zshguide06.html#comp) \`Completion, old and new' for the lowdown on
new-style completion.

There won't be a big difference between 4.0 and 4.1, just bug fixes and
a few evolutionary changes, plus some extra modules. There will be some
notes in [chapter 7](zshguide07.html#ragbag) about new features in 4.1,
but nothing you write for 4.0 is likely to become obsolete in the
foreseeable future.

1.3: Conventions
----------------

Most of what I say will be reasonably self-contained (which means I use
phrases like \`as I said before' and \`as I'll discuss later on' more
than a real stylist would like, and the number times I refer to other
chapters is excessive), but there are some points I should perhaps draw
your attention to before you leap in.

I will often write chunks of code as you would put them in a file for
execution (a \`script' or a \`function', the differences to be discussed
*passim*):

      if [[ $ZSH_VERSION = 3.* ]]; then
        print This is a release of the third version of zsh.
      else
        print This is either very new or very old.
      fi

but sometimes I will show both what you type into a shell interactively,
and what the shell throws back at you:

      % print $ZSH_VERSION
      3.1.9
      % print $CPUTYPE
      i586

Here, \``%`' shows the prompt the shell puts up to tell you it is
expecting input (and the space immediately after is part of it).
Actually, you probably see something before the percent sign like the
name of the machine or your user name, or maybe something fancier. I've
pruned it to the minimum to avoid confusion, and kept it as reminder
that this is the line you type.

If you're reading an electronic version of this guide, and want to copy
lines with the \``%`' in front into a terminal to be executed, there's a
neat way of doing this where you don't even have to edit the line first:

      alias %=' '

Then `%` at the start of a line is turned into nothing whatsoever; the
space just indicates that any following aliases should be expanded. So
the line \``% print $CPUTYPE`' will ignore the \``%`' and execute the
rest of the line. (I hope it's obvious, but your *own* prompt is always
ignored; this is just if you copy the prompts from the guide into the
shell.)

There are lots of different types of object in zsh, but one of the most
common is parameters, which I will always show with a \``$`' sign in
front, like \``$ZSH_VERSION`', to remind you they are parameters. You
need to remember that when you're setting or fiddling with the parameter
itself, rather than its value, you omit the \``$`'. When you do and
don't need it should become clearer as we go along.

The other objects I'll show specially are shell options --- choices
about how the shell is to work --- which I write like this:
\``SH_WORD_SPLIT`', \``NO_NOMATCH`', \``ZLE`'. Again, that's not the
whole story since whenever the shell expects options you can write them
in upper or lower case with as many or as few underscores as you like;
and often in code chunks I'll use the simplest form instead:
\``shwordsplit`', \``nonomatch`', \``zle`'. If you're philosophical you
can think of it as expressing the category difference between talking
about programming and actual programming, but really it's just me being
inconsistent.

You may find it odd that I use three hyphens to signify a dash. That's
actually a convention used in the printed version of this guide, which
is made with LaTeX. One day, I will turn this into a macro and it will
appear properly in other versions; but then, one day the universe will
come to an end.

1.4: Acknowledgments
--------------------

I am grateful for comments from various zsh users. In particular, I have
had detailed comments and corrections from Bart Schaefer, Sven \`Mr
Completion' Wischnowsky and Oliver Kiddle. It's usual to add that any
remaining errors are my own, but that's so stark staringly obvious as to
be ridiculous. I mean, who wrote this? Never mind.

Most of this written on one or another release of Linux Mandrake (a
derivative of Red Hat), with the usual GNU and XFree86 tools. Since all
of this was free, it only seems fair to say \`thank you' for the gift.
It also works a lot better than the operating system that came with this
particular PC.

