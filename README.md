# ECE 198 Project - Hot Drink Temperature Tracker

This project is for the ECE 198 Project Studio course in 1A Computer Engineering at the University of Waterloo, created by Wilson Cheng and William Zhang.

Its purpose is to track the temprature of a hot mug, and ring a buzzer when it has reached a preset desired temperature. 

All this was done by programming an STM32F401RE with a breadboard, resistors, wires, and the peripherals mentioned below. 

## Getting started

To utilize this, follow the documents in the "Design Files" directory to assemble the circuits and systems.

Then, to flash the code, plug the STM32 into an electronic source, clone this repository, open an STM CubeIDE, and press the build hammer and the run button.

The display should prompt you to press the button, after which the thermometer will take an initial measurement. Then, you should be prompted to turn the potentiometer knob to set a temperature. Push the button again to set your desired temperature, and wait for the buzz. You will see regular temperature updates. 

## Files Involved

The file `ECE_198.ioc` in the home directory allows me to set configurations for the STM32. Not only can I activate certain settings on specific pins or clocks with the click of a button, but I can also use it to generate code in the project directory that configures the desired settings.

Most of the work was done in the directory `Core/Src`, with the vast majority of code being written in `main.c`, the file that is first called when the board's chip begins running. That's where the procedure described below starts and is controlled. 

In addition, the files `liquidcrystal_i2c.c` and `liquidcrystal_i2c.h` were created as part of a library used to simplify functions for the LCD display, originally from [this](https://github.com/SayidHosseini/STM32LiquidCrystal_I2C) repository.

Finally, the files `mlx90614.c` and `mlx90614.h` are part of a library from [this](https://github.com/dinamitemic/mlx90614) repository, which could simplify the calling of the functions used to collect temperature data. However, it failed to work, so we implemented the same procedure ourselves in `main.c`. 

## How does this work?

The project will involve the control of 5 peripherals: an LCD display, an infrared thermometer, a push button, a buzzer, and a potentiometer.

This is accomplished with GPIO input/output control, ADC input, and the I2C communication protocol. 

### How is each peripheral controlled?

1) The display: we used the "LiquidCrystal_I2C" library for STM32 to call basic functions more easily. All that was required was connecting the circuits, checking the correct external I2C handle `hi2c1` in the library, and generating code to set up the SDA and SCL pins. 

2) The thermometer: since the library did not work, we implemented the procedure laid out in [this](https://www.youtube.com/watch?v=Qw4ScK2CZqI&pp=ygUTbWx4OTA2MTQgc3RtMzIgIGkyYw%3D%3D) Youtube video. To summarize how it works, the peripherals have set unique addresses on the I2C bus, and the `HAL_I2C_Mem_Read` function in the STM32 HAL (Hardware Abstraction Layer) library reads raw data collected by the sensor from the address, and stores it in a temporary data buffer (which comes as a character array). After certain numerical conversions with the given elements in the buffer, one obtains a temperature of a floating-point value. In order for this to be printed to the display, it must be casted to an integer with a simple `(int)` operation.

3) The potentiometer: here basic ADC was used. A potentiometer works by splitting the input voltage into two. The potential difference between the outer two pins remains the same, however, turning the knob will adjust the resistance and the output voltage in the inner pin (connected to a pin set to `ADC`). The potentiometer used in this project was linear, so we used a linear interpolation equation allowing adjustment on integer values from 20 degrees (standard room temperature) and the initial measurement, with single degree increments. See main.c comments.

4) The buzzer: this was a very simple peripheral, with no resistors needed, only a GPIO output, and a ground pin. When it was time to buzz, the GPIO output was set, and kept on for half a second with `HAL_Delay(500)` until the pin was reset. 

5) The push button: this peripheral was slightly complex, as the pins had to be bent straight to stick all the way in the breadboard for the button to work! The way this was set up was through a 5V to Ground connection with a 10k Ohm pull-up resistor in between, and an input pin branched out to the STM32 microcontroller. If the input pin detected a 0 state, which indicates that the button is pressed down, actions were taken accordingly, depending on the stage of the procedure.

### How is this all put together?

We use the power of enumerations! In the C programming language, enums are user-defined data types with user-defined readable names for the set of possible constants. 

With firmware and embedded systems programming, after you flash the code to the microcontroller, the microcontroller will continuously run the `while(1) {}` loop. For the special case of the STM32, when you open up the CubeIDE, you choose a board type, for which code is immediately set up by the IDE for that board and more code can be generated. 

Then, there are "user code" comments, between which one should write code to preserve it through code regenerations. To set pins to a certain state and configure them, one simply opens the `.ioc` file eponymous with the user-defined project name, sets configurations, saves the `.ioc` file, and the CubeIDE will generate code for you to configure everything.

Therefore, this project will have to run continuous, possibly infinite, rounds of the "measure-set temperature-check temperature-buzz" steps. In each step, the same while loop must be continuously traversed to monitor user input. So with different sets of sets of steps in the same loop, we simply associate enum constants such as `reset`, `set-temp`, etc with user-defined enums constants and certain commands via control flow. Side joke: ChatGPT was not used for that.

## Further Notes

An extra feature that was added to stop infinite runs is to check if the measured temperature is lower than the desired temperature. This occurs outside of existing conditions when both the desired temperature and cup temperature are much higher than room temperature, and the cup is removed before the buzzer sounds. 

The design files include the design document, the circuit diagrams, the 3D model of the stand. Note that the changelog markdown stores all version changes. 

To view a demonstration of a successful implementation, see the folder called 'Demo Videos'. Apologies for the poor filming quality and display resolution.

The Internet is full of good resources on the I2C communication protocol, including [(1)](https://www.ti.com/lit/an/slva704/slva704.pdf?ts=1704107523196&ref_url=https%253A%252F%252Fwww.google.com%252F), and [(2)](https://learn.sparkfun.com/tutorials/i2c/all).
