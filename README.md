# Esp smart water heater
Esp8266 based pcb to transform your boiler into a smart one and to integrate it into your autonomous home. 
Any possibility of improvement is accepted.

You can find on Youtube an italian [video tutorial](https://youtu.be/jwHNBpo81dQ) about this project.

## Table of contents
* [General info](#general-info)
* [Hardware](#hardware)
* [Software](#software)
* [Pcb](#pcb)
* [Other usefull things about](#other-usefull-things-about)
  * [Custom software component power on time](#custom-software-component-power-on-time)
  * [Home assistant automations](#home-assistant-automations)

# General info
With this board you can: 
  - control the power on and off of the heater thanks to a relay 
  - measure the temperature of the water 
  - visualize data to an LCD 
  - have a Wi-Fi capability to integrate the heater in your home network 

I've used an esp8266 board powered from an AC-DC transformer. The temperature is measured with a digital thermometer. The LCD is a simple 8x2 display with I2C interface, used to visualize the temperature and the state of the water heater.

The total cost is less than 20€, or 30€ if you also buy the pcb board.

![alt text](/images/water-heater.png)

# Hardware
The module is composed by:
* esp8266
* 8x2 lcd display with i2c interface
* DS18B20 temperature Sensor
* 20A external relay module
* HLk-5M05 transformer
* 4.7k ohm resistor
* 10k ohm resistor
* 10uF capacitor
* 2N3904 transistor
* external button

The temperature sensor, the relay, the lcd display and the button are external components, connected to the pcb board thanks to 2.54mm pins or directly soldered in the board. The esp is connected thanks to female pins which allow a rapid change of the esp in case of failures. You can think to use jst connectors to avoid orientation problems.

I've used an esp8266 board to reduce the costs. However also the esp32 could be used if made the necessary modifications to the [pcb](#pcb).

The relay I've used is 20A maximum which is sufficient for a 1200W water heater. Please change it in function of your power consumption. 

The transformer allows a more compact design of the pcb without the need to add an external one. You can even think to take the ac L and N from the boiler itself with proper care of short circuits. My connections are made with JST connector and a cable of 22awg.

You can easily change the 8x2 lcd with any other i2c display just by modifying correspondently the code.

![alt text](/images/pcb-completed.jpg)
![alt text](/images/pcb-completed-2.jpg)

# Software
The software I’ve chosen is EspHome so that I could do Over The Air updates and integrate it to my Home Assistant server.

To the default code you just need to add the lines for the control of the relay, the temperature sensor and the display. 

Below you can find the parts to add to the default EspHome code or you can find a complete example of the code [here](/water-heater.yaml.example):

```
# DS18B20 Temperature Sensor
dallas:
  - pin: 
      number: 2
      mode: 
        output: False
        analog: False
        input: True
        open_drain: False
        pullup: False
        pulldown: False
      inverted: False
    
switch:
  - platform: gpio
    name: "Water heater switch"
    pin: GPIO12
    id: rele_boiler
    restore_mode: ALWAYS_OFF

sensor:
  - platform: dallas
    address: 0xd601131bd9e6e328
    name: "Water heater temperature"
    #update_interval: update_time
    id: temperature_sensor
    
i2c:
  sda: GPIO4
  scl: GPIO5
  
display:
  - platform: lcd_pcf8574
    dimensions: 16x2
    id: mydisplay_b
    address: 0x27
    lambda: |-
      static bool bDisplay = true;
      auto now = id(my_time).now(); // local time
      if ((now.hour >= 23 || now.hour < 7) && bDisplay) {
        id(mydisplay_b).no_backlight();;
        bDisplay = false;
      } else if ((now.hour >= 7 && now.hour < 23) && !bDisplay) {
        id(mydisplay_b).backlight();;
        bDisplay = true;
      }
      //it.print("Hello world!");
      it.printf(0,0,"Stat %s", id(rele_boiler).state ? "ON" : "OFF");
      it.printf(0,1,"T %4.1f C", id(sensore_temperatura).state);
      
time:
- platform: sntp
  id: my_time
```

This is the home assistant interface I have created for the water heater:

![alt text](/images/ha-water-heater-view.jpg)

# Pcb
The pcb design has been made with Eagle and with the intention to have all the components in just one side of the board. For better communications you should add a 3-way JST connector also to the temperature sensor. 

I’ve added the place for an external button in case of some future implementations.

In case you will need some libraries to modify the files, just tell me.

You can find the Eagle's files [here](/smart_water_heater).

![alt text](/images/circuit.png)
![alt text](/images/pcb.png)

# Other usefull things about
## Home assistant automations
To automate the on/off of the boiler I've created some automations which you can find [here](https://github.com/zioCristia/energy-saver-ha-automations#water-heater-automations). They're possible thanks to another [component](https://github.com/zioCristia/esp-energy-monitor) that measure the power consumption of my house as well as the solar power production.

## Custom software component power on time
I've added to my Home Assistant configuration a custom component to monitor the time the water heater has been on. This does not exactly correspond to the time when the water heater consumes 1200W because the heating element turns on and off according to the temperature around it. 
So, especially when we have reached the final temperature, the boiler is on but not active.

Here it is the code:
```
- platform: history_stats
    name: Power on time wh
    entity_id: switch.switch_boiler
    state: "on"
    type: time
    start: "{{ now().replace(hour=0, minute=0, second=0) }}"
    end: "{{ now() }}"
```

