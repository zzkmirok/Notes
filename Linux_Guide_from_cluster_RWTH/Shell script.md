# Shell script

`#!/bin/bash` Always the first line

`ls \` 

`-l` . `\` line break

**Variables**

- assign with `=` no space
- Always text
- Quotes necessary when white space, specail characters in value
- Retrieve with `$`
- brackets

`var='bla'`, will save the text `bla` to var

`var='$bla'`will save the text $bla to var

`var="$bla"` will look for variable named bla

`var=$(bla)` will execute command bla and save its output to var

- command line arguments:
    - `$0` - `$9`  `${10}`
    - Loops and if statements
    - `for file in $( ls ); do`
        
               `echo item: $file`
        
        `done`
        
        `if [ -e $filename ]; then`
        
        `echo "$filename exits,`
        
        `fi`