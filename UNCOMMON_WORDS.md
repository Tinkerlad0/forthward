# Uncommon Words
All of the words I write will be documented here in order to keep a history and allow for looking them up later.

## Math Custom Words

* Square - Usage ( 2 ^2 . -> 4 ok )
```
: ^2 ( n -- n^2 ) dup * ;
```

* Exponential -Usage ( 2 4 ** . -> 16 ok )
```
: ** ( n N -- n^N ) 1 swap 0 ?do over * loop nip ;
```

## Unit Conversions

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

#### Floating Point Arithmetic

Note that you will need to have numbers on the floating point stack instead of the integer stack here.

