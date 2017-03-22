# WARNING - ANSWERS, DO NOT COPY

## Lab 1

### Question 1

Word   | Example       | Word Function
------ | ------------- | -------------
SWAP   | 3 4 SWAP      | Swaps TOS and NOS
DUP    | 3 4 DUP       | Duplicates TOS to a new value which is put on TOS
OVER   | 3 4 OVER      | Duplicates NOS to a new TOS
ROT    | 3 4 5 ROT     | Move TOS to NOS, NOS to s3, and s3 to TOS
-ROT   | 3 4 5 –ROT    | Same as ROT, but opposite direction
2DUP   | 1 2 2DUP      | Duplicates TOS and NOS
+UNDER | 2 3 9 +UNDER  | Increments NOS by 1
DROP   | 1 2 3 DROP    | Remove TOS
<      | 3 2 <         | Test if n1 < n2. Puts result on the stack
ABS    | 8 -12 ABS     | Converts TOS to positive value of TOS
MIN    | 2 3 4 MIN     | Removes the larger of TOS and NOS
/      | 13 2 /        | Integer Division.
mod    | 14 3 MOD      | Integer Division Remainder / Modulus
within | 3 2 12 WITHIN | ( n1 n2 n3 -- n4 ) checks if n1 is in range n2 - n3. returns true or false
0<>    | 12 0<>        | == 0 , returns true if TOS not 0 (consumes TOS)
NOT    | 8 NOT         | Bitwise Inversion (At a guess, not in my version of FORTH)
OR     | 8 4 OR        | Bitwise OR (For each bit, if input is 1, output is 1, else 0)
XOR    | 8 4 XOR       | Bitwise XOR, (For each bit, if the inputs are different then output is 1 else 0.)
0>     | 3 0> -3 0>    | Greater then 0. True if TOS (consumed) is greater than 0
HEX    | 32 HEX .      | Converts working radix to base 16
DECIMAL| 32 HEX DUP . DECIMAL . | Converts working radix to base 10


### Question 2: What does */ do? Why did we not do the multiplication and then the division?  Why would it be even worse to do the division first?

*/ requires the stack to be at least 3 items large, the second and third stack items are multiplied and then divided by the top stack item. It places on the stack the quotient. The multiplication is done first in order to avoid a loss of precision associated with the integer division. The reason this is done in one command is because it is more than simply a multiply action and the divide. The multiplication is done using double precision in order to prevent loss of precision.

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

### Question 4: Write a word that can calculate the value of y for the quadratic equation y = 3x2 - 3x + 8, given a value of x on the stack.

For modularity I defined a sub word here

* Square - Usage ( 2 ^2 . -> 4 ok )
```
: ^2 ( n -- n^2 ) dup * ;
```

Which led me to:

* Question 4 word
````
: q4 ( x -- 3x^2 - 3x + 8 ) dup ^2 - 3 * 8 + ;
````

### Question 5: Fancy printing

I realise this is a clunky solution, but it works.... .BIN would be neater in a loop, however this is tecchnically more efficient.

```
: .HEX ( d1 -- ) BASE @ -ROT 16 BASE  ! <# # # # # 32 HOLD 58 HOLD 32 HOLD # # # # 32 HOLD 36 HOLD #> TYPE SPACE BASE ! ;
```

```
: .BIN ( d1 -- ) BASE @ -ROT 2 BASE  ! <# # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD # # # # 32 HOLD 37 HOLD #> TYPE SPACE BASE ! ;
```

```
: .ALL ( d1 -- ) 2DUP CR .HEX CR .BIN CR;
```

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

