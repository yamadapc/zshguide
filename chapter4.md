

Chapter 4: The Z-Shell Line Editor
==================================

The zsh line editor is probably the first part of the shell you ever
used, when you started typing in commands. Even the most basic shells,
such as sh, provide some kind of editing capability, although in that
case probably just what the system itself does --- enter characters,
delete the last character, delete the entire line. Most shells you're
likely to use nowadays do quite a lot more. With zsh you can even extend
the set of editor commands using shell functions.

4.1: Introducing zle
--------------------

The zsh line editor is usually abbreviated to \`zle'. Normally it fires
itself up for any interative shell; you don't have to do anything
special until you decide you need to change its behaviour. If everything
looks OK and you're not interested in how zle is started up, skip to the
next subsection.

Nowadays, zle lives in its own loadable module, `zsh/zle`, which saves
all the overhead of having an editor if the shell isn't interactive.
However, you normally won't need to worry about that; I'll say more
about modules in [chapter 7](zshguide07.html#ragbag), but the shell
knows when you need zle and gives you it automatically. Usually the
module is in a directory with a name like
\``/usr/local/lib/zsh/4.0.4/zsh/zle.so`', where the \``4.0.4`' is the
shell's version number, the same as the value of the parameter
`$ZSH_VERSION`, and everything after that apart from the suffix \``.so`'
is the module name. The suffix may be \``.sl`' (HP-UX) or \``.dll`'
(Cygwin), but \``.so`' is by far the most common form. It differs
because zsh keeps the same convention for dynamically loadable
libraries, or \`shared objects' in UNIX-speak, as the operating system.

If the shell is badly installed, you sometimes see error messages that
it, or a command such as \`bindkey', couldn't be loaded. That means the
shell couldn't find \``zsh/zle`' anywhere in the module load path, the
array `$module_path`. Then you need to complain to your system
administrator. If you've just compiled zsh and are having this problem,
it's because you have to install the modules before you run the shell,
even if the shell itself isn't installed. You can do that by saying
\``make install.modules`'. Then the compiled zsh should run from where
it is.

Note that unlike bash's line editor, readline, which is an entirely
separate library, zle is an integral part of the shell. Hence you
configure it by sticking commands in your `.zshrc` --- as it's only
useful for an interactive shell, only `/etc/zshrc` and `.zshrc` make
sense for this purpose.

One tip if you're looking at the zsh manual using info, either with the
command of that name or `\C-h i` within Emacs, which I find the most
convenient way: the entry for zle is called \`Zsh Line Editor', in full,
not just \`Zle'. Have fun looking for \`Shell Builtin Commands' (not
\`Builtins') while you're at it.

### 4.1.1: The simple facts

As with any editor later than `ed`, you can move around the line and
change it using various \`keystrokes', in other words one or more sets
of keys you type at once. For example, the keystroke to move back a word
is (maybe) `ESC b`. This means you first hit the escape key; nothing
happens yet. Then you hit \`b', and the cursor instantly jumps back to
the start of the word. (I'll have more to say on what zle thinks is a
\`word' --- it's not necessarily the same as what the rest of the shell
thinks is a word.)

It will probably help if I introduce the shell's way of describing
keystrokes right away; then when you need to enter them you can just
copy them straight in. The escape key is \``\e`', so that keystroke
would be \``\eb`'. Other common keystrokes include holding down the
control key, probably near the bottom left of the keyboard, and typing
another key at the same time. The simplest way of indicate a control key
is just to put \``^`' in front; so for example \``^x^x`' means hold down
control, and press \`x' twice with control still held down. It has
exactly the same effect as \``^X^X`'. (You may find each time you do
that it takes you to the start of the line and back to where you were.)

I've already introduced the weasel word \`maybe' to try to avoid lying.
This is because actually zle has two modes of operation, one (the
default) like Emacs, the other like vi. If you don't know either of
those venerable UNIX editors, I suggest you stick to Emacs mode, since
it tends to interfere a little less with what you're doing, and
furthermore completion is a little easier. Completion is an offshoot of
zle behaviour which is described in [chapter 6](zshguide06.html#comp)
(which, you will notice, is longer than this one).

If you normally use vi, you may have one or both of the environment
variables `$EDITOR` or `$VISUAL` set to \``vi`'. (It all works the same
way if you use \``vim`' instead, or any editor that happens to contain
\``vi`' such as \``elvis`'.) In that case, zle will start up in its
\``vi`' mode, where the keystrokes are rather different. That's why you
might have found that \``\eb`' didn't do what I said, even though you
had made no attempt to configure zle. You can make zle always use either
emacs or vi mode by putting either

      bindkey -e

or

      bindkey -v

in your `.zshrc`. This is just one of many uses of `bindkey`.

If you're not familiar with this use of the word \`bind', it just means
\`make a keystroke execute a particular editor command'. Commands have
long-winded names with hyphens which give you quite a good description
of what they do, such as \``backward-delete-char`'. Normal keys which
correspond to printable characters are usually \`bound to
`self-insert`', a perverse way of saying they do what you expect and
show up the character you typed. However, you can actually bind them to
something else. In vi command mode, this is perfectly normal.

Actually, if you use a windowing system, you might want to say
\``bindkey -me`', which binds a whole set of \`meta' keys. In X Windows,
one of the keys on your keyboard, possibly `ALT`, may be designated a
\`meta' key which has a special effect similar to the control key.
Bindings with the meta key held down are described a bit like they are
in Emacs, \``\M-b`'. (You can specify control keys similarly, in fact,
like \``\C-x`', but \``^x`' is shorter.) Using the \``-m`' option to
`bindkey` tells zsh that wherever it binds an escape sequence like
\``\eb`', it should also bind the corresponding meta sequence like
\``\M-b`'. Emacs always ties these together, but zsh doesn't --- you can
rebind them separately. and if you want both sequences to be bound to a
new command, you have to bind them both explicitly.

You need to be careful with \``bindkey -m`', however; the shell can't
tell whether you are typing a character with the top bit set, or
executing a command. This is likely to become worse as the UTF-8
encoding for characters becomes more popular, since a non-ASCII
character then consists of a whole series of bytes with the top bit set.

If you are interested in binding function keys, you may already have
found the key sequences they send apparently don't make any sense; see
[the section below](zshguide04.html#fkeys) for more information. This
will introduce the function called `zkbd` which can make the process
less painful. The function also helps with \`meta' and \`ALT' keys.

### 4.1.2: Vi mode

I'm going to concentrate on Emacs mode for various reasons: firstly,
because I use it myself; secondly, because the most likely reason for
you using vi mode is that you are already familiar with vi and don't
need to be told how it works; thirdly, because most of the commands are
the same in both modes, just bound differently; and finally because if
you *don't* already know vi, you will quite likely find vi editing mode
rather counterintuitive and difficult to use.

However, here are a few remarks on it just to get it out of the way.
Like the real vi editor, there are two basic modes, insert mode, where
you type in text, and command mode, where the same keystrokes which
insert characters are bound instead to editing commands. Unlike the real
vi, the line editor starts in insert mode on every new command you edit.
This means you can often simply type a line up to the \`return' at the
end and forget you are in vi mode at all.

To enter command mode, you hit \`escape', again just like normal vi. At
this point you are in the magic world of vi commands, where typing an
ordinary character can have any effect whatsoever. However, the bindings
are similar to normal vi, so \``h`' and \``l`' move left and right. When
you want to insert more text, you can use any of the normal vi commands
which allow you to do that, such as \``i`' (`vi-insert`) or \``a`'
(`vi-add-next`).

Apart from the separate command and insert modes and the completely
different set of key bindings, there is no basic difference between
Emacs mode and vi mode. You can bind keys in both the vi modes --- they
don't *have* to correspond to `self-insert` in insert mode. Below, I'll
describe \`keymaps', a complete set of descriptions for what all the
keys will do (less impressive than it sounds, since a lot of keys may be
set to \``undefined-key`, which means they don't do anything useful),
and you will see how to change the behaviour in both modes.

4.2: Basic editing
------------------

If you know Emacs or vi, you will very soon find out how to do simple
commands like moving the cursor, going up and down the history list, and
deleting and copying words. If you don't, you should read the `zshzle`
manual page for a concise description of what it can do. Here is a
summary for Emacs mode.

### 4.2.1: Moving

You can move forwards and backwards along the line using the cursor
keys. The are a variety of different conventions as to what keystrokes
the cursor keys produce. You might naively expect that pressing, say,
cursor right, sends a signal along the lines of \`cursor right' to the
application. Unfortunately, there is no such character in the ASCII
character set, so programmes which read input as a string of characters
like zsh have to be given an arbitrary string of characters. (It's
different for programmes which understand other forms of input, like
windowing systems.)

The two most common conventions for cursor keys are where the up key
sends \``\e[A`' and the other three the same with `B`, `C` and `D` at
the end, and the convention where the \``[`' is replaced by an \``O`'
(uppercase letter \`O'). In old versions of zsh, the only convention
supported was the first of those two. The second, and any other
convention, were not supported at all and you had to bind the keys
yourself. This was done by something like:

      bindkey "\eOA" up-line-or-history
      bindkey "\eOB" down-line-or-history
      bindkey "\eOC" forward-char
      bindkey "\eOD" backward-char

The shell tries harder now, and provided your system has the correct
information about your terminal (zsh uses an old system called
\`termcap' which has largely been superseded by another called
\`terminfo') you should be lucky. If the shell thinks your keys are too
perverse --- in particular, if the keystroke it wants to bind the
function too is already defined by zsh --- you will still have to do it
by hand. The list above should serve as a template.

Instead of the cursor keys, traditional Emacs keys are available: `^b`
and `^f` for backward and forward, `^p` and `^n` for previous line and
next line, so you can continue even if the cursor keys don't work.

Moving longer distances is done by `\eb` and `\ef` for a word backwards
or forwards (or, as you saw, `\M-b` and `\M-f`), and `^a` and `^e` for
the start and the end of the line. That just about exhausts the ones you
will use the most frequently.

### 4.2.2: Deleting

For deleting, backspace or the delete key will delete backwards. There
is an eternal battle over these keys owing to the fact that on PC
keyboards the key at the top left of the central keyboard section is
\`backspace' which is the character `^h`, while on traditional UNIX
keyboards it is \`delete' which is the character 127, often written as
`^?` (which zsh also understands). When you are in the system's own
primitive line editing mode, as with sh (unless your sh is really bash),
only one of these is \`bound', although it's not really a key binding,
it's a translation made by the system's terminal driver, and it's
usually the wrong one. Hence you often find the system prints \``^h`' on
the screen when you want it to delete. You can change the key using

      stty erase '^h'

but zsh protects you from all that --- both `^h` (backspace) and `^?`
(delete) will delete backwards one character. Note, by the way, that zsh
doesn't understand smart names for any keystrokes -- if you try to bind
a key called \``backspace`' zsh will bind a command to that sequence of
characters, not a key of that name. See comments on \``bindkey -s`' for
when something like this might even be useful.

To confuse matters further, the key often marked as \`Delete' on a 101-
or 102-key PC keyboard in the group of 6 above the cursor keys is
completely different again, and probably doesn't send either of those
sequences. On my keyboard it sends the sequence \``\e[3~`'. I find it
convenient to have this delete the next charater, which is its tradional
role in the PC world, which I do by

      bindkey '\e[3~' delete-char

However, the tradtional *Emacs* way of deleting the next character is to
use \``^d`', which zsh binds for you by default. If you look at the
binding, which you can do by not giving bindkey an editor command to
bind,

      % bindkey '^d'
      delete-char-or-list

you'll see it doesn't *quite* do what I suggested. The \``-or-list`'
part is for completion, and you'll find out about it in the next
chapter. The first shell I know of to have this odd combination was
tcsh.

Since I enjoy confusion, I might as well point out that usually `^d` has
another use, which is to tell the terminal driver you've reached the end
of a file. In the case of, say, a file on a disk, the system knows this
by itself, but if you are supplying a stream of characters, the only way
of telling it is to send a special character. The default is usually
`^d`. You will notice that if you type \``^d`' at the start of the line,
you see the message

      zsh: use 'exit' to exit.

That's because zsh recognises the `^d` as end-of-file in that position.
By default the shell warns you; you can turn this off by setting the
option `IGNORE_EOF`. You can tell the system you don't ever want to send
an end-of-file in this way with `stty`, again: the following are
equivalent in Linux but your system way want one or the other:

      stty eof '^-'
      stty eof undef

Remember that `stty` is not part of the shell; it's a way of controlling
the state of the system's terminal driver. This means it survives as
long as the terminal or terminal window is still connected, even if you
start a new shell or exit one that isn't the login shell.

By the way, if you need to refer to a character by its number, the
easiest way is probably to use the syntax \``\x??`', where the \``??`'
are the two hex digits for the key. In the case of delete, it is
\``\x7f`'. You can confirm this by:

      % bindkey '\x7f'
      "^?" backward-delete-char

### 4.2.3: More deletion

You can delete larger areas with \``\ed`' to delete the next word and
\``\e^h`' or \``\e^?`' (escape followed by delete backwards) to delete
the previous word. \``^u`' usually removes the entire line, before and
after the cursor --- this is not like Emacs, where `^u` introduces digit
arguments as I will describe in the next subsection. It is, however,
like another of those primitive editing commands the terminal driver
itself provides, this one being known to `stty` as \``kill`'. The most
common use of this outside zsh is for deleting your password when you
login, when you know you've typed it wrong but can't see how many !@?\*!
characters you've typed, and maybe can't rely on the terminal agreeing
with you as to which of `^h` or `^?` will delete a single one.

Strictly speaking, all the keystrokes in the previous paragraph perform
a \`kill' (zsh-speak, not to be confused with the `stty` \`kill') rather
than a \`delete' (or deletion, as we used to say when we had a distinct
between nouning and verbing). The difference is the same as in Emacs ---
\`killed' text is saved for later \`yanking' back somewhere else, which
you do with the `^y` key, whereas \`deleted' text as with `^?` and `^d`
is gone forever. This is what everyone not brought up under Emacs calls
\`cut' and \`paste' (although since Emacs dates back to the seventies,
it could be everyone else that's wrong). Another feature borrowed from
Emacs is that if you do multiple \`kills' without any other editing in
between, the killed text is joined together and you can yank it all back
in one go. I will say more when I talk about point and mark (another
Emacs idea).

Actually, even deleted text isn't gone forever: zsh has an Emacs-like
editing history, and you can undo the previous commands on the line.
This is usually bound to `^xu` and `^x^u`, and there is shorter binding
which is described rather confusingly as \``^_`' --- confusingly,
because on all not-completely-spaced-out keyboards I've ever used you
actually generate that sequence by holding down control and pressing the
\``/`' key. Zsh doesn't use `^z` by default and, if you are used to
Windows, that is another suitable binding for `undo`.

Zsh scores in one way over Emacs --- it also has \``redo`', not bound by
default. This means that if you undo to much, you can put back what you
just undid by repeatedly using the `redo` command.

4.3: Fancier editing
--------------------

### 4.3.1: Options controlling zle

Unlike completion, `zle` doesn't have many options associated with it;
most of the control is done by key bindings and builtin commands. Only
two are really useful; both control beeps. The option `beep` can be
unset to tell the shell never to make a noise on an error; the option
`histbeep` can be unset to disable beeps only in the case of trying to
go back before the first or forward after the last history entry.

The not-very-useful options are `zle` and `singlelinezle`. The former
controls whether zle is active at all and isn't that useful because it's
usually on automatically whenever you need it, in other words in
interative shells, and off whenever you don't. It's sometimes useful to
test via \``[[ -o zle ]]`', however; this lets you make a function do
something cleverer in an interative shell.

The option `singlelinezle` restricts editing to one line; if it gets too
long, it will be truncated and a \``$`' printed where the missing bits
are. It's only there for compatibility with ksh and as a safeguard if
your terminal is really screwed up, though even in that case zsh tries
to guess whether everything it needs is available.

Other functions that affect zle include the history functions. These
were described back in [chapter 2](zshguide02.html#init); once you've
set it off, searching through the history works basically the same way
in zle as with the \``!`' history commands.

### 4.3.2: The minibuffer and extended commands

The \`minibuffer' is yet another Emacs concept; it is a prompt that
appears just under the command line for you to enter some edit required
by the editor itself. Usually, it comes and goes as it pleases and you
don't need to think about it. The most common uses are entering text for
searches, and entering a command which isn't bound to a string. That's
yet another Emacs feature: `\ex` prompts you to enter the name of a
command. Luckily, since the names tend to be rather long, completion is
available. So typing \``echo foo<ESC>xba<TAB>w<TAB>`' ends up with:

      % echo foo
      execute: backward-word

and hitting return executes that function, taking you to the start of
the `foo`; you might be able to think of easier ways of doing that. This
does provide a way of running commands you don't often use.

(I hope my notation isn't too confusing. I write things like `<TAB>`
when I'm showing a single character you hit, to make it stand out from
the surrounding text. However, when I'm not showing text being entered,
I would write that as \``\t`', which is how you would enter the
character into a key sequence to be bound, or a string to be printed.)

The minibuffer only handles a very limited set of editing commands.
Typing one it doesn't understand usually exits whatever you were trying
to do with the minibuffer, then executes the keystroke. However, in this
particular case, it won't let you exit until you have finished typing a
command; your only other option is to abort. The usual zle abort
character is `^g`, \``send-break`'. This is different from the more
drastic `^c`, which sends the shell itself an interrupt signal. Quite
often they have the same effect in zle, however. (You'll notice `^c` is
actually \`bound to `undefined-key`', in other words zle doesn't
consider it does anything. However, the terminal driver probably causes
it to send an interrupt, and zle does respond to that.)

Another feature useful with rare commands is \``where-is`'. Surprise!
it's not bound by default, so typing \``<ESC>xwhere-is`' is then the way
of runing it. Then you type another editor command at the \``Where is:`'
prompt, and the shell will tell you what keystrokes, if any, are bound
to it. You can also simply use `grep` on the output of `bindkey`, which,
with no arguments, lists all bindings.

### 4.3.3: Prefix (digit) arguments

Many commands can be repeated by giving them a numeric prefix or digit
argument. For example, at the end of a long line of text, type
\``<ESC>4<ESC>b`'. The \``<ESC>b`' on its own would take you one word
backwards. The \``<ESC>4`' passes it the number four and it moves four
words backwards. Generally speaking, this works any time it make sense
to repeat a command. It works for `self-insert`, too, just repeatedly
inserting the character. If it doesn't work, the prefix argument is
simply ignored.

You can build up long or negative arguments by repeating both the `\e`
and the digit or \``-`' after it; for example, \``<ESC>-<ESC>1<ESC>0`'
specifies minus ten. It varies from command to command how useful
negative numbers are, but they generally switch from backwards to
forwards or similar: \``<ESC>-<ESC>4<ESC>\f`' is a pointless way of
executing the same as \``<ESC>4<ESC>b`'.

The shell also has Emacs' \``universal-argument`' feature, but it's not
bound by default --- in Emacs it is `\C-u`, but as we've seen that's
already in use. This is an alternative to all those escapes. If you bind
the command to a keystroke (it's absolutely pointless as a shortcut
otherwise), and type that key, then an option minus followed by any
digits are remembered as a prefix. The next keystroke which is not one
of those is then executed as a command, with the prefix formed by the
number typed after `universal-argument`.

For example, on my keyboard, the key `F12` sends the key sequence
\``\e[[24~`' --- see below for how to find out what functions keys send.
Hence I use

      bindkey '\e[[24~' universal-argument

Then if I hit the characters `F12`, `4`, `0`, `a`, a row of forty \`a's
is inserted onto the command line. I'm not claiming this example is
particularly useful.

### 4.3.4: Words, regions and marks

Words are handled a bit differently in zsh from the way they are in most
editors. First, there is a difference between what Emacs mode and vi
mode consider words. That is to say, there is a difference between the
functions bound by default in those modes; you can use the same
functions in either mode by rebinding the keys.

In both vi and Emacs modes, the same logic about words applies whether
you are moving forward or backward a number of words, or deleting or
killing them; the same amount of text is removed when killing as the
cursor would move in the other case.

In vi mode, words are basically the same as what vi considers words to
be: a sequence of alphanumeric characters together with underscores ---
essentially, characters that can occur in identifiers, and in fact
that's how zsh internally recognises vi \`word characters'. There is one
slight oddity about vi's wordwise behaviour, however, which you can
easily see if you type \``/a/filename/path/`', leave insert mode with
`ESC`, and use \``w`' or \``b`' to go forward or backward by words over
it. It alternates between moving over the characters in a word, and the
characters in the separator \``/`'.

In Emacs, however, it is done a bit differently. The vi \`word
characters' are always considered parts of a word, but there is a
parameter `$WORDCHARS` which gives a string of characters which are
*also* part of a word. This is perhaps opposite to what you would
expect; given that alphanumerics are always part of a word, you might
expect there to be a parameter to which you add characters you *don't*
want to be part of a word. But it's not like that.

Also unlike vi, jumping a word always means jumping to a word character
at the start of a word. There is no extra \`turn' used up in jumping
over the non-word characters.

The default value for `$WORDCHARS` is

      *?_-.[]~=/&;!#$%^(){}<>

i.e. pretty much everything and the kitchen sink. Usually, therefore,
you will want to remove characters which you don't want to be considered
parts of words; \``-`', \``/`' and \``.`' are particularly likely
possibilities. If you want to remove individual characters, you can do
it with some pattern matching trickery (next chapter):

      % WORDCHARS=${WORDCHARS//[&.;]}
      % print $WORDCHARS
      *?_-[]~=/!#$%^(){}<>

shows that the operation has removed those three characters in the
group, i.e. \``&`', \``.`' and \``;`', from `$WORDCHARS`. The \``//`'
indicates a global substitution: any of the characters in the square
brackets is replaced by nothing.

Many other line editors, even those like `readline` with Emacs bindings,
behave as if only identifier characters were part of a word, i.e. as if
`$WORDCHARS` was empty. This is very easy to do with a zle shell
function. Recent versions of zsh supply the functions
\``bash-forward-word`', \``bash-kill-word`', and a set of other similar
ones, for you to bind to keys in order to have that behaviour.

Other behaviours are also possible by writing functions; for example,
you can jump over real shell words (i.e. individual command arguments)
by using some more substitution trickery, or you can consider only
space-delimited words (though that's not so far from what you get with
`$WORDCHARS` by adding \``` "`'\@ ``').

### 4.3.5: Regions and marks

Another useful concept from Emacs is that of regions and marks. In
Emacs-speak \`point' is where the cursor is and \`mark' is somewhere
where you leave a mark to come back to later. The command to set the
mark at the current point is \``^@`' as in Emacs, a hieroglyphic which
usually means holding down the control key and pressing the space key.
On some systems, such as the limited version of `telnet` provided with a
well-known non-UNIX-based windowing system, you can't send this
sequence, and you need to bind a different sequence to
`set-mark-command`. One possibility is \``\e `' (escape followed by
space), as in MicroEMACS. (Some X Windows configurations don't allow
`^@` to work in an xterm, either, though that is usually fixable.)

To continue with Emacs language, the region between point and mark is
described simply as \`the region'. In zsh, you can't have this
highlighted, as you might be used to with editors running directly under
windowing systems, so the easiest way to find out the ends of the region
is with `^x^x`, `exchange-point-and-mark`, which I mentioned before ---
mark, by default, is left at the beginning of the line, hence the
behaviour you saw above.

Various editing commands --- usually those with \``region`' in the name
--- operate on this. The most usual are those which kill or copy the
region. Annoyingly, `kill-region` isn't bound --- in Emacs, it's `^w`,
but zsh follows the tradition of having that bound to
`backward-kill-word`, even though that's also available as the
traditional Emacs binding `\e^?`. So it's probably useful to rebind it.
To copy the region, the usual binding \``\ew`' works.

You then \`yank' back the text copied or killed at another point with
\``^y`'. The shell implements the \`kill ring' feature, which means if
you perform a yank, then type \``<ESC>y`' (`yank-pop`) repeatedly, the
shell cycles back through previously killed or copied text, so that you
have more available than just the last one.

4.4: History and searching
--------------------------

Zle has access to the lines saved in the shell's history, as described
in \`Setting up history' in [chapter 2](zshguide02.html#init). There are
essentially three ways of retrieving bits of the history: moving back
through it line by line, searching back for a matching line, and
extracting individual words from the history. In fact, the first two are
pretty similar, and there are hybrid commands which allow you to move
back step by step but still matching only particular lines.

### 4.4.1: Moving through the history

The simplest behaviour is what you get with the normal cursor key
bindings, \``up-line-or-history`' and \``down-line-or-history`'. If you
are in text which fits into a single line (which may be a continuation
line, i.e. it has a new prompt in the form given by `$PS2` at the start
of the line), this replaces the entire line with the line before or
after in the history. The history is not circular, it has a beginning
and an end. The beginning is the first line still remembered by the
shell (i.e. `$HISTSIZE` lines back, taking into account that the actual
number of lines present will be modified by the effects of any special
history options you have set to remove unwanted lines); the end is the
line you are typing. You can use `\e<` and `\e>` to go to the first and
last line in the history.

The last phrase sounds trivial but isn't quite. Type
\``echo This is the last line`', go back a few lines with the up arrow,
and then back down to the end, and you will see what I mean --- the
shell has remembered the line you were typing, even though it hadn't
been entered, so you can scroll up and down the history and still come
back to it.

Of course, you can edit any of the earlier history lines, and hit
\`return' so that they are executed --- that's the whole point of being
able to scroll back through the history. What is maybe not so obvious is
that the shell will remember changes you make to these lines, too, until
you hit \`return'.

For example, type \``echo this is the last line`' at a new shell prompt,
but don't hit return. Now hit the up arrow once, and edit the previous
line to say \``echo this is the previous line`'. If you scroll down and
up, you will see that the shell has kept both of those lines. When you
decide which one to use and hit return, that line is executed and added
to the end of the history, and any changes to previous lines in the
history are forgotten.

Sometimes you don't want to add a new line to history, instead
re-execute a whole series of earlier commands one by one. This can be
done with `^o`, `accept-line-and-down-history`. When you hit `^o` on a
line in the history, that is executed, and the line after it in the
history is shown. So you just need to keep hitting it to keep executing
the commands.

There are two other similar commands I don't use as much,
`infer-next-history`, bound to `^x^n`, and
`accept-and-infer-next-history`, not bound by default. \`Inferring' the
next history means that the shell looks at what is in the current line,
whatever its provenance --- you might just have typed it, for example
--- and looks back in the history for a matching line; the \`inferred'
next history line is the one following that line. In the first case, you
are simply shown that line; in the second case, the current line is
executed first, then you are shown the inferred line. Feel free to drop
me a line if you find this is the best thing since sliced bread.

One slight confusion about the history is that it can be hard to
remember quite where you are in it, for example, if you were editing a
line and had to scroll back to look for something else. In cases like
this, `\e>` is your friend, as it takes you the last line. Also,
whenever you hit return, you are guaranteed to be at the end of the
history, even if you were editing a line some back in the history,
unlike certain other systems (though `accept-line-and-down-history` can
emulate those). So it's usually not too hard to stay unconfused about
what you're editing.

### 4.4.2: Searching through the history

Zsh has the commands you would expect to search through the history,
i.e. where you hit a search key and then type in the words to search
for. However, it also has other features, probably more used by the zsh
community, where the search is based on some feature of the current
line, in particular the first word or the line up to the cursor
position. These typically enable you to search backwards more quickly,
since you don't need to tell the shell what you are looking for.

**Ordinary searching**\
\

The standard search commands, by which I mean the ones your are probably
most familiar with from ordinary text editors (if either Emacs or vi can
be so called), are designed to make Emacs and vi users feel at home.

In Emacs mode, you have incremental search: `^r` to search backwards ---
this is usually what you want, since you usually start at the end ---
and `^s` to search forwards. Note that `^s` is another keystroke which
is often intercepted by the terminal driver; in this case, it usually
freezes output to the terminal until you type `^q` to turn it back on.
If you don't like this, you can either use \``stty stop`' and
\``stty start`' to change the characters, or simply
\``unsetopt flowcontrol`' to turn that feature off altogether. However,
the command bound to `^s`, `history-incremental-search-forward`, is also
bound to `^xs`, so you can use that instead.

As in Emacs, for each character that you type, incremental search takes
you to the nearest history entry that matches all the characters, until
the match fails. Typing the search keystroke again at any point takes
you to the next match for the characters in the minibuffer.

In vi command mode, the keystrokes available by default are the familiar
\``/`' and \``?`'. There are various differences from vi, however. First
of all, it is \``/`' that searches backwards --- this is the one you
will use more often. Secondly, you can't search for regular expressions
(patterns); the only exception is that the character \``^`' at the start
anchors the search to the start of a line. Everything else is just a
plain string.

The other two standard vi search keystrokes are also present: \``n`'
searches for the next match of the current string, and \``N`' does the
same but reverses the direction of the search.

**Search for the first word**\
\

The next sort of search is probably the most commonly used, but is only
bound in Emacs mode: `\ep` and `\en` search forward or backward for the
next history line with the same first word as the current line. So often
to reuse a command you will type just the command name itself, and hit
`\ep` until the command line you want appears. These commands are called
simply \``history-search-backward`' and \``history-search-forward`'; the
name doesn't really describe the function all that well.

**Prefix searching**\
\

Finally, you can search backwards for a line which has the entire
starting point up to the cursor position the same as the current line.
This gives you a little more control than `history-search-`*direction*.
The corresponding commands, `history-beginning-search-backward` and
`history-beginning-search-forward`, are not bound by default. I find it
useful to have them bound to `^xp` and `^xn` since this is similar to
the initial-word searches:

      bindkey '^xp' history-beginning-search-backward
      bindkey '^xn' history-beginning-search-forward

**Other search commands based on functions**\
\

Search commands are one of the types most often customised by writing
shell functions. Some are supplied with the latest versions of the
shell; have a look in the ZLE section of the `zshcontrib` manual page.
You should find the functions themselves installed somewhere in your
`$fpath`, typically

      /usr/local/share/zsh/$ZSH_VERSION/functions

or in the subdirectory `Zle` of that directory, depending how your
version of zsh was installed. If the shell was pre-installed, the most
likely location is

      /usr/share/zsh/$ZSH_VERSION/functions/Zle

These should guide you in writing your own.

One point to note is that when called from a function the
`history-search-`*direction* and
`history-incremental-search-`*direction* can take a string argument
specifying what to search for. In the first case, this is just a one off
search, while in the second, you remain in incremental search and the
string is used to prime the minibuffer, so you can edit it. I will later
say much more about writing zle functions, but calling a search command
from a user-defined editing function is as simple as:

      zle history-search-backward search-string

and you can test the return status to see if the search succeeded.

### 4.4.3: Extracting words from the history

Sometimes instead of editing a previous line you just want to extract a
word from it into the current line. This is particularly easy if the
word is the last on the line, and the line isn't far back in the
history: just hit `\e.` repeatedly, and the shell will cycle through the
last word on previous lines. You can give this a prefix argument to pick
the *N*th from last word on the line just above the last line you picked
a word from. As you can tell from the description, this gets a little
hairy; version 4.1 of the shell will probably provide a slightly more
flexible version.

Although it's strictly not to do with the history, you can copy the
previous word on the current line with `copy-prev-word`, which for some
reason is bound to `\e^_`, escape followed (probably) by control and
slash. I have this bound to `\e=` instead (in some versions of ksh that
key sequence is taken by the equivalent of `list-choices`). This copies
words delimited by whitespace, but you can copy what the shell would see
as the previous complete argument by using `copy-prev-shell-word`
instead. This isn't bound by default, as it is newer than the other one,
but it is arguably more useful.

Sometimes you want to complete a word from the history; this is possible
using the completion system, described in the next chapter.

4.5: Binding keys and handling keymaps
--------------------------------------

There are two topics to cover under the heading of key bindings: first,
how to bind keys themselves, and secondly, keymaps and how to use them.
Manipulating both key bindings and keymaps is done with the `bindkey`
command. The first topic is the more immediately useful, so I'll start
with that.

### 4.5.1: Simple key bindings

You've already seen basic use of `bindkey` to link editing commands to a
particular sequence of keys. You've seen the shorthand for naming keys,
with `\e` being escape and `^x` the character `x` pressed while the
control key is held down. I even said something about \`meta' key
bindings.

Let me now launch into a little more detail. When you bind a key
sequence, which you do with \``bindkey` *key-sequence*
*editor-command*', the *key-sequence* can consist of as many characters
as you like. It doesn't even matter (much) if some initial set of the
key sequence is already bound. For example, you can do,

      bindkey '\eA' backward-word
      bindkey '\eAA' beginning-of-line

Here, I'll follow the shell documentation in referring to `\eA` as the
prefix of `\eAA`.

This introduces two points. First, note that the binding for `\eA` is
distinct from that for `\ea`; you will see the latter still does
`accept-and-hold` (in Emacs mode), which means it excutes the current
line, then gives it back to you for editing --- useful for doing a lot
of quite similar tasks. Meanwhile, `\eA` takes you back a word.

This case sensitivity only applies to alphabetic characters which are a
complete key in their own right, not to those characters with the
control key held down --- `^x` and `^X` are identical. (You may have
found there are ways to bind both separately in Emacs when running under
a windowing system, since the windowing system can tell Emacs if the
shift key is held down with the others; it's not that simple if you are
using an ordinary terminal.)

If you entered both those `bindkey` commands, you may notice that there
is a short pause before `\eA` takes effect. That's because it's waiting
to see if you type another `A`. If you do type the extra `A` during that
pause, you will be taken to the beginning of the line instead. That
pause is how the shell decides whether to execute the prefix on its own.

The time it waits is configurable and is given by the parameter
`$KEYTIMEOUT`, which is the delay in hundredths of a second. The default
is 40, i.e. four tenths of a second. Its use is usually down to personal
preference; if you don't type very fast, you might want to increase it,
at the expense of a longer delay when you are waiting for the prefix to
be executed. If you are editing on a remote machine over a very slow
link, you also may need to increase it to be able to get full key
sequences which have such a prefix to work at all.

However, the shell only has this ambivalent behaviour if a prefix is
bound in its own right; if the initial key or keys don't mean anything
on their own, it will wait as long as you like for you to type a full
sequence which is bound. This is by far the normal case. The only common
example of a separately bound prefix is in vi insert mode, where `<ESC>`
takes you back to command mode, while there may be other bindings
starting with `\e` such as the cursor keys. We'll see below how you can
remove those if they offend your sense of vi purity. (Don't laugh, vi
users are strange.)

Note that if the whole sequence is not bound, after all, the shell will
abort as soon as it reads a complete key sequence which is no longer a
prefix. For example, if you type \``\e[`', the chances are the shell is
waiting for more, but if you add a \``/`', say, it will probably decide
you are being silly and abort. The next key you type then starts a new
sequence.

### 4.5.2: Removing key bindings

If you want to remove a key binding, you can simply bind it to something
else. Near all uses of the `bindkey` and `zle` commands are smart about
removing dead wood in such cases. However you can also use
\``bindkey -r` *key-sequence*' to remove the binding explicitly. You can
also simply bind the sequence to the command `undefined-key`; this has
exactly the same effect --- even down to pruning completely any bindings
for long sequences. For example, suppose you bind \``\e\C-x\C-x`' to a
command, then to `undefined-key`. All memory that \``\e\C-x\C-x`' was
ever bound is removed; `\e\C-x` will no longer be marked as a prefix
key, unless you had some other binding with that prefix.

You can remove all bindings starting with a given prefix by adding the
\``-p` option. The example given in the manual,

      bindkey -rpM viins '\e'

(except it uses the equivalent form \``^[`') is amongst the most useful,
as it will remove the annoying delay after you type \``\e`' to enter vi
command mode. The delay is there because the cursor keys usually also
start with `\e` and the shell is waiting to see if you actually typed
one of those. So if you can make do without cursor keys in vi insert
mode you may want to consider this.

Note that any binding for the prefix itself is not removed. In this
example, `\e` stays bound as it was in the `viins` keymap, presumably to
`vi-cmd-mode`.

All manipulations like this are specific to one particular keymap. You
need to repeat them with a different `-M` *...* option argument, which
is described below, to have the same effect in other keymaps.

### 4.5.3: Function keys and so on

It's usually possible to bind the function keys on your keyboard,
including the specially named ones such as \`Home' and \`Page Up'. It
depends a good deal on how your windowing system or terminal driver
handles them, but these days it's nearly always the case that a well
set-up system will allow the function keys to send a string of
characters to the terminal. To bind the keys you need to find out what
that string is.

Luckily, you are usually aided by the fact that only the first character
of the string is \`funny', i.e. does something other than insert a
character. So there is a trick for finding out what the sequence is. In
a shell window, hit `^v` (if you are using vi bindings, you will need to
be in insert mode), then the function key in question. You will probably
see a string like \``^[OP`' --- this is what I get from the F1 key. A
note in my `.zshrc` suggests I used to get \``\e[11~`', so be prepared
for something different, even if, like me, you are using a standard
xterm terminal emulator. A quick poll of terminal emulators on this
Linux/GNU/XFree86 system suggests these two possibilities are by far the
most popular.

You may even be able to get different sequences by holding down shift or
control as well (after pressing `^v`, of course). On my keyboard,
combining F1 with shift gives me \``^[O2P`', with control \``^[O5P`' and
with both \``^[O6P`'. Again, your system may do something completely
different.

If you move the cursor back over that \``^[`', you'll find it's a single
character --- you can position the cursor over the \``^`', but not the
\``[`'. This is zsh's way of inserting a real, live escape character
into the line. In fact, if you type

      bindkey '

then `^v`, the function key, and the other single quote, you have a
perfectly acceptable way of binding the key on the command line. Zsh is
generally quite relaxed about your use of unprintable characters; they
may not show up correctly on your terminal, but the shell is able to
handle all single-byte characters. It doesn't yet have support for those
longer than a single byte, however.

You can also do the same in your `.zshrc`; the shell will handle strange
characters in input without a murmur. You can also use the two
characters \``^[`', which is just another way of entering an escape key.
However, the kosher thing to do is to turn it into \``\e`'. For example,

      bindkey '\e[OP'  where-is           # F1
      bindkey '\e[O2P' universal-argument # shift F1

and so on. Using this, you can give sensible meanings to \`Home',
\`End', etc. Note the windowing system's sensible way of avoiding the
problem with prefixes --- any extra characters are inserted before the
final character, so the shell can easily tell when the sequence is
complete without having to wait and see if there is more to follow.

There is a utility supplied with zsh called `zkbd` which can help with
all of this by finding out and remembering the definitions for you. You
can probably use it simply by autoloading it and running it, as it is
usually installed with the other functions. It should be reasonably
self-explanatory, else consult the `zshcontrib` manual.

If you are using X Windows and are educated enough, you can tinker with
your `.Xdefaults` file to tweak how keys are interpreted by the terminal
emulator. For example, the following turns the backspace key into a
delete key in anything using a \`VT100 widget', which is the basis of
xterm's usual mode of operation:

      *VT100.Translations: #override \ 
      <Key>BackSpace: string(0x7F)

Part of the reason for showing this is that it makes zsh's key binding
system look wonderfully streamlined by comparison. However, tinkering
around at this level gives you very much more control over the use of
key modifiers (shift, alt, meta, control, and maybe even super and hyper
if you're lucky). This is far beyond the scope of this guide --- which I
say, as you probably realise by now, to cover up for not knowing much
about it. Here's another example from Oliver Kiddle, though; it uses
control with the left-cursor key to send an escape sequence: insert

      Ctrl<Key>Left: string(0x1b) string("[159q") \n\

into the middle of the example above --- this shows how multiple
definitions are handled. Modern xterms already send special escape
sequences which you can investigate and bind to as I've described.

### 4.5.4: Binding strings instead of commands

It's possible to assign an arbitrary string of characters to a key
sequence instead of an editor command by giving `bindkey` the option
`-s`. One of the good things about this is that the string of characters
are reinterpreted by zle, so they can contain active key sequences. In
the old days, this was quite often used as a basic form of macro, to
string together editor commands. For example, the following is a simple
way of moving backward two words by repeating the Emacs mode bindings.
I've used my F1 binding again; yours may be completely different.

      bindkey -s '\e[OP' '\eb\eb'

It's not a good idea to bind a key sequence to another string which
includes itself.

This method has the obvious drawback that if someone comes along and
rebinds \``\eb`', then F1 will stop working, too. Nowadays, this sort of
task can be done much more flexibly and clearly by writing a
user-defined widget, which is described in a later section. So bindings
of this sort are going a little out of fashion. However, they do provide
quick shortcuts. Two from Oliver Kiddle:

      bindkey -s '^[[072q' '^V^I'                       # Ctrl-Tab
      bindkey -s "\C-x\C-z" "\eqsuspend\n"

You can also quite easily do some of the things you can do with global
aliases.

Remember that \`ordinary' characters can be rebound, too; they just
usually happen to have a binding which makes them be inserted directly.
As a particularly pointless example, consider:

      bindkey -s secret 'Oh no!'

If you type \``secret`' fast enough the letters are swallowed up and
\``Oh no!`' appears instead. If you pause long enough anywhere in the
middle, the word is inserted just as normal. That's because all parts of
it can be interpreted as prefixes in their own right, so `$KEYTIMEOUT`
applies at every intervening stage. Less pointlessly, you could use this
as a way of defining abbreviations.

### 4.5.5: Keymaps

So far, all I've said about keymaps is that there are three standard
ones, one for Emacs mode and two for vi mode, and that \``bindkey -e`'
and \``bindkey -v`' pick Emacs or vi insert mode bindings. There's no
simple way of picking vi command mode bindings, since that's not usually
directly available but entered by the `vi-cmd-mode` command, usually
bound to `\e`, in vi insert mode. (There is a \``bindkey -a`', but that
doesn't pick the keymap for normal use; it's equivalent to, but less
clear than, \``bindkey -M vicmd`'.)

Most handling of keymaps is done through `bindkey`. The keymaps have
short names, `emacs`, `viins` and `vicmd`, for use with `bindkey`. There
is also a keymap `.safe` which you don't usually need but which never
changes, so can be used if your experimentation has completely ruined
every other keymap. It only has bindings for `self-insert` (most keys)
and `accept-line` (`^j` and `^m`), but that's enough to enter commands.

The names are most useful in two places. First, you can use
\``bindkey -M` *keymap*' to define keys in a particular map:

      bindkey -M vicmd "\e[OA" up-line-or-history

binds the usual up-cursor key in `vicmd` mode, whatever keymap is
currently set. Actually, any version of the shell which understands the
`-M` option probably has that bound already.

Secondly, you can force zle to use a particular keymap. This is done in
a slightly non-obvious way: zle always uses the keymap `main` as the
current keymap (except when it's off in vi command mode, which is
handled a bit specially). To use your own, you need to make `main` an
alias for that with \``bindkey -A`'. The order after this is the same as
that after `ln`: the existing keymap you want to refer to comes first,
then what you want to make an alias for it, in this case `main`. This
means that

      bindkey -A emacs main

has the same effect as

      bindkey -e

but is more explicit, if a little more baroque. Don't link `vicmd` to
main, since then you can't use `viins`, which is bad. Note that
\``bindkey -M emacs`' doesn't have this effect; it simply lists the
bindings in the `emacs` keymap.

You can create your own keymaps, too. The easiest way is to copy an
existing keymap, such as

      bindkey -N mymap emacs

which creates (or replaces) `mymap` and initialises it with the bindings
from `emacs`. Now you can use `mymap` just like `emacs`. The bindings in
each are completely separate. If you finish with a keymap, you can
remove it with \``bindkey -D keymap`', although you'd better make sure
it's not linked to `main` first.

You can omit \``emacs`' to create an empty keymap; this might be
appropriate if your keymap is only going to be used in certain special
places and you want complete control on what goes into it. Currently the
shell isn't very good at letting you apply your own keymaps just in
certain places, however.

There are various other keymaps you might encounter, used in special
circumstances. If you list all keymaps, which is done by
\``bindkey -l`', you may see `listscroll` and `menuselect`. These are
used by the new completion system, so if that isn't active, you probably
won't see them. They reside in the module `zsh/complist`. There will be
more about their effects in [chapter 6](zshguide06.html#comp);
`listscroll` allows you to move up and down completion lists which more
than fill the terminal window, and `menuselect` allows you to select
items interactively from a displayed list. You can bind keys in them as
with any other keymap.

4.6: Advanced editing
---------------------

(In physics, the \`advanced wave' is a hypothetical wave which moves
backwards in time. Unfortunately, however useful it would be to meet
deadlines, that's not what I meant by \`advanced editing'.)

Here are are a few bits and pieces which go beyond ordinary line editing
of shell commands. Although they haven't been widespread in shells up to
now, I use all of them every day, so they aren't just for the
postgraduate zsh scholar.

### 4.6.1: Multi-line editing

All Bourne-like shells allow you to edit continuation lines; that is, if
the shell can work out for sure that you haven't finished typing, it
will show you a new prompt, given by `$PS2`, and allow you to continue
where you left off from the previous line. In zsh, you can even see what
it is the shell is waiting for. For a simple example, type
\``array=`(`first`' and then \`return'. The shell is waiting for the
final parenthesis of the array, and prints \``array>`' at you, unless
you have altered `$PS2`. You can continue to add elements to the array
until you close the parentheses.

Shells derived from csh are less happy about continuation lines;
historically, this is because they try to evaluate everything in one go,
and became confused if they couldn't. The original csh didn't have a
particularly sophisticated parser. For once, zsh doesn't have an option
to match the csh behaviour; you just have to get used to the idea that
things work in zsh.

Where zsh improves over other shells is that you aren't just limited to
editing a single continuation line; you can actually edit a whole block
of lines on screen as you would in a full screen editor --- although you
can't scroll off the chunk of lines you're editing, which wouldn't make
sense.

The easiest way of doing this is to hit escape before you type a newline
at the point where you haven't finished typing. Actually, you can do
this any time, even if the line so far is complete. For example,

      % print This is line one<ESC><RET>
      print This is line two

where those angle brackets at the end of the line means you type escape,
then return. Nothing happens, and there is no new prompt; you just type
blithely on. Hit return, unescaped this time, and both lines will be
executed. Note there is no implicit backslash, or anything like that;
when zsh reads the whole caboodle, that escaped carriage return became a
real carriage return, just as the shell would have read it from a
script.

This works because \``\e\r`' is actually bound to the command
`self-insert-unmeta`' which means \`insert the character you get by
stripping the escape or top bit from what I just typed' --- in other
words, a literal carriage return. You would have got exactly the same
effect by typing `^v^j`, since the `^v` likewise escapes the `^j`
(newline), as it does any other character.

(Aside for the terminally curious only: Why newline here and not
carriage return --- the \`enter' key --- as you might expect? That's a
rather grotesque story. It turns out that for mostly historical reasons
UNIX terminal drivers like to swap newline and carriage return, so when
you type carriage return (sent both by that key and by `^m`, which is
the same as the character represented by `\r`), it comes out as newline
(on most keyboards, just sent by `^j`, which is the same as the
character represented by `\n`). It is the newline character which is the
one you \`see' at the end of the line (by virtue of the fact it is the
end of the line). However, `^v` sees through this and if you type `^m`
after it, it inserts a literal `^m`, which just looks like a `^m`
because that's how zsh outputs it. So that's why that doesn't work.
Actually, `self-insert-unmeta` would see the `^m`, too, because that's
what you get when you strip off the `\e`, but it has a little extra code
to make UNIX users feel at home, and behaves as if it were a newline.
Normally, `^j` and `^m` are treated the same way (`accept-line`), but
the literal characters have different behaviours. If you're now very
confused, just be thankful I haven't told you about the additional
contortions which go on when outputting a newline.)

It probably doesn't seem particularly useful yet, because all you've
done is miss out a new prompt. What makes it so is that you can now go
up and down between the two (or more) lines just using the cursor keys.
I'm assuming you haven't rebound the cursor keys, your terminal isn't a
dumb one which doesn't support cursor up, and the option `singlelinezle`
isn't in effect --- just unset it if it is, you'll be grateful later.

So for example, type

      % if [[ true = false ]]; then<ESC><RET>
        print Fuzzy logic rules<ESC><RET>
      fi

where I indented that second line just with spaces, because I usually do
inside an \`if'. There are no continuation prompts here, just the
original `$PS1`; that's not a misprint. Now, before hitting return, move
up two lines, and edit `false` to `true`. You can see how this can be
useful. Entering functions at the command line is probably a more
typical example.

Suppose you've already gone through a few continuation lines in the
normal way with `$PS2`'s? You can't scroll back then, even though the
block hasn't yet been edited. There's a magic way of turning all those
continuation lines into a single block: the editor command
`push-line-or-edit`. If you're not on a continuation line, it acts like
the normal `push-line` command, which we'll meet below, but for present
purpose you use it when you are on a continuation line. You are
presented with a seamless block of text from the (redrawn) prompt to the
end which you can edit as one. It's quite reasonable to bind
`push-line-or-edit` instead of `push-line`, to either `^q` or `\eq` (in
Emacs mode, which I will assume, as usual). Be careful with `^q`, though
--- if the option `flowcontrol` is set it will probably be swallowed up
by the terminal driver and not get through to the shell, the same
problem I mentioned above for `^s`.

### 4.6.2: The builtin vared and the function zed

I mentioned the `vared` command in [chapter 3](zshguide03.html#syntax);
it uses the normal line editor to editor a variable, typically a long
one you don't want to have to type in completely like `$path`, although
you need to remember *not* to put the \``$`' in front or the shell will
substitute it before `vared` is run. However, since it's just a piece of
text like any other input, this, too, can have multiple lines, which you
enter in the same way --- and since a shell parameter can contain
anything at all, you have a pretty general purpose editor. The shell
function \``zed`' is supplied with the shell and allows you to edit a
file using all the now-familiar commands. Since when editing files you
don't expect a carriage return to dump you out of the editor, just to
insert a new line, zed rebinds carriage return to `self-insert-unmeta`
(the \``-unmeta`' here is just to get the swapping behaviour of turning
the carriage return into a newline). To save and exit, you can type
`^j`, or, if your terminal does something odd with that, you can also
use `^x^w`, which is designed to look like Emacs' way of writing a file.

If you look at `zed`, you will see it has a few bells and whistles ---
for example, \``zed -f`' allows you to edit a function --- but the code
to read a file into a parameter, edit the parameter, and write the
parameter back to the file is extremely simple; all the hard editing
code is already handled within `vared`. Indeed, `zed` is essentially a
completely general purpose editor, though it quickly becomes inefficient
with long files, particularly if they are larger than a single screen;
as you would expect, zle was written to cope efficiently with short
chunks of text.

It would probably be nice if you could make key bindings that only
applied within vared by using a special keymap. That may happen one day.

By the way, note that you can edit arrays with vared and it will handle
the different elements sensibly. As usual, whitespace separates
elements; when it presents you with an array which contains whitespace
within elements, vared will precede it with a backslash to show it isn't
a separator. You can insert quoted spaces with backslashes yourself.
Only whitespace characters need this quoting, and only backslashes work.

For example,

      array=('one word' 'two or more words')
      vared array

presents you with \``one\ word two\ or\ more\ words`'. If you add
\`` and\ some\ more.`', hit return, and type \``print -l $array`' to
show one element per line you will see

      one word
      two or more words
      and some more.

Some older versions of the shell were less careful about spaces within
elements.

### 4.6.3: The buffer stack

The mysterious other use for `push-line-or-edit` will now be explained.
Let's stick to `push-line`, in fact, since I've already dealt with the
`-or-edit` bit.

Type

      print I was just in the directory

(no newline). Oh dear, which directory were you just in? You don't want
to interrupt the flow of text to find out. Hit \``\eq`'; the line you've
been typing disappears --- but don't worry, it hasn't gone. Now type

      dirs

Two things happen: that last line is executed, of course, showing the
list of directories on the directory stack (your use of `pushd` and
`popd`), but also the line you got rid of before has reappeared, so you
can continue to edit it.

You may not realise straight away quite how useful this is, but I used
it several times just while I was writing the previous paragraph. For
example, I was alternating directories between the zle source code and
the directory where I keep this guide, and I started typing a \``grep`'
command before realising I was in the wrong directory. All I need to do
is type `\eq`, then `pushd`, to put me where I want to be, and finish
off the `grep`.

The \`buffer stack', which is the jargon for this mechanism, can go as
deep as you like. It's a last-in-first-out (LIFO) stack, so the line
pushed onto it by the most recently typed `\eq` will reappear first,
followed by the back numbers in reverse order. You can even prime the
buffer stack from a function --- not necessarily a zle function, though
that works too --- with \``print -z` *command-line*'.

You can pull something explicitly off the stack, if you want, by typing
`\eg`, but that has the same effect as clearing the current line and
hitting return. You can of course push the same line multiple times: if
you need to do a whole series of things before executing it, just hit
`\eq` again each time the line pops back up.

I lied a little bit, to avoid confusion. The cleverness of
`push-line-or-edit` about multi-line buffers extends to the this case,
too. If you do a normal `push-line` on a multi-line buffer, only the
current single line is pushed; the command to push the whole lot, which
is probably what you want, is `push-input`. But if you have
`push-line-or-edit` bound, you can forget the distinction, since it will
do that for you. If you've been paying attention you can work out the
following sequence (assuming `\eq` has been rebound to
`push-line-or-edit`):

      % if [[ no = yes ]]; then
      then> print<ESC>q<ESC>q

The first `\eq` turns the two lines into a single buffer, then the
second pushes the whole lot onto the buffer stack. This saves a lot of
thinking about bindings. Hence I would recommend users of Emacs mode add

      bindkey '\eq' push-line-or-edit

to their `.zshrc` and forget the distinctions.

4.7: Extending zle
------------------

We now come to the newest and most flexible part of zle, the ability to
create new editing commands, as complicated as you like, using shell
functions. This was originally introduced by Andrew Main (\`Zefram') in
zsh 3.1 and so is standard in all versions of zsh 4, although work goes
on.

### 4.7.1: Widgets

If you don't speak English as you first language, first of all,
congratulations for getting this far. Secondly, you may think of
\`widget' only as a technical word applied to the object which realises
some computational idea, like the thing that implements text editing in
a window system, for example. However, to most English speakers,
\`widget' is a humorous word for an object, a bit like
\`whatyoumacallit' or \`thingummybob', as in \`where's that clever
widget that undoes the foil and takes out the cork in one go'. Zsh's use
has always seemed to me closer to the second, non-technical version, but
I may be biased by the fact that the internal object introduced by
Zefram to represent a widget, and never seen by the user, is called a
\`thingy', which I won't refer to again since you don't need to know.

Anyway, a \`widget' is essentially what I've been calling an editor
command up to now, something you can bind to a key sequence. The reason
the more precise terminology is useful is that as soon as you have shell
functions flying around, the word \`command' is hopelessly non-specific,
since functions are full of commands which may or may not be widgets. So
I make no apology for using the word.

So now we are introducing a second type of widget: one which, instead of
something handled by code built into the shell, is handled by a function
written by the user. They are completely equivalent; `bindkey` and
company don't care which it is. All you need to do to create a widget is

      zle -N widget-name function-name

then *widget-name* can be used in `bindkey`, or `execute-named-cmd`, and
the function *function-name* will be run. If the `widget-name` and
`function-name` are the same, which is often the simplest thing to do,
you just need one of them.

You can list the existing widgets by using \``zle -l`', although often
\``zle -lL`' is a better choice since the output format is then the same
as the form you would use to define the widget. If you see lots of
\``zle -C`' widgets when you do that, ignore them for now; they are
completion widgets, handled a bit differently and described in [chapter
6](zshguide06.html#comp).

Now you need to know what should go into the function.

### 4.7.2: Executing other widgets

The simplest thing you can do inside a function implementing a widget is
call an existing function. So,

      my-widget() {
        zle backward-word
      }
      zle -N my-widget

creates a widget called `my-widget` which behaves in every respect
(except speed) like the builtin widget `backward-word`. You can even
give it a prefix argument, which is passed down; `\e3` then whatever you
bound the widget to (or `\exmy-widget`) will go backward three words.

Suppose you wanted to pass your own prefix argument to `backward-word`,
instead of what the user typed? Or suppose you want to take account of
the prefix argument, but do something different with it? Both are
possible.

Let's take the first of those. You can supply a prefix argument for this
command alone by putting `-n` *argument* after the widget name (note
this is not where most options go).

      my-widget() {
        zle backward-word -n 2
      }

This always goes backwards two words, overriding any numeric argument
given by the user. (You can redefine the function without telling zle
about it, by the way; zle just calls whatever function happens to be
defined when the widget is run.) If you put just `-N` after the name
instead, it will cancel out any prefix given by the user, without
introducing a new one.

The other part of prefix handling --- intercepting the one the user
specified and maybe modifying it --- introduces one of the most
important parts of user-defined widgets. Zle provides various parameters
which can be read and often written to alter the behaviour of the editor
or even the text being edited. In this case, the parameter is `$PREFIX`.
For example,

      my-widget() {
        zle backward-word -n $(( ${NUMERIC:-1} * 2 ))
      }

This uses an arithmetic substitution to provide an argument to
`backward-word` which is twice what the user gave. Note that
`${NUMERIC:-1}` notation, which is important: most of the time, you
don't give a numeric argument to a command at all, and in that case zle
naturally enough treats `$NUMERIC` as if it wasn't set. This would mess
up the arithmetic substitution.

By the way, if you do make an error in a shell function, you won't see
it; you'll just get a beep, unless you've turned that off with
`setopt nobeep`. The output from such functions is junked, since it
would mess up the display. So you should do any basic debugging before
turning the function into a widget, for example, stick a `print` in
front and run it directly --- you can't execute widgets from outside the
editor.

The following also works:

      my-widget() {
        (( NUMERIC = ${NUMERIC:-1} * 2 ))
        zle backward-word
      }

because you can alter `$NUMERIC` directly, and unless overridden by the
`-n` argument it is used by any widgets called from the function. If you
called more widgets inside the function --- and you can call as many as
you like --- the same argument would apply to all the ones that didn't
have an explicit `-n` or `-N`.

Some widgets allow you to specify non-numeric arguments. At the moment
these are mainly search functions, which you can give an explicit search
string. Usually, however, you want to specify a new search string each
time. The most useful way of using this I can see is to provide an
initial argument for incremental search commands. Later, I'll show you
how you can read in characters in a similar fashion to Emacs mode's `^r`
binding, `history-incremental-search-backwards`.

### 4.7.3: Some special builtin widgets and their uses

There are some things you might want to do with the editor in a zle
function which wouldn't be useful executed directly from zle. One is to
cause an error in the same way as a normal widget does. You can do that
with \``zle beep`'. However, this doesn't automatically stop your
function at that point; it's up to you to return from it.

It's possible to redefine a builtin widget just by declaring it with
\``zle -N`' and defining the corresponding function. From now on, all
existing bindings which refer to that widget will cause yours to be run
instead of the builtin one. This happens because zle doesn't actually
care what a widget does until it is run. You can see this by using
`bindkey` to define a key sequence to call an undefined widget such as
`any-old-string`. The shell doesn't complain until you actually hit the
key sequence.

Sometimes, however, you want to be sure to call the builtin widget, even
if the behaviour has been redefined. You can do this by putting a \``.`'
in front of the name of the widget; \``zle .up-line-or-history`' always
calls the builtin widget usually referred to as `up-line-or-history`,
even if the latter has been redefined. One use for this is to rebind
\``accept-line`' to do something whenever zle is about to pass a line up
to the shell, but to accept the line anyway: you write your own widget
`accept-line`, make sure it calls \``zle .accept-line` just before it
finishes, and then use \``zle -N accept-line`. Here's a trivial but not
entirely stupid example:

      accept-line() {
        print -n "\e]2;Executing $BUFFER\a"
        zle .accept-line
      }
      zle -N accept-line

Now every time you hit return to execute a command, that `print` command
will be executed first. As written, it puts \``Executing`' and then the
contents of the command line (see below) into the title of your xterm
window, assuming it understands the usual xterm escape sequences. In
fact, this particular example is usually handled with the special shell
function (not zle function) \``preexec`' which is passed a command line
about to be executed as an argument instead of in `$BUFFER`. There seems
to be a side effect of rebinding `accept-line` that the return key stops
working in the minibuffer under some circumstances.

Note that to undo the fact that return executes your new widget, you
need to alias `accept-line` back to `.accept-line`:

      zle -A .accept-line accept-line

If you have trouble remembering the order, as with most alias or rename
commands in zsh and UNIX generally, including `ln` and `bindkey -A`, the
existing command, the one whose properties you want to keep, comes
first, while the new name for it comes second. Also, as with those
commands, it doesn't matter if the second name on the line currently
means something else; that will be replaced by the new meaning.
Afterwards, you don't need to worry about your own `accept-line` widget;
zle handles the details of removing widgets when they're no longer
referred to. The function's still there, however, since as far as the
rest of the shell is concerned it's just an ordinary shell function
which you need to \``unfunction`' to remove.

Do remember, however, not to delete a widget which redefines a basic
internal widget by the obvious command

      # Noooo!
      zle -D accept-line

which stops the return key having any effect other than complaining
there's no such widget. If you get into real trouble,
\``\ex.accept-line`' should work, as you can use the \``.`'-widgets
anywhere you can use any other except where they would redefine or
delete a \``.`' widget. Use the \``zle -A`' command above with the
extended-command form of \``.accept-line`' to return to normality. If
you try to redefine or delete a \``.`' widget, zle will tell you it's
protected. You can remove any other widget in this way, however, even if
it is still bound to a key sequence; you will then see an error if you
type that sequence.

One point to note about `accept-line` is that the line isn't passed up
to zsh instantly, only when your own function exits. This is pretty
obvious when you think about it; zle is called from the main shell, and
if your own zle widget hasn't finished executing, the main shell hasn't
got control back yet. But it does mean, for example, that if you modify
the command line after a call to `accept-line` or `.accept-line`, those
changes are reflected in the line passed up to the shell:

      # Noooo! to this one too.
      accept-line() {
        zle .accept-line
        BUFFER='Ha ha!'
      }

This always returns the string \``Ha ha!`' to the main shell. This is
not particularly useful unless you are constructing a Samuel Beckett
shell for display at an installation in a Parisian art gallery.

### 4.7.4: Special parameters: normal text

The shell makes various parameters available for easy manipulation of
the command line. You've already seen `$NUMERIC`. You may wonder what
happens if you have your own parmeter called `$NUMERIC`; after all, it's
a fairly simple string to use as a name. The good news is you don't need
to worry; when the shell runs a zle function, it simply hides any
existing occurrences of a parameter and makes its special parameters
available. Then when it exits, the original parameter is reenabled. So
all you have to worry about is making sure you don't use these special
parameters for anything else while you are inside a zle widget.

There are four particularly common zle parameters.

First, there are three ways of referring to the text on the command
line: `$BUFFER` is the entire line as a string, `$LBUFFER` is the line
left of the cursor position, and `$RBUFFER` is the line after it
including the character under the cursor, so that the division is always
at the point where the next inserted character would go. Any or all of
these may be empty, and `$BUFFER` is always the string
`$LBUFFER$RBUFFER`.

The necessary counterpart to these is `$CURSOR`, which is the cursor
position with 1 being the first character. If you know how the shell
handles substrings in parameter substitutions, you will be able to see
that `$LBUFFER` is `$BUFFER[1,$CURSOR-1]`, while `$RBUFFER` is
`$BUFFER[$CURSOR,-1]` (unless you are using the option `KSH_ARRAYS` for
compatibility of indexes with ksh --- this isn't recommended for
implementing zle or completion widgets as it causes confusion with the
ones supplied with the shell).

The really useful thing about these is that they are modifiable. If you
modify `$LBUFFER` or `$RBUFFER`, then `$BUFFER` and `$CURSOR` will be
modified appropriately; lengthening or shortening `$LBUFFER` increases
or decreases `$CURSOR`. If you modify `$BUFFER`, you may need to set
`$CURSOR` yourself as the shell can't tell for sure where the cursor
should be. If you alter `$CURSOR`, characters will be moved between
`$LBUFFER` and `$RBUFFER`, but `$BUFFER` will remain the same.

This makes tasks along the lines of basic movement and deletion commands
extremely simple, often just a matter of pattern matching. However, it
definitely pays to know about zsh's more sophisticated pattern matching
and parameter substitution features, described in the next chapter. For
example, if you start a widget function with

      emulate -L zsh
      setopt extendedglob
      LBUFFER=${LBUFFER%%[^[:blank:]]##}

then `$LBUFFER` contains the line left of the cursor stripped of all the
non-blank characters (usually anything except space or tab) immediately
to the left of the cursor.

This function uses the parameter substitution feature
\``${`*param*`%%`*pattern*`}`' which removes the longest match of
*pattern* from the end of `$`*param*. The \``emulate -L zsh`' ensures
the shell options are set appropriately for the function and makes all
option settings local, and \``setopt extendedglob`' which turns on the
extended pattern matching features; it is this that makes the sequence
\``##`' appearing in the pattern mean \`at least one repetition of the
previous pattern element'. The previous pattern element is \`anything
except a blank character'. Hence, all occurrences of non-blank
characters are removed from the end of `$LBUFFER`.

If you want to move the cursor over those characters, you can tweak the
function slightly:

      emulate -L zsh
      setopt extendedglob
      chars=${(M)LBUFFER%%[^[:blank:]]##}
      (( CURSOR -= ${#chars} ))

The string \``(M)`' has appeared at the start of the parameter
substitution. This is part of zsh's unique system of parameter flags;
this one means \`insert the matched portion of the substitution'. In
other words, instead of returning `$LBUFFER` stripped of non-blank
characters at the end, the substitution returns those very characters
which it would have stripped. To skip over them is now a simple matter
of decreasing `$CURSOR` by the length of that string.

You'll find if you try these examples that they probably don't do quite
what you want. In particular, they don't handle any blank characters
found next to the non-blank ones which normal word-orientated functions
do. However, you now have enough information to add tests for that
yourself.

If you get more sophisticated, you can then add handling for `$NUMERIC`.
Remember this isn't set unless the user gave it explicitly, so it's up
to you to treat it as 1 in that case.

### 4.7.5: Other special parameters

A large fraction of what you are likely to want to do can be done with
the parameters we've already met. Here are some hints as to how you
might want to use some of the other parameters available. As always, for
a complete list with rather less in the way of hints see the manual.

`$KEYS` tells you the keys which were used to call the widget; it's a
string of those raw characters, not turned into the `bindkey` format. In
other words, if it was a single key (including possibly a control key or
a meta key), `$KEYS` will just contain a single character. So you can
change the widget's behaviour for different keys. Here's a very (very)
simple function like `self-insert`:

      LBUFFER=$LBUFFER$KEYS

Note this doesn't work very well with `\ex` extended command handling;
you just get the `^m` from the end of the line. You need to make sure
any widgets which use `$KEYS` are sensibly bound. This also doesn't
handle numeric arguments to repeat characters; it's a fairly simple
exercise (particularly given zsh's \``repeat`' loop) to add that.

`$WIDGET` and `$LASTWIDGET` tell you the name of the current widget
being executed and the one before that. These don't sound all that
useful at first hearing. However, you can use `$WIDGET` together with
the fact that a widget doesn't need to have the same name as the
function that defines it. You can define

      zle -N this-widget function
      zle -N that-widget function

and test `$WIDGET` inside `function` to see if it contains `this-widget`
or `that-widget`. If these have a lot of shared code, that is a
considerable simplification without having to write extra functions.

`$LASTWIDGET` tends to be used for a slightly different purpose:
checking whether the last command to be executed was the same as the
current one, or maybe was just friendly with it. Here are edited
highlights of the function `up-line-or-beginning-search`, a sort of
cross between `up-line-or-search` and
`history-beginning-search-backward` which has been added to the shell
distribution for `4.1`. If there are previous lines in the buffer, it
moves up through them; else if it's the first in a sequence of calls to
this function it remembers the cursor position and looks backwards for a
line with the same text from the start up to that point, and puts the
cursor at the end of the line; else if the same widget has just been
executed, it uses the old cursor position to search for another match
further back in the history.

      if [[ $LBUFFER == *$'\n'* ]]; then
        zle .up-line-or-history
        __searching=''
      else
        if [[ $LASTWIDGET = $__searching ]]; then
          CURSOR=$__savecursor
        else
          __savecursor=$CURSOR
        fi
        __searching=$WIDGET
        zle .history-beginning-search-backward
        zle .end-of-line
      fi

We test `$__searching` instead of `$WIDGET` directly to be able to tell
the case when we are moving lines instead of searching. `$__savecursor`
gives the position for the backward search, after which we put the
cursor at the end of the line. The parameters beginning \``__`' aren't
local to the function, because we need to test them from the previous
execution, so they have been given underscores in front to try to
distinguish them from other parameters which might be around.

You'll see that the actual function supplied in the distribution is a
little more complicated than this; for one thing, it uses styles set by
the user to decide it's behaviour. Styles are described for use with
completion widgets in [chapter 6](zshguide06.html#comp), but you can use
them exactly the same way in zle functions.

The full version of `up-line-or-beginning-search` uses another
parameter, `$PREBUFFER`. This contains any text already absorbed by
`zle` which you can no longer edit --- in other words, text read in
before the shell prompted with `$PS2` for the remainder. Testing
\``[[ -n $PREBUFFER ]]`' therefore effectively tests whether you are at
the `$PS2`. You can use this to implement behaviour after the fashion of
`push-line-or-edit`.

### 4.7.6: Reading keys and using the minibuffer

Every now and then you want the editor to do a sequence of operations
with user input in the middle. This is usually done by a combination of
two commands.

First, you may need to prompt the user in the minibuffer, just like
`\ex` does. You can do this with \``zle -R`'. Its basic function is to
redisplay the command line, flushing all the changes you have made in
your function so far, but you can give it a string argument which
appears in the minibuffer, just below the command line. You can give it
a list of other strings after that, which appear in a similar way to
lists of possible completions, but have no special significance to zle
in this case.

To get input back from the user, you can use \``read -k`' which reads a
single key (not a sequence; no lookup takes place). This command is
always available in the shell, but in this case it is handled by zle
itself. The key is returned as a raw byte. Two facilities of arithmetic
evaluation are useful for handling this key: \``#key`' returns the ASCII
code for the first character of `$key`, while \``##`*key*' returns the
ASCII code for *key*, which is in the form that `bindkey` would
understand. For example,

      read -k key
      if (( #key == ##\C-g )); then
         ...

makes the use of arithmetic evaluation. The form on the left turns the
first character in `$key` into a number, the second turns the literal
bindkey-style string `\C-g` into a number (ASCII 7, since 1 to 26 are
just `\C-a` to `\C-z`). Don't confuse either of these forms with
\``$#key`', which is the length of the string in the parameter, in this
case almost certainly 1 for a single byte; this form works both inside
and outside arithmetic substitution, the other forms only inside. The
\``(( ... ))`' form is recommended for arithmetic substitutions whenever
possibly; you can do it with the basic \``[[ ... ]]`' form, since
\``-eq`' and similar tests treat both sides as arithmetic, though you
may need extra quoting; however, the only good reason I know for doing
that is to avoid using two types of condition syntax in the same complex
test.

These tricks are only really useful for quite complicated functions. For
an example, look at the function `incremental-complete-word` supplied
with the zsh source distribution. This function doesn't add to clarity
by using the form \``#\\C-g`' instead of \``##\C-g`'; it does the same
thing but the double backslash is very confusing, which is why the other
form was introduced.

### 4.7.7: Examples

**transpose-words-about-point**\
\

This function is a variant on `transpose-words`. It has various twists.
First, the words in question are always space-delimited, neither shell
words nor words in the `$WORDCHARS` sense. This makes it fairly
predictable.

Second, it will transpose words about the current point (hence its name)
even if the character under the cursor is not a whitespace character. I
find this useful because I am eternally typing compound words a bit like
\``function_name`' only to find that what I should have typed was
\``name_function`'. Now I just position the cursor over the underscore
and execute this widget.

      emulate -L zsh
      setopt extendedglob

      local match mbegin mend pat1 pat2 word1 word2 ws1 ws2

      pat1=${LBUFFER%%(#b)([^[:blank:]]##)([[:blank:]]#)}
      word1=$match[1]
      ws1=$match[2]

      match=()
      pat2=${RBUFFER##(#b)(?[[:blank:]]#)([^[:blank:]]##)}
      ws2=$match[1]
      word2=$match[2]

      if [[ -n $word1 && -n $word2 ]]; then
        LBUFFER="$pat1$word2$ws1"
        RBUFFER="$ws2$word1$pat2"
      else
        zle beep
      fi

The only clever stuff here is the pattern matching. It makes a great
deal of use of \`backreferences' an extended globbing feature which is
used in all forms of pattern matching including, as in this case,
parameter substitution. It will be described fully in the next chapter.
The key things to look for are the \``(#b)`', which activates
backreferences if the option `EXTENDED_GLOB` is turned on, the
parentheses following that, which mark out the bits you want to refer
to, and the references to elements of the array `$match`, which store
those bits. The shell also sets `$mbegin` and `$mend` to give the
positions of the start and end of those matches, which is why those
parameters are made local; we want to preserve them from being seen
outside the function even though we don't actually use them.

You might also need to know about the \``#`' characters: one after a
pattern means \`zero or more repetitions', and two mean \`one or more
repetitions'. Finally, \``[:blank:]`' in a character class refers to any
blank character; when negated, as in the character class
\``[^[:blank:]]`', it means any non-blank character. With the \``#`'s we
match a series blank or non-blank characters. Given that, you can work
out the rest of what's going on.

Here's a more sophisticated version of that. If you found the previous
one heavy going. you probably don't want to look too closely at this.

      emulate -L zsh
      setopt extendedglob

      local wordstyle blankpat wordpat1 wordpat2
      local match mbegin mend pat1 pat2 word1 word2 ws1 ws2

      zstyle -s ':zle:transpose-words-about-point' word-style wordstyle

      case $wordstyle in
        (shell) local bufwords
                # This splits the line into words as the shell understands them.
                bufwords=(${(z)LBUFFER})
                wordpat1="${(q)bufwords[-1]}"
                # Take substring of RBUFFER to skip over first character,
                # which is the one under the cursor.
                bufwords=(${(z)RBUFFER[2,-1]})
                wordpat2="${(q)bufwords[1]}"
                blankpat='[[:blank:]]#'
                ;;
        (space) blankpat='[[:blank:]]#'
                wordpat1='[^[:blank:]]##'
                wordpat2=$wordpat1
                ;;
        (*) local wc=$WORDCHARS
            if [[ $wc = (#b)(?*)-(*) ]]; then
              # We need to bring any `-' to the front to avoid confusing
              # character classes... we get away with `]' since in zsh
              # this isn't a pattern character if it's quoted.
              wc=-$match[1]$match[2]
            fi
            # A blank is anything not in the character class consisting
            # of alphanumerics and the characters in $wc.
            # Quote $wc where necessary, because we don't want those
            # characters to be considered as pattern characters later on.
            blankpat="[^${(q)wc}a-zA-Z0-9]#"
            # and a word character is anything else.
            wordpat1="[${(q)wc}a-zA-Z0-9]##"
            wordpat2=$wordpat1
            ;;
      esac

      # The eval makes any special characters in the parameters active.
      # In particular, we need the surrounding `[' s to be `real'.
      # This is why we quoted the wordpats in the `shell' option, where
      # they have to be treated as literal strings at this point.
      eval pat1='${LBUFFER%%(#b)('${wordpat1}')('${blankpat}')}'
      word1=$match[1]
      ws1=$match[2]

      match=()
      eval pat2='${RBUFFER##(#b)(?'${blankpat}')('${wordpat2}')}'
      ws2=$match[1]
      word2=$match[2]

      if [[ -n $word1 && -n $word2 ]]; then
        LBUFFER="$pat1$word2$ws1"
        RBUFFER="$ws2$word1$pat2"
      else
        zle beep
      fi

What has been added is the ability to use a style to define how the
shell finds a \`word'. By default, words are the same as what the shell
usually thinks of as a word; this is handled by the branch of the case
statement which uses \``$WORDCHARS`' and a little extra trickery to get
a pattern which matches the set of characters considered parts of a
word. We used the `eval`'s because it allowed us to have some bits of
`$wordpat1` and friends active as pattern characters while others were
quoted.

This introduces two types of parameter expansion flags:
`${(q)`*param*`}` adds backslashes to quote special characters in
`$`*param*, so that when the parameter appears after `eval` the result
is just the original string. `${(z)`*param*`}` splits the parameter just
as if it were a shell command line being split into a command and words,
so the result is an array; \``z`' stands for zsh-splitting or just
zplitting as you fancy.

If you set

      zstyle ':zle:*' word-style space

you get back to the behaviour of the original function.

Finally, if you replace \``space`' with \``shell`' in that `zstyle`
command, you will get words as they are split for normal use within the
shell; for example try

      echo execute the widget 'between these' 'two quoted expressions'

and the entire quoted expressions will be transposed. You may find that
if you do this in the middle of a quoted expression, you don't get a
sensible result; that's because the `(z)`-splitting doesn't know what to
do with the improperly completed quotes to its left and right. Some
versions of the shell have a bug (fixed in 4.0.5) that the expressions
which couldn't be split properly, because the quotes weren't complete,
have an extra space character at the end.

**insert-numeric**\
\

Here's a widget which allows you to insert an ASCII character which you
know by number. I can't for the life of me remember where it came from,
but it's been lying around apparently for two and a half years (please
do email me if you think you wrote it, otherwise I'll assume I did). You
can give it a numeric prefix (that's the easy part of the function),
else it will prompt you for a number. If you type \``x`' or \``o`' the
number is treated as hexadecimal or octal, respectively, else as
decimal.

      # Set up standard options.
      # Important for portability.
      emulate -L zsh
      
      # x must display in hexadecimal
      typeset -i 16 x
      
      if (( ${+NUMERIC} )); then
        # Numeric prefix given; just use that.
        x=$NUMERIC
      else
        # We need to read the ASCII code.
        local msg modes key mode=dec code char
        # Prompt for and read a base.
        integer base=10
        zle -R "ASCII code (o -> oct, x -> hex) [$mode]: "
        read -k key
        case $key in
          (o) base=8 mode=oct
              zle -R "ASCII code [$mode]: "
              read -k key
              ;;
          (x) base=16 mode=hex
              zle -R "ASCII code [$mode]: "
              read -k key
             ;;
        esac
        # Now we are looking for numbers in that base.
        # Loop until newline or return.
        while [[ '#key' -ne '##\n' && '#key' -ne '##\r' ]]; do
          if [[ '#key' -eq '##^?' || '#key' -eq '##^h' ]]; then
            # Delete a character
            [[ -n $code ]] && code=${code[1,-2]}
          elif [[ ($mode == hex && $key != [0-9a-fA-f]) ||
                ($mode == dec && $key != [0-9]) ||
                ($mode == oct && $key != [0-7]) ]]; then
            # Character not in range, beep
            zle beep
          elif [[ '#key' -eq '##\C-g' ]]; then
            # Abort: returning 1 signals to zle that this
            # is an abnormal termination.
            return 1
          else
            code="${code}${key}"
          fi
          char=
          if [[ -n $code ]]; then
            # Work out the character using the
            # numbers typed so far.
            (( x = ${base}#${code} ))
            if (( x > 255 )); then
              zle beep
              code=${code[1,-2]}
              [[ -n $code ]] && (( x = ${base}#${code} ))
            fi
            [[ -n $code ]] && eval char=\$\'\\x${x##???}\'
          fi
      
          # Prompt for any more digits, showing
          # the character as it would be inserted.
          zle -R "ASCII code [$mode]: $code${char:+ = $char}"
          read -k key || return 1
        done
        # If aborted with no code, return
        [[ -z $code ]] && return 0
        # Now we have the ASCII code.
        (( x = ${base}#${code} ))
      fi
      
      # Finally, if we have a single-byte character,
      # insert it to the left of the cursor
      if (( x < 0 || x > 255 )); then
        return 1
      else
        eval LBUFFER=\$LBUFFER\$\'\\x${x##???}\'
      fi

This shows how to do interactive input. The \``zle -R`'s prompt the
user, while the \``read -k`'s accept a character at a time. As an extra
feature, while you are typing the number, the character that would be
inserted if you hit return is shown. The widget also handles deletion
with backspace or the (UNIX-style, not PC-style) delete key.

One blight on this is the way of turning the number in x into a
character, which is done by all those `eval`s and backslashes. It uses
the feature that e.g. `$'\x41'` is the character `0x41` (an ASCII \`A').
To use this, we must make sure the character (stored in x) appears as
hexadecimal, and following ksh zsh outputs hexadecimal numbers as
\``16#41`' or similar. (The new option `C_BASES` shows hexadecimal
numbers as 0x41 and similar, but here we need the plain number in any
case.) Hence we strip the \``16#`' and construct our `$'\x41'`. Now we
need to persuade the shell to interpret this as a quoted string by
passing it to `eval` with the special characters (`$`, `\`, `'`) quoted
with a backslash so that they aren't interpreted too early.

By the way, note that zsh only handles ordinary 8-bit characters at the
moment. It doesn't matter if some do-gooder on your system has set
things up to use UTF-8 (a UNIX-friendly version of the international
standard for multi-byte characters, Unicode) to appeal to the
international market, I'm afraid zsh is stuck with ISO 8859 and similar
character sets for now.

