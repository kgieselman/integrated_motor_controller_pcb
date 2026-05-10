# Integrated Motor Controller — Bring-Up Notes

Sequencing and constraints for first-board bring-up.

---

## Power-On Sequence

### Pre-power checklist

1. Confirm no shorts on +BATT, 5V, 5V_AUX, 3V3 rails (continuity to GND).
2. Confirm USB-C (J11) and battery (J13) should not both be connected on first power-on.
3. Confirm Y1 (ASE-25.000MHZ) orientation — pin 1 marked with dot; OE pin must be driven or tied appropriately.

### Rail verification (battery path)

1. Connect bench supply to J13 (BATT_IN / XT60) at 7.4 V (2S nominal), current-limited to 500 mA.
2. Press SW1 (power switch).
3. Verify with multimeter:
   - TP9 = 5V ± 100 mV (U4 buck, main rail)
   - TP6 = 5V_AUX ± 100 mV (U2 buck, servo/aux rail)
   - TP8 = 3V3 ± 50 mV (U3 LDO)
   - TP7 = BATT_DIV — should be approximately V_BATT × R_low / (R_high + R_low). JLC resistors are intentionally kept here; per-board ADC calibration (see below) compensates for any divider tolerance.
4. Check current draw at idle (no MCU firmware) — expect < 50 mA.

### Rail verification (USB path)

1. Connect USB-C (J11) with no battery attached.
2. Verify TP8 = 3V3 (logic rails only; 5V bucks do not run from VBUS).
3. Verify TP9 and TP6 are near 0V (bucks off without battery).

---

## STM32 Programming

### SWD (J2) — primary path

J2 pinout (2×3, pin 1 top-left):

```
1  VCC_TARGET (3.3V)   2  SWDIO
3  GND                 4  SWDCLK
5  GND                 6  NRST
```

1. Connect J-Link or ST-Link to J2.
2. Supply power (SW1 on, battery connected, or USB-C connected for 3.3V).
3. Flash with OpenOCD or STM32CubeProgrammer targeting STM32H563RIT6.
4. To force BOOT0 (system bootloader for DFU): hold SW3 (BOOT), press SW2 (RST), release SW2, then release SW3.

---

## Motor Driver Bring-Up (DRV8874)

Both motor channels (U7 for Motor 0, U8 for Motor 1) share the same sheet — verify each independently.

1. Before connecting motors, confirm IPROPI resistor values are correct for expected current range.
2. With MCU firmware loaded, confirm PMODE is driven high (IN1/IN2 control mode).
3. Connect a small test motor to J14 (Motor 0) or J15 (Motor 1) and verify direction control and IPROPI feedback.
4. Do not exceed DRV8874 absolute maximum V_VM (65V); 2S/3S is well within range.

---

## External Receiver (J1)

J1 pinout (1×4, pin 1 = square pad):

```
1  GND
2  +5V
3  CRSF_RX  (MCU←, IO to receiver TX)
4  CRSF_TX  (MCU→, IO to receiver RX)
```

JP1 and JP2 are bridged by default — the CRSF UART is a direct MCU↔J1 connection. No jumper changes required for normal operation.

To connect an ELRS receiver: wire receiver TX → J1 pin 3, receiver RX → J1 pin 4.

---

## Battery Voltage Sense Calibration

The VBAT_SENSE divider uses JLC resistors (not precision-tolerance parts). Each board must be calibrated individually to remove the divider error from ADC readings.

**Procedure (per board):**

1. Connect the calibrated DC supply to J8. Set a known voltage in the 6–9 V range (typical 2S window).
2. Run the ADC calibration routine in the bringup firmware (trigger TBD).
3. The routine samples `ADC1_INP4` (PC4), computes the scale factor against the known supply voltage, and stores the result in EEPROM (address TBD).
4. `Battery.hpp` must apply this stored scale factor at runtime instead of using nominal divider math.
5. Repeat at a second setpoint to verify linearity before signing off.

> **Note:** The calibration trigger mechanism has not been decided yet. Options include a dedicated console command (`cal vbat`), a button-hold sequence at boot, or a flag byte in EEPROM. Update this section once the approach is settled.

---

## Notes

- **SW2** = MCU RST (active low, pulls NRST)
- **SW3** = MCU BOOT0 (hold during reset to enter system bootloader)
- **SW4** = USR (user debug button)
- **SW1** = board power (series with Q1 gate drive)
- **D6** = LED0, **D7** = LED1, **D8** = LED2 — debug LEDs driven by MCU GPIOs; **D9** = PWR indicator LED
- **BZ** = buzzer driven by MCU GPIO; D10 is flyback protection
- **J12** = VMeter — exposes +BATT and GND for an external battery voltage display
