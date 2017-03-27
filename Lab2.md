### Question 1

```
: EnPort ( n1 -- ) RCC_APB2ENR @ OR RCC_APB2ENR W! ;
: DisPort ( n1 -- ) INVERT RCC_APB2ENR @ AND RCC_APB2ENR W!

\ Define the port we can change state of.

: PortToBit ( n1 -- )
```




```
: portNum ( n1 -- ) $40010800 - 1024 / ;
: PortA $40010800 ;
: PortB $40010C00 ;
: PortC $40011000 ;
: PortD $40011400 ;
: PortE $40011800 ;
: PortF $40011C00 ;
: PortG $40012000 ;

: EnPort ( n1 -- ) portNum 2 + 1 swap lshift RCC_APB2ENR @ OR RCC_APB2ENR W! ;
: DisPort ( n1 -- ) portNum 2 + 1 swap lshift INVERT RCC_APB2ENR @ AND RCC_APP2ENR W! ;



```