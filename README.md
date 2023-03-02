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
- in ESPHome use the sensor class only for addresses that are read-only
- for addresses that are read/write use the "number" class (you can then change their values in lovelace)
- for each register you want to have in HA you have to create a separate sensor in ESPHome
- you can write the address to the sensor in decimal or hex

## Lovelace:
![lovelace](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/IMG/Lovelace.png?raw=true)


## Components:
- ESP8266 / ESP32
- RS485/TTL converter: [SHOP](https://www.laskakit.cz/prevodnik-ttl-na-rs-485--max485/) 


## Schematic ESP32:
![Schema](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/IMG/schematic2.png?raw=true)

## Schematic ESP8266 - Wemos D1 mini:
![Schema](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/IMG/schematic_wemos.png?raw=true)

## ESPHome code:
```
uart:
  id: mod_bus
  tx_pin: GPIO16 # D7 for Wemos
  rx_pin: GPIO17 # D6 for Wemos
  baud_rate: 9600
  stop_bits: 2

modbus:
  flow_control_pin: GPIO23 # D5 for Wemos
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
    name: "Gree_versati_tank_temp_set"
    address: 13
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    
sensor:
  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "Gree_versati_tank_temp_target"
    address: 13
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1 

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "Gree_versati_tank_temp"
    address: 128
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1
    
  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "Gree_versati_outside_temp"
    address: 118
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    
    
  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "Gree_versati_water_in_temp"
    address: 127
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "Gree_versati_water_out_temp"
    address: 125
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    
```    


