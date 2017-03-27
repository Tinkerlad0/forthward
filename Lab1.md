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
: ** ( n N -- n^N ) 1 swap 0 ?do over * loop nip ;
VARIABLE L
: LINE ( b1 -- ) L !  6 0 DO L @  2 I ** AND 0> IF 42 EMIT ELSE SPACE THEN LOOP CR ;
```

We can then define words for printing differents letters.
```
: starA ( -- ) %100001 %100001 %111111 %100001 %010010 %001100 CR %110 0 DO LINE LOOP CR ;
: starB ( -- ) %111111 %100001 %011111 %100001 %100001 %111111 CR %110 0 DO LINE LOOP CR ;
: starC ( -- ) %111100 %000010 %000001 %000001 %000010 %111100 CR %110 0 DO LINE LOOP CR ;
: starD ( -- ) %001111 %010001 %100001 %100001 %010001 %001111 CR %110 0 DO LINE LOOP CR ;
: starE ( -- ) %011111 %000001 %000001 %000111 %000001 %011111 CR %110 0 DO LINE LOOP CR ;
```
Lastly we can simply print the word ADD by using the following:
```
: starADD1 ( -- ) starA starD starD ;
```

#### Explanation

First up the ** word will take two numbers off the stack and return to the stack NOS^TOS. This is achieved by looping for TOS times, and each loop, multiplying NOS by itself. (Note when I say NOS and TOS I refer to at the start of the operation, not during)

I then decalre a Variable `L` in order to use in `LINE` to store the binary number that `LINE` is working on. The reason I use a variable here is that the return stack is in use with the loop, and it is more convenient to be off of the main stack.

`LINE` expects a number on the stack, which it will read the 6 least significant bits of in binary and output a string representation, where 1 results in a `*` char, and 0 results in a space.
Iterating over the bits is achieved by looping from 0 to 6. Each iteration `I` is updated to the value that the loop is currently cycling in. By using `2 I **` we can create avalue where the only bit set to 1 is the bit we are interested in looking at.
Then by using the logical `AND` with the given binary number (retreived from L) and checking if the result is greater than 0, we are effectively checking if the bit is set to one or zero.
If one we can print a `*`, else we can print a space and then continue on looping.

Each of the starX words simply put the correct numbers on the stack and calls the `LINE` word repeatedly as required.
The final piece in this puzzle was the `starADD` word which merely calls `starA` `starD` `starD`.

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

## Flashing LEDs

### Required Words:


* PortInit \ Initialise Port C with pin 10 as output
* PC10On   \ Turn on pin PC10
* PC10Off  \ Turn off pin PC10

* Flash    \ Flash the LED on then off with 200ms delay
* Flashes  \ Flash the LED the number of time determined by TOS

So lets define the contents of these words

```
: PortInit ( -- ) $40021018 constant RCC_APB2ENR $40011004 constant GPIOC_CRH $4001100C constant GPIOC_ODR $0010 RCC_APB2ENR ! $0000 GPIOC_ODR W! $44444344 GPIOC_CRH ! DROP DROP DROP ;
```

```
: PCPOn ( n1 -- ) GPIOC_ODR @ OR GPIOC_ODR W! ;
: PC10On ( -- ) $400 PCPOn ;
```

```
: PCPOff ( n1 -- ) INVERT GPIOC_ODR AND GPIOC_ODR W! ;
: PC10Off ( -- ) $400 PCPOff ;   
```

```
: Flash ( -- ) PC10On 200 ms PC10Off  200 ms ;
: Flashes ( n1 -- ) ABS 0 ?DO Flash LOOP ;
```

### Questions

#### Sub Questions:

##### What is the value you would need to write?
In order to simply turn the LED off without bothering about what any other pin is doing, then simply writing $0 to GPIOC_ODR will turn off the LED.
However this is not effective in the long term, as it will turn off all other pins on port C as well. My code below does not face this issue.

##### What happens if you type -5 Flashes?

In the given code this will flash for a very long time, as it would start at 0, counting up 1 for each loop cycle. It will stop when the counter overflows positive into negative numbers adn eventually counts up to -5.
This is avoided in my code as I have an ABS funcrion which will automatically make the result positive. NOted that the overflow could be intended in order to achieve longer loops.

#### Question 1 : How would you modify the code to use PB1 rather than PC10 to drive the LED?

Firstly the value assigned to RCC_APB2ENR would have to be changed to $8 as opposed to $10 as this would enable Port B instead of Port C.
Next you would want to define GPIOB_ODR at the address $4001000C and GPIOB_CRL to $4001000
Lastly the value stored in the GPIOB_CRL will have to change as it is pin specific. In order to use pin 1 the value needing to be stored in this register is $44444443

Lastly the methods for changing to PB1 would look like:
```
: PortInit ( -- ) $40021018 constant RCC_APB2ENR $40010004 constant GPIOB_CRL $4001000C constant GPIOB_ODR $0008 RCC_APB2ENR ! $0000 GPIOB_ODR W! $44444443 GPIOB_CRL ! DROP DROP DROP ;
```

```
: PBPOn ( n1 -- ) GPIOB_ODR @ OR GPIOB_ODR W! ;
: PB1On ( -- ) $1 PCPOn ;
```

```
: PBPOff ( n1 -- ) INVERT GPIOB_ODR AND GPIOB_ODR W! ;
: PB1Off ( -- ) $1 PCPOff ;   
```

#### Question 2 : What would change in your code if you were to drive the LED with the port acting as a sink rather than as a source?
To sink current effectively the pin should be configured to be an open drain. This is achieved by modifying the PortInit word.
In order to configure open drain we change the value we are storing in the GPIOB_CRl CNF01 bits to %01 meaning the value we wold write would be $44444447 

#### Question 3 : With the current code, if you had LEDs on both PC9 and PC10, and you went to switch PC10 on or off, the PC9 port would be reset to zero.  How might you change the code to ensure that the settings for one port did not influence the existing settings for others

Well I appear to have beaten the questions here..... My code will only affect individual pins when changing pin state.
The original code writes a value directly to the ODR which will replace the existing vlaue. This will modify all of the pins on the port, whereas in my code only the pin I am interested in will be modified.
 This is because I read in the original state of the