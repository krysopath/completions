# basics
```
$ compgen -W 'aaa bbb ccc' b
bbb

$ compgen -W 'aaa bba bbc bbc ccc' b
bba
bbc
bbc

$ compgen -W 'aaa bba bbc bbc ccc' bb
bba
bbc
bbc

$ compgen -W 'aaa bba bbc bbc ccc' bbc
bbc
bbc
```
> -W means to understand the value of $1 as words

> constructing string in shell is easy

To connect this with an executable:

```
# have example target
touch magic; chmod +x magic

# have a completion magic
_magic_completions(){ $MAGICWORDS }

# tell shell to use <TAB> magic for ./magic
complete -F _magic_completions magic
```

# very simple: sequential arrays of string

```
_magic_completions()
{
  COMPREPLY+=("aaaa")
  COMPREPLY+=("bbba")
  COMPREPLY+=("bbbb")
  COMPREPLY+=("cccc")
}

complete -F _magic_completions magic
```

> COMPREPLY is the array that holds the options you currently can complete
> into. Populating it by any means, is what shell completion is about.

# magic variables

- `COMP_WORDS`: an array of all the words typed after the name of the program the compspec belongs to
- `COMP_CWORD`: an index of the COMP_WORDS array pointing to the word the current cursor is at - in other words, the index of the word the cursor was when the tab key was pressed
- `COMP_LINE`: the current command line


> We can access these arrays via bash notations. e.g.

```
# the count of all completed words
${#COMP_WORDS[@]}

# the current word
${COMP_WORDS[COMP_CWORD]}

# the previous word
${COMP_WORDS[COMP_CWORD-1]}
```

> it is magic


# A simple bash skeleton:

```
_magic_completions()
{
  COMPREPLY=($(compgen -W "yolo yeeees yet xylo" -- "${COMP_WORDS[COMP_CWORD]}"))
}

complete -F _magic_completions magic
```

source the file and try tab magic:

```
. magic-completion
./magic y<TAB><TAB>
yolo yeeees yet
```

# A useful bash golem
```
__get_corrections() {
  echo "$1" \
  | aspell -a 2>/dev/null \
  | sed 1d \
  | awk -F: '{ print $2 }' \
  | tr -d '[:blank:]'\
  | tr ',' '\n'
}

_magic_completions()
{
  local current
  current="${COMP_WORDS[COMP_CWORD]}"
  COMPREPLY=($(compgen -W "$(__get_corrections "$current" )" -- "$current" ))
  return 0
}


complete -F _magic_completions magic
```

# A proper unix golem
```
# watchmon completion                                      -*- shell-script -*-

__get_battery_devices()
{
    find /sys/class/power_supply -name 'BAT*' | xargs basename
}

_watchmon_completions()
{
    local cur prev OPTS
    _init_completion || return

    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"

    case $prev in
        '-bat')
            COMPREPLY=( $(compgen -W  "$(__get_battery_devices)" -- $cur) )
            return 0
            ;;
        '-sqlite')
            _filedir
            return 0
            ;;
    esac

    case $cur in
        *)
            OPTS="-bat
            -dbcreate
            -dump
            -sqlite"

            COMPREPLY=( $(compgen -W "${OPTS[*]}" -- $cur) )
            return  0
            ;;
    esac

    return 0
}

complete -o bashdefault -o nospace -F _watchmon_completions watchmon
```

