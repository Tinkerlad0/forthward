# Assignment 1
## Michael Brock - z5056704

### Question 1

i. Which of the following is an example of signal conditioning
(a) Adding a current limiting resistor to an LED
(b) Filtering an input signal before an analog-to-digital converter
(c) Acquiring data through an analog-to-digital converter
(d) Displaying the input signal on an LCD

ii. A modifying input
(a) Can not be removed from a measurement system
(b) Changes the sensitivity for a measurement
(c) Adds an offset to a measured value
(d) Aids in signal conditioning

iii. Most sensors have a built-in analog-to-digital converter (ADC) because
(a) It standardizes the output signal
(b) It is cheaper to manufacture sensors with an ADC than without one
(c) Microcontrollers tend not to have ADCs on board
(d) The signal is less susceptible to noise after analog-to-digital conversion

iv. A transducer is a device that
(a) Converts a voltage to a digital value
(b) Displays a measured physical quantity
(c) Converts a physical quantity into a mechanical movement
(d) Converts a physical quantity into an electrical quantity

#### Answers

* i) B
* ii) B
* iii) D
* iv) D

### Question 2

```V = 2 * ( p + 0.01 * K ) + Noise```

Therefore the ouput voltage would be ``` 2 * ( 5 + 3 ) + 0.1 = 16.1V```
However the noise should be considered to ±0.1V and hence our output is 16.0±0.1V

By looking at the gradient of `Vout` with respect to each input other than pressure we can see if they are modifying or interfering outputs.
The derivative of `Vout` with respect to temperature yields ```dV/dK = 0.02 K```. As the gradient of the original graph changes due to temperature we can see that temperature is a modifying input.
The derivative of `Vout` with respect to the noise yeilds ```dV/Dk = 1``` showing that noise is interfering. 

### Question 3

#### (a) According to the data sheet, is temperature a modifying or an interfering input to the pressure measurement?  Justify your answer in terms of the calibration factors for the transducer. (4 marks)
<p align="center"><img aligh="middle" src="https://s3-ap-southeast-2.amazonaws.com/brocky-storage/assign1.png"> </p>

 The graph above was created by entering the entire formula for pressure calculation into excel and plotting the calculated pressure values against raw pressure readings for a constant temperature.
 This was plotted for three different temperature values, so that we can examine the effect that temperature has on the calculation of pressure. 
 As can be seen by looking at the three graphs, the change in temperature both interferes by adding an offset as well modifying the sensitivity by changing the gradient.

Temperature is therefore both a modifying and interfering input.

#### (b) On page 13 there is a box that calculates the true temperature from input parameters read from the device.  Write a set of forth words that perform this calculation. (4 marks)

I am going to assume that all of the calibration data has been read into the variables of their equivalent name. I am storing the default values here to allow me to use the formula without having to have the actual chip available.

```
VARIABLE AC1 408 AC1 !
VARIABLE AC2 -72 AC2 !
VARIABLE AC3 -14383 AC3 !
VARIABLE AC4 32741 AC4 !
VARIABLE AC5 32757 AC5 !
VARIABLE AC6 23153 AC6 !
VARIABLE B1 6190 B1 !
VARIABLE B2 4 B2 !
VARIABLE B5
VARIABLE MB -327688 MB !
VARIABLE MC -8711 MC !
VARIABLE MD 2868 MD !
```

Therefore I can define the following forth word in order to convert an unscaled temperature value to a compenstaed temperature value.

```
: TrueTemp ( n1 -- n2 ) AC6 @ - AC5 @ $8000 */ >R MC @ $800 * R@ MD @ + / R> + DUP B5 ! $8 + $10 / ;
```

##### Explanation
This follows the formulae given in the datasheet for the BMP085 to convert the raw temperature to the compensated temperature. There is no new concepts here so I will not be explaining the code.

#### (c) If I wanted to use this device to provide temperature and pressure measurements together, and I had to acquire each of the two quantities and then transmit at the highest possible rate, how many measurements could I transfer per second? (2 marks)

The maximum sample rate of the device in standard mode is 128 samples per second. Taking the first sample as temperature this leaves us with 127 pressure samples in a second. 
Normally I would take into account the speed that I2C could transfer the data out of the chip, however this is insignificant in this case. To begin a sample requires 29 clock cycles of I2C communication, whilst reading back the data requires 47 cycles.
Together 76 cycles when running at 3.4MHz is only 0.02 ms worth of communication time per sample. For 128 total samples this is approximately 2.9ms of communication per second, making the I2C communication time insignificant.

If you calculate the maximum theoretical samples that the chip can achieve by using the time taken to sample you can theoretically get up to ```(1000 - 4.5) / 7.5 = 132 samples per second``` maximum.
However it clearly states on page 11 of the datasheet "All modes can be performed at higher speeds, e.g. up to 128 times per second for standard mode"

### Question 4
Despite the question referring to using port B, I used port C as this port is broken out in full on the external headers, whilst port B is only partially on the headers.
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

: InitPortBars ( -- ) PortC EnPort $33333333 PortC ! $33333333 PortC CRH ! ;

: PinOn ( port pin -- ) SWAP ODR DUP -ROT @ OR W! ;
: PinOff ( port pin -- ) INVERT SWAP ODR DUP -ROT @  AND W! ;

: ** ( n N -- n^N ) 1 SWAP 0 ?DO OVER * LOOP NIP ;

BARS ( c1 -- ) 0 PortC ODR W! 31 + 32** / 7 MIN 0 DO 2 I ** PortC ODR @ OR PortC ODR W! LOOP ;

InitPortBars

0 BARS
255 BARS
132 BARS
154 BARS
```
##### Explanation

The `portNum` word converts a port base address into the number port it is where port A is 0 and port G is 6.
Each of the port definitions defines the basse address for the port, which coincidentally is also the CRL register for the port. All the other relevant registers are defined as words to be called after the port which will apply the correct offset.
`EnPort` allows me to enable any given port, and I use this in `InitPortBars` to setup the port with all pins as outputs.
`BARS` initially sets the PortC ODR to 0 before starting. It takes the top number off the stack, adds 31 then divides by 32.
This scales values between 0 and 255 to values between 0 and 7 (inclusive). However if a value larger than 255 appears we dont want the loop to go longer, so `7 MIN` ensures that the largest number possible is 7. 
The loop will go from 0 to the previously calculated value, each time turning on the next LED (starting from 0) until the loop finishes. 
