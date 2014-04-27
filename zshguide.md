A User's Guide to the Z-Shell
=============================

Peter Stephenson
----------------

2003/03/23
----------

- - -

Table of Contents
=================

[Chapter 1: A short introduction](chapter1.md)
--------------------------------------------------------------------------------------

### [1.1: Other shells and other guides](chapter1.md#11-other-shells-and-other-guides)

### [1.2: Versions of zsh](chapter1.md#12-versions-of-zsh)

### [1.3: Conventions](chapter1.md#13-conventions)

### [1.4: Acknowledgments](chapter1.md#14-acknowledgments)

[Chapter 2: What to put in your startup files](chapter2.md)
---------------------------------------------------------------------------------------------------

### [2.1: Types of shell: interactive and login shells](chapter2.md#21-types-of-shell-interactive-and-login-shells)

[2.1.1: What is a login shell? Simple tests](chapter2.md#211-what-is-a-login-shell?-simple-tests)

### [2.2: All the startup files](chapter2.md#22-all-the-startup-files)

### [2.3: Options](chapter2.md#23-options)

### [2.4: Parameters](chapter2.md#24-parameters)

[2.4.1: Arrays](chapter2.md#241-arrays)

### [2.5: What to put in your startup files](chapter2.md#25-what-to-put-in-your-startup-files)

[2.5.1: Compatibility options: `SH_WORD_SPLIT` and others](chapter2.md#251-compatibility-options-sh_word_split-and-others)

[2.5.2: Options for csh junkies](chapter2.md#252-options-for-csh-junkies)

[2.5.3: The history mechanism: types of history](chapter2.md#253-the-history-mechanism-types-of-history)

[2.5.4: Setting up history](chapter2.md#254-setting-up-history)

[2.5.5: History options](chapter2.md#255-history-options)

[2.5.6: Prompts](chapter2.md#256-prompts)

[2.5.7: Named directories](chapter2.md#257-named-directories)

[2.5.8: \`Go faster' options for power users](chapter2.md#258-\go-faster'-options-for-power-users)

[2.5.9: aliases](chapter2.md#259-aliases)

[2.5.10: Environment variables](chapter2.md#2510-environment-variables)

[2.5.11: Path](chapter2.md#2511-path)

[2.5.12: Mail](chapter2.md#2512-mail)

[2.5.13: Other path-like things](chapter2.md#2513-other-path-like-things)

[2.5.14: Version-specific things](chapter2.md#2514-version-specific-things)

[2.5.15: Everything else](chapter2.md#2515-everything-else)

[Chapter 3: Dealing with basic shell syntax](chapter3.md)
--------------------------------------------------------------------------------------------------

### [3.1: External commands](chapter3.md#31-external-commands)

### [3.2: Builtin commands](chapter3.md#32-builtin-commands)

[3.2.1: Builtins for printing](chapter3.md#321-builtins-for-printing)

[3.2.2: Other builtins just for speed](chapter3.md#322-other-builtins-just-for-speed)

[3.2.3: Builtins which change the shell's state](chapter3.md#323-builtins-which-change-the-shell's-state)

[3.2.4: cd and friends](chapter3.md#324-cd-and-friends)

[3.2.5: Command control and information commands](chapter3.md#325-command-control-and-information-commands)

[3.2.6: Parameter control](chapter3.md#326-parameter-control)

[3.2.7: History control commands](chapter3.md#327-history-control-commands)

[3.2.8: Job control and process control](chapter3.md#328-job-control-and-process-control)

[3.2.9: Terminals, users, etc.](chapter3.md#329-terminals,-users,-etc)

[3.2.10: Syntactic oddments](chapter3.md#3210-syntactic-oddments)

[3.2.11: More precommand modifiers: `exec`, `noglob`](chapter3.md#3211-more-precommand-modifiers-exec,-noglob)

[3.2.12: Testing things](chapter3.md#3212-testing-things)

[3.2.13: Handling options to functions and scripts](chapter3.md#3213-handling-options-to-functions-and-scripts)

[3.2.14: Random file control things](chapter3.md#3214-random-file-control-things)

[3.2.15: Don't watch this space, watch some other](chapter3.md#3215-don't-watch-this-space,-watch-some-other)

[3.2.16: And also](chapter3.md#3216-and-also)

### [3.3: Functions](chapter3.md#33-functions)

[3.3.1: Loading functions](chapter3.md#331-loading-functions)

[3.3.2: Function parameters](chapter3.md#332-function-parameters)

[3.3.3: Compiling functions](chapter3.md#333-compiling-functions)

### [3.4: Aliases](chapter3.md#34-aliases)

### [3.5: Command summary](chapter3.md#35-command-summary)

### [3.6: Expansions and quotes](chapter3.md#36-expansions-and-quotes)

[3.6.1: History expansion](chapter3.md#361-history-expansion)

[3.6.2: Alias expansion](chapter3.md#362-alias-expansion)

[3.6.3: Process, parameter, command, arithmetic and brace expansion](chapter3.md#363-process,-parameter,-command,-arithmetic-and-brace-expansion)

[3.6.4: Filename Expansion](chapter3.md#364-filename-expansion)

[3.6.5: Filename Generation](chapter3.md#365-filename-generation)

### [3.7: Redirection: greater-thans and less-thans](chapter3.md#37-redirection-greater-thans-and-less-thans)

[3.7.1: Clobber](chapter3.md#371-clobber)

[3.7.2: File descriptors](chapter3.md#372-file-descriptors)

[3.7.3: Appending, here documents, here strings, read write](chapter3.md#373-appending,-here-documents,-here-strings,-read-write)

[3.7.4: Clever tricks: exec and other file descriptors](chapter3.md#374-clever-tricks-exec-and-other-file-descriptors)

[3.7.5: Multios](chapter3.md#375-multios)

### [3.8: Shell syntax: loops, (chapter3.md#38-shell-syntax-loops,-(sub)shells-and-so-on)

[3.8.1: Logical command connectors](chapter3.md#381-logical-command-connectors)

[3.8.2: Structures](chapter3.md#382-structures)

[3.8.3: Subshells and current shell constructs](chapter3.md#383-subshells-and-current-shell-constructs)

[3.8.4: Subshells and current shells](chapter3.md#384-subshells-and-current-shells)

### [3.9: Emulation and portability](chapter3.md#39-emulation-and-portability)

[3.9.1: Differences in detail](chapter3.md#391-differences-in-detail)

[3.9.2: Making your own scripts and functions portable](chapter3.md#392-making-your-own-scripts-and-functions-portable)

### [3.10: Running scripts](chapter3.md#310-running-scripts)

[Chapter 4: The Z-Shell Line Editor](chapter4.md)
------------------------------------------------------------------------------------------

### [4.1: Introducing zle](chapter4.md#41-introducing-zle)

[4.1.1: The simple facts](chapter4.md#411-the-simple-facts)

[4.1.2: Vi mode](chapter4.md#412-vi-mode)

### [4.2: Basic editing](chapter4.md#42-basic-editing)

[4.2.1: Moving](chapter4.md#421-moving)

[4.2.2: Deleting](chapter4.md#422-deleting)

[4.2.3: More deletion](chapter4.md#423-more-deletion)

### [4.3: Fancier editing](chapter4.md#43-fancier-editing)

[4.3.1: Options controlling zle](chapter4.md#431-options-controlling-zle)

[4.3.2: The minibuffer and extended commands](chapter4.md#432-the-minibuffer-and-extended-commands)

[4.3.3: Prefix (chapter4.md#433-prefix-(digit)-arguments)

[4.3.4: Words, regions and marks](chapter4.md#434-words,-regions-and-marks)

[4.3.5: Regions and marks](chapter4.md#435-regions-and-marks)

### [4.4: History and searching](chapter4.md#44-history-and-searching)

[4.4.1: Moving through the history](chapter4.md#441-moving-through-the-history)

[4.4.2: Searching through the history](chapter4.md#442-searching-through-the-history)

[4.4.3: Extracting words from the history](chapter4.md#443-extracting-words-from-the-history)

### [4.5: Binding keys and handling keymaps](chapter4.md#45-binding-keys-and-handling-keymaps)

[4.5.1: Simple key bindings](chapter4.md#451-simple-key-bindings)

[4.5.2: Removing key bindings](chapter4.md#452-removing-key-bindings)

[4.5.3: Function keys and so on](chapter4.md#453-function-keys-and-so-on)

[4.5.4: Binding strings instead of commands](chapter4.md#454-binding-strings-instead-of-commands)

[4.5.5: Keymaps](chapter4.md#455-keymaps)

### [4.6: Advanced editing](chapter4.md#46-advanced-editing)

[4.6.1: Multi-line editing](chapter4.md#461-multi-line-editing)

[4.6.2: The builtin vared and the function zed](chapter4.md#462-the-builtin-vared-and-the-function-zed)

[4.6.3: The buffer stack](chapter4.md#463-the-buffer-stack)

### [4.7: Extending zle](chapter4.md#47-extending-zle)

[4.7.1: Widgets](chapter4.md#471-widgets)

[4.7.2: Executing other widgets](chapter4.md#472-executing-other-widgets)

[4.7.3: Some special builtin widgets and their uses](chapter4.md#473-some-special-builtin-widgets-and-their-uses)

[4.7.4: Special parameters: normal text](chapter4.md#474-special-parameters-normal-text)

[4.7.5: Other special parameters](chapter4.md#475-other-special-parameters)

[4.7.6: Reading keys and using the minibuffer](chapter4.md#476-reading-keys-and-using-the-minibuffer)

[4.7.7: Examples](chapter4.md#477-examples)

[Chapter 5: Substitutions](chapter5.md)
---------------------------------------------------------------------------------

### [5.1: Quoting](chapter5.md#51-quoting)

[5.1.1: Backslashes](chapter5.md#511-backslashes)

[5.1.2: Single quotes](chapter5.md#512-single-quotes)

[5.1.3: POSIX quotes](chapter5.md#513-posix-quotes)

[5.1.4: Double quotes](chapter5.md#514-double-quotes)

[5.1.5: Backquotes](chapter5.md#515-backquotes)

### [5.2: Modifiers and what they modify](chapter5.md#52-modifiers-and-what-they-modify)

### [5.3: Process Substitution](chapter5.md#53-process-substitution)

### [5.4: Parameter substitution](chapter5.md#54-parameter-substitution)

[5.4.1: Using arrays](chapter5.md#541-using-arrays)

[5.4.2: Using associative arrays](chapter5.md#542-using-associative-arrays)

[5.4.3: Substituted substitutions, top- and tailing, etc.](chapter5.md#543-substituted-substitutions,-top--and-tailing,-etc)

[5.4.4: Flags for options: splitting and joining](chapter5.md#544-flags-for-options-splitting-and-joining)

[5.4.5: Flags for options: `GLOB_SUBST` and `RC_EXPAND_PARAM`](chapter5.md#545-flags-for-options-glob_subst-and-rc_expand_param)

[5.4.6: Yet more parameter flags](chapter5.md#546-yet-more-parameter-flags)

[5.4.7: A couple of parameter substitution tricks](chapter5.md#547-a-couple-of-parameter-substitution-tricks)

[5.4.8: Nested parameter substitutions](chapter5.md#548-nested-parameter-substitutions)

### [5.5: That substitution again](chapter5.md#55-that-substitution-again)

### [5.6: Arithmetic Expansion](chapter5.md#56-arithmetic-expansion)

[5.6.1: Entering and outputting bases](chapter5.md#561-entering-and-outputting-bases)

[5.6.2: Parameter typing](chapter5.md#562-parameter-typing)

### [5.7: Brace Expansion and Arrays](chapter5.md#57-brace-expansion-and-arrays)

### [5.8: Filename Expansion](chapter5.md#58-filename-expansion)

### [5.9: Filename Generation and Pattern Matching](chapter5.md#59-filename-generation-and-pattern-matching)

[5.9.1: Comparing patterns and regular expressions](chapter5.md#591-comparing-patterns-and-regular-expressions)

[5.9.2: Standard features](chapter5.md#592-standard-features)

[5.9.3: Extensions usually available](chapter5.md#593-extensions-usually-available)

[5.9.4: Extensions requiring `EXTENDED_GLOB`](chapter5.md#594-extensions-requiring-extended_glob)

[5.9.5: Recursive globbing](chapter5.md#595-recursive-globbing)

[5.9.6: Glob qualifiers](chapter5.md#596-glob-qualifiers)

[5.9.7: Globbing flags: alter the behaviour of matches](chapter5.md#597-globbing-flags-alter-the-behaviour-of-matches)

[5.9.8: The function `zmv`](chapter5.md#598-the-function-zmv)

[Chapter 6: Completion, old and new](chapter6.md)
-------------------------------------------------------------------------------------------

### [6.1: Completion and expansion](chapter6.md#61-completion-and-expansion)

### [6.2: Configuring completion using shell options](chapter6.md#62-configuring-completion-using-shell-options)

[6.2.1: Ambiguous completions](chapter6.md#621-ambiguous-completions)

[6.2.2: `ALWAYS_LAST_PROMPT`](chapter6.md#622-always_last_prompt)

[6.2.3: Menu completion and menu selection](chapter6.md#623-menu-completion-and-menu-selection)

[6.2.4: Other ways of changing completion behaviour](chapter6.md#624-other-ways-of-changing-completion-behaviour)

[6.2.5: Changing the way completions are displayed](chapter6.md#625-changing-the-way-completions-are-displayed)

### [6.3: Getting started with new completion](chapter6.md#63-getting-started-with-new-completion)

### [6.4: How the shell finds the right completions](chapter6.md#64-how-the-shell-finds-the-right-completions)

[6.4.1: Contexts](chapter6.md#641-contexts)

[6.4.2: Tags](chapter6.md#642-tags)

### [6.5: Configuring completion using styles](chapter6.md#65-configuring-completion-using-styles)

[6.5.1: Specifying completers and their options](chapter6.md#651-specifying-completers-and-their-options)

[6.5.2: Changing the format of listings: groups etc.](chapter6.md#652-changing-the-format-of-listings-groups-etc)

[6.5.3: Styles affecting particular completions](chapter6.md#653-styles-affecting-particular-completions)

### [6.6: Command widgets](chapter6.md#66-command-widgets)

[6.6.1: `_complete_help`](chapter6.md#661-_complete_help)

[6.6.2: `_correct_word`, `_correct_filename`, `_expand_word`](chapter6.md#662-_correct_word,-_correct_filename,-_expand_word)

[6.6.3: `_history_complete_word`](chapter6.md#663-_history_complete_word)

[6.6.4: `_most_recent_file`](chapter6.md#664-_most_recent_file)

[6.6.5: `_next_tags`](chapter6.md#665-_next_tags)

[6.6.6: `_bash_completions`](chapter6.md#666-_bash_completions)

[6.6.7: `_read_comp`](chapter6.md#667-_read_comp)

[6.6.8: `_generic`](chapter6.md#668-_generic)

[6.6.9: `predict-on`, `incremental-complete-word`](chapter6.md#669-predict-on,-incremental-complete-word)

### [6.7: Matching control and controlling where things are inserted](chapter6.md#67-matching-control-and-controlling-where-things-are-inserted)

[6.7.1: Case-insensitive matching](chapter6.md#671-case-insensitive-matching)

[6.7.2: Matching option names](chapter6.md#672-matching-option-names)

[6.7.3: Partial word completion](chapter6.md#673-partial-word-completion)

[6.7.4: Substring completion](chapter6.md#674-substring-completion)

[6.7.5: Partial words with capitals](chapter6.md#675-partial-words-with-capitals)

[6.7.6: Final notes](chapter6.md#676-final-notes)

### [6.8: Tutorial](chapter6.md#68-tutorial)

[6.8.1: The dispatcher](chapter6.md#681-the-dispatcher)

[6.8.2: Subcommand completion: `_arguments`](chapter6.md#682-subcommand-completion-_arguments)

[6.8.3: Completing particular argument types](chapter6.md#683-completing-particular-argument-types)

[6.8.4: The rest](chapter6.md#684-the-rest)

### [6.9: Writing new completion functions and widgets](chapter6.md#69-writing-new-completion-functions-and-widgets)

[6.9.1: Loading completion functions: `compdef`](chapter6.md#691-loading-completion-functions-compdef)

[6.9.2: Adding a set of completions: `compadd`](chapter6.md#692-adding-a-set-of-completions-compadd)

[6.9.3: Functions for generating filenames, etc.](chapter6.md#693-functions-for-generating-filenames,-etc)

[6.9.4: The `zsh/parameter` module](chapter6.md#694-the-zsh/parameter-module)

[6.9.5: Special completion parameters and `compset`](chapter6.md#695-special-completion-parameters-and-compset)

[6.9.6: Fancier completion: using the tags and styles mechanism](chapter6.md#696-fancier-completion-using-the-tags-and-styles-mechanism)

[6.9.7: Getting the work done for you: handling arguments etc.](chapter6.md#697-getting-the-work-done-for-you-handling-arguments-etc)

[6.9.8: More completion utility functions](chapter6.md#698-more-completion-utility-functions)

### [6.10: Finally](chapter6.md#610-finally)

[Chapter 7: Modules and other bits and pieces *Not written*](chapter7.md)
-------------------------------------------------------------------------------------------------------------------

### [7.1: Control over modules: `zmodload`](chapter7.md#71-control-over-modules-zmodload)

[7.1.1: Modules defining parameters](chapter7.md#711-modules-defining-parameters)

[7.1.2: Low-level system interaction](chapter7.md#712-low-level-system-interaction)

[7.1.3: ZFTP](chapter7.md#713-zftp)

### [7.2: Contributed bits](chapter7.md#72-contributed-bits)

[7.2.1: Prompt themes](chapter7.md#721-prompt-themes)

### [7.3: What's new in 4.1](chapter7.md#73-what's-new-in-41)

