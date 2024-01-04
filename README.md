# ECE 198 Project - Temperature Aware Mug

This project is for the ECE 198 Project Studio course in 1A Computer Engineering at the University of Waterloo, created by Wilson Cheng and William Zhang.

Its purpose is to track the temperature of a hot mug, and ring a buzzer when it has reached a preset desired temperature. 

All this was done by programming an STM32F401RE with a breadboard, resistors, wires, and the peripherals mentioned below. 

## Getting started

To utilize this, follow the documents in the "Design Files" directory to assemble the circuits and systems.

Then, to flash the code, plug the STM32 into an electronic source, clone this repository, open an STM CubeIDE, and press the build hammer and the run button.

The display should prompt you to press the button, after which the thermometer will take an initial measurement. Then, you should be prompted to turn the potentiometer knob to set a temperature. Push the button again to set your desired temperature, and wait for the buzz. You will see regular temperature updates. 

## Files Involved

The coding was done in `Core/Src/main.c`, the file that is first called when the board's chip begins running. Please refer to the "How is this all put together?" section.

The file `ECE_198.ioc` in the home directory allows me to set configurations for the STM32. By activating certain settings on specific pins or clocks, I can use it to generate code in `Core/Src/main.c` that configures the desired settings and allows me to set or reset pins by calling GPIO or HAL functions on a small number of parameters.

In addition, the files `liquidcrystal_i2c.c` and `liquidcrystal_i2c.h` were imported as part of a library used to simplify functions for the LCD display, originally from [this](https://github.com/SayidHosseini/STM32LiquidCrystal_I2C) repository.

Finally, the files `mlx90614.c` and `mlx90614.h` are part of a library from [this](https://github.com/dinamitemic/mlx90614) repository, which could simplify the calling of the functions used to collect temperature data. It was used to verify the address of the thermometer on the I2C bus, by calling the function `MLX90614_ScanDevices()`.

## How does this work?

The project will involve the control of 5 peripherals: an LCD display, an infrared thermometer, a push button, a buzzer, and a potentiometer.

All the implementation was done in `Core/Src/main.c`.

### How is each peripheral controlled?

1) The display: we used the "LiquidCrystal_I2C" library for STM32 to call basic functions more easily. All that was required was connecting the circuits, checking the correct external I2C handle `hi2c1` in the library, and configuring the `ECE_198.ioc` file to generate code that would set up the I2C bus interface. 

2) The thermometer: since the library did not work, we implemented the procedure laid out in [this](https://www.youtube.com/watch?v=Qw4ScK2CZqI&pp=ygUTbWx4OTA2MTQgc3RtMzIgIGkyYw%3D%3D) Youtube video. To summarize how it works, the peripherals have set unique addresses on the I2C bus, and the `HAL_I2C_Mem_Read` function in the STM32 HAL (Hardware Abstraction Layer) library reads raw data collected by the sensor from the address, and stores it in a temporary data buffer (which comes as a character array containing 2 elements). After certain numerical conversions with the given elements in the buffer, as outlined in the video, one obtains a temperature of a floating-point value. In order for this to be printed to the display, it must be cast to an integer with a simple `(int)` prefix.

3) The potentiometer: here basic ADC was used. A potentiometer works by splitting the input voltage into two. The potential difference between the outer two pins remains the same, however, turning the knob will adjust the resistance and the output voltage in the inner pin (connected to a pin set to `ADC`). The potentiometer used in this project was linear, so we used a linear interpolation equation allowing adjustment on integer values from 20 degrees (standard room temperature) and the initial measurement, with single-degree increments. See main.c comments.

4) The buzzer: this was a very simple peripheral, with no resistors needed, only a GPIO output and a ground pin. When it was time to buzz, the GPIO output was set on pin 9 of port A (PA9) and kept on for half a second with the Hardware Abstraction Layer (HAL) function `HAL_Delay(500)` until the pin was reset. 

5) The push button: this peripheral was slightly complex, as the pins had to be bent straight to stick all the way in the breadboard for the button to work! The way this was set up was through a 5V to Ground connection with a 10k Ohm pull-up resistor in between, and an input pin branched out to the STM32 microcontroller. If the input pin detected a 0 state, which indicates that the button is pressed down, actions were taken accordingly, depending on the stage of the procedure.

### How is this all put together?

We use the power of enumerations! In the C programming language, enums are user-defined data types with user-defined readable names for the set of possible constants. 

With firmware and embedded systems programming, the microcontroller will only run once after you flash the code to the microcontroller and has to be maintained with an infinite loop. For the special case of the STM32, when you open up the CubeIDE, you choose a board type, for which the IDE immediately sets up default library functions and a development environment. After that, the user may continue to add code, which will be flashed together with the default code each time the project is built. 

Due to the setup of the STM32, this procedure will have to run infinite rounds of the "measure-set temperature-check temperature-buzz" steps. The same while loop must be continuously traversed in each step to monitor user input or temperature. So with different sets of steps in the same loop, we simply associate states such as `reset`, `set-temp`, etc., with special enum constants via control flow. 

The procedure will run as follows:
1) In the 'reset' state, the microcontroller will check if the user has pressed the button and thus activated the '0' state on it. If the button is not pressed, the display will continue, prompting the user to press the button. As soon as the button is pressed, the system will take a temperature measurement, print it to the display, and switch to the next state.
2) In the 'set-temp' state, the microcontroller checks the state of the potentiometer. Depending on its position, the ADC (Analog-to-Digital Converter) value may be different, which corresponds to a certain temperature value between 20 degrees Celsius and the initial temperature, which is assumed to be warmer than 20 degrees. The board will continuously print the current set temperature onto the display, which will update in real-time as the knob is turned. To enter the preferred temperature and move onto the 'watch-temp' state, the user has to press the button.
3) In the 'watch-temp' state, the board will measure the temperature every 10 seconds using a character buffer, the `HAL_I2C_Mem_Read` function, and conversions as described above. Every new temperature reading is printed on the display. However, one second after measurement, the controller will check if the measured temperature is within 5 degrees of or lower than the desired temperature. If so, it will transition to the 'buzz' state. Otherwise, it will continue the delay for 9 more seconds.
4) In the 'buzz' state, the controller will ring the buzzer every 0.5 seconds until the user has pressed the button, which will revert it to the 'reset' state.

## Further Notes

An extra feature added to stop infinite runs checks if the measured temperature is lower than the desired temperature. This occurs outside of existing conditions when both the desired temperature and cup temperature are much higher than room temperature, and the cup is removed before the buzzer sounds. 

The design files include the design document, the circuit diagrams, and the 3D model of the stand. Note that the changelog markdown stores all version changes. 

To view a demonstration of a successful implementation, see the folder called 'Demo Videos'. While there is no demonstration of waiting for the drink to cool by itself, the user-preferred temperature was set to 24 degrees, which is not far from the room's temperature. To speed up the process further, removing the cup brought the measured temperature down from 34 to 22 degrees, within 5 degrees of 24 degrees.

The Internet is full of good resources on the I2C communication protocol, including [(1)](https://www.ti.com/lit/an/slva704/slva704.pdf?ts=1704107523196&ref_url=https%253A%252F%252Fwww.google.com%252F), and [(2)](https://learn.sparkfun.com/tutorials/i2c/all).
