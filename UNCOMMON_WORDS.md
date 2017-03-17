# Uncommon Words
All of the words I write will be documented here in order to keep a history and allow for looking them up later.

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
