# CHANGELOG

Documentation of changes to the project. 

## [v.0.3] - 2023-11-19

## Added:
- new redundancy feature to check if measured temperature is below desired temperature
- merged to main as last revision in `a234316e`

## [v.0.2] - 2023-11-19

### Added:
- A 1 second delay after final temperature measurement so user can verify that it is close to desired temperature.
- merged to main in `b8ecdb88`

### Changed:
- Changed from 30 second delay to 10 second delay

## [v.0.1] - 2023-11-16

### Added:
- *Mechanical*: new stand configuration, from the file added in commit `07b0906e`. It has a cardboard guard rail, and a box to lower the thermometer.

### Changed:
- fixed potentiometer linear interpolation formula to T = ((ADC_Value)*(init_temp-20)/4095)+20, where 
    - T is temperature mapped by potentiometer
    - init_temp is initial temperature measurement
    - ADC_Value is raw potentiometer value received by ADC 
- merged to main in `673a10b9`

## [v.0.0] - 2023-11-16

### Added:

- Made procedure work according to original design
- Configured loops and delays
- In commit `f562d5b0`
- *Schematic*: two significant feature changes in commit `514a44f2`. This includes:
    - 10k Ohm Resistors added to SDA/SCL connection with 5V
    - Voltage and GPIO pin changes on button and potentiometer
- *Mechanical*: stand configuration with rail according to original design, in commit `cbe37fea`