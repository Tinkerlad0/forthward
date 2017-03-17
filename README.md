# forthward
Forth Hints


# Math

 * `.s` - Show the stack
 * `+`, `-`, `*`, `mod` - Math operators
 * `/mod` - performs both / and mod

# Stack manipulation
 * `drop` and `2drop` - drop a stack item (once / twice)
 * `dup` - duplicate a stack item
 * `rot` - rotate the stack
 * `nip` - Removes the 2nd to last item on the stack
 * `tuck` - duplicates the 2nd to last item on the stack to the front of the stack

# Working with Files
 * `s" file.fs" included` - Loads file.fs into memory
 * `gforth code.fs tests.fs -e bye` - Load a file and exit if something goes wrong (instead of the command line)

# Comments
 * `(` and `\` - Comments. Comments are significant in Forth, so keep a space between them.

# Compiling words
By convention the comment after the name of a definition describes the stack effect: The part in front of the '--' describes the state of the stack before the execution of       
   the definition, i.e., the parameters that are passed into the colon definition; the part behind the '--' is the state of the stack after the execution of the definition,         
   i.e., the results of the definition. The stack comment only shows the top stack items that the definition accesses and/or changes.                                                
                                                                                                                                                                                     
  You should put a correct stack effect on every definition, even if it is just ( -- ). You should also add some descriptive comment to more complicated words (I usually do        
   this in the lines following :). If you don't do this, your code becomes unreadable (because you have to work through every definition before you can understand any).

## Conventions for stack effect comments 

 * `n` - signed integer
 * `u` - unsigned integer
 * `c` - character
 * `f` - Boolean flags, i.e. false or true.
 * `a-addr`,`a-` Cell-aligned address
 * `c-addr`,`c-` - Char-aligned address (note that a Char may have two bytes in Windows NT)
 * `xt` - Execution token, same size as Cell 
 * `w,x` - Cell, can contain an integer or an address. It usually takes 32, 64 or 16 bits (depending on your platform and Forth system). A cell is more commonly known as a machine word, but the term word already means something different in Forth.
 * `d` - signed double-cell integer 
 * `ud` - unsigned double-cell integer 
 * `r` - Float (on the FP stack)
 
# (De)Compiling words
 ```forth
 : squared ( n -- n^2 ) \ The parenthesis are just a convention to explain the function.
   dup * ;
 ```
 * `see` - 'decompiles' a word to see source.

# "Types"
Forth doesn't do any type checking whatsoever. It's not a big deal though, but you should be aware of it.
 * `(none)` - signed integer 
 * `u` - unsigned integer 
 * `c` - character
 * `d` - signed double-cell integer 
 * `ud`, `du` - unsigned double-cell integer
 * `2` - two cells (not-necessarily double-cell numbers)
 * `m`, `um` - mixed single-cell and double-cell operations 
 * `f` - floating-point (note that in stack comments 'f' represents flags, and 'r' represents FP numbers).
 
# Locals (Local variable definitions)
 Curly braces in a word definition allow you to create local variables in the definition.

```forth
: swap { a b -- b a }
b a ;
```

# Conditionals

Conditionals can only be used *within a colon definition*.

```forth
 : abs ( n1 -- +n2 ) 
   dup 0 < if
   negate
 endif ;
```

**NOTE:** In Forth, `endif` is less commonly used. Typically, the word `then` is used. But that's really confusing for non-forth programmers.

Another example:

```forth
: min ( n1 n2 -- n )
 2dup < if 
  drop
 else
  nip
endif;
```

# Boolean expressions
 * `true` is often represented by `-1` (a 11111111 byte)
 * `false` is 0
 * In many contexts, non zero values are treated as `true`

# Loops
 * `begin` does nothing at run-time, `again` jumps back to `begin`.

```forth
 : endless ( -- )
   0 begin
     dup . 1+
   again ;
 endless
```
use `leave` to get out of a loop and `exit` to leave the definition.

# Recursion

Use `recurse` to call the word being defined.

# The Return Stack

 Aside from the main stack ('the stack'), there is also a 'return stack'. Typically, the return stack is used to store temporary variables. **Miscounting items on the return stack usually causes a crash**

 * `>r` Pushes data stack element onto return stack (data -> return)
 * `r>` Move from return stack to data stack (return -> data)
 * `r@` pushes a copy of the top of the return stack on the data stack

# Memory and Global Variables
 * `variable v` initialize variable 'v'
 * `v` Push the **address** onto the stack (not the value)
 * `5 v !` Set the **value** of 'v' to 5.
 * `v @` Push the **value** of 'v' to the stack
 * `v 2 cells` "Grab the contents of 2 cells of memory, starting at the location of variable v"
 * `create v2 20 cells allot` Allocate 20 cells of memory and store the starting address to variable v2

# Working with Floating Point Numbers

Write floats with scientific notation.

`1.23e` -> 1.23
`1e0` -> 1.0

Most Forth implementations create a seperate stack for FP operations. This includes Gforth. Usually, you can just prefix data stack operators with an 'f' to operate in the float stack. Eg: `f.` to pop off the fp stack. Other operators include `f+` `f-` `f*` `f/` 
`f**` `f@` and `f!`.

# Common Used Word Definitions


 * +UNDER ( n1 n2 -- n1 n2 )- Increments the NOS by 1
 ```
 : +UNDER swap 1+ swap ;
 ```

 * CONSTANT ( n1 -- ; n1 ) - defines a constant variable, which when used after definition will put the constant on the top of the stack
 ```
 : CONSTANT ( n1 -- ; n1 ) CREATE , DOES> @ ;
 ```

 * ARRAY - Creates an array of length specified by TOS
 ```
 : ARRAY            \size --; [child] n -- addr ; cell array
 CREATE             \Create the data word
 CELL *             \Calculate Size
 HERE OVER ERASE    \clear memory
 ALLOT              \Allocate Memory
 DOES>              \run time gives address of data
 SWAP CELL * +      \index in array
 ;
 ```

    USAGE
    ```
    10 ARRAY Var3
    3 5 Var3 !  \Store a 3 in cell 5 of Var3
    5 Var3 @    \Fetch the 5th value in Var3
    ```