# Chapter 1. Improve Command-line Productivity

Goal: run commands more efficiently with Bash features, shell scripts and RHEL utilities.

Sections:

1. Write Simple Bash Scripts
2. Loops and Conditional Constructs in Scripts
3. Match Text in Command Output with Regular Expressions

## 1. Write Simple Bash Scripts

- A Bash script = an executable file with a list of commands (and maybe logic like `if` and loops).
- Use `vim` or `emacs`: they highlight syntax and help find missing quotes and brackets.
- First line (shebang) sets the interpreter:

```bash
#!/usr/bin/bash
```

### Run a script

- The file needs execute permission: `chmod +x script`.
- If the script is in a `$PATH` directory (for example `~/bin`), run it by name.
- If not, use `./script` or the absolute path.
- `PATH` runs the first match, so do not name a script like an existing command.

```bash
which hello        # ~/bin/hello
echo $PATH
./hello            # script in the current directory
```

### Quoting

| Quote   | Meaning                                                                  |
| ------- | ------------------------------------------------------------------------ |
| `\`     | removes the special meaning of the next character                        |
| `'...'` | everything is literal (no globbing, no variable or command substitution) |
| `"..."` | no globbing, but `$var`, `${var}` and `$(cmd)` still work                |

```bash
echo # not a comment            # empty line (# starts a comment)
echo \# not a comment           # # not a comment
echo '# not a comment #'        # # not a comment #
var=$(hostname -s); echo $var   # host
echo "hostname is ${var}"       # hostname is host
echo 'hostname is ${var}'       # hostname is ${var}
echo "\"Hello, world\""         # "Hello, world"
```

### Output

- `echo` prints to STDOUT by default.
- Send error messages to STDERR with `>&2`.
- Use `echo` to debug: print variable values to see what the script does.

```bash
#!/usr/bin/bash

