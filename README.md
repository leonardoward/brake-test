# brake-test

**Medidor de Aceleraciones en la Punta de Eje** — an axle-tip acceleration logger built to record brake-test data for the **Baja SAE USB** off-road prototype (Universidad Simón Bolívar, División de Electrónica). Designed and tested by Leonardo Ward, December 2013.

## What it does

A small metal enclosure bolts directly onto a wheel hub (using the wheel's own lug holes) and logs 3-axis acceleration to a microSD card while the car is driven and braked. The X/Y/Z axes are hand-marked in pen on the enclosure so the raw log can be matched back to the car's real-world orientation during analysis.

The intent was to instrument brake tests directly at the axle tip — closer to the wheel/road interface than a chassis-mounted IMU — and log locally to SD rather than requiring a live telemetry link. It's a simpler, standalone alternative to the team's existing telemetry system (a prior thesis project using Freescale MCUs, GPS, gyro, XBee/ZigBee and CAN, all logged to a PC over LabVIEW).

## Hardware

- **MCU:** ATmega328P (on an Arduino Uno board), 16 MHz.
- **Accelerometer:** [ADXL335](Pruebas%20de%20frenos/adxl335.pdf) (±3 g, 3-axis, analog output), on a small breakout board.
- **Logging:** a microSD "SD Card Shield" (SPI) stacked on the Arduino.
- **Power:** a single 9 V battery — an `L7805CV` linear regulator supplies 5 V to the Arduino/logger, an `LD33V` supplies 3.3 V to the accelerometer, each with 4.7 nF bypass caps.
- **Filtering:** each accelerometer axis (X, Y, Z) feeds an external RC low-pass filter (10 kΩ + 1 µF, cutoff ≈ 15.9 Hz) before reaching the ADC, on top of the ADXL335's own internal filter (its 32 kΩ internal resistors + the breakout board's 0.1 µF caps give ≈ 49.7 Hz). A pull-down resistor holds the ATmega's RESET pin, and a bench pushbutton/switch controls logging.
- **Enclosure:** hand-formed sheet-metal box, with the sensor circuit built on a bakelite perfboard.

See the [bill of materials](Pruebas%20de%20frenos/LISTA%20DE%20COMPONENTES%20MEDIDOR%20DE%20ACELERACIONES%20EN%20LA%20PUNTA%20DE%20EJE.docx) and the [full build report](Pruebas%20de%20frenos/Informe%20Medidor%20de%20Aceleraciones%20en%20las%20Puntas%20de%20Eje.docx) (both in Spanish) for the complete schematic description, component list and calculations.

## Calibration

The ADXL335 is a raw-voltage sensor, so each axis needs a per-device scale and offset before its ADC counts mean anything in m/s². The calibration procedure used here: with the sensor at rest, read the raw ADC count for each axis in two orientations 180° apart (i.e. +1 g and −1 g on that axis). The midpoint of the two readings is the axis's zero-g offset; half their difference is the 1 g scale. Both are recorded in [`Cálculos Medición de aceleraciones.xlsx`](Pruebas%20de%20frenos/C%C3%A1lculos%20Medici%C3%B3n%20de%20aceleraciones.xlsx) and then applied to every raw sample:

```
acceleration [m/s²] = (raw_count − zero_offset) / scale × 9.80665
```

## Data

Raw logs are plain CSV — `TIEMPO,ACEX,ACEY,ACEZ` (a running sample index, and the three raw ADC counts) — one file per run, in [`Pruebas de frenos/`](Pruebas%20de%20frenos/):

- [`Rueda Delantera Derecha/`](Pruebas%20de%20frenos/Rueda%20Delantera%20Derecha) — front-right wheel
- [`Rueda Delantera Izquierda/`](Pruebas%20de%20frenos/Rueda%20Delantera%20Izquierda) — front-left wheel
- [`Rueda Trasera Derecha/`](Pruebas%20de%20frenos/Rueda%20Trasera%20Derecha) — rear-right wheel

Each folder's `Cálculos...xlsx` applies the calibration above to its raw `ACEXYZ.CSV`. [`Pruebas de Frenos.xlsx`](Pruebas%20de%20frenos/Pruebas%20de%20Frenos.xlsx) collects the processed runs into one workbook — a reconnaissance lap plus several braking laps per wheel — each sheet giving time (s) and X/Y/Z acceleration (m/s²) at a 0.5 s sample interval. Tests were driven around Centro San Ignacio, Chacao, Caracas.

There is no firmware source in this repository — the logger's Arduino sketch was not preserved, only its output.

## Repository layout

```
Pruebas de frenos/
├── Informe Medidor de Aceleraciones en las Puntas de Eje.docx   full build report (theory, schematic, BOM)
├── LISTA DE COMPONENTES...docx                                  bill of materials
├── Cálculos Medición de aceleraciones.xlsx                      ADXL335 zero-offset / scale calibration
├── Pruebas de Frenos.xlsx                                       processed test runs (all wheels)
├── Rueda Delantera Derecha/, Rueda Delantera Izquierda/,
│   Rueda Trasera Derecha/                                       raw + per-wheel-calculated CSV logs
├── adxl335.pdf, ADXL335_v13.pdf, Acelerómetro...pdf              ADXL335 reference datasheets
├── Filtro RC.png                                                 RC low-pass filter design/response
└── San Ignacio.png, *.jpg                                        test-site map and build/mounting photos
```

## License

MIT — see [LICENSE](LICENSE).
