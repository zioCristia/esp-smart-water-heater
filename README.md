# Esp smart water heater
Esp8266 based pcb to transform your boiler in a smart one and to integrate it into your autonomous house. 
Any possibility of improvement is accepted. 

## Table of contenets
* [General info](#general-info)
* [Hardware](#hardware)
* [Software](#software)
* [Pcb](#pcb)

# General info
With this board you can: 
  - control the power on and off of the heater thanks to a relay 
  - measure the temperature of the water 
  - visualize data to an LCD 
  - have a Wi-Fi capability to integrate the heater in your home network 

I've used an esp8266 board powered from an AC-DC transformer. The temperature is measured with a digital thermometer. The LCD is a simple 8x2 display with I2C interface, used to visualize the temperature and the state of the water heater.

# Hardware
I've used an esp8266 board to reduce the costs. However also the esp32 could be used if made the necessary modifications to the [pcb](#pcb).

The relay I've used is 20A maximum which is sufficient for a 1200W water heater. Please change it in function of your power consumption. 

The transformer allows to have a more compact design of the pcb without the need to add an external one. You can even think to take the ac L and N from the boiler itself with proper care of short circuits. My connections are made with JST connector and a cable of 22awg. 

# Software
The software I’ve chosen is EspHome so that I could do Over The Air updates and integrate it to my Home Assistant server.

To the default code you just need to add the lines for the control of the relay, the temperature sensor and the display. 

Below you can find a complete exemple of the code:

### Home assistanta automations
To automate the on/off of the boiler I've created some automations which you can find [here](https://github.com/zioCristia/energy-saver-ha-automations#water-heater-automations). They're possible thanks to another [component](https://github.com/zioCristia/esp-energy-monitor) that measure the power consumption of my house as well as the solar power production.

# Pcb
The pcb design has been made with Eagle and with the intention to have all the components in just one side of the board. For a better communication you should add a 3-way JST connector also to the temperature sensor. 

I’ve added the place for an external button in case of some future implementations. 
