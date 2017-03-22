# Lab 1 - Intro to Forth and the STM32

## Michael Brock - z5056704

### Question 1

For the following words describe their finction:

Word   | Example       | Word Function
------ | ------------- | -------------
SWAP   | 3 4 SWAP      | Swaps TOS and NOS
DUP    | 3 4 DUP       | Duplicates TOS to a new value which is put onto the stack
OVER   | 3 4 OVER      | Duplicates NOS to the top of the stack
ROT    | 3 4 5 ROT     | Move TOS to NOS, NOS to s3, and s3(third item on stack) to TOS
-ROT   | 3 4 5 –ROT    | Same as ROT, but opposite direction
2DUP   | 1 2 2DUP      | Duplicates TOS and NOS to the top of the stack.
+UNDER | 2 3 9 +UNDER  | Increments NOS by 1
DROP   | 1 2 3 DROP    | Remove TOS
<      | 3 2 <         | Test if n1 < n2. Puts result on the stack (will remove TOS and NOS to do this)
ABS    | 8 -12 ABS     | Converts TOS to positive value of TOS
MIN    | 2 3 4 MIN     | Removes the larger of TOS and NOS
/      | 13 2 /        | Integer Division.
MOD    | 14 3 MOD      | Integer Division Remainder / Modulus
WITHIN | 3 2 12 WITHIN | ( n1 n2 n3 -- n4 ) checks if n1 is in range n2 - n3. returns true or false. Consumes n1 n2 and n3 from stack.
0<>    | 12 0<>        | == 0 , returns true if TOS not 0 (consumes TOS)
NOT    | 8 NOT         | Bitwise Inversion (At a guess, not in my version of FORTH)
OR     | 8 4 OR        | Bitwise OR (For each bit, if input is 1, output is 1, else 0)
XOR    | 8 4 XOR       | Bitwise XOR, (For each bit, if the inputs are different then output is 1 else 0.)
0>     | 3 0> -3 0>    | Greater than 0. True if TOS (consumed) is greater than 0
HEX    | 32 HEX .      | Converts working radix to base 16
DECIMAL| 32 HEX DUP . DECIMAL . | Converts working radix to base 10


### Question 2: What does */ do? Why did we not do the multiplication and then the division?  Why would it be even worse to do the division first?

*/ requires the stack to be at least 3 items large, the second and third stack items are multiplied and then divided by the top stack item.
It places on the stack the quotient. The multiplication is done first in order to avoid a loss of precision associated with the integer division.
If you were to divided by two then multiply by 100, the multiplication is also multiplying the inherent error in integer division.
The reason this is done in one command is because it is more than simply a multiply action and the divide.
The multiplication is done using double numbers in order to allow for larger numbers to be created in the process which then are divided down again (larger means larger than single numbers would allow)

### Question 3: Now see if you can write a word F2C doing the inverse calculation for a temperature in Fahrenheit.  Write similar words for conversion to Kelvin from Fahrenheit or Celcius.

### Temperature

#### Fixed Point Arithmetic

* Celsius to Farenheit
```
: C2F ( n1 -- n2 ) 18 10 */ 32 + ;
```

* Celsius to Kelvin
```
: C2K ( n1 -- n2 ) 273 + ;
```

* Fahrenheit to Celsius
```
: F2C ( n1 -- n2 ) 32 - 5 9 */ ;
```

* Fahrenheit to Kelvin
```
: F2K ( n1 -- n2 ) 32 - 5 9 */ 273 + ;
```

* Kelvin to Celsius
```
: K2C ( n1 -- n2 ) 273 - ;
```

* Kelvin to Fahrenheit
```
: K2F ( n1 -- n2 ) 273 - 9 5 */ 32 + ;
```

##### Explanation

All of the above words use nothing that needs further explaining, and the formulae themselves are pretty straightforward.

### Question 4: Write a word that can calculate the value of y for the quadratic equation y = 3x2 - 3x + 8, given a value of x on the stack.

* Question 4 word
````
: q4 ( x -- 3x^2 - 3x + 8 ) dup 1 - 3 * * 8 + ;
````

##### Explanation

The assumptions this word makes is that there is a number on the stack which will be consumed as 'x' in the above formula.
Firstly I simplified the formula to `y = 3 * x * (x - 1) + 8` to utilise less operations in the reverse polish notation used by forth.

I duplicate the TOS value as we will need it twice. I put one on the stack so that I can then take that away from the TOS (x) leaving TOS as `x - 1`.
NExt 3 is put on the stack so that it can be multiplied by TOS, leaving TOS as `3(x-1)`. Then TOS and NOS are multiplied to give TOS as `3x(x-1)`.
All that remains is to put 8 on the stack and add TOS and NOS to leave TOS as `3x(x-1)+8`, our final required answer.

### Question 5: Fancy printing

