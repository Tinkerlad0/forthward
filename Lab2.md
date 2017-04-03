# This is all untested on the device at this stage, and written whilst I was quite tired....

### Question 1

```
: EnPort ( n1 -- ) RCC_APB2ENR @ OR RCC_APB2ENR W! ;
: DisPort ( n1 -- ) INVERT RCC_APB2ENR @ AND RCC_APB2ENR W!
: PCPOn ( n1 -- ) GPIOC_ODR @ OR GPIOC_ODR W! ;
: PCPOff ( n1 -- ) INVERT GPIOC_ODR AND GPIOC_ODR W! ;


\ Define the port we can change state of.
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

: ODR ( n1 -- n2 ) $0C + ; \ Convert base port address to a ODR address
: CRH ( n1 -- n2 ) $04 + ; \ Convert base port address to a CRH address
: IDR ( n1 -- n2 ) $8H + ; \ Convert base port address to a IDR address

: NULL ( -- ) ;

: portNum ( n1 -- ) $40010800 - 1024 / ;
: EnPort ( n1 -- ) portNum 2 + 1 SWAP LSHIFT RCC_APB2ENR @ OR RCC_APB2ENR W! ;
: DisPort ( n1 -- ) portNum 2 + 1 SWAP LSHIFT INVERT RCC_APB2ENR @ AND RCC_APP2ENR W! ;

: PinAnalogInput ( port pin -- ) DUP 7 > IF 7 - SWAP CRH SWAP THEN 1 - $F LSHIFT INVERT SWAP DUP -ROT @ AND SWAP W! ;
: PinOutput ( port pin -- ) DUP 7 > IF 7 - SWAP CRH SWAP THEN 1 - DUP >R $3 LSHIFT SWAP DUP -ROT @ OR SWAP DUP -ROT W! $C R> SWAP LSHIFT INVERT SWAP DUP -ROT @ AND SWAP W! ;

: PinOn ( port pin -- ) SWAP ODR DUP -ROT @ OR W! ;
: PinOff ( port pin -- ) INVERT SWAP ODR DUP -ROT @  AND W! ;
```


### Question 2

Assuming that the Question 1 words are defined

```
: ReadPin ( port pin -- ) 1 - 1 SWAP LSHIFT SWAP IDR @ AND 0 > ;
: PWM ( n1 -- ) ABS R> PortC 8 PinAnalogInput BEGIN PortC 8 ReadPin IF NULL ELSE LEAVE THEN R@ 5 / DUP PortC 10 PinOn MS 20 SWAP - MS AGAIN R> DROP;
```