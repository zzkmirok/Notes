# VIM 使用

Press `:` $\to$ command Mode

Press `i` `a` `o` $\to$ insert mode

**Opening and quiting**

`:w <filename>` write as new filename

`:wq` or `:x` or `ZZ` Write file and quit

`:q!`close file without saving

**Movement**

`h` `j` `k` `l` cursor left, down, up, right

`$` Move to end of line

`gg` Move cursor to the first line

`G` Move cursor to the last line

`w` jump forward to next word

`b` jump backward to previous word

`%` jump to matching character (default pairs: `()`, `{}`, `[]`)

**Editing**

`u` Undo last change

`<ctrl-r>` Redo last change

`.` Repeat last command

`x` delete character

`dd` delete (cut) entire line

`yy` or `Y` Yank (copy) entire line

`p` paste after cursor

`/pattern` Forward search for regular expression

`?pattern` Backward search for regular expression

`n` Repeat last search

`N` Repeat last search in opposite direction

`:s/old/new` Replace `old` pattern with `new` pattern on current line

`:s/old/new/g` Replace `old` pattern in the entire file

`R` enter replace mode

`: $number` turn to `$number`th line 