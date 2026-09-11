# NASA advanced diagnostics

This document collects additional NASA MessageSet values that can be exposed through the existing `custom_sensor` support.

The values are deliberately kept as custom sensors instead of hard-coded component fields until they have been validated against the specific outdoor unit and indoor units.

## Indoor units

Use these on each `20.xx.xx` indoor device:

| Message | Meaning | Conversion |
|---|---|---|
| `0x4211` | Capacity request | raw / 8.6 = kW |
| `0x4212` | Absolute capacity | raw / 8.6 = kW |
| `0x4217` | Indoor EEV 1 current position | raw value |
| `0x4218` | Indoor EEV 2 current position | raw value |
| `0x4219` | Indoor EEV 3 current position | raw value |
| `0x421A` | Indoor EEV 4 current position | raw value |
| `0x4205` | EVA inlet temperature | raw / 10 = °C |
| `0x4206` | EVA outlet temperature | raw / 10 = °C |

The capacity conversion is based on the NASA protocol notes and existing decoder behavior. The EEV values are intentionally published raw first; their exact range/meaning should be validated from live data on the target system.

## Outdoor unit

Use these on the `10.xx.xx` outdoor device:

| Message | Meaning | Conversion |
|---|---|---|
| `0x8204` | Outdoor air temperature | existing sensor |
| `0x8206` | High pressure | raw/diagnostic initially |
| `0x8208` | Low pressure | raw/diagnostic initially |
| `0x820A` | Discharge temperature 1 | raw / 10 = °C |
| `0x8217` | Compressor 1 current | existing sensor |
| `0x8229` | Main EEV 1 | raw value |
| `0x822A` | Main EEV 2 | raw value |
| `0x822B` | Main EEV 3 | raw value |
| `0x822C` | Main EEV 4 | raw value |
| `0x822D` | Main EEV 5 | raw value |
| `0x822E` | EVI EEV | raw value |
| `0x822F` | HR EEV | raw value |
| `0x8233` | Outdoor operation capacity sum | raw / 8.6 = kW |
| `0x8236` | Compressor 1 order frequency | Hz |
| `0x8237` | Compressor 1 target frequency | Hz |
| `0x8238` | Compressor 1 current frequency | Hz |
| `0x823D` | Outdoor fan 1 speed | rpm |
| `0x823E` | Outdoor fan 2 speed | rpm |
| `0x829F` | High-pressure saturation temperature | validate before adding engineering unit |
| `0x82A0` | Low-pressure saturation temperature | validate before adding engineering unit |

## Example

```yaml
custom_sensor:
  - name: "Living room capacity request"
    message: 0x4211
    unit_of_measurement: "kW"
    accuracy_decimals: 2
    state_class: measurement
    filters:
      - lambda: return x / 8.6;

  - name: "Living room absolute capacity"
    message: 0x4212
    unit_of_measurement: "kW"
    accuracy_decimals: 2
    state_class: measurement
    filters:
      - lambda: return x / 8.6;

  - name: "Living room EEV"
    message: 0x4217
    accuracy_decimals: 0
    entity_category: diagnostic

  - name: "Outdoor discharge temperature"
    message: 0x820A
    unit_of_measurement: "°C"
    accuracy_decimals: 1
    device_class: temperature
    state_class: measurement
    filters:
      - multiply: 0.1

  - name: "Outdoor compressor frequency"
    message: 0x8238
    unit_of_measurement: "Hz"
    accuracy_decimals: 1
    state_class: measurement

  - name: "Outdoor main EEV 1"
    message: 0x8229
    accuracy_decimals: 0
    entity_category: diagnostic
```

## Validation strategy

Do not convert `0x8206`/`0x8208` to bar yet. Capture their raw values while the system is heating and cooling and compare them with operating conditions. Likewise, first record the `0x4217`-`0x421A` and `0x8229`-`0x822F` ranges before assigning an engineering scale.

Once live data from the AJ100TXJ5KG/EU is available, the next step is to add proper named sensors and validated conversions to the component itself.