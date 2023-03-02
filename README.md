# ESPHome-modbus-heatpump-Gree-Versati-III

** Info: **

Tento návod popisuje jak připojit zařízení do Home Assistant přes modbus protokol pomocí ESP a RS485/TTL převodníku.
V tomto případě jde o připojení tepelného čerpadla Gree Versati III.

** Komponenty: **

- ESP8266 / ESP32
- RS485/TTL converter: [SHOP](https://www.laskakit.cz/prevodnik-ttl-na-rs-485--max485/) 

## ESPHome code
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