I realise this is a clunky solution, but it works.... .BIN would be neater in a loop, however this is technically more efficient.
Also this works for single length numbers and not double length. If you want to use double length then remove the 0 succeeding the stack comment.

```
: .HEX ( n1 -- ) 0 BASE @ -ROT 16 BASE  ! <# # # # # 32 HOLD 58 HOLD 32 HOLD # # # # 32 HOLD 36 HOLD #> TYPE SPACE BASE ! ;
```

```
: .BIN ( n1 -- ) 0 BASE @ -ROT 2 BASE  ! <# # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD 37 HOLD #> TYPE SPACE BASE ! ;
```

```
: .ALL ( n1 -- ) 0 2DUP CR .HEX CR .BIN CR;
```

##### Explanation

Each word assumes you have a single length number on the stack which you want to display.
We start each definition by putting 0 on the stack, this effectively means you now have a double length number.
Using `BASE @` I place the current base/radix on the stack and immediatly push it beneath my double length number(`-ROT`), it will be useful later.
Then I store the relevant base/radix in base using `x BASE !` to change how the numbers will be displayed (ones and zeroes for binary, etc.)

`<#` begins the number formatting, which is ended by `#>` Each time you see a # it will display a digit (starting at LSB) and moving up, if there is no digits remaining in the number it wil display 0.
Where you see `32 HOLD` or `36 HOLD` I am inserting an ASCII character into the output where the number corresponds to the ASCII code for the char.
Once done the `TYPE` will output this as a string. The last `SPACE` command will just move the 'ok' response from forth away from the output.
The final command is `BASE !` which will look at the stack, see the stored base value from earlier and restore the base/radix to what it was prior to the word being called.

### Question 6: Explain how each of the four words (KEY, KEY?, ACCEPT, WORD, TYPE), and give an example piece of code that produces the following exchange:
#### Please tell me your name:  Trevor
#### Hi, Trevor, pleased to meet you.


* KEY

This word will block until a character is entered into the current input device/is recieved from a preexisting entry.

* KEY?

This world returns a boolean stating whether there is a character available from the current input device.

* ACCEPT

Takes two arguments, NOS is addr and TOS is len. ACCEPT will read in a string of maximum size len and store it at addr.
Will return the actual size of the string read in. It stops when it encounters a carriage return or the max limit. Whichever comes first.

* WORD

Reads one word from the input device, using the character (usually blank) as a delimiter. moves the string to the address with the count in the first byte (counted string), leaving the address on the stack.

* TYPE

Takes two inputs, NOS being the address of the string to start printing and TOS being the length. Will then print the string referenced by the address ans length.
In the case of formatting numbers ( <# #> ) These stack items are provided already.

#### Example Code

: HELLOFORTH CR ." Please tell me your name: " TIB 80 ACCEPT CR ." Hi, " TIB SWAP TYPE ." , pleased to meet you!" CR ;

### Question 7:
Make an array Letters that contains binary encoded data for 6x6-element letters that can be made with stars like in the previous example.  The code for E would contain the data in binary as 111110 100000 111000 100000 100000 111110.  Just do examples from A-E.  Then write another word that takes a string array like “ADD” and outputs each of the letters like

```
  **
 *  *
*    *
******
*    *
*    *

****
*   *
*    *
*    *
*   *
****

****
*   *
*    *
*    *
*   *
****
```

This is my solution to the problem, I would be very happy to see better solutions.

```
: ** ( n N -- n^N ) 1 swap 0 ?do over * loop nip ; \ Convenient word for exponentials
Variable L \ Need to define this for the line word which will be defined next.
: line ( b1 -- ) L !  6 0 DO L @  2 I ** AND 0> IF 42 EMIT ELSE SPACE THEN LOOP ; \ Will print a row of stars representing a binary string
```


### Question 8:

Write a word Quadratic that takes 4 items from the stack and uses them as x value coefficients in a polynomial expression.  Your word should evaluate for y given a known x
eg  2 3 -4 5 Quadratic evaluates 2x^2+3x-4 at x=5.

First off I defined a word to square conveniently for me.
```
: ^2 ( n -- n^2 ) dup * ;
```
Next I defined the following word to solve the problem which solves the quadratic ` a * x^2 + b * x + c

```
: q8 ( a b c x -- y) >R -ROT SWAP R@ ^2 * SWAP  R> * + + ;
```

#### Explanation:

Firstly I store x on the return stack. Two reasons for this, I need to use x more than once, and I want to reorder the stack without x on it.
Then I change the order of the stack from `a b c` to `c b a`. This is so as I remove the coefficients from the stack, they are in the correct mathematical order. This is achieved by `-ROT SWAP`.
Next I duplicate x off the return stack. ^2 is a function I defined earlier to square the top stack item. This is then multiplied by a.
x is once again retreived and then multiplied by b and then the result added to the previous result before c is finally added. The remaining number on the stack is the result of the quadratic.
