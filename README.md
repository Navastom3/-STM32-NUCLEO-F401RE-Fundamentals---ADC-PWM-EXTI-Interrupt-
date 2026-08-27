# STM32 Peripheral Integration

**Author:** TOMAS NAVAS
**Date:** 08/2026
**Hardware:** STM32 NUCLEO-F401RE
**Language:** C
**Framework:** STM32 Hardware Abstraction Layer (HAL)

---

## 1. Project Overview

I started this project to apply the Embedded C knowledge I acquired through my **Embedded Systems Development with C Coursera certification**, and to progressively learn STM32 peripherals through hands-on implementation.

The project began by reading a potentiometer with ADC1 to learn ADC operation and CubeMX configuration. I then added PWM control, mapping the ADC reading to an LED's duty cycle, followed by an EXTI interrupt that toggles between controlling the LED and controlling a DC motor.

For V2, I replaced the LED-only output with a DC motor controlled through PWM using a MOSFET and flyback diode. The potentiometer now controls either the motor speed or the LED brightness depending on the interrupt state.

---

## 2. Demonstration

[▶ Watch the STM32 ADC/PWM/EXTI Demo on YouTube](https://www.youtube.com/watch?v=J1Mxn0nGeRE)

[▶ Watch the STM32 Motor Control Demo on YouTube]((https://youtu.be/AvxG5Rzgmmk)

### Current Behavior

1. Potentiometer produces an analog voltage.
2. ADC1 converts the voltage into a 12-bit digital value.
3. The ADC reading is used as the PWM compare value.
4. When the LED is disabled, the potentiometer controls the DC motor speed.
5. When the interrupt enables the LED, the motor is turned off and the potentiometer controls the LED brightness.
6. Button_1 external interrupt switches between the two output modes.

---

## 3. Hardware

### Main Components

* STM32 NUCLEO-F401RE
* Potentiometer
* Red LED
* 220 Ω resistor
* DC motor
* Logic-level N-channel MOSFET
* Flyback diode
* 9V battery for motor powering 
* Breadboard
* Jumper wires

---

## 4. Software Architecture

### ADC1

ADC1 starts a 12-bit analog-to-digital conversion and samples the voltage produced by the potentiometer. The conversion result is stored in `pot_reading` as a value from 0–4095 and is updated once per loop iteration.

This value is then used as the PWM compare value for either the LED or the motor.

### PWM

The timer counter period is set to `4095` to match the 12-bit ADC range, allowing `pot_reading` to be used directly as the compare value.

The PWM duty cycle is therefore controlled directly by the potentiometer reading.

When the motor is enabled, the PWM signal controls the gate of the MOSFET and changes the motor speed.

When the LED is enabled, the PWM signal controls the brightness of the LED.

### EXTI

The built-in push button on PC13 is configured as an external interrupt using EXTI.

When a button edge is detected, the interrupt temporarily interrupts normal program execution and HAL calls `HAL_GPIO_EXTI_Callback()`.

The callback toggles the `led_enabled` flag before execution returns to the main loop. The main loop uses this flag to decide whether the potentiometer controls the LED or the motor.

---

## 5. Design Decisions

### Decision 1: Use STM32 HAL Rather Than Implementing Every Peripheral Directly Through Registers

**Reason:**

In my previous learning, I created LED blink logic and then added a button to create a toggle LED using direct register manipulation. Although I enjoyed the bitwise operations and direct control over the hardware, I realized the amount of effort required to implement increasingly complex features.

Since the main purpose of this project is to learn STM32 fundamentals one peripheral at a time, I decided to use the STM32 HAL drivers.

### Decision 2: Match the PWM Counter Period to the 12-bit ADC Range

**Reason:**

The PWM timer was initially configured with its 32-bit counter period set to `4294967295`.

Since ADC1 produces values from 0–4095, directly using the ADC reading as the PWM compare value resulted in an extremely small duty cycle across the entire potentiometer range.

I changed the timer period to `4095` so the ADC reading could be used directly as the PWM compare value.

### Decision 3: Keep the CubeMX-Generated Modular Peripheral Structure

**Reason:**

I kept the CubeMX-generated separation of peripheral configuration rather than consolidating all peripheral initialization into `main.c`.

This keeps peripheral initialization separate from the main application logic, making the project easier to navigate, debug, and extend as additional peripherals and features are introduced.

### Decision 4: Use a MOSFET to Control the DC Motor

**Reason:**

The STM32 GPIO pins cannot supply the current required to power the DC motor directly.

I used an N-channel MOSFET as a low-side switch so the STM32 only provides the control signal while the motor receives power from an external supply.

A flyback diode is connected across the motor to protect the circuit from the voltage spike created when current through the motor is switched off.

---

## 6. Problems / Challenges

### Challenge 1: PWM Did Not Visibly Change LED Brightness

**Cause:**

The timer counter period was configured as `4294967295`, while ADC values ranged only from 0–4095.

**Solution:**

Configured the timer period to `4095`, allowing ADC readings to be used directly as PWM compare values.

### Challenge 2: External Interrupt Did Not Affect the LED State

**Cause:**

The interrupt callback was checking the wrong generated pin identifier. The project used `Button_1_Pin`, while the callback was originally checking `B1_Pin`.

I also initially declared the LED enable flag locally inside `main()`, even though it needed to be accessed and modified by the interrupt callback.

**Solution:**

Moved `led_enabled` to global scope and declared it as `volatile`, allowing both the main loop and interrupt callback to access it. I also changed the callback to check `Button_1_Pin`.

The EXTI callback now toggles the flag, while the main loop uses its value to switch between LED and motor control.

### Challenge 3: Motor PWM Did Not Work Correctly

**Cause:**
When I first added the motor, it would not turn on. After checking the circuit with a multimeter, I found that current was reaching the motor, but there was almost no voltage difference across its terminals. When I rewired one side to ground, the motor stayed fully on instead of responding to changes in PWM duty cycle. This made me look more closely at the transistor stage, where I realized the MOSFET was not switching the motor correctly from the STM32’s 3.3 V gate signal.

**Solution:**

I replaced the IRF520 with an IRLZ44N n-logic-level MOSFET that can be driven properly from the STM32.

After changing the MOSFET, the motor responded correctly to changes in the potentiometer and PWM duty cycle.

This also helped me understand the difference between MOSFET gate threshold voltage and the gate voltage required for useful conduction.

---

## 7. Future Improvements

* UART telemetry
* SPI / I2C peripherals
* Moving into a CAN-Capable stm32 or using a CAN controller for communication
* Temperature sensor integration
* Closed-loop motor speed control
* Custom PCB implementation

---

## 8. Project Status

* [x] GPIO
* [x] ADC
* [x] PWM
* [x] External interrupts
* [x] DC motor control
* [x] MOSFET motor driver
* [x] Hardware verification
* [ ] UART
* [ ] SPI / I2C
* [ ] CAN

**Current Version:** V2.0 — ADC / PWM / EXTI / DC Motor Integration

