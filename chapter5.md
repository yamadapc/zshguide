

Chapter 5: Substitutions
========================

This chapter will appeal above all to people who are excited by the fact
that

      print ${array[(r)${(l.${#${(O@)array//?/X}[1]}..?.)}]}

prints out the longest element of the array `$array`. For the
overwhelming majority that forms the rest of the population, however,
there should be plenty that is useful before we reach that stage.
Anyway, it should be immediately apparent why there is no obfuscated zsh
code competition.

For those who don't do a lot of function writing and spend most of the
time at the shell prompt, the most useful section of this chapter is
probably that on filename generation (i.e. globbing) at the end of the
chapter. This will teach you how to avoid wasting your time with `find`
and the like when you want to select files for a command.

5.1: Quoting
------------

I've been using quotes of some sort throughout this guide, but I've
never gone into the detail. It's about time I did, since using quotes is
an important part of controlling the effects of the shell's various
substitutions. Here are the basic quoting types.

### 5.1.1: Backslashes

The main point to make about backslashes is that they are really
trivial. You can quote any character whatsoever from the shell with a
backslash, even if it didn't mean anything unquoted; so if the worst
comes to the worst, you can take any old string at all, whatever it has
in it --- random collections of quotes, backslashes, unprintable
characters --- quote every single character with a backslash, and the
shell will treat it as a plain string:

      print \T\h\i\s\ \i\s\ \*\p\o\i\n\t\l\e\s\s\*\ \ 
          \-\ \b\u\t\ \v\a\l\i\d\!

Remember, too that, this means you need an extra layer of quotation to
pass a \``\n`', or whatever, down to `print`.

However, zsh has an easier way of making sure everything is quoted with
a backslash when that's needed. It's a special form of parameter
substitution, just one of many tricks you can do by supplying flags in
parentheses:

      % read string
      This is a *string* with various `special' characters
      % print -r -- ${(q)string}
      This\ is\ a\ \*string\*\ with\ various\ \`special\'\ characters

The `read` builtin didn't do anything to what you typed, so `$string`
contains just those characters. The `-r` flag to print told it to print
out what came after it in raw fashion, and here's the special part:
`${(q)string}` tells the shell to output the parameter with backslashes
where needed to prevent special characters being interpreted. All
parameter flags are specific to zsh; no other shell has them.

The flag is not very useful there, because zsh usually (remember the
`GLOB_SUBST` option?) doesn't do anything special to characters from
substitutions anyway. Where it *is* extremely useful is if you are going
to re-evaluate the text in the substitution but still want it treated as
a plain string. So after the above,

      % eval print -r -- ${(q)string}
      This is a *string* with various `special' characters

and you get back what you started with, because at the `eval` of the
command line the backslashes put in by the `(q)` flag meant that the
value was treated as a plain string.

You can strip off quotes in parameters, too; the flag `(Q)` does this.
It doesn't care whether backslashes or single or double quotes are used,
it treats them all the way the shell's parser would. You only need this
when the parameter has somehow acquired quotes in its value. One way
this can happen is if you try reading a file containing shell commands,
and for this there's another trick: the `(z)` flag splits a line into an
array in the same way as if the line had been read in and was, say,
being assigned to an array. Here's an example:

      % cat file
      print 'a quoted string' and\ another\ argument
      % read -r line <file
      % for word in ${(z)line}; do
      for> print -r "quoted:    $word"
      for> print -r "unquoted:  ${(Q)word}"
      for> done
      quoted:    print
      unquoted:  print
      quoted:    'a quoted string'
      unquoted:  a quoted string
      quoted:    and\ another\ argument
      unquoted:  and another argument

You will notice that the `(z)` doesn't remove any of the quotes from the
words read in, but the `(Q)` flag does. Note the `-r` flags to both
`read` and `print`: the first prevents the backslashes being absorbed by
`read`, and the second prevents them being absorbed by `print`. I'm
afraid backslashes can be a bit of a pain in the neck.

### 5.1.2: Single quotes

The only thing you can't quote with single quotes is another single
quote. However, there's an option `RC_QUOTES`, where two single quotes
inside a single-quoted string are turned into one. Apparently \``RC`'
refers to the shell `rc` which appeared in plan9; it seems to be one of
those programmes that some people get fanatically worked up about while
the rest of us can't quite work out why. Zsh users may sympathise. (This
was corrected by Oliver Kiddle and Bart Schaefer after I guessed
incorrectly that `RC` stood for recursive, although you're welcome to
think of it that way anyway. It doesn't really work for
`RC_EXPAND_PARAM`, however, which is definitely from the `rc` shell, and
if you look at the source code you will find a variable called
\``plan9`' which is tested to see if that option is in effect.)

You might remember something like this from BASIC, although in that case
with double quotes --- in zsh, it works only with single quotes, for
some reason. So,

      print -r 'A ''quoted'' string'

would usually give you the output \``A quoted string`', but with the
option set it prints \``A 'quoted' string`'. The `-r` option to `print`
doesn't do anything here, it's just to show I'm not hiding anything.
This is usually a useful and harmless option to have set, since there's
no other good reason for having two quotes together within quotes.

The standard way of quoting single quotes is to end the quote, insert a
backslashed single quote, and restart quotes again:

      print -r 'A '\''quoted'\'' string'

which is unaffected by the option setting, since the quotes immediately
after the backslashes are always treated as an ordinary printable
character. What you *can't* ever do is use backslashes as a way of
quoting characters inside single quotes; they are just treated as
ordinary characters there.

You can make parameter flags produce strings quoted with single quotes
instead of backslashes by doubling the \``q`': \``${(qq)param}`' instead
of \``${(q)param}`'. The main use for this is that the result is shorter
if you know there are a lot of special characters in the string, and
it's also a bit more easy to read for humans rather than machines, but
usually it gains nothing over the other form. It can tell whether you
have `RC_QUOTES` set and uses that to make the string even shorter, so
be careful if you might use the resulting string somewhere where the
option isn't set.

### 5.1.3: POSIX quotes

There's a relative of single quotes which uses the syntax `$'` to
introduce a quoted string and `'` to end it; I refer to them as \`POSIX
quotes' because they appear in the POSIX standard and I don't know what
else to call them; \`string quotes' is one possibility, but sounds a bit
vague (what else would you quote?) The difference from single quotes is
that they understand the same backslash sequences as the print builtin.
Hence you can have the convenience of using \``\n`' for newline, \``\e`'
for escape, \``\xFF`' for an arbitrary character in hexadecimal, and so
on, for any command:

      % cat <<<$'Line\tone\nLine\ttwo'
      Line    one
      Line    two

Remember the \`here string' notation \``<<<`', which supplies standard
input for the command. Hence the output shows exactly how the quoted
string is being interpreted. It is the same as

      % print 'Line\tone\n\Line\ttwo'
      Line    one
      Line    two

but there the interpretation is done inside `print`, which isn't always
convenient. POSIX quotes are currently rather underused.

This is as good a point as any to mention that the shell is completely
\`eight-bit clean', which means you can have any of the 256 possible
characters anywhere in your string. For example, `$'foo\000bar'` has an
embedded ASCII NUL in it (that's not a misprint --- officially, ASCII
non-printing characters have two- or three-letter abbreviations).
Usually this terminates a string, but the shell works around this when
you are using it internally; when you try and pass it as an argument to
an external programme, however, all bets are off. Almost certainly the
first NUL in that case will cause the programme to think the string is
finished, because no information about the length of arguments is passed
down and there's nothing the shell can do about it. Hence, for example:

      % echo $'foo\000bar'
      foobar
      % /bin/echo $'foo\000bar'
      foo

The shell's `echo` knows about the shell's 8-bit conventions, and prints
out the NUL, which the terminal doesn't show, then the remainder of the
string. The external version of `echo` didn't know any better than to
stop when it reached the NUL.

There are actually uses for embedded NULs: some versions of `find` and
`xargs`, for example, will put or accept NULs instead of newlines
between their bits of input and output (as distinct from command line
arguments), which is much safer if there's a chance the input or output
can contain a live newline. Using `$'\000'` allows the shell to fit in
very comfortably with these. If you want to try this, the corresponding
options are `-print0` for `find` (print with a NUL terminator instead of
newline) and `-0` for `xargs` (read input assuming a NUL terminator).

In older versions of the shell, characters with the top bit set, such as
those from non-English character sets found in ISO 8859 fonts, could
cause problems, since the shell also uses such characters internally to
represent its own special characters, but recent versions of the shell
(from about 3.0) side-step this problem in the same way as for NULs. Any
remaining problems --- it's quite tricky to handle this completely
consistently --- are bugs and should be reported.

You can force parameters to be quoted with POSIX quotes by the somewhat
absurd expedient of making the `q` in the quote flag appear a total of
four times. I can't think why you would ever want to do that, except
that it will turn newlines into \``\n`' and hence the result will fit on
a single (maybe rather long) line. Plus you get the replacement of funny
characters with escape sequences.

### 5.1.4: Double quotes

Double quotes allow some, but not all, forms of substitution inside.
More specifically, they allow parameter expansion, command substitution
and arithmetic substitution, but not any of the others: process
substitution doesn't happen, braces and initial tildes and equals signs
are not expanded and patterns are not special. Here's a table; each
expression on the left is some command line argument, and the results
show what is substituted if it appears outside quotes, or in double
quotes.

      Expression      Outside quotes  In double quotes
      ------------------------------------------------
      =(echo hi mum)  /tmp/zshTiqpL     =(echo hi mum)
      $ZSH_VERSION    4.0.1             4.0.1
      $(echo hi mum)  hi mum            hi mum
      $((6**2 + 6))   42                42
      {a,b}cd         acd bcd           {a,b}cd
      ~/foo           /home/pws/foo     ~/foo
      .zl*            .zlogin .zlogout  .zl*

That \``/tmp/zshTiqpL`' could be any temporary filename, and indeed
several of the other substitutions will be different in your case.

You might already have guessed that \``${(qqq)string}`' forces `$string`
to use double quotes to quote its special characters. As with the other
forms, this is all properly handled --- the shell knows just which
characters need quoting inside double quotes, and which don't.

**Word-splitting in double quotes**\
\

Where the substitutions are allowed, the (almost) invariable side effect
of double quotes is that word-splitting is suppressed. You can see this
using \``print -l`', which prints one argument per line:

      % array=(one two)
      % print -l $(echo foo bar) $array
      foo
      bar
      one
      two
      % print -l "$(echo foo bar) $array"
      foo bar one two

The reason this is \`almost' invariable is that parameter substitution
allows you to specify that normal word-splitting will occur. There are
two ways of doing this; both use the symbol \``@`'. You probably
remember this from the parameter \``$@`' which has just that effect when
it appears in double quotes: the arguments to the script or function are
split into words like a normal array, except that empty arguments are
not removed. I covered this at some length in [chapter
3](zshguide03.html#syntax).

This is extended for other parameters in the following way:

      % array=(one two three)
      % print -l "${array[@]}"
      one
      two
      three

and more generally for all forms of substitution using another flag,
`(@)`:

      % print -l "${(@)array}"
      one
      two
      three

**Digression on subscripts**\
\

The version with flags is perhaps less clear than the other, but it can
appear in lots of different places. For example, here is how you pick a
slice of an array in zsh:

      % print -l ${array[2,-1]}
      two
      three

where negative numbers count from the end of the array. The numbers in
square brackets are referred to as subscripts. This can get the `(@)`
treatment, too:

      % print -l "${(@)array[2,-1]}"
      two
      three

Although it's probably not obvious, you can use the other notation in
this case:

      % print -l "${array[@][2,-1]}"
      two
      three

The shell will actually handle arbitrary numbers of subscripts in
parameter substitutions, not just one; each applies to the result of the
previous one:

      % print -l "${array[@][2,-1][1]}"
      two

What you have to watch out for is that that last subscript selected a
single word. You can continue to apply subscripts, but they will apply
only on the *characters* in that word, not on array elements:

      % print -l "${array[@][2,1][1][2,-1]}"
      wo

We've now strayed severely off topic: the subscripts will of course work
quite independently from whether the word is being split or appears in
double quotes. Despite the joining of words that occurs in double
quotes, subscripts of arrays still select array elements. This is a
consequence of the order in which the rules of parameter expansion
apply. There is a long, involved section on this in the `zshexpn` manual
entry (look for the heading \`Rules' there or in the \`Parameter
Expansion' node of the corresponding Info or HTML file).

**Word-splitting of quoted command substitutions**\
\

Zsh has the useful feature that you can force the shell to apply the
rules of parameter expansion to the result of a command substitution. To
see where that might be useful, consider the case of the special
\`command substitution' (although it's handled entirely in the shell,
not by running an external command) which puts the contents of a file on
the command line:

      % args() { print $#; }    # report number of arguments
      % cat file
      Words on line one
      Words on line two
      % args $(<file)
      8
      % args "$(<file)"
      1

The unquoted substitution split the file into individual words; the
quoted substitution didn't split it at all. These are the standard shell
rules.

It's very common, however, that you want one line per argument, not
splitting on spaces within the line. This is where parameter expansion
can come in. There is a flag `(f)` which says \`split the result of the
expansion, one word per line'. Here's how to use it in this case:

      % args "${(f)$(<file)}"
      2

Where you would usually put the name of a parameter, you put the command
substitution instead, and the shell operates on the result of that (note
that it does not treat the result as the name of a parameter, but as a
value --- this is discussed in more detail below). The double quotes
were necessary because otherwise the file would already have been split
into individual words by the time the parameter substitution came to
look at the result. You can easily verify that the two arguments are the
individual lines of the file. I don't remember what the \``f`' stands
for, but we were already using up flag codes quite fast when it came
along; Bart Schaefer believes it stands for \`fold', which might at
least help you remember it.

### 5.1.5: Backquotes

The main thing to say about backquotes is that you should use the other
form of command substitution instead. There are two good reasons.

First, the other form can be nested:

      % print $(print $(print a word))
      a word

Obviously that's a silly example, but the main point is that the only
time parentheses should occur unquoted in the shell is in pairs (the
patterns in case statements are an exception, but pairs of parentheses
around patterns are valid, too, and I have used that form in this
guide). Thus you can be confident that any piece of well-formatted shell
code can appear inside the command substitution.

This is clearly not true with `` `...` ``, even though the basic effect
is the same. Any unquoted `` ` `` which happens to appear in a chunk of
code within the backquotes will be treated as the end of the quotes.

The second reason, which is closely related, is that it can be quite
difficult to decide how many levels of quotes are required inside a
backquoted expression. Consider:

      % print "`echo \"hello\"`"
      hello
      % print "$(echo \"hello\")"
      "hello"

It's hard to explain quite what the difference here is without waving my
hands, which prevents me from typing, but the essential point is really
the same one about nesting: you can't do it with backquotes, because the
start and end symbols are the same, but you can do it with parentheses.
So in the second case there is no doubt that the embedded command line,
\``echo \"hello\"`', is to be treated exactly as if that had appeared
outside the command substitution; whereas in the first place, the quotes
within quotes had to be, um, quoted.

As a consequence, in

      % print "$(echo "hello")"
      hello

you need to be careful: at first glance, the pairs of double quotes
surround \``$`(`echo `' and \`)', but they don't, they are nested by
virtue of the substitution. You see the same thing with parameter
substitution:

      % unset foo
      % print "${foo:-"a string"}"
      a string

A third, less good, reason for using the form with parentheses is that
your more sophisticated friends will laugh at you otherwise. Peer
pressure is so important in this complex world.

That's all I have to say about command substitution, since I already
said a lot about it when I discussed the basic syntax in [chapter
3](zshguide03.html#syntax).

5.2: Modifiers and what they modify
-----------------------------------

Modifiers were introduced in [chapter 2](zshguide02.html#init) when I
talked about \`bang history', since that's where they came from. In zsh,
however, they can be used in a couple of other places. They have the
same form in each case: a colon, followed by a letter which is the code
for what the modifier does, possibly (in the case of substitutions)
followed by some other string. So, to jog your memory, unless you have
`NO_BANG_HIST` set:

      % print ~/file
      /home/pws/file
      % print !-1:t
      file

where \``:t`' takes the tail (non-directory part) of the filename.

The second use is in parameters. This follows on very naturally. Note
that neither this nor any of the later uses of modifiers rely on the
`NO_BANG_HIST` option; that's purely for history.

      % param=~/file
      % print ${param:t}
      file

Normally you can miss out the braces in the parameter substitution, but
I tend to use them with modifiers for the sake of clarity. The fact that
the same parts of the shell are used for modifiers wherever they come
from has certain consequences:

      % print foo
      foo
      % ^foo^bar
      bar
      % param='this sentence contains a foo.'
      % print ${param:&}
      this sentence contains a bar.

The ampersand repeats the last substitution, which is the same for
parameter modifiers as for history modifiers. I find parameter modifiers
even more useful than history ones; extracting the head or tail of a
path is a very common operation on parameters.

Modifiers are also smart enough to handle arrays in a useful fashion.
Note this is not true of sets of arguments in history expansions;
\``:t`' will only extract one tail in that case, which may not be quite
what you're expecting:

      % print a sentence with a /real/live/bogus/path in it.
      % print !!:t
      path in it.

However, arrays *are* handled the way you might hope:

      % array=(~/.zshenv ~/.zshrc ~/.zlogout)
      % print ${array:t}
      .zshenv .zshrc .zlogout

The same logic is applied with substitutions. This means that the first
match in every element of the array is replaced:

      % array=('a bar of chocolate' 'a bar of barflies' 
      array> 'a barrier of barns')
      % print ${array:s/bar/car/}
      a car of chocolate a car of barflies a carrier of barns

unless, of course, you do a global replacement:

      % print ${array:gs/bar/car/}
      a car of chocolate a car of carflies a carrier of carns

Note, however, that parameter substitution has its own *much* more
powerful equivalent, which does pattern matching, partial replacement of
modified parts of the original string, and so on. We'll come to this all
in good time.

The final use of modifiers is in filename generation, i.e. globbing.
Since this usually works by having special characters on the command
line, and modifiers just consist of ordinary characters, the syntax is a
little different:

      % print *.c
      parser.c lexer.c input.c output.c
      % print *.c(:r)
      parser lexer input output

so you need parentheses around them. This is a special case of \`glob
qualifiers' which you'll meet below; you can mix them, but the modifiers
must appear at the end. For example,

      % print -l ~/stuff/*
      /home/pws/stuff/onefile.c
      /home/pws/stuff/twofile.c
      /home/pws/stuff/subdir
      % print ~/stuff/*(.:r:t)
      onefile twofile

The globbing qualifier \``.`' specifies that files must be regular, i.e.
not directories nor some form of special file. The \``:r`' removes the
suffix from the result, and the \``:t`' takes away the directory part.
Consequently, filename modifiers will be turned off if you set the
option `NO_BARE_GLOB_QUAL`.

Two final points to note about modifiers with filenames. First, it is
the only form of globbing where the result is no longer a filename; it
is always performed right at the end, after all normal filename
generation. Presumably, in the examples above, the word which was
inserted into the command line doesn't actually correspond to a real
file any more.

Second, although it *does* work if the word on the command line isn't a
pattern but an ordinary word with a modifier tacked on, it *doesn't*
work if that pattern, before modification, doesn't correspond to a real
file. So \``foo.c(:r)`' will only strip off the suffix if `foo.c` is
there in the current directory. This is perfectly logical given that the
attempt to match a file kicks the globbing system, including modifiers,
into action. If this is a problem for you, there are ways round; for
example, insert the right value by hand in a simple case like this, or
more realistically store the value in a parameter and apply the modifier
to that.

5.3: Process Substitution
-------------------------

I don't have much new to say on process substitution, but I do have an
example of where I find it useful. If you use the pager \`less' you may
know it has the facility to preprocess the files you look at, for
example uncompressing files temporarily via the environment variable
`$LESSOPEN` (and maybe `$LESSCLOSE`). Zsh can very easily and, to my
thoroughly unbiased way of looking, more conveniently do the same thing.
Here's a subset of my zsh function front-end to less --- or indeed any
pager, which is given here by the standard environment variable `$PAGER`
with the default `less`. You can hard-wire any file-displaying command
at that point if you prefer.

      integer i=1
      local args arg
      args=($*)

      for arg in $*; do
        case $arg in
          (*.bz2) args[$i]="=(bunzip2 -c ${(q)arg})"
                  ;;
          # this assumes your zcat is the one installed with gzip:
          (*.(gz|Z)) args[$i]="=(zcat ${(q)arg})"
                     ;;
          (*) args=${(q)arg}
              ;;
        esac
        (( i++ ))
      done

      eval command ${PAGER:-less} $args

The main pieces of interest is how elements of the array `$args` were
replaced. The reason each argument was given an extra layer of quotes
via `(q)` is the `eval` at the end; `$args` is turned into an array of
literal characters first, which hence need quoting to protect special
characters. Without that, filenames with spaces or asterisks or whatever
wouldn't be shown properly.

The reason the `eval` is there is so that the process substitutions are
evaluated on the command line when the pager is run, and not before.
They are assigned back to elements of `$args` in quotes, so don't get
evaluated at that point. The effect will be to turn:

      less file.gz file.txt

into

      less =(zcat file.gz) file.txt

The \``command`' at the end of the function is there just in case the
function has the same name as the pager (i.e. \`less' in this example);
it forces the external command to be called rather than the function.
The process substitution is ideal in this context; it provides `less`
with the name of a file to which the decompressed contents of `file.gz`
have been sent, and it deletes the file after the command exits.
Furthermore, the substitution happens in such a way that you can still
specify multiple files on the command line as you usually can with less.
The only problem is that the filename that appears in the \``less`'
prompt is meaningless.

In case you haven't come across it, `bzip2` is a programme very similar
to `gzip`, and it is used almost identically, but it provides better
compression.

There's an infelicity in output process substitutions, just as there is
with multios.

      echo hello > >(sed s/hello/goodbye)

The shell spawns the `sed` process to handle the output from the command
line --- and then forgets about it. It does not wait for it (at least,
not until after it exits, when it will use the `wait` system call to
tidy up). So it is dangerous to rely on the result of the process being
available in the next command. If you try it interactively, in fact, you
may well find that the next prompt is printed before the output from
`sed` shows up on the terminal. This can probably be considered a bug,
but it is quite difficult to fix.

5.4: Parameter substitution
---------------------------

You can probably see from the above that parameter substitutions are at
the heart of much of the power available to transform zsh command lines.
What's more, we haven't covered even a significant fraction of what's on
offer.

### 5.4.1: Using arrays

The array syntax in zsh is quite powerful (surprised?); just don't
expect it to be as efficient as, say, perl. Like other features of zsh,
it exists to make users' lives easier, not to make your computer run
blindingly fast.

I've covered, somewhat sporadically, how to set arrays, and how to
extract bits of them --- the following illustrates this:

      % array=(one two three four)
      % print ${array}
      one two three four
      % print ${array[3]}
      three
      % print ${array[2,-1]}
      two three four

Remember you need \``typeset`' or equivalent if you want the array to be
local to a function. The neat way is \``typeset -a`', which creates an
empty array, but as long as you assign to the array before trying to use
it any old `typeset` will do.

You can use the array index and array slice notations for assigning to
arrays, in other words on the left-hand side of an \``=`':

      % array=(what kind of fool am i)
      % array[2]=species
      % print $array
      what species of fool am i
      % array[2]=(a piece)
      % print $array
      what a piece of fool am i
      % array[-3,-1]=(work is a man)
      % print $array
      what a piece of work is a man

So you can replace a single element of an array by a single element, or
by an array slice; likewise you can replace a slice in one go by a slice
of a different length --- only the bits you explicitly tell it to
replace are changed, the rest is left intact and maybe shifted along to
make way. This is similar to perl's \`splice' command, only for once
maybe a bit more memorable. Note that you shouldn't supply any braces on
the left hand side. The appearance of the expression in an assignment is
enough to trigger the special behaviour of subscripts, even if
`KSH_ARRAYS` is in effect --- though you need to subtract one from your
subscripts in that case.

You can remove bits in the middle, too, but note you should use an empty
array:

      % array=(one two three four)
      % print $#array
      4
      % array[2]=
      % print $#array
      4
      % array[2]=()
      % print $#array
      3

The first assignment set element 2 to the empty string, it didn't remove
it. The second replaced the array element with an array of length zero,
which did remove it.

Just as parameter substitutions have flags for special purposes, so do
subscripts. You can force them to search through arrays, matching on the
values. You can return the value matched ((r)everse subscripting):

      % array=(se vuol ballare signor contino)
      % print ${array[(r)s*]}
      se
      % print ${array[(R)s*]}
      signor

The `(r)` flag takes a pattern and substitutes the first element of the
array matched, while the `(R)` flag does the same but starting from the
end of the array. If nothing matched, you get the empty string; as usual
with parameters, this will be omitted if it's the only thing in an
unquoted argument. Using our `args` function to count the arguments
passed to a command again:

      % array=(some words)
      % args() { print $#; }
      % args ${array[(r)s*]}
      1
      % args ${array[(r)X*]}
      0
      % args "${array[(r)X*]}"
      1

where in the last case the empty string was quoted, and passed down as a
single, empty argument.

You can also return the index matched; `(i)` to start matching from the
beginning, and `(I)` to start from the end.

      % array=(se vuol venire nella mia scuola)
      % print ${array[(i)v*]}
      2
      % print ${array[(I)v*]}
      3  

matching \`vuol' the first time and \`venire' the second. What happens
if they don't match may be a little unexpected, but is reasonably
logical: you get the next index along. In other words, failing to match
at the end gives you the length of the array plus one, and failing to
match at the beginning gives you zero, so:

      array=(three egregious words)
      for pat in '*e*e*' '*a*a*'; do
        if [[ ${array[(i)$pat]} -le ${#array} ]]; then
          print "Pattern $pat matched in array: ${array[(r)$pat]}."
        else
          print "Pattern $pat failed to match in array"
        fi
      done

prints:

      Pattern *e*e* matched in array: three.
      Pattern *a*a* failed to match in array

If you adapt that chunk of code, you'll see you get the indices 1 and 4
returned. Note that the characters in `$pat` were treated as a pattern
even though putting `$pat` on the command line would normally just
produce the characters themselves. Subscripts are special in that way;
trying to keep the syntax under control at this point is a little hairy.
There is a more detailed description of this in the manual in the
section \`Subscript Parsing' of the `zshparam` manual page or the
\`Array Parameters' info node; to quote the characters in `pat`, you
would actually have to supply the command line strings `'\*e\*e\*'` and
`'\*a\*a\*'`. Just go round mumbling \`extra layer of pattern expansion'
and everyone will think you know what you're talking about (it works for
me, fitfully).

There is currently no way of extracting a complete set of matches from
an ordinary array with subscript flags. We'll see other ways of doing
that below, however.

### 5.4.2: Using associative arrays

Look back at [chapter 3](zshguide03.html#syntax) if you've forgotten
about associative arrays. These take subscripts, like ordinary arrays
do, but here the subscripts are arbitrary strings (or keys) associated
with the value stored in the element of the array. Remember, you need to
use \``typeset -A`' to create one, or one of `typeset`'s relatives with
the same option. This means that if you created it inside a function it
will be limited to the local scope, so if you want to create a global
associative array you will need to give the `-g` flag as well. This is
particularly common with associative arrays, which are often used to
store global information such as configuration details.

Retrieving information from associative arrays can get you into some of
the problems already hinted at in the use of subscript flags with
arrays. However, since normal subscripting doesn't make patterns active,
there is a way round here: make the subscript into another parameter:

      % typeset -A assoc
      % assoc=(key value Shlüssel Wert clavis valor)
      % subscript='key'
      % print ${assoc[$subscript]}
      value  

I used fairly boring keys here, but they can be any string of
characters:

      % assoc=(']' right\ square\ bracket '*' asterisk '@' at\ sign)
      % subscript=']'
      % print ${assoc[$subscript]}
      right square bracket

and *that* is harder to get the other way. Nonetheless, if you define
your own keys you will often use simple words, and in that case they can
happily appear directly in the square brackets.

I introduced two parameter flags, `(k)` and `(v)` in [chapter
3](zshguide03.html#syntax):

      % print ${(k)assoc}
      * ] @

prints out keys, while

      % print ${(kv)assoc}
      * asterisk ] right square bracket @ at sign

and the remaining two possibilities do the same thing:

      % print ${(v)assoc}
      asterisk right square bracket at sign
      % print ${assoc}
      asterisk right square bracket at sign

You now know these are part of a much larger family of tricks to apply
to substitutions. There's nothing to stop you combining flags:

      % print -r ${(qkv)assoc}
      \* asterisk \] right\ square\ bracket @ at\ sign

which helps see the wordbreaks. Don't forget the \``print -l`' trick for
separating out different words, and hence elements of arrays and
associative arrays:

      % print -l ${(kv)assoc}
      *
      asterisk
      ]
      right square bracket
      @
      at sign

which is quite a lot clearer. As always, this will fail if you engage in
un-zsh activities with `SH_WORD_SPLIT`, but judicious use of `@`,
whether as a flag or a subscript, and double quotes, will always work:

      % print -l "${(@kv)assoc}"
      *
      asterisk
      ]
      right square bracket
      @
      at sign

regardless of the option setting.

Apart from the subscripts, the second major difference between
associative and ordinary arrays is that the former don't have any order
defined. This will be entirely familiar if you have used Perl; the
principle here is identical. However, zsh has no notion at all, even as
a convenience, of slices of associative arrays. You can assign
individual elements or whole associative arrays --- remembering that in
the second case the right hand side must consist of key/value pairs ---
but you can't assign subgroups. Any attempt to use the slice notation
with commas will be met by a stern error message.

What zsh does have, however, is extra subscript flags for you to match
and retrieve one or more elements. If instead of an ordinary subscript
you use a subscript preceded by the flag `(i)`, the shell will search
for a matching key (not value) with the pattern given and return that.
This is deliberately the same as searching an ordinary array to get its
key (which in that case is just a number, the index), but note this time
it doesn't match on the value, it really does match, as well as return,
the key:

      % typeset -A assoc
      % assoc=(fred third\ man finnbar slip roger gully trevor long\ off)
      % print ${assoc[(i)f*]}
      fred

You can still use the parameter flags `(k)` and `(v)` to tell the shell
which part of the key and/or value to return:

      % print ${(kv)assoc[(i)f*]}
      fred third man

Note the division of labour. The subscript flag tells the shell what to
match against, while the parameter flags tell it which bit of the
matched element(s) you actually want to see.

Because of the essentially random ordering of associative arrays, you
couldn't tell here whether fred or finnbar would be chosen. However, you
can use the capital form `(I)` to tell the shell to retrieve all
matches. This time, let's see the values of the elements for which the
keys were matched:

      % print -l ${(v)assoc[(I)f*]}
      third man
      slip

and here we also got the position occupied by `finnbar`. The same rules
about patterns apply as with `(r)` in ordinary arrays --- a subscript is
treated as a pattern even if it came from a parameter substitution
itself.

You probably aren't surprised to hear that the subscript flags `(r)` and
`(R)` try to match the values of the associative array rather than its
keys. These, too, print out the actual part matched, here the value,
unless you use the parameter flags.

      % print ${assoc[(r)*i*]}
      third man
      % print ${(k)assoc[(R)*i*]}
      fred finnbar

There's one more pair of subscript flags of particular relevance to
associative arrays, `(k)` and `(K)`. These work a bit like a case
statement: the subscripts are treated as strings, and the keys of the
associative arrays as patterns, instead of the other way around. With
`(k)`, the value of the first key which matches the subscript is
substituted; with `(K)`, the values of all matching keys are substituted

      % typeset -A assoc
      % assoc=('[0-9]' digit '[a-zA-Z]' letter '[^0-9a-zA-Z]' neither)
      % print ${assoc[(k)0]}
      digit
      % print ${assoc[(k)_]}
      neither

In case you're still confused, the \``0`' in the first subscript was
taken as a string and all the keys in `$assoc` were treated as patterns
in turn, a little like

      case 0 in
        ([0-9]) print digit
                ;;
        ([a-zA-Z]) print letter
                   ;;
        ([^0-9a-zA-Z]) print neither
                       ;;
      esac

One important way in which this is *not* like the selection in a case
statement is that you can't rely on the order of the comparison, so you
can't rely on more general patterns being matched after more specific
ones. You just have to use keys which are sufficiently explicit to match
just the strings you want to match and no others. That's why we picked
the pattern \``[^0-9a-zA-Z]`' instead of just \``*`' as we would
probably have used in the case statement.

I said storing information about configuration was a common use of
associative arrays, but the shell has a more powerful way of doing that:
styles, which will figure prominently in the discussion of programmable
completion in the next chapter. The major advantage of styles over
associative arrays is that they can be made context-sensitive; you can
easily make the same style return the same value globally, or make it
have a default but with a different value in one particular context, or
give it a whole load of different values in different places. Each shell
application can decide what is meant by a \`context'; you are not tied
to the same scheme as the completion system uses, or anything like it.
Use of hierarchical contexts in the manner of the completion system does
mean that it is easy to create sets of styles for different modules
which don't clash.

Here, finally, is a comparison of some of the uses of associative arrays
in perl and zsh.

          perl                          zsh
      -----------------------------------------------------------------
      %hash = qw(key value);         typeset -A hash; hash=(key value)
      $hash{key}                     ${hash[key]}
      keys %hash                     ${(k)hash}
      values %hash                   ${(v)hash}
      %hash2 = %hash;                typeset -A hash2; hash2=("${(@kv)hash}")
      unset %hash;                   unset hash
      if (exists $hash{key}) {       if (( ${+hash[key]} )); then
        ...                            ...
      }                              fi

One final reminder: if you are creating associative arrays inside a
function which need to last beyond the end of the function, you should
create them with \``typeset -gA`' which puts them into the surrounding
scope. The \``-g`' flag is of course useful with all types of parameter,
but the associative array is the only type that doesn't automatically
spring into existence when you assign to it in the right context; hence
the flag is particularly worthy of note here.

### 5.4.3: Substituted substitutions, top- and tailing, etc.

There are many transformations which you can do on the result of a
parameter substitution. The most powerful involve the use of patterns.
For this, the more you know about patterns, the better, so I will
reserve explanation of some of the whackiest until after I have gone
into more detail on patterns. In particular, it's useful if you know how
to tell the shell to mark subexpressions which it has matched for future
extraction. However, you can do some very useful things with just the
basic patterns common to all shells.

**Standard forms: lengths**\
\

I'll separate out zsh-specific forms, and start off with some which
appear in all shells derived from the Bourne shell. A more compact
(read: terse) list is given in the manual, as always.

A few simple forms don't use patterns. First, the substitution
`${#`*param*`}` outputs the length of `$`*param*. In zsh, you don't need
the braces here, though in most other shells with this feature you do.
Note that `${#}` on its own is the number of parameters in the command
line argument array, which is why explicit use of braces is clearer.

`$#` works differently on scalar values and array values; in the former
case, it gives the length in characters, and in the latter case the
length in elements. Note that I said \`values', not \`parameters' ---
you have to work out whether the substitution is giving you a scalar or
an array:

      % print ${#path}
      8
      % print ${#path[1]}
      13

The first result shows I have 8 directories in my path, the latter that
the first directory (actually \``/home/pws/bin`') has 13 characters. You
should bear this in mind with nested substitutions, as discussed below,
which can also return either an array or a scalar.

Earlier versions of zsh always returned a character count if the
expression was in double quotes, or anywhere the shell evalauted the
expression as a single word, but that doesn't happen any more; it
depends only on the type of the value. However, you can force the shell
to count characters by using the `(c)` flag, and to count words (even in
scalars, which it will split if necessary) by using `(w)`:

      % print ${#PATH}
      84
      % print ${(c)#path}
      84
      % foo="three scalar words"
      % print ${(w)#foo}
      3

Comparing the first two, you will see that character count with arrays
includes the space used for separating (equal to the number of colons
separating the elements in `$PATH`). There's a relative of `(w)` called
`(W)`, which treats multiple word separators as having zero-length words
in between:

      % foo="three  well-spaced  word"
      % print ${(w)#foo}
      3
      % print ${(W)#foo}
      5

giving two extra words over `(w)`, which treats the groups of spaces in
the same way as one. Being parameter flags, these modifications of the
syntax are specific to zsh.

Note that if you use lengths in an arithmetic context (inside `((...))`
or `$((...))`), you must include the leading \``$`', which you don't
need for substituting the parameters themselves. That's because
\``#foo`' means something different here --- the number in the ASCII
character set (or whatever extension of it you are using if it is an
extended character set) of the first character in `$foo`.

**Standard forms: conditional substitutions**\
\

The next group of substitutions is a whole series where the parameter is
followed by an option colon and then \``-`', \``=`', \``+`' or \``?`'.
The colon has the same effect in each case: without a colon, the shell
tests whether the parameter is set before performing the operation,
while with the colon it tests whether the parameter has non-zero length.

The simplest is \``${`*param*`:-`*value*`}`'. If `$param` has non-zero
length (without the colon, if it is set at all), use its value, else use
the *value* supplied. Suppose `$foo` wasn't set at the start of the
following (however unlikely that may seem):

      % print ${foo-bar}
      bar
      % foo=''
      % print ${foo-bar}
      
      % print ${foo:-bar}
      bar
      % foo='please no anything but bar'
      % print ${foo:-bar}
      please no anything but bar

It's more usual to use the form with the colon. One reason for that is
that in functions you will often create the parameter with a `typeset`
before using it, in which case it always exists, initially with zero
length, so that the other form would never use the default value. I'll
use the colon for describing the other three types.

\``${`*param*`:=`*value*`}`' is similar to the previous type. but in
this case the shell will not only substitute *value* into the line, it
will assign it to *param* if (and only if) it does so. This leads to the
following common idiom in scripts and functions:

      : ${MYPARAM:=default}  ${OTHERPARAM:=otherdefault}

If the user has already set `$MYPARAM`, nothing happens, otherwise it
will be set to \``default`', and similarly for `${OTHERPARAM}`. The
\``:`' command does nothing but return true after the command line has
been processed.

\``${`*param*`:+`*value*`}`' is the opposite of \``:-`', logically
enough: the *value* is substituted if the parameter *doesn't* have zero
length. In this case, *value* will often be another parameter
substitution:

      print ${value:+"the value of value is $value"}

prints the string only if `$#value` is greater than zero. Note that what
can appear after the \``+`' is pretty much any single word the shell can
parse; all the usual single-word substitutions (so globbing is excluded)
will be applied to it, and quotes will work just the same as usual. This
applies to the values after \``:-`' and \``:=`', too. One other commonly
seen trick might be worth mentioning:

      print ${1+"$@"}

substitutes all the positional parameters as they were passed if the
first one was set (here you don't want the colon). This was necessary in
some old shells because `"$@"` on its own gave you a single empty
argument instead of no arguments when no arguments were passed. This
workaround isn't necessary in zsh, nor in most modern Bourne-derived
shells. There's a bug in zsh's handling, however; see the section on
function parameters in chapter 3.

The final type isn't that often used (meaning I never have):
`${`*param*`?`*message*`}` tests if *param* is set (no colon), and if it
isn't, prints the message and exits the shell. An interactive shell
won't exit, but it will return you immediately to the prompt, skipping
anything else stored up for execution. It's a rudimentary safety
feature, a little bit like \`assert' in C programmes; most shell
programmers seem to cover the case of missing parameter settings by more
verbose tests. It's quite neat in short shell functions for interactive
use:

      mless() { mtype ${@:?missing filename} | $PAGER }

**Standard forms: pattern removal**\
\

Most of the more sophisticated Bourne-like shells define two pairs of
pattern operators, which I shall call \`top and tail' operators. One
pair (using \``#`' and \``##`') removes a given pattern from the head of
the string, returning the rest, while the other pair (using \``%`' and
\``%%`') removes a pattern from the tail of the string. In each case,
the form with one symbol removes the shortest matching pattern, while
the one with two symbols removes the longest matching pattern. Two
typical uses are:

      % print $HOME
      /home/pws
      % print ${HOME##*/}
      pws
      % print ${HOME%/*}
      /home

which here have the same effect of `${HOME:t}` and and `${HOME:h}`, and
in zsh you would be more likely to use the latter. However, as you can
see the pattern forms are much more general. Note the difference from:

      % print ${HOME#*/}
      home/pws
      % print ${HOME%%/*}

where the shortest match of \``*/`' at the head was just the first
slash, since \``*`' can match an empty string, while the longest match
of \``/*`' at the tail was the entire string, right back to the first
slash. Although these are standard forms, remember that the full power
of zsh patterns is available.

How do you remember which operator does what? The fact that the longer
form does the longer match is probably easy. Remembering that \``#`'
removes at the head and \``%`' at the tail is harder. Try to think of
\`hash' and \`head' (if you call it a \`pound sign', when it's nothing
of the sort since a pound since looks like \`£', you will get no
sympathy from me), and \`percent' and \`posterior'. It never worked for
me, but maybe I just don't have the mental discipline. Oliver Kiddle
points out that \``#`' is further to the left (head) on a standard US
keyboard. On my UK keyboard, \``#`' is right next to the return key,
unfortunately, although here the confusion with \`pound sign' will jog
your memory.

The most important thing to remember is: this notation is not our fault.
Sorry, anyway. By the way, notice there's no funny business with colons
in the case of the pattern operators. (Well --- except for the zsh
variant noted below.)

**Zsh-specific parameter substitutions**\
\

Now for some enhancements that zsh has for using the forms of parameter
substitution I've just given as well as some similar but different ones.

One simple enhancement is that in addition to
\``${`*param*`=`*value*`}`' and \``${`*param*`:=`*value*`}`', zsh has
\``${`*param*`::=`*value*`}`' which performs an unconditional assignment
as well as sticking the value on the command line. It's not really any
different from using a normal assignment, then a normal parameter
substitution, except that zsh users like densely packed code.

All the assignment types are affected by the parameter flags \``A`' and
\``AA`' which tell the shell to perform array and associative array
assignment (in the second case, you need pairs of key/value elements as
usual). You need to be a little bit careful with array elements and word
splitting, however:

      % print -l ${(A)foo::=one two three four}
      one two three four
      % print ${#foo}
      1

That made `$foo` an array all right, but treated the argument as a
scalar value and assigned it to the first element. There's a way round
this:

      % print -l ${(A)=foo::=one two three four}
      one
      two
      three
      four
      % print ${#foo}
      4

Here, the \``=`' *before* the parameter name has a completely different
effect from the others: it turns on word-splitting, just as if the
option `SH_WORD_SPLIT` is in effect. You may remember I went into this
in appalling detail in the section \`Function parameters' in [chapter
3](zshguide03.html#syntax).

You should be careful, however, as more sophisticated attempts at
putting arrays inside parameter values can easily lead you astray. It's
usually much easier to use the \`*array*`=`(*...*)' or \``set -A` *...*'
notations.

One extremely useful zsh enhancement is the notation \``${+foo}`' which
returns 1 if `$foo` is set and 0 if it isn't. You can use this in
arithmetic expressions. This is a much more flexible way of dealing with
possibly unset parameters than the more standard \``${foo?goodbye}`'
notation, and consequently is better used by zsh programmers. The
notation \`plus foo' for \`foo is set' should be fairly memorable, too.
A more standard way of doing this (noted by David Korn) is
\``0${foo+1}`', giving 0 if `$foo` is not set and 01 if it is.

**Parameter flags and pattern substitutions**\
\

Zsh increases the usefulness of the \`top and tail' operators with some
of its parameter flags. Usually these show you what's left after the
removal of some matched portion. However, with the flag `(M)` the shell
will instead show you the matched portion itself. The flag `(R)` is the
opposite and shows the rest: that's not all that useful in the normal
case, since you get that by default. It only starts being useful when
you combine it with other flags.

Next, zsh allows you to match on substrings, not just on the head or
tail. You can do this by giving the flag `(S)` with either of the \``#`'
or \``%`' pattern-matching forms. The difference here is whether the
shell starts searching for a matching substring at the start or end of
the full string. Let's take

      foo='where I was huge lizards walked here and there'

and see what we get matching on \``h*e`':

      % print -l ${(S)foo#h*e} ${(S)foo##h*e} ${(S)foo%h*e} ${(S)foo%%h*e}
      wre I was huge lizards walked here and there
      w
      where I was huge lizards walked here and tre
      where I was huge lizards walked here and t

There are some odd discrepancies at first sight, but here's what
happens. In the first case, \``#`' the shell looks forward until it
finds a match for \``h*e`', and takes the shortest, which is the \``he`'
in the first word. With \``##`', the match succeeds at the same point,
but the longest match extends to the \``e`' right at the end of the
string. With the other two forms, the shell starts scanning backwards
from the end, and stops as soon as it reaches a starting point which has
a match. For both \``%`' and \``%%`' this is the last \``h`', but the
former matches \``he`' and the latter matches \``here`'.

You can extend this by using the `(I)` flag to specify a numeric index.
The index needs to be delimited, conventionally, although not
necessarily, by colons. The shell will then scan forward or backward,
depending on the form used, until it has found the `(I)`'th match. Note
that it only ever counts a single match from each position, either the
longest or the shortest, so the `(I)`'th match starts from the `(I)`'th
position which has any match. Here's what happens when we remove all the
matches for \``#`' using the example above.

      % for (( i = 1; i <= 5; i++ )); do
      for> print ${(SI:$i:)foo#h*e}
      for> done
      wre I was huge lizards walked here and there
      where I was  lizards walked here and there
      where I was huge lizards walked re and there
      where I was huge lizards walked here and tre
      where I was huge lizards walked here and there

Each time we match and remove one of the possible \``h*e`' sets where
there is no \``e`' in the middle, moving from left to right. The last
time there was nothing left to match and the complete string was
returned. Note that the index we used was itself a parameter.

It's obvious what happens with \``##`': it will find matches at all the
same points, but they will all extend to the \``e`' at the end of the
string. It's probably less obvious what happens with \``%%`' and \``%`',
but if you try it you will find they produce just the same set of
matches as \``##`' and \``#`', respectively, but with the indices in the
reverse order (4 for 1, 3 for 2, etc.).

You can use the \``M`' flag to leave the matched portion rather than the
rest of the string, if you like. There are three other flags which let
you get the indices associated with the match instead of the string:
`(B)` for the beginning, using the usual zsh convention where the first
character is 1, `(E)` for the character *after* the end, and `(N)` for
the length, simply `B-E`. You can even have more than one of these; the
value substituted is a string with the given values with spaces between,
always in the order beginning, end, length.

There is a sort of opposite to the \``(S)`' flag, which instead of
matching substrings will only match the whole string; to do this, put a
colon before the \``#`'. Hence:

      % print ${foo:#w*g}
      where I was huge lizards walked here and there
      % print ${foo:#w*e}

      % 

The first one didn't match, because the \``g`' is not at the end; the
second one did, because there is an \``e`' at the end.

**Pattern replacement**\
\

The most powerful of the parameter pattern-matching forms has been
borrowed from bash and ksh93; it doesn't occur in traditional Bourne
shells. Here, you use a pair of \``/`'s to indicate a pattern to be
replaced, and its replacement. Lets use the lizards again:

      % print ${foo/h*e/urgh}
      wurgh

A bit incomprehensible: that's because like most pattern matchers it
takes the longest match unless told otherwise. In this case the `(S)`
flag has been pressed into service to mean not a substring (that's
automatic) but the shortest match:

      % print ${(S)foo/h*e/urgh}
      wurghre I was huge lizards walked here and there

That only replace the first match. This is where \``//`' comes in; it
replaces every match:

      % print ${(S)foo//h*e/urgh}
      wurghre I was urgh lizards walked urghre and turghre

(No doubt you're starting to feel like a typical anachronistic Hollywood
cave-dweller already.) Note the syntax: it's a little bit like
substitution in `sed` or perl, but there's no slash at the end, and with
\``//`' only the first slash is doubled. It's a bit confusing that with
the other pattern expressions the single and double forms mean the
shortest and longest match, while here it's the flag `(S)` that makes
the difference.

The index flag `(I)` is useful here, too. In the case of \``/`', it
tells the shell which single match to substitute, and in the case of
\``//`' it tells the shell at which match to start: all matches starting
from that are replaced.

Overlapping matches are never replaced by \``//`'; once it has put the
new text in for a match, that section is not considered further and the
text just to its right is examined for matches. This is probably
familiar from other substitution schemes.

You may well be thinking \`wouldn't it be good to be able to use the
matched text, or some part of it, in the replacment text?' This is what
you can do in sed with \``\1`' or \``\&`' and in perl with \``$1`' and
\``$&`'. It turns out this *is* possible with zsh, due to part of the
more sophisticated pattern matching features. I'll talk about this when
we come on to patterns, since it's not really part of parameter
substitution, although it's designed to work well with that.

### 5.4.4: Flags for options: splitting and joining

There are three types of flag that don't look like flags, for historical
reasons; you've already seen them in [chapter
3](zshguide03.html#syntax). The first is the one that turns on the
`SH_WORD_SPLIT` option, `${=foo}`. Note that you can mix this with flags
that *do* look like flags, in parentheses, in which case the \``=`' must
come after the closing parenthesis. You can force the option to be
turned off for a single substitution by doubling the symbol:
\``${==foo}`'. However, you wouldn't do that unless the option was
already set, in which case you are probably trying to be compatible with
some other shell, and wouldn't want to use that form.

More control over splitting and joining is possible with three of the
more standard type of flags, `(s)`, `(j)` and `(z)`. These do splitting
on a given string, joining with a given string, and splitting just the
way the shell does it, respectively. In the first two cases, you need to
specify the string in the same way as you specified the index for the
`(I)` flag. So, for example, here's how to turn `$PATH` into an ordinary
array without using `$path`:

      % print -l ${(s.:.)PATH}
      /home/pws/bin
      /usr/local/bin
      /usr/sbin
      /sbin
      /bin
      /usr/bin
      /usr/X11R6/bin
      /usr/games

Any character can follow the `(s)` or `(j)`; the string argument lasts
until the matching character, here \``.`'. If the character is one of
the bracket-like characters including \``<`', the \`matching' character
is the corresponding right bracket, e.g. \``${(s<:>)PATH}`' and
\``${(s(:))PATH}`' are both valid. This applies to all flags that need
arguments, including `(I)`.

Although the split or join string isn't a pattern, it doesn't have to be
a single character:

      % foo=(array of words)
      % print ${(j.**.)foo}
      array**of**words

The `(z)` flag doesn't take an argument. As it handles splitting on the
full shell definition of a word, it goes naturally with quoted
expressions, and I discussed above its use with the `(Q)` flag for
extracting words from a line with the quotes removed.

It's possible for the same parameter expression to have both splitting
and joining applied to it. This always occurs in the same order,
regardless of how you specify the flags: joining first, then splitting.
This is described in the (rather hairy) complete set of rules in the
manual entry for parameter substitution. There are one or two occasions
where this can be a bit surprising. One is when you have `SH_WORD_SPLIT`
set and try to join a string:

      % setopt shwordsplit    
      % foo=('another array' of 'words with spaces')
      % print -l ${(j.:.)foo}
      another
      array:of:words
      with
      spaces

You might not have noticed if you didn't use the \``-l` option to print,
but the spaces still caused word-spliting even though you asked for the
array to be joined with colons. To avoid this, either don't use
`SH_WORD_SPLIT` (my personal preference), or use quotes:

      % print -l "${(j.:.)foo}"
      another array:of:words with spaces

The elements of an array would normally be joined by spaces in this
case, but the character specified by the `(j)` flag takes precedence. In
just the same way, if `SH_WORD_SPLIT` is in effect, any splitting string
given by `(s)` is used instead of the normal set of characters, which
are any characters that occur in the string `$IFS`, by default space,
tab, newline and NUL.

Specifying a split for a particular parameter substitution not only sets
the string to split on, but also ensures the split will take place even
if the expression is quoted:

      % array=('element one' 'element two' 'element three')
      % print -l "${=array}"
      element
      one
      element
      two
      element
      three

To be clear about what's happening here: the quotes force the elements
to be joined with spaces, giving a single string, which is then split on
the original spaces as well as the one used to join the elements of the
array.

I will talk shortly about nested parameter substitution; you should also
note that splitting and joining will if necessary take place at all
levels of a nested substitution, not just the outermost one:

      % foo="three blind words"
      % print ${#${(z)foo}}
      3

This prints the length of the innermost expression; because of the
zplit, that has produced a three-element array.

### 5.4.5: Flags for options: `GLOB_SUBST` and `RC_EXPAND_PARAM`

The other two flags that don't use parentheses affect options for single
substitutions, too. The second is the \``~`' flag that turns on
`GLOB_SUBST`, making the result of a parameter substitution eligible for
pattern matching. As the notation is supposed to indicate, it also makes
filename expansion possible, so

      % foo='~'
      % print ${~foo}
      /home/pws

It's that first \``~`' which is giving the home directory; the one in
the parameter expansion simply allows that to happen. If you have
`GLOB_SUBST` set, you can use \``${~~foo}`' to turn it off for one
substitution.

There's one other of these option flags: \``^`' forces on
`RC_EXPAND_PARAM` for the current substitution, and \``^^`' forces it
off. In [chapter 3](zshguide03.html#syntax), I showed how parameters
expanded with this option on fitted in with brace expansions.

### 5.4.6: Yet more parameter flags

Here are a few other parameter flags; I'm repeating some of these. A
very useful one is \``t`' to tell you the type of a parameter. This came
up in [chapter 3](zshguide03.html#syntax) as well. It's most common use
is to test the basic type of the parameter before trying to use it:

      if [[ ${(t)myparam} != *assoc* ]]; then
        # $myparam is not an associative array.  Do something about it.
      fi

Another very useful type is for left or right padding of a string, to a
specified length, and optionally with a specified fill string to use
instead of space; you can even specify a one-off string to go right next
to the string in question.

      foo='abcdefghij'
      for (( i = 1; i <= 10; i++ )); do
       goo=${foo[1,$i]}
       print ${(l:10::X::Y:)goo} ${(r:10::X::Y:)goo}
      done

prints out the rather pretty:

      XXXXXXXXYa aYXXXXXXXX
      XXXXXXXYab abYXXXXXXX
      XXXXXXYabc abcYXXXXXX
      XXXXXYabcd abcdYXXXXX
      XXXXYabcde abcdeYXXXX
      XXXYabcdef abcdefYXXX
      XXYabcdefg abcdefgYXX
      XYabcdefgh abcdefghYX
      Yabcdefghi abcdefghiY
      abcdefghij abcdefghij

Note that those colons (which can be other characters, as I explained
for the `(s)` and `(j)` flags) always occur in pairs before and after
the argument, so that with three arguments, the colons in between are
doubled. You can miss out the \``:Y:`' part and the \``:X:`' part and
see what happens. The fill strings don't need to be single characters;
if they don't fit an exact number of times into the filler space, the
last repetition will be truncated on the end furthest from the parameter
argument being inserted.

Two parameters tell the shell that you want something special done with
the value of the parameter substitution. The `(P)` flag forces the value
to be treated as a parameter name, so that you get the effect of a
double substitution:

      % final=string
      % intermediate=final
      % print ${(P)intermediate}
      string

This is a bit as if `$intermediate` were what in ksh is called a
\`nameref', a parameter that is marked as a reference to another
parameter. Zsh may eventually have those, too; there are places where
they are a good deal more convenient than the \``(P)`' flag.

A more powerful flag is `(e)`, which forces the value to be rescanned
for all forms of single-word substitution. For example,

      % foo='$(print $ZSH_VERSION)'
      % print ${(e)foo}
      4.0.2

made the value of `$foo` be re-examined, at which point the command
substitution was found and executed.

The remaining flags are a few simple special formatting tricks: order
array elements in normal lexical (character) order with `(o)`, order in
reverse order with `(O)`, do the same case-independently with `(oi)` or
`(Oi)` respectively, expand prompt \``%`'-escapes with `(%)` (easy to
remember), expand backslash escapes as `print` does with `p`, force all
characters to uppercase with `(U)` or lowercase with `(L)`, capitalise
the first character of the string or each array element with `(C)`, show
up special characters as escape sequences with `(V)`. That should be
enough to be getting on with.

### 5.4.7: A couple of parameter substitution tricks

I can't resist describing a couple of extras.

Zsh can do so much on parameter expressions that sometimes it's useful
even without a parameter! For example, here's how to get the length of a
fixed string without needing to put it into a parameter:

      % print ${#:-abcdefghijklm}
      13

If the parameter whose name you haven't given has a zero length (it
does, because there isn't one), use the string after the \``:-`'
instead, and take it's length. Note you need the colon, else you are
asking the shell to test whether a parameter is set, and it becomes
rather upset when it realises there isn't one to test. Other shells are
unlikely to tolerate any such syntactic outrages at all; the `#` in that
case is likely to be treated as `$#`, the number of shell arguments. But
zsh knows that's not going to have zero length, and assumes you know
what you're doing with the extra part; this is useful, but technically a
violation of the rules.

Sometimes you don't need anything more than the flags. The most useful
case is making the \`fill' flags generate repeated words, with the
effect of perl's \``x`' operator (for those not familiar with perl, the
expression \``"string" x 3`' produces the string \`stringstringstring'.
Here, you need to remember that the fill width you specify is the total
width, not the number of repetitions, so you need to multiply it by the
length of the string:

      % print ${(l.18..string.)}
      stringstringstring

### 5.4.8: Nested parameter substitutions

Zsh has a system for multiple nested parameter substitutions. Whereas in
most shells or other scripting languages you would do something like:

      % p=/directory/file.ext
      % p2=${p##*/}            # remove longest match of */ from head
      % print $p2
      file.ext
      % print ${p%.*}          # remove shortest match of .* from tail
      file

in zsh you can do this in one substitution:

      % p=/directory/file.ext
      % print ${${p##*/}%.*}
      file

saving the temporary parameter in the middle. (Again, you are more
likely to use `${p:t:r}` in this particular case.) Where this becomes a
major advantage is with arrays: if `$p` is an array, all the
substitutions are applied to every element of the array:

      % p=(/dir1/file1.ext1 /dir2/file2.ext2)
      % print ${${p##*/}%.*}
      file1 file2

This can result in some considerable reductions in the code for
processing arrays. It's a way of getting round the fact that an ordinary
command line interface like zsh, designed originally for direct
interaction with the user, doesn't have all the sophistication of a
non-interactive language like perl, whose \``map`' function would
probably be the neatest way of doing the same thing:

       # Perl code.
       @p = qw(/dir1/file1.ext1 /dir2/file2.ext2);
       @q = map { m%^(?:.*/)(.*?)(?:\.[^.]*|)$%; } @p;
       print "@q\n";'

or numerous possible variants. In a shell, there's no way of putting
functions like that into the command line without complicating the basic
\`command, arguments' syntax; so we resort to trickery with
substitutions. Note, however, that this degree of brevity makes for a
certain lack of readability even in Perl. Furthermore, zsh is so
optimised for common cases that

      print ${p:t:r}

will work for both arrays and scalars: the `:t` takes only the tail of
the filename, stripping the directories, and the `:r` removes the
suffix. These two operators could have slightly unexpected effects in
versions of zsh before 4.0.1, removing \`suffixes' which contained
directory paths, for example (though this is what the pattern forms
taken separately do, too).

Note one feature of the nested substitution: you might have expected the
\``${...}`' inside the other one to do a full parameter substitution, so
that the outer one would act on the value of that --- that's what you'd
get if the substitution was on its own, after all. However, that's not
what happens: the \``${...}`' inside is simply a syntactic trick to say
\`here come more operations on the parameter'. This means that

      bar='this doesn'\''t get substituted'
      foo='bar'
      print ${${foo}}

simply prints \``bar`', not the value of `$bar`. This is the same case
we had before but without any of the extra \``##`' and \``%`' bits. The
reason is historical: when the extremely useful nested substitution
feature was added, it was much simpler to have the leading \``$`'
indicate to the shell that it should call the substitution function
again than find another syntax. You can make the value be re-interpreted
as another parameter substitution, using the `(P)` substitution flag
described above. Just remember that `${${foo}}` and `${(P)foo}` are
different.

5.5: That substitution again
----------------------------

Finally, here is a brief explanation of how to read the expression at
the top of the chapter. This is for advanced students only (nutcases, if
you ask me). You can find all the bits in the manual, if you try hard
enough, even the ones I didn't get around to explaining above. As an
example, let's suppose the array contains

      array=(long longer longest short brief)

and see what

      print ${array[(r)${(l.${#${(O@)array//?/X}[1]}..?.)}]}

gives.

1.  Always start from the inside. The innermost expression here is

            ${(O@)array//?/X}

    Not much clearer? Start from the inside again: there's the parameter
    we're operating on, whose name is `array`. Before that there are two
    flags in parenthesis: (`O`) says sort the result in descending
    alphabetic order, (`@`) treat the result as an array, which is
    necessary because this inner substitution occurs where a scalar
    value (actually, an arithmetic expression) would usually occur, and
    we need to take an array element. After the array name, \``//?/X`'
    is a global substitution: take the pattern \``?`' (any character)
    wherever it occurs, and replace it with the string \``X`'. The
    result of this is an array like `$array`, but with all the elements
    turned into strings consisting of \``X`'s in place of the original
    characters, and with the longest first, because that's how reverse
    alphabetic order works for strings with the same character. So

            long longer longest short brief

    would have become

            XXXXXXX XXXXXX XXXXX XXXXX XXXX

2.  Next, we have \``${#`*result*`[1]}`' wrapped around that. That means
    that we take the first element of the array we arrived at above (the
    \``[1]`': that's why we had to make sure it was treated as an
    array), and then take the length of that (the \``#`'). We will end
    up in this case with 7, the length of the first (and longest
    element). We're finally getting somewhere.
3.  The next step is the \``${`(`l.`*result*`..?.`)`}`'. Our previous
    *result* appears as an argument to the \``(l)`' flag of the
    substitution. That's a rather special case of nested substitution:
    at this point, the shell expects an arithmetical expression, giving
    the minimum length of a string to be filled on the left. The
    previous substitution was evaluated because arithmetic expressions
    undergo parameter substitution. So it is the result of that, 7,
    which appears here, giving the more manageable

            ${(l.7..?.)}

    The expression for the \``(l)`' flag in full says \`fill the result
    of this parameter substitution to a minimum width of 7 using the
    fill character \``?`'. What is the substitution we are filling? It's
    empty: zsh is smart enough to assume you know what you're doing when
    you don't give a parameter name, and just puts in an empty string
    instead. So the empty string is filled out to length 7 with question
    marks, giving \``???????`'.

4.  Now we have \``${array[(r)???????]}`'. It may not be obvious
    (congratulations if the rest is), but the question marks are active
    as a pattern. Subscripts are treated specially in this respect. The
    subscript flag \``(r)`' means \`reverse match', not reverse as in
    backwards, but as in the opposite way round: search the array itself
    for a matching value, rather than taking this as an index. The only
    thing that will match this is a string of length 7. Bingo! that must
    be the element \`longest' in this case. If there were other elements
    of the same length, you would only get the first of that length; I
    haven't thought of a way of getting all the elements of that length
    substituted by a single expression without turning `$array` into an
    associative array, so if you have, you should feel smug.

After I wrote this, Sven Wischnowsky (who is responsible for a large
fraction of the similar hieroglyphics in the completion functions)
pointed out that a similar way of achieving this is:

      print ${(M)array:#${~${(O@)array//?/?}[1]}}

which does indeed show all the elements of the maximum length. A brief
summary of how this works is that the innermost expression produces an
array of \``?`' corresponding to the elements, longest first in the way
we did above, turning the \``?`' into pattern match characters. The next
expansion picks the longest. Finally, the outermost expansion goes
through `$array` to find elements which match the complete string of
\``?`' and selects out those that do match.

If you are wondering about how to do that in perl in a single
expression, probably sorting on length is the easiest:

      # Perl code
      @array = qw(long longer longest short brief);
      @array = sort { length $b <=> length $a } @array;

and taking out the first element or first few elements of `@array`.
However, in a highly-optimized scripting language you would almost
certainly do it some other way: for example, avoid sorting and just
remember the longest element:

      # Perl code
      $elt = '';
      $l = 0;
      foreach (@array) {
        $newl = length $_;
        $elt = $_, $l = $newl  if $l > $newl;
      }
      print $elt, "\n";

You can do just the same thing in zsh easily enough in this case;

      local val elt
      integer l newl
      for val in $array; do
        newl=${#val}
        if (( newl > l )); then
          elt=$val
          (( l = newl ))
        fi
      done
      print $elt

so this probably isn't a particularly good use for nested substitution,
even though it illustrates its power.

If you enjoyed that expression, there are many more like it in the
completion function suite for you to goggle at.

5.6: Arithmetic Expansion
-------------------------

Performing mathematics within the shell was first described in [chapter
3](zshguide03.html#syntax) where I showed how to create numeric
parameters with variants of \``typeset`', and said a little about
arithmetic substitution.

In addition to the math library, loadable with
\``zmodload zsh/mathfunc`', zsh has essentially all the operators you
expect from C and other languages derived from it. In other words,
things like

      (( foo = bar ? 3 : 1, ++brr ))

are accepted. The comma operator works just as in C; all the arguments
are evaluated, in this case \``foo = bar ? 3 : 1`' assigns 3 or 1 to
`$foo` depending whether or not `bar` is non-zero, and then `$brr` is
incremented by 1. The return status is determined by the final
expression, so if `$brr` is zero after increment the return status is
one, else it is zero (integers may be negative).

One extra operator has been borrowed from FORTRAN, or maybe Perl, the
exponentiation operator, \``**`'. This can take either integers or
floating point numbers, though a negative exponent will cause a floating
point number to be returned, so \``$(( 2 ** -1 ))`' gives you 0.5, not
rounded down to zero. This is why the standard library function `pow` is
missing from `zsh/mathfunc` --- it's already there in that other form.
Pure integer exponentiation, however, is done by repeated multiplication
--- up to arbitrary sizes, so instead of \``2 ** 100`', you should use
\``1 << 100`', and for powers of any other integer where you don't need
an exact result, you should use floating point numbers. For this
purpose, the `zsh/mathfunc` library makes \`casts' available;
\``float`(*num*)' forces the expression *num* to interpreted as a
floating point number, whatever it would otherwise have given, although
the trick of adding \``0.0`' to a number works as well. Note that,
although this works like a cast in C, the syntax is that of an ordinary
function call. Likewise, \``int`(*num*)' causes the number to be
interpreted as an integer --- rounding towards zero; you can use `floor`
and `ceil` to round down or up, and `rint` to round to the nearest
integer, although these three actually produce floating point numbers.
They are standard C library functions.

For completeness, the assignment form of exponentiation \``**=`' also
works. I can't remember ever using it.

The range of integers depends on how zsh was configured on your machine.
The primary goal is to make sure integers are large enough to represent
indexes into files; on some systems where the hardware usually deals
with 32-bit integers, file sizes may be given by 64-bit integers, and
zsh will try to use 64-bit integers as well. However, zsh will test for
large integers even if no large file support is available; usually it
just requires that your compiler has some easy to recognise way of
defining 64-bit integers, such as \``long long`' which may be handled by
gcc even if it isn't by the native compiler. You can easily test; if
your zsh supports 64-bit integers, the largest available integer is:

      % print $(( 0x7FFFFFFFFFFFFFFF ))
      9223372036854775807

and if you try adding something positive to that, you will get a
negative result due to two's complement arithmetic. This should be large
enough to count most things.

The range of floating point numbers is always that of a C \``double`',
which is usually also 64 bits, and internally the number is highly
likely to be in the IEEE standard form, which also affects the precision
and range you can get, though that's system specific, too. On most
systems, the math library functions handle `double`s rather than single
precision `float`s, so this is the natural choice. The cast function is
called \``float`' because, unlike C, the representation of a floating
point number is chosen for you, so the generic name is used.

### 5.6.1: Entering and outputting bases

I'll say a word or two about bases. I already said you could enter a
number with any small base in a form like \``2#101010`' or \``16#ffff`',
and that the latter could also be \``0xffff`' as in C. You can't,
however, enter octal numbers just by using a leading \``0`', which you
might expect from C. Here's an example of why not. Let's set:

      % foo=${(%):-%D}
      % print $foo
      01-08-06

The first line is another of those bogus parameter substitutions where
we gave it a literal string and a blank parameter. We also gave it the
flag \``(%)`', which forces prompt escapes to be expanded, and in
prompts \``(%D)`' is the date as *yy*-*mm*-*dd*. Let's write a short
program to find out what the date after `$foo` is. We have the luxury of
99 years to worry about the century wrapping, so we'll ignore it (and
the Gregorian calendar).

      mlens=(31 28 31 30 31 30 31 31 30 31 30 31)
      date=(${(s.-.)foo})    #  splits to array (01 08 23)
      typeset -Z 2 incr
      if (( ${date[3]} < ${mlens[${date[2]}]} )); then
        # just increment day
        (( incr = ${date[3]} + 1 ))
        date[3]=$incr
      else
        # go to first of next month
        date[3]=01
        if (( ${date[2]} < 12 )); then
          (( incr = ${date[2]} + 1 ))
          date[2]=$incr
        else
          # happy new year
          date[2]=01
          (( incr = ${date[3]} + 1 ))
          date[3]=$incr
        fi
      fi
      print ${date[1]}-${date[2]}-${date[3]}

This will print \``01-08-07`'. Before I get to the point, various other
explanations. We forced `$foo` to be split on any \``-`' in it, giving a
three-part array. The next trick was \``typeset -Z 2 incr`', which tells
the shell that `$incr` is to be at least two characters, filled with
leading zeroes. That's how we got the \``07`' at the end, instead of
just \``7`'. There's another way of doing this: replace

     
      typeset -Z 2 incr
      (( incr = ${date[2]} + 1 ))
      date[2]=$incr

with:

      date[2]=${(l.2..0.)$(( ${date[2]} + 1 ))}

This uses the `(l)` parameter flag to fill up to two characters with a
zero (the default is a space, so we need to specify the \``0`' this
time), using the fact that parameter operations can have a nested
`$`-substution. This second form is less standard, however.

Now, finally, the point. In that \`\$(( \${date[2]} + 1 ))', the
\``${date[2]}`' is simply the *scalar* \``08`' --- the result of
splitting an arbitrary string into an array. Suppose we used leading
zeroes to signify octal numbers. We would get something like:

      % print $(( ${date[2]} + 1 ))
      zsh: bad math expression: operator expected at `8 + 1 '

because the expression in the substitution becomes \``08 + 1`' and an 8
can't appear in an octal number. So we would have to strip off any
otherwise harmless leading zeroes. Parsing dates, or indeed strings with
leading zeroes as padding, is a fairly common thing for a shell to do,
and octal arithmetic isn't. So by default leading zeroes don't have that
effect.

However, there is an option you can set, `OCTAL_ZEROES`; this is
required for compatibility with the POSIX standard. That's how I got the
error message in the previous paragraph, in fact.

Floating point numbers are never octal, always decimal:

      % setopt octalzeroes
      % print $(( 077 ))
      63
      % print $(( 077.43 ))
      77.430000000000007

The other option to do with bases is `C_BASES`, which makes hexadecimal
(and, if you have `OCTAL_ZEROES` set, octal) numbers appear in the form
that you would use as input to a C (or, once again, Perl) program.

How do you persuade the shell to print out numbers in a particular base
anyway? There are two ways. The first is to associate a base with a
parameter, which you do with an argument after the \``-i`' option to
typeset:

      % typeset -i 16 hexnum=32
      % print $hexnum
      16#20

This is the standard way. By the way, there's a slight catch with bases,
taken over from ksh: if you *don't* specify a base, the first assignment
will do the job for you.

      % integer anynum
      % (( anynum = 16#20 ))
      % print $anynum
      16#20

Only constants with explicit bases in an expression produce this effect;
the first time \``anynum`' comes into contact with a \`*base*`#`*num*',
or a hexadecimal or (where applicable) octal expression in the standard
C form, it will acquire a default output base. So you need to use
\``typeset -i 10`' if you don't like that.

Often, however, you just want to print out an expression in, say,
hexadecimal. Zsh has a shorthand for this, which is only in recent
versions (and not in other shells). Preceding an expression by
\``[#`*base*`]`' causes the default output base to be set to `base` with
the the usual prefix showing the base, and \``[##`*base*`]`' will do the
same but without the prefix, i.e. \``$(( [##16]255 ))`' is simply
\``FF`'. This has no effect on assignments to a parameter, not even on
the parameter's default output base, but it will affect the result of a
direct substitution using `$((...))`.

### 5.6.2: Parameter typing

Just as creating a parameter with an ordinary assignment makes it a
scalar, so creating it in an arithmetic substitution makes it either an
integer or a floating point parameter, according to the value assigned.
This is likely to be a floating point number if there was a floating
point number in the expression on the right hand side, and an integer
otherwise. However, there are reasons why a floating point number on the
right may not have this effect --- use of `int`, for example, since it
produces an integer.

However, relying on implicit typing in this fashion is bad. One of the
reasons is explained in the manual entry, and I can't do better than use
that example (since I wrote it):

      for (( f = 0; f < 1; f += 0.1 )); do
        print $f
      done

If you try this, and `$f` does not already exist, you will see an
endless stream of zeroes. What's happening is that the original
assignment creates `$f` as an integer to store the integer `0` in. After
printing this, `$f` is incremented by adding `0.1` to it. But once
created, `$f` remains an integer, so the resulting `0.1` is cast back to
an integer, and the resulting zero is stored back in `$f`. The result is
that `$f` is never incremented.

You could turn the first `0` into `0.0`, but a better way is to declare
\``float f`' before the loop. In a function, this also ensures `$f` is
local to the function.

If you use a scalar to store an integer or floating point, everything
will work. You don't have the problem just described, since although
`$f` contains what looks like an integer to start with, it has no
numeric type associated with it, and when you store `0.1` into `$f`, it
will happily overwrite the string \``0`'. It's a bit more inefficient to
use scalars, but actually not that much. You can't specify an output
base or precision, and in versions of zsh up to 4.0.x, there is a
problem when the parameter already has a string in it which doesn't make
sense as a numeric expression:

      % foo='/file/name'
      % (( foo = 3 ))
      zsh: bad math expression: operand expected at `/file/name'

The unexpected error comes because \``/file/name/`' is evaluated even
though the shell is about to overwrite the contents of `$foo`. Versions
of the shell from 4.1.1 have a fix for this, and the integer assignment
works as expected.

You need to be careful with scalars that might contain an empty string.
If you declare \``integer i`', it will immediately contain the value 0,
but if you declare \``typeset s`', the scalar `$s` will just contain the
empty string. You get away with this if you use the parameter without a
\``$`' in front:

      % typeset s
      % print $(( 3 * s ))
      0

because the math code tries to retrieve `$s`, and when it fails puts a
`0` there. However, if you explicitly use `$s`, the math code gets
confused:

      % print $(( 3 * $s ))
      zsh: bad math expression: operand expected at `'

because \``$s`' evaluates to an empty string before the arithmetic
evaluation proper, which spoils the syntax. There's one common case
where you need to do that, and that's with positional parameters:

      % fn() { print "Twice $1 is $(( 2 * $1 ))"; }
      % fn 3
      Twice 3 is 6
      % fn
      fn: bad math expression: operand expected at `'

Obviously turning the \``$1`' into \``1`' means something completely
different. You can guard against this with default values:

      % fn() { print "Twice ${1:=0} is $(( 2 * $1 ))"; }
      % fn
      Twice 0 is 0

This assigns a default value for `$0` if one was not set. Since
parameter expansion is performed in one go from left to right, the
second reference to `$1` will pick up that value.

Note that you need to do this even if it doesn't look like the number
will be needed:

      % fn() { print $(( ${1:-0} ? $1 : 3 )); }
      % fn
      fn: bad math expression: operand expected at `: 3 '

The expression before the \``?`' evaluates to zero if `$1` is not
present, and you expect the expression after the colon to be used in
that case. But actually it's too late by then; the arithmetic expression
parser has received \``0 ? : 3`', which doesn't make sense to it, hence
the error. So you need to put in \``${1:-0}`' for the second `$1`, too
--- or `${1:-32}`, or any other number, since it won't be evaluated if
`$1` is empty, it just needs to be parsed.

You should note that just as you can put numbers into scalar parameters
without needing any special handling, you can also do all the usual
string-related tricks on numeric parameters, since there is automatic
conversion in the other direction, too:

      % float foo
      % zmodload -i zsh/mathfunc
      % (( foo = 4 * atan(1.0) ))
      % print $foo
      3.141592654e+00
      % print ${foo%%.*}${foo##*.[0-9]##}
      3e+00

The argument `-i` to `zmodload` tells it not to complain if the math
library is already loaded. This gives us access to `atan`. Remember,
\``float`' declares a parameter whose output includes an exponent ---
you can actually convert it to a fixed point format on the fly using
\``typeset -F foo`', which retains the value but alters the output type.
The substitution uses some `EXTENDED_GLOB` chicanery: the final
\``[0-9]##`' matches one or more occurrences of any decimal digit. So
the head of the string value of `$foo` up to the last digit after the
decimal point is removed, and the remainder appended to whatever appears
before the decimal point.

Starting from 4.1.1, a calculator function called `zcalc` is bundled
with the shell. You type a standard arithmetic expression and the shell
evaluates the formula and prints it out. Lines already entered are
prefixed by a number, and you can use the positional parameter
corresponding to that number to retrieve that result for use in a new
formula. The function uses `vared` to read the formulae, so the full
shell editing mechanism is available. It will also read in
`zsh/mathfunc` if that is present.

5.7: Brace Expansion and Arrays
-------------------------------

Brace expansion, which you met in [chapter 3](zshguide03.html#syntax),
appears in all csh derivatives, in some versions of ksh, and in bash, so
is fairly standard. However, there are some features and aspects of it
which are only found in zsh, which I'll describe here.

A complication occurs when arrays are involved. Normally, unquoted
arrays are put into a command line as if there is a break between
arguments when there is a new element, so

      % array=(three separate words)
      % print -l before${array}after
      beforethree
      separate
      wordsafter

unless the `RC_EXPAND_PARAM` option is set, which combines the before
and after parts with *each* element, so you get:

      % print -l before${^array}after
      beforethreeafter
      beforeseparateafter
      beforewordsafter

--- the \``^`' character turns on the option just for that expansion, as
\``=`' does with `SH_WORD_SPLIT`. If you think of the character as a
correction to a proof, meaning \`insert a new word between the others
here', it might help you remember (this was suggested by Bart Schaefer).

These two ways of expanding arrays interact differently with braces; the
more useful version here is when the `RC_EXPAND_PARAM` option is on.
Here the array acts as sort of additional nesting:

      % array=(two three)
      % print X{one,${^array}}Y
      XoneY XtwoY XoneY XthreeY

with the `XoneY` tacked on each time, but because of the braces it
appears as a separate word, so there are four altogether.

If `RC_EXPAND_PARAM` is not set, you get something at first sight
slightly odd:

      % array=(two three)
      % print X{one,$array}Y
      X{one,two three}Y

What has happened here is that the `$array` has produced two words; the
first has \``X{one,`' tacked in front of the array's \``two`', while the
second likewise has \``}Y`' on the end of the array's \``three`'. So by
the time the shell comes to think about brace expansion, the braces are
in different words and don't do anything useful.

There's no obvious simple way of forcing the `$array` to be embedded in
the braces at the same level, instead of like an additional set of
braces. There are more complicated ways, of course.

      % array=(two three)
      % print X${^=:-one $array}Y
      XoneY XtwoY XthreeY

Yuk. We gave parameter substitution a string of words, the array with
`one` stuck in front, and told it to split them on spaces (this will
split on any extra spaces in elements of `$array`, unfortunately), while
setting `RC_EXPAND_PARAM`. The parameter flags are \``^=`'; the \``:-`'
is the usual \`insert the following if the substitution has zero length'
operator. It's probably better just to create your own temporary array
and apply `RX_EXPAND_PARAM` to that. By the way, if you had
`RC_EXPAND_PARAM` set already, the last result would have been different
becuase the embedded `$array` would have been expanded together with the
\``one `' in front of it.

Braces allow numeric expressions; this works a little like in Perl:

      % print {1..10}a
      1a 2a 3a 4a 5a 6a 7a 8a 9a 10a

and you can ask the numbers to be padded with zeroes:

      % print {01..10}b
      01b 02b 03b 04b 05b 06b 07b 08b 09b 10b

or have them in descending order:

      % print {10..1}c
      10c 9c 8c 7c 6c 5c 4c 3c 2c 1c

Nesting this within other braces works in the expected way, but you
can't have any extra braces inside: the syntax is fixed to number, two
dots, number, and the numbers must be positive.

There's also an option `BRACE_CCL` which, if the braces aren't in either
of the above forms, expands single letters and ranges of letters:

      % setopt braceccl
      % print 1{abw-z}2
      1a2 1b2 1w2 1x2 1y2 1z2

An important point to be made about braces is that they are *not* part
of filename generation; they have nothing to do with pattern matching at
all. The shell blindly generates all the arguments you specify. If you
want to generate only some arguments, depending on what files are
matched, you should use the alternative-match syntax. Compare:

      % ls
      file1
      % print file(1|2)
      file1
      % print file{1,2}
      file1 file2

The first matches any of \``file1`' or \``file2`' it happens to find in
the directory (regardless of other files). The second doesn't look at
files in the directory at all; it simply expands the braces according to
the rules given above.

This point is particularly worthy of note if you have come from a
C-shell world, or use the `CSH_NULL_GLOB` option:

      csh% echo file{1,2}
      file1 file2
      csh% echo f*{1,2}
      file1

(\``csh%`' is the prompt, to remind you if you're skipping through
without reading the text), where the difference occurs because in the
first case there was no pattern, so brace expansion was done on ordinary
words, while in the second case the \``*`' made pattern expansion
happen. In zsh, the sequence would be: \``f*{1,2}`' becomes
\``f*1 f*2`'; the first becomes `file1` and the second fails to match.
With `CSH_NULL_GLOB` set, the failed match is simply removed; there is
no error because one pattern has succeeded in matching. This is
presumably the logic usually followed by the C shell. If you stick with
\``file(1|2)`' and \``f*(1|2)`' --- in this case you can simplify them
to \``file[12]`' and \``f*[12]`', but that's not true if you have more
than one character in either branch --- you are protected from this
difference.

5.8: Filename Expansion
-----------------------

Filename expansions consists of just \``~/...`', \``~user/...`',
\``~namedir/...`' and \``=prog`', where the \``~`' and \``=`' must be
the first character of a word, and the option `EQUALS` must be set (it
is by default) for the \``=`' to be special. I told you about all this
in [chapter 3](zshguide03.html#syntax).

There's really only one thing to add, and that's the behaviour of the
`MAGIC_EQUAL_SUBST` option. Assignments after `typeset` and similar
statements are handled as follows

      % typeset foo=~pws
      % print $foo
      /home/pws
      % typeset PATH=$PATH:~pws/bin
      % print ${path[-1]}
      /home/pws/bin

It may not be obvious why this is not obvious. The point is that
\``typeset`' is an ordinary command which happens to be a shell builtin;
the arguments of ordinary commands are not assignments. However, a
special case is made here for `typeset` and its friends so that this
works, even though, as I've said repeatedly, array assignments can't be
done after `typeset`. The parameter `$PATH` isn't handled differently
from any other --- any colon in an assignment to any variable is special
in the way shown.

It's often useful to have this feature with commands of your own. There
is an option, `MAGIC_EQUAL_SUBST`, which spots the forms \``...=~...`'
and \``...=...:~...`' for any command at all and expands
`~`-expressions. Commands where this is particularly useful include
`make` and the GNU `configure` command used for setting up the
compilation of a software package from scratch.

A related new option appeared in version 4.0.2 when it became clear
there was an annoying difference between zsh and other shells such as
ksh and bash. Consider:

      export FOO=`echo hello there`

In ksh and bash, this exports `$foo` with the value \``hello there`'. In
zsh, however, an unquoted backquote expression forces wordsplitting, so
the line becomes

      export FOO=hello there

and exports `$FOO` with the value \``hello`', and `$there` with any
value it happens to have already or none if it didn't exist. This is
actually perfectly logical according to the rules, but you can set the
option `KSH_TYPESET` to have the other interpretation.

Normally, `KSH_TYPESET` applies only after parameter declaration
builtins, and then only in the values of an assignment. However, in
combination with `MAGIC_EQUAL_SUBST`, you will get the same behaviour
with any command argument that looks like an assignment --- actually,
anything following an \``=`' which wasn't at the start of the word, so
\``"hello mother, => I'm home "$(echo right now)`' qualifies.

It seems that bash behaves as if both `KSH_TYPESET` *and*
`MAGIC_EQUAL_SUBST` are always in effect.

5.9: Filename Generation and Pattern Matching
---------------------------------------------

The final topic is perhaps the biggest, even richer than parameter
expansion. I'm finally going to explain the wonderful world of zsh
pattern matching. In addition to patterns as such, you will learn such
things as how to find all files in all subdirectories, searching
recursively, which have a given name, case insensitive, are at least 50
KB large, no more than a week old and owned by the root user, and
allowing up to a single error in the spelling of the name. In fact, the
required expression looks like this:

      **/(#ia1)name(LK+50mw-1u0)

which might appear, at first sight, a mite impenetrable. We'll work up
to it gradually.

To repeat: filename generation is just the same as globbing, only
longer. I use the terms interchangeably.

### 5.9.1: Comparing patterns and regular expressions

It can be confusing that there are two rather different sorts of pattern
around, those used for matching files on a command line as in zsh and
other shells, and those used for matching text inside files as in
`grep`, `sed`, `emacs`, `perl` and many other utilities, each of which,
typically, has a slightly different form for patterns (called in this
case \`regular expressions', because UNIX was designed by computer
scientists). There are even some utilities like TCL which provide both
forms.

Zsh deals exclusively with the shell form, which I've been calling by
its colloquial name, \`globbing', and consequently I won't talk about
regular expressions in any detail. Here are the two classic differences
to note. First, in a shell, \``*`' on its own matches any set of
characters, while in a regular expression it always refers to the
previous pattern, and says that that can be repeated any number of
times. Second, in a shell \``.`' is an ordinary (and much used)
character, while in a regular expression it means \`any character',
which is specified by \``?`' in the shell. Put this together, and what a
shell calls \``*`' is given by \``.*`' in a regular expression. \``*`'
in the latter case is called a \`Kleene closure': it's those computer
scientists again. In zsh, art rather than science tends to be in
evidence.

In fact, zsh does have many of the features available in regular
expressions, as well as some which aren't. Remember that anywhere in zsh
where you need a pattern, it's of the same form, whether it's matching
files on the command line or a string in a `case` statement. There are a
few features which only fit well into one or another use of patterns;
for example the feature that selects files by examining their type,
owner, age, etc. (the final parenthesis in the expression I showed
above) are no use in matching against a string.

### 5.9.2: Standard features

There is one thing to note about the simple pattern matching features
\``*`' and \``?`', which is that when matching file names (not in other
places patterns are used, however) they never match a leading \``.`'.
This is a convention in UNIX-like systems to hide certain files which
are not interesting to most users. You may have got the impression that
files begining with \``.`' are somehow special, but that's not so; only
the files \``.`' (the current directory) and \``..`' (the parent
directory, or the current directory in `/`) are special to the system.
Other files beginning with \``.`' only appear special because of a
conspiracy between the shell (the rule I've just given) and the command
`ls`, which, when it lists a directory, doesn't show files beginning
\``.`' unless you give the \``-a`' option. Otherwise \``.`'-files are
perfectly normal files.

You can suppress the special rule for an initial \``.`' by setting the
option `GLOB_DOTS`, in which case \``*`' will match every single file
and directory except for \``.`' and \``..`'.

In addition to \``*`' and \``?`', which are so basic that even DOS had
them (though I never *quite* worked out exactly what it was doing with
them a lot of the time), the pattern consisting of a set of characters
in square brackets appears in all shells. This feature happens to be
pretty much the same as in regular expressions. \``[abc]`' matches any
one of those three characters; \``[a-z]`' matches any character between
`a` and `z`, inclusive; \``[^a-z]`' matches any single character
*except* those 26 --- but notice it still matches a single character.

A recent common enhancement to character ranges features in zsh, which
is to specify types of characters instead of listing them; I'm just
repeating the manual entry here, which you should consult for more
detail. The special syntax is like \``[:`*spec*`:]`', where the square
brackets there are in addition to the ones specifying the range. If you
are familiar with the \`ctype' macros use in C programmes, you will
probably recognise the things that *spec* can be: `alnum`, `alpha`,
`blank`, `cntrl`, `digit`, `graph`, `lower`, `print`, `punct`, `space`,
`upper`, `xdigit`. The similarity to C macros isn't just for show: the
shell really does call the macro (or function) \``isalpha`' to test for
`[:alpha:]`ness, and so on. On most modern systems which support
internationalization this means the shell can tell you whether a
character is, say, an alphabetic letter in the character set in use on
your machine. By the way, zsh doesn't use international character set
support for sorting matches --- this turned out to produce too many
unexpected effects.

So \``[^[:digit:]]`' matches any single character other than a decimal
digit. Standards say you should use \``!`' instead of \``^`' to signify
negation, but most people I know don't; also, this can clash with
history substitution. However, it is accepted by zsh anywhere where
history substitution doesn't get its hands on the \``!`' first (which
includes all scripts and autoloaded functions).

### 5.9.3: Extensions usually available

Now we reach the bits specific to zsh. I've divided these into two
parts, since some require the option \``EXTENDED_GLOB`' to be set ---
those which are most likely to clash with other uses of the characters
in question.

**Numeric ranges**\
\

One possibility that is always available is the syntax for numeric
ranges in the form \``<`*num1*`-`*num2*`>`'. You can omit either *num1*,
which defaults to zero, or *num2*, which defaults to infinity, or both,
in which case any set of digits will be matched. Note that this really
*does* mean infinity, despite the finite range of integers; missing out
*num2* is treated as a special case and the shell will simply advance
over any number of digits. (In *very* old versions of zsh you had to use
\``<>`' to get that effect, but that has been removed and \``<>`' is now
a redirection operator, as in other shells; \``<->`' is what you need
for any set of digits.)

I repeat another warning from the manual: this test

      [[ 342 = <1-30>* ]]

succeeds, even though the number isn't in the range 1 to 30. That's
because \``<1-30>`' matches \``3`' and \``*`' matches 42. There's no use
moaning, it's a consequence of the usual rule for patterns of all types
in shells or utilities: pattern operators are tried independently, and
each \`uses up' the longest piece of the string it is matching without
causing the rest of the match to fail. We would have to break this
simple and well-tried rule to stop numeric ranges matching if there is
another digit left. You can test for that yourself, of course:

      [[ 342 = <1-30>(|[^[:digit:]]*) ]]

fails. I wrote it so that it would match any number between 1 and 30,
either not followed by anything, or followed by something which doesn't
start with a digit; I will explain what the parentheses and the vertical
bar are doing in the next section. By the way, leading zeroes are
correctly handled (and never force octal interpretation); so
\``00000003NaN`' would successfully match the pattern.

The numbers in the range are always positive integers; you need extra
pattern trickery to match floating point. Here's one attempt, which uses
`EXTENDED_GLOB` operators, so come back and look when you've read the
rest of this section if it doesn't make sense now:

      isfloat() {
        setopt localoptions extendedglob
        if [[ $1 = ([-+]|)([0-9]##.[0-9]#|[0-9]#.[0-9]##)\ 
    ([eE]([-+]|)[0-9]##|) ]]; then
          print -r -- "$1 is a floating point number"
        else
          print -r -- "$1 is not a floating point number"
        fi
      }

I've split it over two lines to fit. The first parenthesis matches an
optional minus or plus sign --- careful with \``-`' in square brackets,
since if it occurs in the middle it's taken as a range, and if you want
it to match itself, it has to be at the start or end. The second
parenthesis contains an alternative because \``.`' isn't a floating
point number (at least, not in my book, and not in zsh's, either), but
both \``0.`' and \``.0`' *are* properly formed numbers. So we need at
least one digit, either before or after the decimal point; the \``##`'
means \`at least one occurrence of the previous expression', while the
\``#`' means \`zero or more occurrences of the previous expression'. The
expresion on the next line matches an exponent; here you need at least
one digit, too. So \``3.14159E+00`' is successfully matched, and indeed
you'll find that zsh's arithmetic operations handle it properly.

The range operator is the only special zsh operator that you can't turn
off with an option. This is usually not a problem, but in principle a
string like \``<3-10>`' is ambiguous, since in another shell it would be
read as \``<3-10 >`', meaning \`take input from file `3-10`, and send
output to the file formed by whatever comes after the expression'. It's
very unlikely you will run across this in practice, however, since shell
code writers nearly alwys put a space after the end of a file name for
redirection if something else follows on the command line, and that's
enough to differentiate it from a range operator.

**Parentheses**\
\

Parentheses are quite natural in zsh if you've used extended regular
expressions. They are usually available, and only turned off if you set
the \``SH_GLOB`' option to ensure compatibility with shells that don't
have it. The key part of the expression is the vertical bar, which
specifies an alternative. It can occur as many times as necessary;
\``(a|b|c|d|e|f|g|h|i|j|k|l|m)`' is a rather idiosyncratic way of
writing \``[a-m]`'. If you don't include the vertical bar (we'll see
reasons for not doing so later), and you are generating filenames, you
should be careful that the expression doesn't occur at the end of the
pattern, else it would be taken as a \`glob qualifier', as described
below. The rather unsightly hack of putting \``(|)`' (match the empty
string or the empty string --- guess what this matches?) right at the
end will get around that problem.

The vertical bar usually needs to be inside parentheses so that the
shell doesn't take it as a pipe, but in some contexts where this won't
happen, such as a case statement label, you can omit any parentheses
that would completely surround the pattern. So in

      case $foo in
        (bar|rod|pipe) print "foo represents a piece of metal"
                  ;;
        (*) print "Are you trying to be different?"
            ;;
      esac

the surrounding parentheses are the required syntax for `case`, rather
than pattern parentheses --- the same syntax works in other shells. Then
\``bar|rod`' is an ordinary zsh expression matching either `bar` or
`rod`, in a context where the \``|`' can't be mistaken for a pipe. In
fact, this whole example works with `ksh` --- but there the use of
\``|`' is a special case, while in zsh it fits in with the standard
pattern rules.

Indeed, ksh has slightly different ways of specifying patterns: to make
the use of parentheses less ambiguous, it requires a character before
the left parenthesis. The corresponding form for a simple alternative is
\``@(this|that)`'. The \``@`' can also be a \``?`', for zero or one
occurrences of what's in the parentheses; \``*`' for any number of
repetitions, for example \``thisthisthatthis`'; or \``!`' for anything
except what's in the parentheses. Zsh allows this syntax if you set the
option `KSH_GLOB`. Note that this is independent of the option
`SH_GLOB`; if you set `KSH_GLOB` but not `SH_GLOB`, you can actually use
both forms for pattern matching, with the ksh form taking precedence in
the case of ambiguities. This is probably to be avoided. In ksh
emulation, both options are set; this is the only sensible reason I know
of for using these options at all. I'll show some comparisons in the
next section.

An important thing to note is that when you are matching files, you
can't put directory separators inside parentheses:

      # Doesn't work!
      print (foo/bar|bar/foo)/file.c

doesn't work. The reason is that it's simply too difficult to write;
pattern matching would be bound in a highly intricate way with searching
the directory hierarchy, with the end of a group sending you back up to
try another bit of the pattern on a directory you'd already visited.
It's probably not impossible, but the pattern code maintainer (me) isn't
all that enthusiastic about it.

### 5.9.4: Extensions requiring `EXTENDED_GLOB`

Setting `EXTENDED_GLOB` makes three new types of operator available:
those which excluded a particular pattern from matching; those which
specify that a pattern may occur a repeated number of times; and a set
of \`globbing flags', a little bit like parameter flags which I'll
describe in a later section since they are really the icing on the cake.

**Negative matches or exclusions**\
\

The simpler of the two exclusions uses \``^`' to introduce a pattern
which must *not* be matched. So a trivial example (I will assume for
much of the rest of the chapter that the option `EXTENDED_GLOB` is set)
is:

      [[ foo = ^foo ]]
      [[ bar = ^foo ]]

The first test fails, the second succeeds. It's important to realise
that that the pattern demands nothing else whatever about the relevant
part of the test string other than it doesn't match the pattern that
follows: it doesn't say what length the matched string should have, for
example. So

      [[ foo = *^foo ]]

actually *does* match: `*` swallows up the whole string, and the
remaining empty string successfully fails to be \``foo`'. Remember the
mantra: each part of the pattern matches the longest possible substring
that causes the remainder of the pattern not to fail (unless, of course,
failure is unavoidable).

Note that the `^` applies to the whole pattern to its right, either to
the end of the string, or to the end of the nearest enclosing
parenthesis. Here's a couple more examples:

      [[ foo = ^foo* ]]

Overall, this fails to match: the pattern \``foo*`' always matches the
string on the left, so negating means it always fails.

      [[ foo = (^foo)* ]]

This is similar to the last example but one. The expression in the
parenthesis first matches against `foo`; this causes the overall match
to fail because of the `^`, so it backs up one character and tries
again. Now \``fo`' is successfully matched by `^foo` and the remaining
\``o`' is matched by the `*`, so the overall match succeeds. When you
know about backreferences, you will be able to confirm that, indeed, the
expression in parentheses matches \``fo`'. This is a quite subtle point:
it's easy to imagine that \``^foo`' says \`match any three letter string
except the one I've given you', but actually there is no requirement
that it match three letters, or indeed any.

In filename generation, the `^` has a lower precedence than a slash:

      % print /*/tmp
      /data/tmp /home/tmp /usr/tmp /var/tmp
      % print /^usr/tmp
      /data/tmp /home/tmp /var/tmp

successfully caused the first level of directories to match anything but
\``usr`'. A typical use of this with files is \``^*.o`' to match
everything in a directory except files which end with \``.o`'.

Note one point mentioned in the FAQ --- probably indicating the reason
that \``^`' is only available with `EXTENDED_GLOB` switched on. Some
commands use an initial \``^`' to indicate a control character; in fact,
zsh's `bindkey` builtin does this:

      bindkey '^z' backward-delete-word

which attaches the given function to the keystroke `Ctrl-z`. You must
remember to quote that keystroke expression, otherwise it would expand
to a list of all files in the current directory not called \``z`', very
likely all of them.

There's another reason this isn't available by default: in some versions
of the Bourne shell, \``^`' was used for pipes since \``|`' was missing
on some keyboards.

The other exclusion operator is closely related. \`*pat1*`~`*pat2*'
means \`anything that matches *pat1* as long as it doesn't also match
*pat2*'. If *pat1* is `*`, you have the same effect as \``^`' --- in
fact, that's pretty much how \``^`' is currently implemented.

There's one significant difference between \``*~`*pat*' and \``^`*pat*':
the `~` has a *lower* precedence than \``/`' when matching against
filenames. What's more, the pattern on the right of the `~` is not
treated as a filename at all; it's simply matched against any filename
found on the left, to see if it should be rejected. This sounds like
black magic, but it's actually quite useful, particularly in combination
with the recursive globbing syntax:

       print **/*~*/CVS(/)

matches any subdirectory of the current directory to any depth, except
for directories called `CVS` --- the \``*`' on the right of the \``~`'
will match any character including \``/`'. The final \``(/)`' is a glob
qualifier indicating that only directories are to be allowed to match
--- note that it's a positive assertion, despite being after the \``~`'.
Glob qualifiers do not feel the effect of preceding exclusion operators.

Note that in that example, any subdirectory of a directory called `CVS`
would have matched successfully; you can see from the pattern that the
expression after the \``~`' wouldn't weed it out. Slightly less
obviously, the \``**/*`' matches files in the current directory, while
the \``*/CVS`' never matches a \``CVS`' in the current directory, so
that could appear. If you want to, you can fix that up like this:

       print **/*~(*/|)CVS(/*|)(/)

again relying on the fact that \``/`'s are not special after the \``~`'.
This will ruthlessly weed out any path with a directory component called
`CVS`. An easier, but less instructive, way is

       print ./**/*~*/CVS(/)

You can restrict the range of the tilde operator by putting it in
parentheses, so \``/(*~usr)/tmp`' is equivalent to \``/^usr/tmp`'.

A \``~`' at the beginning is never treated as excluding what follows; as
you already know, it has other uses. Also, a \``~`' at the end of a
pattern isn't special either; this is lucky, because Emacs produces
backup files by adding a \``~`' to the end of the file name. You may
have problems if you use Emacs's facility for numbered backup files,
however, since then there is a \``~`' in the middle of the file name,
which will need to be quoted when used in the shell.

**Closures or repeated matches**\
\

The extended globbing symbols \``#`' and \``##`', when they occur in a
pattern, are equivalent to \``*`' and \``+`' in extended regular
expressions: \``#`' allows the previous pattern to match any number of
times, including zero, while with \``##`' it must match at least once.
Note that this pattern does not extend beyond two hashes --- there is no
special symbol \``###`', which is not recognised as a pattern at all.

The \`previous pattern' is the smallest possible item which could be
considered a complete pattern. Very often it is something in
parentheses, but it could be a group in square or angle brackets, or a
single ordinary character. Note particularly that in

      # fails
      [[ foofoo = foo# ]]

the test fails, because the \``#`' only refers to the final \``o`', not
the entire string. What you need is

      # succeeds
      [[ foofoo = (foo)# ]]

It might worry you that \``#`' also introduces comments. Since a
well-formatted pattern never has \``#`' at the start, however, this
isn't a problem unless you expect comments to start in the middle of a
word. It turns out that doesn't even happen in other shells --- \``#`'
must be at the start of a line, or be unquoted and have space in front
of it, to be recognised as introducing a comment. So in fact there is no
clash at all here. There is, of course, a clash if you expect
\``.#foo.c.1.131`' (probably a file produced by the version control
system CVS while attempting to resolve a conflict) to be a plain string,
hence the dependence on the `EXTENDED_GLOB` option.

That's probably all you need to know; the \``#`' operators are generally
much easier to understand than the exclusion operators. Just in case you
are confused, I might as well point out that repeating a *pattern* is
not the same as repeating a *string*, so

      [[ onetwothreetwoone = (one|two|three)## ]]

successfully matches; the string is different for each repetition of the
pattern, but that doesn't matter.

We now have enough information to construct a list of correspondences
between zsh's normal pattern operators and the ksh ones, available with
`KSH_GLOB`. Be careful with \``!`(*...*)'; it seems to have a slightly
different behaviour to the zsh near-equivalent. The following table is
lifted directly from the zsh FAQ.

    ----------------------------------------------------------------------
          ksh              zsh         Meaning
         ------           ------       ---------
         !(foo)            ^foo        Anything but foo.
                    or   foo1~foo2     Anything matching foo1 but foo2.
    @(foo1|foo2|...)  (foo1|foo2|...)  One of foo1 or foo2 or ...
         ?(foo)           (foo|)       Zero or one occurrences of foo.
         *(foo)           (foo)#       Zero or more occurrences of foo.
         +(foo)           (foo)##      One or more occurrences of foo.
    ----------------------------------------------------------------------

In both languages, the vertical bar for alternatives can appear inside
any set of parentheses. Beware of the precedences of `^foo` and
\``foo1~foo2`'; surround them with parentheses, too, if necessary.

### 5.9.5: Recursive globbing

One of the most used special features of zsh, and one I've already used
a couple of times in this section, is recursive globbing, the ability to
match any directory in an arbitrarily deep (or, as we say in English,
tall) tree of directories. There are two forms: \``**/`' matches a set
of directories to any depth, including the top directory, what you get
by replacing \``**/`' by \``./`, i.e. `**/foo` can match `foo` in the
current directory, but also `bar/foo`, `bar/bar/bar/foo`,
`bar/bar/bar/poor/little/lambs/foo` nad so on. \``***/`' does the same,
but follows symbolic links; this can land you in infinite loops if the
link points higher up in the same directory hierarchy --- an odd thing
to do, but it can happen.

The \``**/`' or \``***/`' can't appear in parentheses; there's no way of
specifying them as alternatives. As already noticed, however, the
precedence of the exclusion operator \``~`' provides a useful way of
removing matches you don't want. Remember, too, the recursion operators
don't need to be at the start of the pattern:

      print ~/**/*.txt

prints the name of all the files under your home directory ending with
\``.txt`'. Don't expect it to be particularly fast; it's not as well
optimised as the standard UNIX command `find`, although it is a whole
lot more convenient. The traditional way of searching a file which may
be anywhere in a directory tree is along the lines of:

      find ~/src -name '*.c' -print | xargs grep pattern

which is horrendously cumbersome. What's happening is that `find`
outputs a newline-separated list of all the files it finds, and `xargs`
assembles these as additional arguments to the command
\``grep pattern`'. It simplifies in zsh to the much more natural

      grep pattern ~/src/**/*.c

In fact, strictly speaking you probably ought to use

      find ~/src -name '*.c' -print0 | xargs -0 grep pattern

for the other form --- this passes null-terminated strings around, which
is safer since any character other than a NUL or a slash can occur in a
filename. But of course you don't need that now.

Do remember that this includes the current directory in the search, so
in that last example \``foo.c`' in the directory where you typed the
command would be searched. This isn't completely obvious because of the
\``/`' in the pattern, which erroneously seems to suggest at least one
directory.

It's a little known fact that this is a special case of a more general
syntax, \`(*pat*`/`)`#`'. This syntax isn't perfect, either; it's the
only time where a \``/`' can usefully occur in parentheses. The pattern
*pat* is matched against each directory; if it succeeds, *pat* is
matched against each of the subdirectories, and so on, again to
arbitrary depth. As this uses the character \``#`', it requires the
`EXTENDED_GLOB` option, which the more common syntax doesn't, since
no-one would write two `*`'s in a row for any other reason.

You should consider the \``/`)' to be in effect a single pattern token;
for example in

      % print (F*|B*/)#*.txt
      FOO/BAR/thingy.txt

both \``F*`' and \``B*`' are possible directory names, not just the
\``B*`' next to the slash. The difference between \``#`' and \``##`' is
respected here --- with the former, zero occurrences of the pattern may
be matched (i.e. \``*.txt`'), while with the latter, at least one level
of subdirectories is required. Thus \``(*/)##*.txt`' is equivalent to
\``*/**/*.txt`', except that the first \``*`' in the second pattern will
match a symbolic link to a directory; there's no way of forcing the
other syntax to follow symbolic links.

Fairly obviously, this syntax is only useful with files. Other uses of
patterns treat slashes as ordinary characters and \``**`' or \``***`'
the same as a single \``*`'. It's not an error to use multiple \``*`'s,
though, just pointless.

### 5.9.6: Glob qualifiers

Another very widely used zsh enhancement is the ability to select types
of file by using \`glob qualifiers', a group of (rather terse) flags in
parentheses at the end of the pattern. Like recursive globbing, this
feature only applies for filename generation in the command line
(including an array assignment), not for other uses of patterns.

This feature requires the `BARE_GLOB_QUAL` option to be turned on, which
it usually is; the name implies that one day there may be another,
perhaps more ksh-like, way of doing the same thing with a more
indicative syntax than just a pair of parentheses.

**File types**\
\

The simplest glob qualifiers are similar to what the completion system
appends at the end of file names when the `LIST_TYPES` option is on;
these are in turn similar to the indications used by \``ls -F`'. So

      % print *(.)
      file1 file2 cmd1 cmd2
      % print *(/)
      dir1 dir2
      % print *(*)
      cmd1 cmd2
      % print *(@)
      symlink1 symlink2

where I've invented unlikely filenames with obvious types. `file1` and
`file2` were supposed to be just any old file; `(.)` picks up those but
also executable files. Sockets `(=)`, named pipes `(p)`, and device
files `(%)` including block `(%b)` and character `(%c)` special files
are the other types of file you can detect.

Associated with type, you can also specify the number of hard links to a
file: `(l2)` specifies exactly 2 links, `(l+3)` more than 3 links,
`(l-5)` fewer than 5.

**File permissions**\
\

Actually, the `(*)` qualifier really applies to the file's permissions,
not it's type, although it does require the file to be an executable
non-special file, not a directory nor anything wackier. More basic
qualifiers which apply just to the permissions of the files are `(r)`,
`(w)` and `(x)` for files readable, writeable and executable by the
owner; `(R)`, `(W)` and `(X)` correspond to those for world permissions,
while `(A)`, `(I)` and `(E)` do the job for group permissions --- sorry,
the Latin alphabet doesn't have middle case. You can speciy permissions
more exactly with \``(f)`' for file permissions: the expression after
this can take various forms, but the easiest is probably a delimited
string, where the delimiters work just like the arguments for parameter
flags and the arguments, separated by commas, work just like symbolic
arguments to `chmod`; the example from the manual,

      print *(f:gu+w,o-rx:)

picks out files (of any type) which are writeable by the owner (\`user')
and group, and neither readable nor executable by anyone else
(\`other').

**File ownership**\
\

You can match on the other three mode bits, setuid ((s)), setgid ((S))
and sticky ((t)), but I'm not going to go into what those are if you
don't know; your system's manual page for `chmod` may (or may not)
explain.

Next, you can pick out files by owner; `(U)` and `(G)` say that you or
your group, respectively, owns the file --- really the effective user or
group ID, which is usually who you are logged in as, but this may be
altered by tricks such as a programme running setuid or setgid (the
things I'm not going to explain). More generally, `u0` says that the
file is owned by root and `(u501)` says it is owned by user ID 501; you
can use names if you delimiit them, so `(u:pws:)` says that the owner
must be user `pws`; similarly for groups with `(g)`.

**File times**\
\

You can also pick files by modification ((m)) or access ((a)) time,
either before ((-)), at, or after ((+)) a specific time, which may be
measured in days (the default), months ((M)), weeks ((w)), hours ((h)),
minutes ((m)) or seconds ((s)). These must appear in the order `m` or
`a`, optional unit, optional plus or minus, number. Hence:

      print *(m1)

Files that were modified one day ago --- i.e. less than 48 but more than
24 hours ago.

      print *(aw-1)

Files accessed within the last week, i.e. less than 7 days ago.

In addition to `(m)` and ((a)), there is also `(c)`, which is sometimes
said to refer to file creation, but it is actually something a bit less
useful, namely *inode* change. The inode is the structure on disk where
UNIX-like filing systems record the information about the location and
nature of the file. Information here can change when some aspect of the
file information, such as permissions, changes.

**File size**\
\

The qualifier `(L)` refers to the file size (\`L' is actually for
length), by default in bytes, but it can be in kilobytes `(k)`,
megabytes `(m)`, or 512-byte blocks `(p, unfortunately)`. Plus and minus
can be used in the same way as for times, so

      print *(Lk3)

gives files 3k large, i.e. larger than 2k but smaller than 4k, while

      print *(Lm+1)

gives files larger than a megabyte.

Note that file size applies to directories, too, although it's not very
useful. The size of directories is related to the number of slots for
files currently available inside the directory (at the highest level,
i.e. not counting children of children and deeper). This changes
automatically if necessary to make more space available.

**File matching properties**\
\

There are a few qualifiers which affect option settings just for the
match in question: `(N)` turns on `NULL_GLOB`, so that the pattern
simply disappears from the command line if it fails to match; `(D)`
turns on `GLOB_DOTS`, to match even files beginning with a \``.`', as
described above; `(M)` or `(T)` turn on `MARK_DIRS` or `LIST_TYPES`, so
that the result has an extra character showing the type of a directory
only (in the first case) or of any special file (in the second); and
`(n)` turns on `NUMERIC_GLOB_SORT`, so that numbers in the filename are
sorted arithmetically --- so `10` comes after `1A`, because the 1 and 10
are compared before the next character is looked at.

Other than being local to the pattern qualified, there is no difference
in effect from setting the option itself.

**Combining qualifiers**\
\

One of the reasons that some qualifiers have slightly obscure syntax is
that you can chain any number of them together, which requires that the
file has all of the given properties. In other words \``*(UWLk-10)`' are
files owned by you, world writeable and less than 10k in size.

You can negate a set of qualifiers by putting \``^`' in front of those,
so \``*(ULk-10^W)`' would specify the corresponding files which were not
world writeable. The \``^`' applies until the end of the flags, but you
can put in another one to toggle back to assertion instead of negation.

Also, you can specify alternatives; \``*(ULk-10,W)`' are files which
either are owned by you and are less than 10k, or are world writeable
--- note that the \`and' has higher precedence than the \`or'.

You can also toggle whether the assertions or negations made by
qualifiers apply to symbolic links, or the files found by following
symbolic links. The default is the former --- otherwise the `(@)`
qualifier wouldn't work on its own. By preceding qualifiers with `-`,
they will follow symbolic links. So `*(-/)` matches all directories,
including those reached by a symbolic link (or more than one symbolic
link, up to the limit allowed by your system). As with \``^`', you can
toggle this off again with another one \``-`'. To repeat what I said in
[chapter 3](zshguide03.html#syntax), you can't distinguish between the
other sort of links, hard links, and a real file entry, because a hard
link just supplies an alternative but equivalent name for a file.

There's a nice trick to find broken symlinks: the pattern \``**/*(-@)`'.
This is supposed to follow symlinks; but that \``@`' tells it to match
only on symlinks! There is only one case where this can succeed, namely
where the symlink is broken. (This was pointed out to me by Oliver
Kiddle.)

**Sorting and indexing qualifiers**\
\

Normally the result of filename generation is sorted by alphabetic order
of filename. The globbing flags `(o)` and `(O)` allow you to sort in
normal or reverse order of other things: `n` is for names, so `(on)`
gives the default behaviour while `(On)` is reverse order; `L`, `l`,
`m`, `a` and `c` refer to the same thing as the normal flags with those
letters, i.e. file size, number of links, and modification, access and
inode change times. Finally, `d` refers to subdirectory depth; this is
useful with recursive globbing to show a file tree ordered depth-first
(subdirectory contents appear before files in any given directory) or
depth-last.

Note that time ordering produces the most recent first as the standard
ordering (`(om)`, etc.), and oldest first as the reverse ordering
`(OM)`, etc.). With size, smallest first is the normal ordering.

You can combine ordering criteria, with the most important coming first;
each criterion must be preceded by `o` or `O` to distinguish it from an
ordinary globbing flag. Obviously, `n` serves as a complete
discriminator, since no two different files can have the same name, so
this must appear on its own or last. But it's a good idea, when doing
depth-first ordering, to use `odon`, so that files at a particular depth
appear in alphabetical order of names. Try

      print **/*(odon)

to see the effect, preferably somewhere above a fairly shallow directory
tree or it will take a long time.

There's an extra trick you can play with ordered files, which is to
extract a subset of them by indexing. This works just like arrays, with
individual elements and slices.

      print *([1])

This selects a single file, the first in alphabetic order since we
haven't changed the default ordering.

      print *(om[1,5])

This selects the five most recently modified files (or all files, if
there are five or fewer). Negative indices are understood, too:

      print *(om[1,-2])

selects all files but the oldest, assuming there are at least two.

Finally, a reminder that you can stick modifiers after qualifiers, or
indeed in parentheses without any qualifiers:

      print **/*(On:t)

sorts files in subdirectories into reverse order of name, but then
strips off the directory part of that name. Modifiers are applied right
at the end, after all file selection tasks.

**Evaluating code as a test**\
\

The most complicated effect is produced by the `(e)` qualifer. which is
followed by a string delimited in the now-familiar way by either
matching brackets of any of the four sorts or a pair of any other
characters. The string is evaluated as shell code; another layer of
quotes is stripped off, to make it easier to quote the code from
immediate expansion. The expression is evaulated separately for each
match found by the other parts of the pattern, with the parameter
`$REPLY` set to the filename found.

There are two ways to use `(e)`. First, you can simply rely on the
return code. So:


      print *(e:'[[ -d $REPLY ]]':)
      print *(/)

are equivalent. Note that quotes around the expression, which are
necessary in addition to the delimiters (here \``:`') for expressions
with special characters or whitespace. In particular, `$REPLY` would
have been evaluated too early --- before file generation took place ---
if it hadn't been quoted.

Secondly, the function can alter the value of `$REPLY` to alter the name
of the file. What's more, the expression can set `$reply` (which
overrides the use of `$REPLY`) to an array of files to be inserted into
the command line; it may be any size from zero items upward.

Here's the example in the manual:

      print *(e:'reply=(${REPLY}{1,2})':)

Note the string is delimited by colons *and* quoted. This takes each
file in the current directory, and for each returns a match which has
two entires, the filename with \``1`' appended and the filename with
\``2`' appended.

For anything more complicated than this, you should write a shell
function to use `$REPLY` and set that or `$reply`. Then you can replace
the whole expression in quotes with that name.

### 5.9.7: Globbing flags: alter the behaviour of matches

Another `EXTENDED_GLOB` features is \`globbing flags'. These are a bit
like the flags that can appear in perl regular expressions; instead of
making an assertion about the type of the resulting match, like glob
qualifiers do, they affect the way the match is performed. Thus they are
available for all uses of pattern matching --- though some flags are not
particularly useful with filename generation.

The syntax is borrowed from perl, although it's not the same: it looks
like \``(#X)`', where `X` is a letter, possibily followed by an argument
(currently only a number and only if the letter is \``a`'). Perl
actually uses \``?`' instead of \``#`'; what these have in common is
that they can't appear as a valid pattern characters just after an open
parenthesis, since they apply to the pattern before. Zsh doesn't have
the rather technical flags that perl does (lookahead assertions and so
on); not surprisingly, its features are based around the shortcuts often
required by shell users.

**Mixed-case matches**\
\

The simplest sort of globbing flag will serve as an example. You can
make a pattern, or a portion of a pattern, match case-insensitively with
the flag `(#i)`:

      [[ FOO = foo ]]
      [[ FOO = (#i)foo ]]

Assuming you have `EXTENDED_GLOB` set so that the \``#`' is an active
pattern character, the first match fails while the second succeeds. I
mentioned portions of a pattern. You can put the flags at any point in
the pattern, and they last to the end either of the pattern or any
enclosing set of parentheses, so in

      [[ FOO = f(#i)oo ]]
      [[ FOO = F(#i)oo ]]

once more the first match fails and the second succeeds. Alternatively,
you can put them in parentheses to limit their scope:

      [[ FOO = ((#i)fo)o ]]
      [[ FOO = ((#i)fo)O ]]

gives a failure then a success again. Note that you need extra
parentheses; the ones around the flag just delimit that, and have no
grouping effect. This is different from Perl.

There are two flags which work in exactly the same way: `(#l)` says that
only lowercase letters in the pattern match case-insensitively;
uppercase letters in the pattern only match uppercase letters in the
test string. This is a little like Emacs' behaviour when searching case
insensitvely with the `case-fold-search` option variable set; if you
type an uppercase character, it will look only for an uppercase
character. However, Emacs has the additional feature that from that
point on the whole string becomes case-sensitive; zsh doesn't do that,
the flag applies strictly character by character.

The third flag is `(#I)`, which turns case-insensitive matching off from
that point on. You won't often need this, and you can get the same
effect with grouping --- unless you are applying the case-insensitive
flag to multiple directories, since groups can't span more than one
directory. So

      print (#i)/a*/b*/(#I)c*

is equivalent to

      print /[aA]*/[bB]*/c*

Note that case-insensitive searching only applies to characters not in a
special pattern of some sort. In particular, ranges are not
automatically made case-insensitive; instead of \``(#i)[ab]*`', you must
use \``[abAB]*`'. This may be unexpected, but it's consistent with how
other flags, notably approximation, work.

You should be careful with matching multiple directories
case-insensitively. First,

      print (#i)~/.Z*

doesn't work. This is due to the order of expansions: filename expansion
of the tilde happens before pattern matching is ever attempted, and the
\``~`' isn't at the start where filename expansion needs to find it.
It's interpreted as an empty string which doesn't match \``/.Z*`',
case-insensitively --- in other words, it will match any empty string.

Hence you should put \``(#i)`' and any other globbing flags after the
first slash --- unless, for some reason, you *really* want the
expression to match \``/Home/PWS/`' etc. as well as \``/home/pws`'.

Second,

      print (#i)$HOME/.Z*

does work --- prints all files beginning \``.Z`' or \``.z`' in your home
directory --- but is inefficient. Assume `$HOME` expands to my home
directory, `/home/pws`. Then you are telling the shell it can match in
the directories \``/Home/PWS/`', \``/HOME/pWs`' and so on. There's no
quick way of doing this --- the shell has to look at every single entry
first in \``/`' and then in \``/home`' (assuming that's the only match
at that level) to check for matches. In summary, it's a good idea to use
the fact that the flag doesn't have to be at the beginning, and write
this as:

      print ~/(#i).Z*

Of course,

      print ~/.[zZ]*

would be easier and more standard in this oversimplified example.

On `Cygwin`, a UNIX-like layer running on top of, uh, a well known
graphical user interface claiming to be an operating system, filenames
are usually case insensitive anyway. Unfortunately, while Cygwin itself
is wise to this fact, zsh isn't, so it will do all that extra searching
when you give it the `(#i)` flag with an otherwise explicit string.

A piece of good news, however, is that matching of uppercase and
lowercase characters will handle non-ASCII character sets, provided your
system handles locales, (or to use the standard hieroglyphics, \`i18n'
--- count the letters between \`i' and \`n' in \`internationalization',
which may not even be a word anyway, and wince). In that case you or
your system administrator or the shell environment supplied by your
operating system vendor needs to set `$LC_ALL` or `$LC_CTYPE` to the
appropriate locale -- C for the default, `en` for English, `uk` for
Ukrainian (which I remember because it's confusing in the United
Kingdom), and so on.

**\`Backreferences'**\
\

The feature labelled as \`backreferences' in the manual isn't really
that at all, which is my fault. Many regular expression matchers allow
you to refer back to bits already matched. For example, in Perl the
regular expression \``([A-Z]{3})$1`' says \`match three uppercase
characters followed by the same three characters again. The \``$1`' is a
backreference.

Zsh has a similar feature, but in fact you can't use it while matching a
single pattern; it just makes the characters matched by parentheses
available after a successful complete match. In this, it's a bit more
like Emacs's `match-beginning` and `match-end` functions.

You have to turn it on for each pattern with the globbing flag
\``(#b)`'. The reason for this is that it makes matches involving
parentheses a bit slower, and most of the time you use parentheses just
for ordinary filename generation where this feature isn't useful. Like
most of the other globbing flags, it can have a local effect: only
parentheses after the flag produce backreferences, and the effect is
local to enclosing parentheses (which don't feel the effect themselves).
You can also turn it off with \``(#B)`'.

What happens when a pattern with active parentheses matches is that the
elements of the array `$match`, `$mbegin` and `$mend` are set to reflect
each active parenthesis in turn --- names inspired by the corresponding
Emacs feature. The string matching the first pair of parentheses is
stored in the first element of `$match`, its start position in the
string is stored in the first element of `$mbegin`, and its end position
in the string `$mend`. The same happens for later matched parentheses.
The parentheses around any globbing flags do not count.

`$mbegin` and `$mend` use the indexing convention currently in effect,
i.e. zero offset if `KSH_ARRAYS` is set, unit offset otherwise. This
means that if the string matched against is stored in the parameter
`$teststring`, then it will always be true that `${match[1]}` is the
same string as `${teststring[${mbegin[1]},${mend[1]}]}`. and so on. (I'm
assuming, as usual, that `KSH_ARRAYS` isn't set.) Unfortunately, this is
different from the way the `E` parameter flag works --- that substitutes
the character after the end of the matched substring. Sorry! It's my
fault for not following that older convention; I thought the string
subscripting convention was more relevant.

An obvious use for this is to match directory and non-directory parts of
a filename:

      local match mbegin mend
      if [[ /a/file/name = (#b)(*)/([^/]##) ]]; then
        print -l ${match[1]} ${match[2]}
      fi

prints \``/a/file`' and \``name`'. The second parenthesis matches a
slash followed by any number of characters, but at least one, which are
not slashes, while the first matches anything --- remember slashes
aren't special in a pattern match of this form. Note that if this
appears in a function, it is a good idea to make the three parameters
local. You don't have to clear them, or even make them arrays. If the
match fails, they won't be touched.

There's a slightly simpler way of getting information about the match:
the flag `(#m)` puts the matched string, the start index, and the index
for the *whole* match into the scalars `$MATCH`, `$MBEGIN` and `$MEND`.
It may not be all that obvious why this is useful. Surely the whole
pattern always matches the whole string? Actually, you've already seen
cases where this isn't true for parameter substitutions:

      local MATCH MBEGIN MEND string
      string=aLOha
      : ${(S)string##(#m)([A-Z]##)}

You'll find this sets `$MATCH` to `LO`, `$MBEGIN` to 2 and `$MEND` to 3.
In the parameter expansion, the `(S)` is for matching substrings, so
that the \``##`' match isn't anchored to the start of `$string`. The
pattern is `(#m)([A-Z]##)`, which means: turn on full-match
backreferencing and match any number of capital letters, but at least
one. This matches `LO`. Then the match parameters let you see where in
the test parameter the match occurred.

There's nothing to stop you using both these types of backreferences at
once, and you can specify multiple globbing flags in the short form
\``(#bm)`'. This will work with any combination of flags, except that
some such as \``(#bB)`' are obviously silly.

Because ordinary globbing produces a list of files, rather than just
one, this feature isn't very useful and is turned off. However, it *is*
possible to use backreferences in global substitutions and substitutions
on arrays; here are both at once:

      % array=(mananan then in gone June)
      % print ${array//(#m)?n/${(C)MATCH[1]}n}
      mAnAnAn thEn In gOne JUne

The substitution occurs separately on each element of the array, and at
each match in each element `$MATCH` gets set to what was matched. We use
this to capitalize every character that is followed by a lowercase
\``n`'. This will work with the `(#b)` form, too. The perl equivalent of
this is:

      % perl -pe 's/.n/\u$&/g' <<<$array
      mAnAnAn thEn In gOne JUne

(People sometimes say Perl has a difficult syntax to understand; I hope
I'm convincing you how naive that view is when you have zsh.)

Now I can convince you of one point I made about excluded matches above:

      % [[ foo = (#b)(^foo)* ]]  && print $match
      fo

As claimed, the process of making the longest possible match, then
backtracking from the end until the whole thing is successful, leads to
the \``(^foo)`' matching \``fo`'.

**Approximate matching**\
\

To my knowledge, zsh is the first command line interpreter to make use
of approximate matching. This is very useful because it provides the
shell with an easy way of correcting what you've typed. First, some
basics about what I mean by \`approximate matching'.

There are four ways you can make a mistake in typing. You can leave out
a letter which should be there; you can insert a letter which shouldn't;
you can type one letter instead of another; and you can transpose two
letters. The last one involves two different characters, so some systems
for making approximate matches count it as two different errors; but
it's a particularly common one when typing, and quite useful to be able
to handle as a single error. I know people who even have \``mkae`'
aliased to \``make`'.

You can tell zsh how many errors you are willing to allow in a pattern
match by using, for example `(#a1)`, which says only a single error
allowed. The rules for the flag are almost identical to those for
case-insensitive matching, in particular for scoping and the way
approximate matching is handled for a filename expansion with more than
one directory. The number of errors is global; if the shell manages to
match a directory in a path with an error, one fewer error is allowed
for the rest of the path. You can specify as many errors as you like;
the practical limit is that with too many allowed errors the pattern
will match far too many strings. The shell doesn't have a particularly
nifty way of handling approximate matching (unlike, for example, the
program `agrep`), but you are unlikely to encounter problems if the
number of matches stays in a useful range.

The fact that the error count applies to the whole of a filename path is
a bit of a headache, actually, because we have to make sure the shell
matches each directory with the minimum number of errors. With a single
pattern, the shell doesn't care as long as it doesn't use up all the
errors it has, while with multiple directories if it uses up the errors
early on, it may fail to match something it should match. But you don't
have to worry about that; this explanation is just to elicit sympathy.

So the pattern `(#a1)README` will match `README`, `READ.ME`, `READ_ME`,
`LEADME`, `REDME`, `READEM`, and so on. It will not match `_README_`,
`ReadMe`, `READ` or `AAREADME`. However, you can combine it with
case-insensitivity, for which the short form `(#ia1)README` is allowed,
and then it will match `ReadMe`, `Read.Me`, `read_me`, and so on. You
can consider filenames with multiple directories as single strings for
this purpose --- with one exception, that \``foo/bar`' and \``fo/obar`'
are two errors apart, not one. Because the errors are counted separately
in each directory, you can't transpose the \``/`' with another
character. This restriction doesn't apply in other forms of pattern
matching where `/` is not a special character.

Another common feature with case-insensitive matching is that only the
literal parts of the string are handled. So if you have \``[0-9]`' in a
pattern, that character must match a decimal digit even if approximation
is active. This is often useful to impose a particular form at key
points. The main difficulty, as with the \``/`' in a directory, is that
transposing with another character is not allowed, either. In other
words, \``(#a1)ab[0-9]`' will fail to match \``a1b`'; it will match with
two errors, by removing the \``b`' before the digit and inserting it
after.

As an example of what you can do with this feature, here is a simple
function to correct misspelled filenames.

      emulate -LR zsh
      setopt extendedglob

      local file trylist
      integer approx max_approx=6

      file=$1

      if [[ -e $file ]]; then
        # no correction necessary
        print $file
        return
      fi

      for (( approx = 1; approx <= max_approx; approx++ )); do
        trylist=( (#a$approx)"$file"(N) )
        (( $#trylist )) && break
      done
      (( $#trylist )) || return 1

      print $trylist

The function tries to match a file with the minimum possible number of
errors, but in any case no more than 6. As soon as it finds a match, it
will print it out and exit. It's still possible there is more than one
match with that many errors, however, and in this case the complete list
is printed. The function doesn't handle \``~`' in the filename.

It does illustrate the fact that you can specify the number of
approximations as a parameter. This is purely a consequence of the fact
that filename generation happens right at the end of the expansion
sequence, after the parameters have already been substituted away. The
numbers and the letter in the globbing flag aren't special characters,
unlike the parentheses and the \``#`'; if you wanted those to be special
when substituted from a parameter, you would need to set the `KSH_GLOB`
flag, possibly by using the \``~`' parameter flag.

A more complicated version of that function is included with the shell
distribution in the file `Completion/Base/Widget/_correct_filename`.
This is designed to be used either on its own, or as part of the
completion system.

Indeed, the completion system described in the next chapter is where you
are most likely to come across approximate matching, buried inside
approximate completion and correction --- in the first case, you tell
the shell to complete what you have typed, trying to correct mistakes,
and in the second case, you tell the shell that you have finished typing
but possibly made some mistakes which it should correct. If you already
have the new completion system loaded, you can use `^Xc` to correct a
word on the command line; this is context-sensitive, so more
sophisticated than the function I showed.

**Anchors**\
\

The last two globbing flags are probably the least used. They are there
really for completeness. They are `(#s)`, to match only at the start of
a string, and `(#e)`, to match only at the end. Unlike the other flags
they are purely local, just making a statement about the point where
they occur in the pattern.

They correspond to the much more commonly used \``^`' and \``$`' in
regular expressions. The difference is that shell patterns nearly always
match a complete string, so telling the pattern that a certain point is
the start or end isn't usually very useful. There are two occasions when
it is. The first is when the start or end is to be matched as an
alternative to something else. For example,

      [[ $file = *((#s)|/)dirpart((#e)|/)* ]]

succeeds if `dirpart` is a complete path segment of `$file` --- with a
slash or nothing at all before and after it. Remember, once again, that
slashes aren't special in pattern matches unless they're performing
filename generation. The effect of these two flags isn't altered at all
by their being inside another set of parentheses.

The second time these are useful is in parameter matches where the
pattern is not guaranteed to match a complete string. If you use `(#s)`
or `(#e)`, it will force that point to be the start or end despite the
operator in use. So `${`*param*`##`*pattern*`(#e)}` will remove
*pattern* from `$`*param* only if it matches the entire string: the `##`
must match at the head, while the `(#e)` must match at the end.

You can get the effect with `${`*param*`:#`*pattern*`}`, and further
more this is rather faster. The `:#` operator has some global knowledge
about how to match; it knows that since *pattern* will match as far as
it can along the test string, it only needs to try the match once.
However, since \``##`' just needs to match at the head of the string, it
will backtrack along the pattern, trying to match *pattern*`(#e)`,
entirely heedless of the fact that the pattern itself specifically won't
match if it doesn't extend to the end. So it's more efficient to use the
special parameter operators whenever they're available.

### 5.9.8: The function `zmv`

The shell is supplied with a function `zmv`, which may have been
installed into the default `$fpath`, or can be found in the source tree
in the directory `Functions/Misc`. This provides a way of renaming,
copying and linking files based on patterns. The idea is that you give
two arguments, a pattern to match, and a string which uses that pattern.
The pattern to match has backreferences turned on; these are stored in
the positional parameters to make them easy to refer to. The function
tries to be safe: any file whose name is not changed is simply ignored,
and usually overwriting an existing file is an error, too. However, it
doesn't make sure that there is a one to one mapping from source to
target files; it doesn't know if the target file is supposed to be a
directory (though it could be smarter about that).

In the examples, I will use the option `-n`, which forces `zmv` to print
out what it will do without actually doing it. This is a good thing to
try first if you are unsure.

Here's a simple example.

      % ls
      foo
      % zmv -n '(*)' '${(U)1}'
      mv -- foo FOO

The pattern matches anything in the current directory, excluding files
beginning with a \``.`' (the function starts with an \``emulate`', so
`GLOB_DOTS` is forced to be off). The complete string is stored as the
first backreference, which is in turn put into `$1`. Then the second
argument is used and `$1` in uppercase is substituted.

**Essentials of the function**\
\

The basic code in `zmv` is very simple. It boils down to more or less
the following.

      setopt nobareglobqual extendedglob
      local files pattern result f1 f2 match mbegin mend
      
      pattern=$1
      result=$2

      for f1 in ${~pattern}; do
        [[ $f1 = (#b)${~pattern} ]] || continue
        set -- $match
        f2=${(e)result}
        mv -- $f1 $f2
      done

Here's what's going on. We store the arguments as `$pattern` and
`$result`. We then expand the pattern to a list of files --- remember
that `${~pattern}` makes the characters in `$pattern` active for the
purposes of globbing. For each file we find, we match against the
pattern again, but this time with backreferences turned on, so that
parentheses are expanded into the array `$match`. If, for some reason,
the pattern match failed this time, we just skip the file. Then we store
`$match` in the positional parameters; the \``-``-`' for `set` and for
`mv` is in case `$match[1]` begins with a \``-`'.

Then we evaluate the result, assuming that it will refer to the
positional parameters. In our example, `$result` contains argument
\``${(U)1}`' and if we matched \``foo`', then `$1` contains foo. The
effect of \``${(e)result}`' is to perform an extra substitution on the
`${(U)1}`, so `$f2` will be set to `FOO`. Finally, we use the `mv`
command to do the actual renaming. The effect of the `-n` option isn't
shown, but it's essentially to put a \``print`' in front of the `mv`
command line.

Notice I set `nobareglobqual`, turning off the use of glob qualifiers.
That's necessary because of all those parentheses; otherwise, \``(*)`'
would have been interpreted as a qualifier. There is an option, `-Q`,
which will turn qualifiers back on, if you need them. That's still not
quite ideal, since the second pattern match, the one where we actually
use the backreferences, isn't filename generation, just a test against a
string, so doesn't handle glob qualifers. So in that case the code needs
to strip qualifiers off. It does this by a fairly simple pattern match
which will work in simple cases, though you can confuse it if you try
hard enough, particularly if you have extra parentheses in the glob
qualifier.

Note also the use of \``${(e)result}`' to force substitution of
parameters when `$result` is evaluated. This way of doing it safely
ignores other metacharacters which may be around: all `$`-expansions,
plus backquote expansion, are performed, but otherwise `$result` is left
alone.

**More complicated examples**\
\

`zmv` has some special handling for recursive globbing, but only with
the patterns `**/` and `***/`. If you put either of these in parentheses
in the pattern, they will be spotted and used in the normal way. Hence,

      % ls test
      lonely
      % zmv -n '(**/)lonely' '$1solitary'
      mv -- test/lonely test/solitary

Note that, as with other uses of \``**/`', the slash is part of the
recursive match, so you don't need another one. You don't need to
separate `$1` from `solitary` either, since positional parameters are a
special case, but you could use \``${1}solitary`' for the look of it.
Like glob qualifiers, recursive matches are handled by some magic in the
function; in ordinary globbing you can't put these two forms inside
parentheses.

For the lazy, the option `-w` (which means \`with wildcards') will tell
`zmv` to decide for itself where all the patterns are and automatically
add parentheses. The two examples so far become

      zmv -nw '*' '${(U)1}'
      zmv -nw '***/lonely' '$1solitary'

with exactly the same effects.

Another way of getting useful effects is to use the \``${1//foo/bar}`'
substitution in the second argument. This gives you a general way of
substitution bits in filenames. Often, you can then get away with having
\``(*)`' as the first argument:

      zmv '(*)' '${1//(#m)[aeiou]/${(U)MATCH}}'

capitalises all the vowels in all filenames in the current directory.
You may be familiar with a perl script called `rename` which does tricks
like this (though there's another, less powerful, programme of the same
name which simply replaces strings).

**The effect of `zmv`**\
\

In addition to renaming, `zmv` can be made to copy or link files. If you
call it `zcp` or `zln` instead of `zmv`, it will have those effects, and
in the case of `zln` you can use the option `-s` to create symbolic
links, just as with `ln`. Beware the slightly confusing behaviour of
symbolic links containing relative directories, however.

Alternatively, you can force the behavour of `zmv`, `zcp` and `zln` just
by giving the options `-M`, `-C` or `-L` to the function, whatever it is
called. Or you can use \``-p` *prog*' to execute `prog` instead of `mv`,
`cp` or `ln`; *prog* should be able to be run as \`*prog* `-``-`
*oldname* *newname*', whatever it does.

The option `-i` works a bit like the same option to the basic programmes
which `zmv` usually calls, prompting you before any action --- in this
case, not just overwriting, but any action at all. Likewise, `-f` tells
`zmv` to force overwriting of files, which it will usually refuse to do
because of the potential dangers. Although many versions of `mv` etc.
take this option, some don't, so it's not passed down; instead there's a
generic way of passing down options to the programmes executed, using
`-o` followed by a string. For example,

      % ls
      foo
      % zmv -np frud -o'-a -b' '(*)' '${(U)1}'
      frud -a -b -- foo FOO

