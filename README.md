# STM32 Peripheral Integration

**Author:** TOMAS NAVAS  
**Date:** 08/2026  
**Hardware:** STM32 NUCLEO-F401RE  
**Language:** C  
**Framework:** STM32 Hardware Abstraction Layer (HAL)

---

## 1. Project Overview

I started this project to apply the Embedded C knowledge I acquired through my **Embedded Systems Development with C Coursera certification**, and to progressively learn STM32 peripherals through hands-on implementation.

The project began by reading a potentiometer with ADC1 to learn ADC operation and CubeMX configuration. I then added PWM control, mapping the ADC reading to an LED's duty cycle, followed by an EXTI interrupt that toggles whether the PWM output is enabled.

My goal for V2 is to replace the LED with a DC motor and implement PWM-based motor speed control using a transistor and flyback diode.

---

## 2. Demonstration

[▶ Watch the STM32 ADC/PWM/EXTI Demo on YouTube](https://www.youtube.com/watch?v=J1Mxn0nGeRE)

### Current Behavior

1. Potentiometer produces an analog voltage.
2. ADC1 converts the voltage into a 12-bit digital value.
3. TIM2 generates PWM based on the ADC reading.
4. LED brightness changes with the potentiometer.
5. Button_1 external interrupt enables/disables the LED output.

---

## 3. Hardware

### Main Components

- STM32 NUCLEO-F401RE
- Potentiometer
- Red LED
- 220 Ω resistor
- Breadboard
- Jumper wires

---

## 4. Software Architecture

### ADC1

ADC1 starts a 12-bit analog-to-digital conversion and samples the voltage produced by the potentiometer. The conversion result is stored in `pot_reading` as a value from 0–4095 and is continuously updated. This value is then used as the input for PWM control.

### TIM2 / PWM

TIM2 Channel 2 generates a PWM signal on PB3 (D3 on the board). The timer counter period is set to 4095 to match the 12-bit ADC range, allowing `pot_reading` to be used directly as the compare value.

The compare value determines the PWM duty cycle and therefore the perceived brightness of the LED.

### EXTI

The built-in push button on PC13 is configured as an external interrupt using EXTI. When a button edge is detected, the interrupt temporarily interrupts normal program execution and HAL calls `HAL_GPIO_EXTI_Callback()`.

The callback toggles the `led_enabled` flag before execution returns to the main loop. The main loop uses this flag to enable or disable the LED PWM output.

---

## 5. Design Decisions

### Decision 1: Use STM32 HAL Rather Than Implementing Every Peripheral Directly Through Registers

**Reason:**

In my previous learning, I created LED blink logic and then added a button to create a toggle LED using direct register manipulation. Although I enjoyed the bitwise operations and direct control over the hardware, I realized the amount of effort required to implement increasingly complex features.

Since the main purpose of this project is to learn STM32 fundamentals one peripheral at a time, I decided to use the STM32 HAL drivers.

### Decision 2: Match the PWM Counter Period to the 12-bit ADC Range

**Reason:**

TIM2 was initially configured with its 32-bit counter period set to `4294967295`.

Since ADC1 produces values from 0–4095, directly using the ADC reading as the PWM compare value resulted in an extremely small duty cycle across the entire potentiometer range.

I changed the TIM2 counter period to `4095` so the ADC reading could be used directly as the PWM compare value.

### Decision 3: Keep the CubeMX-Generated Modular Peripheral Structure

**Reason:**

I kept the CubeMX-generated separation of peripheral configuration across modules such as `adc.c`, `tim.c`, `gpio.c`, and `usart.c`, rather than consolidating the peripheral configuration into `main.c`.

This keeps peripheral initialization separate from the main application logic, making the project easier to navigate, debug, and extend as additional peripherals and features are introduced.

## 6. Problems / Challenges

### Challenge 1: PWM Did Not Visibly Change LED Brightness

**Cause:**

TIM2's counter period was configured as `4294967295`, while ADC values ranged only from 0–4095.

**Solution:**

Configured the timer period to `4095`, allowing ADC readings to be used directly as PWM compare values.

### Challenge 2: External Interrupt Did Not Affect the LED State

**Cause:**

The interrupt callback was checking the wrong generated pin identifier. The project used `Button_1_Pin`, while the callback was originally checking `B1_Pin`.

I also initially declared the LED enable flag locally inside `main()`, even though it needed to be accessed and modified by the interrupt callback.

**Solution:**

Moved `led_enabled` to global scope and declared it as `volatile`, allowing both the main loop and interrupt callback to access it. I also changed the callback to check `Button_1_Pin`.

The EXTI callback now toggles the flag, while the main loop uses its value to enable or disable the PWM output.

---

## 7. Future Improvements

- UART telemetry
- SPI / I2C peripherals
- CAN communication
- More advanced sensor integration

---

## 8. Project Status

- [x] GPIO
- [x] ADC
- [x] PWM
- [x] External interrupts
- [ ] UART
- [ ] SPI / I2C
- [ ] CAN

**Current Version:** V1.0 — ADC / PWM / EXTI Integration