echo "Hello, world"
echo "ERROR: Houston, we have a problem." >&2
```

```bash
hello 2> hello.log     # only STDERR goes to the file
cat hello.log
```

References: `bash(1)`, `echo(1)`, `echo(1p)`.

### Dev use

```bash
./deploy.sh 2> errors.log   # only STDERR goes to the file
```

- Common bug in deploy scripts and CI: a variable is not expanded because of single quotes.

## 2. Loops and Conditional Constructs in Scripts

Goal: run repetitive tasks with `for` loops, check exit codes, run tests with operators, build `if` structures.

### for loop

```bash
for VARIABLE in LIST; do
COMMAND VARIABLE
done
```

- The loop takes each string from `LIST`, stores it in `VARIABLE`, runs the commands.
- The variable name is arbitrary.
- The list can be typed by hand or come from expansion (variable, brace, file name) or command substitution.

```bash
for HOST in host1 host2 host3; do echo $HOST; done
for HOST in host{1,2,3}; do echo $HOST; done
for HOST in host{1..3}; do echo $HOST; done
for FILE in file{a..c}; do ls $FILE; done
for EVEN in $(seq 2 2 10); do echo "$EVEN"; done   # 2 4 6 8 10
```

```bash
for PACKAGE in $(rpm -qa | grep kernel); \
do echo "$PACKAGE was installed on \
$(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE))"; done
```

### Exit codes

- `exit` leaves the script at once, the rest is skipped.
- `exit N`: N from 0 to 255. `0` = success, nonzero = error. The script author defines the codes (use unique values for different errors).
- `$?` = exit code of the last completed command.
- `exit` without an argument returns the exit code of the last command in the script.

```bash
#!/usr/bin/bash
echo "Hello, world"
exit 0
```

```bash
./hello
echo $?     # 0
```

### test and operators

- Check input (arguments, user input, substitutions, expansions) so the script does not break on unexpected values.
- Syntax: `test EXPR`, `[ EXPR ]`, `[[ EXPR ]]`. Use `[[ ]]` in most cases (it supports globbing and regex matching).
- Result is in `$?`: `0` = true, `1` = false.
- Spaces inside the brackets are mandatory (`[` is a built-in alias for `test`).

| Type    | Operator         | Meaning                        |
| ------- | ---------------- | ------------------------------ |
| Numbers | `-eq` `-ne`      | equal, not equal               |
|         | `-gt` `-ge`      | greater, greater or equal      |
|         | `-lt` `-le`      | less, less or equal            |
| Strings | `=` / `==`, `!=` | equal, not equal               |
|         | `-z`, `-n`       | zero length, non-zero length   |
| Files   | `-f`, `-d`       | regular file, directory exists |
|         | `-L`, `-r`       | symbolic link, read permission |

```bash
test 1 -gt 0 ; echo $?       # 0
test 0 -gt 1 ; echo $?       # 1
[[ 1 -eq 1 ]]; echo $?       # 0
[[ 1 -ne 1 ]]; echo $?       # 1
[[ 8 -gt 2 ]]; echo $?       # 0
[[ 2 -ge 2 ]]; echo $?       # 0
[[ 2 -lt 2 ]]; echo $?       # 1
[[ 1 -lt 2 ]]; echo $?       # 0
[[ abc = abc ]]; echo $?     # 0
[[ abc == def ]]; echo $?    # 1
[[ abc != def ]]; echo $?    # 0
STRING=''; [[ -z "$STRING" ]]; echo $?      # 0
STRING='abc'; [[ -n "$STRING" ]]; echo $?   # 0
```

More operators: `man test`, `man bash`.

### if/then

```bash
if <CONDITION>; then
      <STATEMENT>
fi
```

`fi` closes the construct. Start `psacct` if it is not active:

```bash
systemctl is-active psacct > /dev/null 2>&1
if [[ $? -ne 0 ]]; then sudo systemctl start psacct; fi
```

### if/then/else

```bash
if <CONDITION>; then
      <STATEMENT>
else
      <STATEMENT>
fi
```

Start `psacct` if it is not active, stop it if it is active:

```bash
systemctl is-active psacct > /dev/null 2>&1
if [[ $? -ne 0 ]]; then \
sudo systemctl start psacct; \
else \
sudo systemctl stop psacct; \
fi
```

### if/then/elif/then/else

```bash
if <CONDITION>; then
      <STATEMENT>
elif <CONDITION>; then
      <STATEMENT>
else
      <STATEMENT>
fi
```

- Bash checks conditions in order, runs the first true one, skips the rest.
- If no condition is true, `else` runs.

Run `mysql` if `mariadb` is active, `psql` if `postgresql` is active, otherwise `sqlite3`:

```bash
systemctl is-active mariadb > /dev/null 2>&1
MARIADB_ACTIVE=$?
sudo systemctl is-active postgresql > /dev/null 2>&1
POSTGRESQL_ACTIVE=$?
if [[ "$MARIADB_ACTIVE" -eq 0 ]]; then \
mysql; \
elif [[ "$POSTGRESQL_ACTIVE" -eq 0 ]]; then \
psql; \
else \
sqlite3; \
fi
```

## References: `bash(1)`, `test(1)`, `getopt(3)`.

### Dev use

```bash
[[ -f .env ]] || { echo "no .env" >&2; exit 1; }
```

- A nonzero exit code fails the step in CI.
- Common bug with `[[ ]]`: missing spaces inside the brackets.
  $? и exit те же, что код выхода процесса в CI: ненулевой код роняет шаг пайплайна. Самый частый баг с [[]]: забытые пробелы внутри скобок, и тогда скрипт падает с непонятной ошибкой. Это мой комментарий, в курсе этого нет.

## 3. Match Text in Command Output with Regular Expressions

Goal: write regular expressions, apply them with `grep` to files and piped output.

### Regular expressions

- A regex is a pattern to find text. Used by `vim`, `grep`, `less`, `sed`, `awk`, Perl, Python (syntax differs a bit).
- Simplest regex = exact match. `cat` matches `cat`, `concatenate`, `category`, `educated`, `vindication`.

### Line anchors

| Regex   | Matches                      |
| ------- | ---------------------------- |
| `^cat`  | `cat` at the start of a line |
| `cat$`  | `cat` at the end of a line   |
| `^cat$` | a line with only `cat`       |

### Basic vs extended

- Characters `|`, `+`, `?`, `(`, `)`, `{`, `}`:
  - basic: special only with `\` before them
  - extended: special without `\`
- Basic: `grep`, `sed`, `vim`.
- Extended: `grep -E`, `sed -E`, `less`.

### Wildcards and multipliers

- `.` = any single character (`c.t` matches `cat`, `cut`).
- `[aou]` = one character from the set (`c[aou]t`).
- `*` = zero or more of the previous item (`c.*t` matches `ct`, `cat`, `culvert`).
- `\{n\}` = exactly n times (`'c.\{2\}t'` matches `coat`, `cart`).
- Globbing (file names in the shell) is not the same as regex, even if some characters look the same.

### Syntax table

| Basic     | Extended | Meaning                   |
| --------- | -------- | ------------------------- |
| `.`       | `.`      | any single character      |
| `?`       | `?`      | previous item 0 or 1 time |
| `*`       | `*`      | 0 or more times           |
| `+`       | `+`      | 1 or more times           |
| `\{n\}`   | `{n}`    | exactly n times           |
| `\{n,\}`  | `{n,}`   | n or more times           |
| `\{,m\}`  | `{,m}`   | at most m times           |
| `\{n,m\}` | `{n,m}`  | from n to m times         |

Character classes: `[:alnum:]`, `[:alpha:]`, `[:blank:]`, `[:cntrl:]`, `[:digit:]`, `[:graph:]`, `[:lower:]`, `[:print:]`, `[:punct:]`, `[:space:]`, `[:upper:]`, `[:xdigit:]`.

Word boundaries and shortcuts: `\b` word edge, `\B` not a word edge, `\<` start of word, `\>` end of word, `\w` word character, `\W` non-word, `\s` whitespace, `\S` non-whitespace.

### grep

- `grep 'REGEX' FILE`. With many files it prints `filename:line`.
- Always put the regex in single quotes, so the shell does not touch `$`, `*`, `{}`.
- Works with pipes.

```bash
grep '^computer' /usr/share/dict/words
ps aux | grep chrony
```

| Option | Function                                  |
| ------ | ----------------------------------------- |
| `-i`   | case-insensitive                          |
| `-v`   | only lines that do not match              |
| `-r`   | recursive search in files and directories |
| `-A N` | N lines after the match                   |
| `-B N` | N lines before the match                  |
| `-e`   | several regexes (logical OR)              |
| `-E`   | extended regex syntax                     |

```bash
grep -i serverroot /etc/httpd/conf/httpd.conf
grep -v -i server /etc/hosts
grep -v '^[#;]' /etc/systemd/system/multi-user.target.wants/rsyslog.service   # hide comments
cat /var/log/secure | grep -e 'pam_unix' -e 'user root' -e 'Accepted publickey' | less
```

### Search inside vim and less

- Type `/pattern`, press Enter. Press `n` for the next match.

```bash
vim /var/log/boot.log     # /Daemons
less /var/log/messages    # /device
```

References: `regex(7)`, `grep(1)`.

### Dev use

```bash
grep -rn 'TODO' src/   # search the code, show line numbers
```

- `-n` is not in the course, `-r` is.
- Regex in JS and Python differs from POSIX basic (JS is closer to extended).
