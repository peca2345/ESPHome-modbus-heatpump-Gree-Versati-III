# ESPHome modbus - heatpump Gree Versati III

## Description:

This manual describes how to connect the device to Home Assistant via Modbus protocol using ESP and RS485/TTL converter.
In this case it is about connecting a Gree Versati III 10kw heat pump.



## Info:
- use only shielded cable, otherwise the error "Modbus CRC Check Failed!" may appear in the log.
- put a 120 ohm resistor after the last connected device
- modbus datasheet: [Gree Versati III](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/modbus-versati-iii-en.pdf)
- you have to find out what the heatpump address is - default is 0x1
- you also need to find out the serial port speed - default 9600

## Components:
- ESP8266 / ESP32
- RS485/TTL converter: [SHOP](https://www.laskakit.cz/prevodnik-ttl-na-rs-485--max485/) 

## Schematic:
![Schema](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/IMG/schematic.png?raw=true)

## ESPHome code:
```
uart:
  id: mod_bus
  tx_pin: GPIO16
  rx_pin: GPIO17
  baud_rate: 9600
  stop_bits: 2

modbus:
  flow_control_pin: GPIO23
  send_wait_time: 100ms
  id: modbus_versati3
  

modbus_controller:
  - id: versati3
    ## the Modbus device addr
    address: 0x1
    modbus_id: modbus_versati3
    setup_priority: -10
    update_interval: 15s
    
number:
  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "nasi_komora_modbus_tank_target_temp_set"
    address: 13
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    
sensor:
  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "nasi_komora_modbus_tank_target_temp"
    address: 13
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    # filters:
    # - multiply: 0.1    
```    
