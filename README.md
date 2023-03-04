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
![lovelace](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/IMG/lovelace3.png?raw=true)


## Components:
- ESP8266 / ESP32
- RS485/TTL converter: [SHOP](https://www.laskakit.cz/prevodnik-ttl-na-rs-485--max485/) 


## Schematic ESP32:
![Schema](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/IMG/schematic2.png?raw=true)

## Schematic ESP8266 - Wemos D1 mini:
![Schema](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/IMG/schematic_wemos.png?raw=true)

## ESPHome code:
```
esphome:
  name: versati-modbus-esp32

esp32:
  board: esp32dev
  framework:
    type: arduino

logger:
captive_portal:
api:
  encryption:
    key: "1k0OjFSddafHxtX4c5YzpeTwmWfc4X9aqgsVKzS2ntI="
    
ota:
  password: "af492f27651e0694487dedbdb60e136e"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

web_server:
  port: 80

uart:
  id: mod_bus
  tx_pin: GPIO16
  rx_pin: GPIO17
  baud_rate: 9600
  stop_bits: 1

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



text_sensor:
  - platform: template
    id: versati_117_unit_status_text
    name: "versati_117_unit_status"

  - platform: template
    id: versati_132_thermostat_status_text
    name: "versati_132_thermostat_status"
    
  - platform: template
    id: versati_135_disinfection_status_text
    name: "versati_135_disinfection_status"



number:
  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_2_mode_set"
    id: versati_2_mode_set
    address: 2
    value_type: S_WORD
    entity_category: config
    min_value: 1
    max_value: 5
    step: 1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_3_optional_E_heater_set"
    address: 3
    value_type: S_WORD
    entity_category: config
    step: 1
    min_value: 1
    max_value: 3
    mode: slider  
      # last: 2   
      # 1:1 set/
      # 2:2 sets/
      # 3: Off
      # Default: 1 set

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_4_disinfection_temp_set"
    address: 4
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config
    step: 1
    min_value: 40
    max_value: 70
    mode: slider   
      # last: 60℃
      # Default: 70℃
  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_5_floor_debug_segments_set"
    address: 5
    value_type: S_WORD
    entity_category: config
    step: 1
    min_value: 1
    max_value: 10
    mode: slider  
      # Default : 1 section

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_6_floor_debug_period_1_set"
    address: 6
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config
    step: 1
    min_value: 25
    max_value: 35
    mode: slider  
      # Default: 25℃

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_7_delta_of_segment_temp_set"
    address: 7
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config
    step: 1
    min_value: 2
    max_value: 10
    mode: slider  
      # Default: 5℃

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_8_segment_time_set"
    address: 8
    unit_of_measurement: "h"
    value_type: S_WORD
    entity_category: config  
    step: 1
    min_value: 12
    max_value: 72
    mode: slider  
      # Default: 0 Hour

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_9_WOT_cool_temp_set"
    address: 9
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config        
    step: 1
    min_value: 7
    max_value: 25
    mode: slider  
      # Default: 18℃

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_10_WOT_heat_temp_set"
    address: 10
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config   
    step: 1
    min_value: 20
    max_value: 60
    mode: slider  
      # last: 46℃
      # Actual value:
      # 20~60℃ [High-temp] / 20~55℃[low-temp]
      # Default：
      # 45℃[High-temp]/45℃[Low-temp]

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_11_RT_cool_temp_set"
    address: 11
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 18
    max_value: 30
    mode: slider 
      # Default：24°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_12_RT_heat_temp_set"
    address: 12
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 18
    max_value: 30
    mode: slider 
      # Default：20°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_13_tank_target_temp_set"
    address: 13
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 40
    max_value: 80
    mode: slider 
      # last: 48°C
      # Default：50°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_14_eheater_temp_set"
    address: 14
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: -20
    max_value: 18
    mode: slider 
      # last: -7°C
      # Default：-15°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_15_other_switch_on_temp_set"
    address: 15
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: -20
    max_value: 18
    mode: slider 
      # last: -20°C
      # Default：-20°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_16_HP_max_temp_set"
    address: 16
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 40
    max_value: 55
    mode: slider 
      # last: 50°C
      # Default：50°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_17_upper_AT_heat_temp_set"
    address: 17
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 10
    max_value: 37
    mode: slider 
      # last: 12°C
      # Default：25°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_18_lower_AT_heat_temp_set"
    address: 18
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: -20
    max_value: 9
    mode: slider 
      # last: -12°C
      # Default：-20°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_19_upper_RT_heat_temp_set"
    address: 19
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 22
    max_value: 30
    mode: slider 
      # last: 24°C
      # Default：24°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_20_lower_RT_heat_temp_set"
    address: 20
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 18
    max_value: 21
    mode: slider 
      # last: 18°C
      # Default：18°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_21_upper_WT_heat_temp_set"
    address: 21
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 46
    max_value: 60
    mode: slider 
      # last: 43°C
      # Actual value: 46～60℃[High-temp]/ 46～55℃[Low-temp]
      # 55℃[High-temp]/55℃[Low-temp]

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_22_lower_WT_heat_temp_set"
    address: 22
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 20
    max_value: 45
    mode: slider 
      # last: 33°C
      # Default：40°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_23_upper_AT_cool_temp_set"
    address: 23
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 26
    max_value: 48
    mode: slider 
      # last: 40°C
      # Default：40°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_24_lower_AT_cool_temp_set"
    address: 24
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 10
    max_value: 25
    mode: slider 
      # last: 25°C
      # Default：25°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_25_upper_RT_cool_temp_set"
    address: 25
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 24
    max_value: 30
    mode: slider 
      # last: 27°C
      # Default：27°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_26_lower_RT_cool_temp_set"
    address: 26
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 18
    max_value: 23
    mode: slider 
      # last: 22°C
      # Default：22°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_27_upper_WT_cool_temp_set"
    address: 27
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1
    min_value: 15
    max_value: 25
    mode: slider 
      # last: 15°C
      # Default：15°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_28_lower_WT_cool_temp_set"
    address: 28
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 7
    max_value: 14
    mode: slider 
      # last: 7°C
      # Default：7°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_29_delta_cool_temp_set"
    address: 29
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 2
    max_value: 10
    mode: slider 
      # last: 5°C
      # Default：5°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_30_delta_heat_temp_set"
    address: 30
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 2
    max_value: 10
    mode: slider 
      # last: 10°C
      # Default：10°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_31_delta_hot_water_temp_set"
    address: 31
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 2
    max_value: 8
    mode: slider 
      # last: 5°C
      # Default：5°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_32_delta_room_temp_set"
    address: 32
    unit_of_measurement: "°C"
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 1
    max_value: 5
    mode: slider 
      # last: 2°C
      # Default：2°C

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_33_cool_run_time_set"
    address: 33
    unit_of_measurement: "min"
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 1
    max_value: 10
    mode: slider 
      # last: 3min
      # Default：3min

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_34_heat_run_time_set"
    address: 34
    unit_of_measurement: "min"
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 1
    max_value: 10
    mode: slider 
      # last: 5min
      # Default：5min

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_35_other_thermal_logic_set"
    address: 35
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 1
    max_value: 3
    mode: slider 
      # last: 0
      # Default：1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_36_tank_heater_set"
    address: 36
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 1
    max_value: 2
    mode: slider 
      # last: 1
      # Default：1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_37_optional_E_heater_logic_set"
    address: 37
    value_type: S_WORD
    entity_category: config 
    step: 1    
    min_value: 1
    max_value: 2
    mode: slider 
      # last: 2
      # Default：1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_38_current_limit_value_set"
    address: 38
    unit_of_measurement: "A"
    value_type: S_WORD
    entity_category: config
    step: 1    
    min_value: 0
    max_value: 50
    mode: slider 
      # last: 16A
      # Default：16A

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_39_thermostat_mode_set"
    address: 39
    value_type: S_WORD
    entity_category: config
    step: 1    
    min_value: 0
    max_value: 2
    mode: slider 
      # last: 2
      # 0: Without/
      # 1: Air /
      # 2: Air+hot water
      # Default: 0-Without

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_40_force_mode_set"
    address: 40
    value_type: S_WORD
    entity_category: config
    step: 1    
    min_value: 1
    max_value: 3
    mode: slider 
      # last: 3
      # 1: Force-cool/
      # 2: Force-heat /
      # 3: Off
      # Default: 3 - Off

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_41_air_removal_set"
    address: 41
    value_type: S_WORD
    entity_category: config
    step: 1    
    min_value: 1
    max_value: 3
    mode: slider   
      # last: 3  
      # 1: Air /
      # 2: Water tank/
      # 3: Off

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_42_on85_off170_set"
    address: 42
    value_type: S_WORD
    entity_category: config
    step: 85
    min_value: 85
    max_value: 170
    mode: slider   
      # last: 85
      # 0xAA:On = 85
      # 0x55:Off = 170
      # Default: Off=170

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_43_power_limit_set"
    address: 43
    unit_of_measurement: "kW" 
    value_type: S_WORD
    entity_category: config
    step: 1    
    min_value: 0
    max_value: 10
    mode: slider   
      # last: 42 
      # Actual value:0～10 Kw
      # Default :3 Kw

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_44_error_reset_1=clear"
    address: 44
    value_type: S_WORD
    entity_category: config
    step: 1    
    min_value: 0
    max_value: 1
      # 0: Does not clear fault
      # 1: Clear fault






sensor:
  - platform: modbus_controller
    modbus_controller_id: versati3
    id: versati_117_unit_status_number
    name: versati_117_unit_status_number
    address: 117
    register_type: holding
    value_type: U_WORD
    on_value:
      then:
        - lambda: |-
            int state = id(versati_117_unit_status_number).state;
            std::string text;
            if (state == 1) {
              text = "COOL";
            } else if (state == 2) {
              text = "HEAT";
            } else if (state == 6) {
              text = "HOT WATER";
            } else if (state == 8) {
              text = "OFF";
            } else {
              text = "UNKNOWN";
            }
            id(versati_117_unit_status_text).publish_state(text);

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_118_outdoor_temp"
    address: 118
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_119_discharge_temp"
    address: 119
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_120_defrost_temp"
    address: 120
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_121_suction_temp"
    address: 121
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_122_economizer_in_temp"
    address: 122
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_123_economizer_out_temp"
    address: 123
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_124_discharge_pressure_temp"
    address: 124
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_125_water_out_PE_temp"
    address: 125
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_126_optional_water_sensor_temp"
    address: 126
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_127_water_in_PE_temp"
    address: 127
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_128_tank_control_temp"
    address: 128
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_129_remote_room_temp"
    address: 129
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1    

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_130_gas_pipe_temp"
    address: 130
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_131_liquid_pipe_temp"
    address: 131
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
    - multiply: 0.1

  - platform: modbus_controller
    modbus_controller_id: versati3
    id: "versati_132_thermostat_status_number"
    name: versati_132_thermostat_status_number
    address: 132
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1 
    on_value:
      then:
        - lambda: |-
            int state = id(versati_132_thermostat_status_number).state;
            std::string text;
            if (state == 1) {
              text = "COOL";
            } else if (state == 2) {
              text = "HEAT";
            } else if (state == 3) {
              text = "OFF";
            } else {
              text = "UNKNOWN";
            }
            id(versati_132_thermostat_status_text).publish_state(text);


  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_133_floor_debug"
    address: 133
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1 
    filters:
    - multiply: 0.001    

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_134_debug_time"
    address: 134
    unit_of_measurement: "h" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1 

  - platform: modbus_controller
    modbus_controller_id: versati3
    id: "versati_135_disinfection_status_number"
    name: "versati_135_disinfection_status_number"
    address: 135
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1 
    on_value:
      then:
        - lambda: |-
            int state = id(versati_135_disinfection_status_number).state;
            std::string text;
            if (state == 1) {
              text = "Running";
            } else if (state == 2) {
              text = "Done";
            } else if (state == 3) {
              text = "Failed";
            } else if (state == 0) {
              text = "OFF";
            } else {
              text = "UNKNOWN";
            }
            id(versati_135_disinfection_status_text).publish_state(text);

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_136_error_time_for_floor_debug"
    address: 136
    unit_of_measurement: "s" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1 

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_137_weather_depend_temp"
    address: 137
    unit_of_measurement: "°C" 
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1 

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_142_setting_fruequency_status"
    address: 142
    unit_of_measurement: "Hz" 
    register_type: holding
    value_type: U_WORD    

  - platform: modbus_controller
    modbus_controller_id: versati3
    name: "versati_143_running_frequency_status"
    address: 143
    unit_of_measurement: "Hz" 
    register_type: holding
    value_type: U_WORD    
```    

## Lovelace config:
```    
type: entities
entities:
  - entity: number.versati_3_optional_e_heater_set
    name: Optional E-heater
  - entity: number.versati_5_floor_debug_segments_set
    name: Floor debug segments
  - entity: number.versati_6_floor_debug_period_1_set
    name: Floor debug period 1
  - entity: number.versati_7_delta_of_segment_temp_set
    name: Δ Segment
  - entity: number.versati_8_segment_time_set
    name: Segment time
  - entity: number.versati_13_tank_target_temp_set
    name: Tank target
  - entity: number.versati_14_eheater_temp_set
    name: E-Heater
  - entity: number.versati_15_other_switch_on_temp_set
    name: Other switch on
  - entity: number.versati_33_cool_run_time_set
    name: Cool run time
  - entity: number.versati_35_other_thermal_logic_set
    name: Other thermal logic segment
  - entity: number.versati_36_tank_heater_set
    name: Tank heater
  - entity: number.versati_37_optional_e_heater_logic_set
    name: Optional E-Heater logic
  - entity: number.versati_40_force_mode_set
    name: Force mode
  - entity: number.versati_41_air_removal_set
    name: Air removal




type: entities
entities:
  - entity: sensor.versati_118_outdoor_temp
    name: Outdoor temp
  - entity: sensor.versati_119_discharge_temp
    name: Discharge temp
  - entity: sensor.versati_120_defrost_temp
    name: Defrost temp
  - entity: sensor.versati_121_suction_temp
    name: Suction temp
  - entity: sensor.versati_122_economizer_in_temp
    name: Economizer IN temp
  - entity: sensor.versati_123_economizer_out_temp
    name: Economizer OUT temp
  - entity: sensor.versati_124_discharge_pressure_temp
    name: Discharge pressure temp
  - entity: sensor.versati_126_optional_water_sensor_temp
    name: Optional water sensor temp
  - entity: sensor.versati_128_tank_control_temp
    name: Tank control temp
  - entity: sensor.versati_129_remote_room_temp
    name: Remote room temp
  - entity: sensor.versati_131_liquid_pipe_temp
    name: Liquid pipe temp
  - entity: sensor.versati_132_thermostat_status_2
    name: Thermostat status
  - entity: sensor.versati_132_thermostat_status_number
    name: Thermostat status number
  - entity: sensor.versati_133_floor_debug
    name: Floor debug temp
  - entity: sensor.versati_134_debug_time
    name: Debug time
  - entity: sensor.versati_136_error_time_for_floor_debug
    name: Error time for floor debug
  - entity: sensor.versati_137_weather_depend_temp
    name: Weather depend temp
  - entity: sensor.versati_142_setting_fruequency_status
    name: Setting frequency
  - entity: sensor.versati_143_running_frequency_status
    name: Running frequency



type: entities
entities:
  - entity: number.versati_17_upper_at_heat_temp_set
    name: AT upper heat
  - entity: number.versati_18_lower_at_heat_temp_set
    name: AT lower heat
  - entity: number.versati_19_upper_rt_heat_temp_set
    name: RT upper heat
  - entity: number.versati_20_lower_rt_heat_temp_set
    name: RT lower heat
  - entity: number.versati_21_upper_wt_heat_temp_set
    name: WT upper heat
  - entity: number.versati_22_lower_wt_heat_temp_set
    name: WT lower heat
  - entity: number.versati_23_upper_at_cool_temp_set
    name: AT upper cool
  - entity: number.versati_24_lower_at_cool_temp_set
    name: AT lower cool
  - entity: number.versati_25_upper_rt_cool_temp_set
    name: RT upper cool
  - entity: number.versati_26_lower_rt_cool_temp_set
    name: RT lower cool
  - entity: number.versati_27_upper_wt_cool_temp_set
    name: WT upper cool
  - entity: number.versati_28_lower_wt_cool_temp_set
    name: WT lower cool
  - entity: number.versati_29_delta_cool_temp_set
    name: Δ Cool
  - entity: number.versati_30_delta_heat_temp_set
    name: Δ Heat
  - entity: number.versati_31_delta_hot_water_temp_set
    name: Δ Hot water
  - entity: number.versati_32_delta_room_temp_set
    name: Δ Room
  - entity: number.versati_10_wot_heat_temp_set
    name: WOT heat
  - entity: number.versati_9_wot_cool_temp_set
    name: WOT Cool
  - entity: number.versati_12_rt_heat_temp_set
    name: RT heat
  - entity: number.versati_11_rt_cool_temp_set
    name: RT Cool



type: entities
entities:
  - entity: sensor.versati_135_disinfection_status_2
    name: Disinfection status
  - entity: sensor.versati_117_unit_status_2
    name: Unit status
  - entity: number.versati_42_on85_off170_set
    name: ON (85) / OFF (170)
  - entity: number.versati_39_thermostat_mode_set
    name: Thermostat mode
  - entity: sensor.versati_127_water_in_pe_temp
    name: Water IN temp
  - entity: sensor.versati_125_water_out_pe_temp
    name: Water OUT PE temp
  - entity: number.versati_4_disinfection_temp_set
    name: Disinfection
  - entity: number.versati_13_tank_target_temp_set
    name: Tank target
  - entity: number.versati_16_hp_max_temp_set
    name: HP max
  - entity: number.versati_33_cool_run_time_set
    name: Cool run time
  - entity: number.versati_34_heat_run_time_set
    name: Heat run time
  - entity: number.versati_36_tank_heater_set
    name: Tank heater
  - entity: number.versati_43_power_limit_set
    name: Power limit
  - entity: number.versati_38_current_limit_value_set
    name: Current limit
  - entity: number.versati_44_error_reset_1_clear
    name: Error (1=clear)

```    
