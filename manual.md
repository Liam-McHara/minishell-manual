## The minish loop
![minish loop](assets/minish_loop.png)
![minish diagram](assets/diagram.png)

## 1. Syntax
When _minish_ reads its input, it divides the input into words and operators, employing the quoting rules to select which meanings to assign various words and characters.

Minish then parses these tokens into commands and other constructs, removes the special meaning of certain words or characters, expands others, redirects input and output as needed, executes the specified command, waits for the command’s exit status, and makes that exit status available for further inspection or processing.

### 1.1 Operation
Basically, _minish_ does the following:

1. Reads its input from the user’s terminal.
2. Breaks the input into _words_[^word] and operators, obeying the quoting rules described in [Quoting](#12-quoting). These _tokens_[^token] are separated by _metacharacters_[^meta].
3. Parses the tokens into [commands](#2-commands).
4. Performs the various [expansions](#4-expansion-and-quote-removal), breaking the expanded tokens.
5. Performs any necessary [redirections](#5-redirections) and removes the _redirection operators_[^redir] and their operands from the argument list.
6. [Executes](#6-executing-commands) the command.
7. Optionally waits for the command to complete and collects its [exit status](#65-exit-status).

### 1.2 Quoting
Quoting is used to remove the special meaning of certain characters or words to _minish_. Quoting can be used to disable special treatment for special characters and to prevent variable expansion.

Each of the shell _metacharacters_ has special meaning to _minish_ and must be quoted if it is to represent itself.

## 2. Commands
A command is just a sequence of _word_s[^word] separated by _metacharacters_[^meta], terminated by a `pipe` or a newline. The first word generally specifies a command to be executed, with the rest of the `word`s being that command’s arguments.
The command to be executed can be an executable filename or a builtin name.
The return status of a command is its [exit status](#65-exit-status) as provided by the POSIX 1003.1 waitpid function, or 128+n if the command was terminated by signal n.

### 2.1 Pipelines
A pipeline is a sequence of one or more commands separated by a `pipe` (‘|’).

The format for a pipeline is:
```
command1 [ | command2 ]*
```

The standard output of _command1_ is connected via a pipe to the standard input of _command2_. This connection is performed <ins>before any redirections</ins> specified by the command.


## 3. Variables
A _variable_ is an entity that stores values. It can be a [name](#definitions) or the special character '?'. A variable is set if it has been assigned a value. The null string is a valid value. Once a variable is set, it may be unset only by using the unset [builtin command](#builtins).

In _minish_, the only way a variable may be assigned to is by using the builtin export, which exports it to the current environment:
```
export name=[value]
```
If no value is given, the null string is assigned.

All values undergo [variable expansion](#41-variable-expansion) and [quote removal](#43-quote-removal).

## 4. Expansion and Quote Removal
After the command line has been split into [token](#definition)s, the following steps are performed from left to right, in the following order.
1. variable expansion
2. word splitting
3. quote removal

### 4.1 Variable Expansion
The '$' character introduces variable expansion.

The environment variable name to be expanded (or '?') may be double-quoted including the leading '$' character to protect the variable to be expanded from characters immediately following it which could be interpreted as part of the name.

The basic form of variable expansion is _$variable_. The value of _variable_ is substituted by its value.

### 4.2 Word Splitting
_Minish_ scans the results of variable expansion that did not occur within double quotes for word splitting.

_Minish_ treats each character of `<space>` or `<tab>` as a delimiter, and splits the results of the variable expansion into words using these characters as field terminators. Sequences of `<space>` and `<tab>` at the beginning and end of the results of the variable expansion are ignored.

Explicit null arguments ("" or '') are retained and passed to commands as empty strings. **Unquoted implicit null arguments, resulting from the expansion of variables that have no values, are removed.** If a variable with no value is expanded within double quotes, a null argument results and is retained and passed to a command as an empty string. When a quoted null argument appears as part of a word whose expansion is non-null, the null argument is removed. That is, the word -d'' becomes -d after word splitting and null argument removal.

Note that <ins>if no expansion occurs, no splitting is performed</ins>.

### 4.3 Quote Removal
All unquoted occurrences of the characters ' and " that did not result from one of the above expansions are removed.

## 5. Redirections
Before a command is executed, its input and output may be _redirected_ using a special notation interpreted by _minish_. Redirection allows commands’ file descriptors to refer to different files, and can change the files the command reads from and writes to. Redirection may also be used to modify file descriptors in the current shell execution environment. The following redirection operators may appear anywhere within a command. Redirections are processed in the order they appear, from left to right.

### 5.1 Redirecting Input
The format for redirecting input is:
```
<file
```
Redirection of input causes the file _file_ to be opened for reading on the standard input.


### 5.2 Redirecting Output
The format for redirecting output is:
```
>file
```
Redirection of output causes the file _file_ to be opened for writing on the standard output. If the file does not exist it is created; if it does exist it is truncated to zero size.

### 5.3 Appending Redirected Output
The format for appending output is:
```
>>file
```
Redirection of output in this fashion causes the file _file_ to be opened for appending on the standard output. If the file does not exist it is created.

### 5.4 Here Documents
The format of here documents is:
```
<<word
	here_document
delimiter
```
This type of redirection instructs _minish_ to read input from the current source until a line containing only _delimeter_ (with no trailing spaces or tabs) is seen. All of the lines read up to that point are then used as the standard input for a command.

> [!WARNING]  
> <ins>No variable expansion is performed on word.</ins> If any part of _word_ is quoted, the _delimiter_ is the result of quote removal on _word_, and the lines in the _here_document_ are not expanded. If _word_ is unquoted, all lines of the _here_document_ are subjected to variable expansion.

## 6. Executing Commands
### 6.1 Command Expansion
### 6.2 Command Search and Execution
### 6.3 Command Execution Environment
### 6.4 Environment
### 6.5 Exit Status
### 6.6 Signals

## The minish Loop

## History

## Builtins

[^token]: **token**: A sequence of characters treated as a unit by the shell. Both operators and words are tokens.

[^pipe]: **pipe**: A token that separates one command from the next, connecting the output of the former to the output of the latter. It is ‘|’.

[^redir]: **redirection operator**: A token that performs a redirection function. It is ‘<’, ‘>’, ‘<<’ or ‘>>’.

[^word]: **word**: A sequence of characters treated as a unit by the shell (excluding operators). Words may not include unquoted metacharacters.

[^meta]: **metacharacter**: A character that, when unquoted, separates words. A metacharacter is a space, tab or one of the following characters: ‘|’, ‘<’, or ‘>’.

[^name]: **name**: A word consisting only of alphanumeric characters and underscores, and beginning with an alphabetic character or an underscore. Names are used as shell variable names.