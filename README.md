# <p align="center">Farm Data Relay System
##### <p align="center">[***In loving memory of Gay Holman, an extraordinary woman.***](https://www.facebook.com/CFECI/posts/2967989419953119) #####

The Farm Data Relay System is an easy way to collect data from remote sensors without relying on WiFi. It is based around the ESP-NOW protocol, which is readily available on ESP32 and ESP8266 microcontroller boards. The system can be used to collect and transmit sensor data in situations where it would be too difficult or energy-consuming to provide full WiFi coverage. 

Using an assigned MAC address scheme allows for the whole system to be configured by setting just a handful of values in code. Every wireless gateway is assigned a single-byte identifier, known as the UNIT_MAC. This, along with a set, 5-byte prefix is assigned to the MAC address of the ESP's radio at boot. 

Gateways can be configured to send an ESP-NOW transmission to the serial port using JSON, another ESP-NOW gateway, or broadcast it via LoRa PHY. They can also be configured to transmit incoming serial packets over MQTT, ESP-NOW, or LoRa PHY.

## Getting Started
To use FDRS with Node-Red and MQTT you'll need two ESP devices (gateways) that are connected via UART, plus additional ESP or LoRa devices with sensors connected.

  **Sensors** can use the [“fdrs_sensor.h”](https://github.com/timmbogner/Farm-Data-Relay-System/tree/main/FDRS_Sensor2000) file to set up the device and send FDRS data via ESP-NOW and/or LoRa. 
  
The two **gateways** are programmed using the instructions [found with the Gateway2000 sketch](https://github.com/timmbogner/Farm-Data-Relay-System/tree/main/FDRS_Gateway2000).
 
The Node-RED **front-end** can be set up with these nodes to format and send the data to InfluxDB:
  ```
[{"id":"66d36c0f.cedf94","type":"influxdb out","z":"d7346a99.716ef8","influxdb":"905dd357.34717","name":"","measurement":"DataReading","precision":"","retentionPolicy":"","database":"database","precisionV18FluxV20":"ms","retentionPolicyV18Flux":"","org":"the_organization","bucket":"bkt","x":760,"y":240,"wires":[]},{"id":"93e9822a.3ad59","type":"mqtt in","z":"d7346a99.716ef8","name":"","topic":"esp/fdrs","qos":"2","datatype":"auto","broker":"c513f7e9.760658","nl":false,"rap":true,"rh":0,"x":170,"y":220,"wires":[["d377f9e0.faef98"]]},{"id":"d377f9e0.faef98","type":"json","z":"d7346a99.716ef8","name":"","property":"payload","action":"obj","pretty":false,"x":290,"y":220,"wires":[["ca383562.4014e8"]]},{"id":"ca383562.4014e8","type":"split","z":"d7346a99.716ef8","name":"","splt":"\\n","spltType":"str","arraySplt":1,"arraySpltType":"len","stream":false,"addname":"","x":410,"y":220,"wires":[["6eaba8dd.429e38"]]},{"id":"6eaba8dd.429e38","type":"function","z":"d7346a99.716ef8","name":"Fields","func":"msg.payload = [{\n    data: msg.payload.data\n},{\n    id: msg.payload.id,\n    type: msg.payload.type\n}]\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":530,"y":220,"wires":[["296d0f4b.37a46","66d36c0f.cedf94"]]},{"id":"296d0f4b.37a46","type":"debug","z":"d7346a99.716ef8","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","statusVal":"","statusType":"auto","x":670,"y":200,"wires":[]},{"id":"905dd357.34717","type":"influxdb","hostname":"127.0.0.1","port":"8086","protocol":"http","database":"database","name":"","usetls":false,"tls":"d50d0c9f.31e858","influxdbVersion":"2.0","url":"http://localhost:8086","rejectUnauthorized":true},{"id":"c513f7e9.760658","type":"mqtt-broker","name":"","broker":"localhost","port":"1883","clientid":"","usetls":false,"protocolVersion":"4","keepalive":"60","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","birthMsg":{},"closeTopic":"","closeQos":"0","closePayload":"","closeMsg":{},"willTopic":"","willQos":"0","willPayload":"","willMsg":{},"sessionExpiry":""},{"id":"d50d0c9f.31e858","type":"tls-config","name":"","cert":"","key":"","ca":"","certname":"","keyname":"","caname":"","servername":"","verifyservercert":false}]
```
The following nodes do the same as above, but I added the ability to add an alias to each sensor ID via the dashboard using globals. It could use improvement, but might be helpful to some:
 ```
[{"id":"66d36c0f.cedf94","type":"influxdb out","z":"d7346a99.716ef8","influxdb":"905dd357.34717","name":"","measurement":"DataReading","precision":"","retentionPolicy":"","database":"database","precisionV18FluxV20":"ms","retentionPolicyV18Flux":"","org":"the_organization","bucket":"bkt","x":1160,"y":300,"wires":[]},{"id":"93e9822a.3ad59","type":"mqtt in","z":"d7346a99.716ef8","name":"","topic":"esp/fdrs","qos":"2","datatype":"auto","broker":"c513f7e9.760658","nl":false,"rap":true,"rh":0,"x":270,"y":300,"wires":[["d377f9e0.faef98"]]},{"id":"d377f9e0.faef98","type":"json","z":"d7346a99.716ef8","name":"","property":"payload","action":"obj","pretty":false,"x":390,"y":300,"wires":[["ca383562.4014e8"]]},{"id":"ca383562.4014e8","type":"split","z":"d7346a99.716ef8","name":"","splt":"\\n","spltType":"str","arraySplt":1,"arraySpltType":"len","stream":false,"addname":"","x":530,"y":300,"wires":[["f9edcceb.225e8"]]},{"id":"6eaba8dd.429e38","type":"function","z":"d7346a99.716ef8","name":"Fields","func":"msg.payload = [{\n    data: msg.payload.data\n},{\n    id: msg.payload.id,\n    type: msg.payload.type,\n    alias: msg.payload.alias,\n    model: msg.payload.model,\n    indoor: msg.payload.indoor,\n    outdoor: msg.payload.outdoor\n}]\n\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":810,"y":300,"wires":[["66d36c0f.cedf94","296d0f4b.37a46"]]},{"id":"296d0f4b.37a46","type":"debug","z":"d7346a99.716ef8","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","targetType":"msg","statusVal":"","statusType":"auto","x":1030,"y":240,"wires":[]},{"id":"edec396f.607608","type":"ui_form","z":"d7346a99.716ef8","name":"","label":"Enter ID and Alias","group":"361e2c11.6c24e4","order":0,"width":0,"height":0,"options":[{"label":"Device ID","value":"id","type":"number","required":true,"rows":null},{"label":"Desired Alias","value":"alias","type":"text","required":true,"rows":null},{"label":"Sensor Type","value":"model","type":"text","required":false,"rows":null},{"label":"Indoor","value":"indoor","type":"checkbox","required":false,"rows":null},{"label":"Outdoor","value":"outdoor","type":"checkbox","required":false,"rows":null}],"formValue":{"id":"","alias":"","model":"","indoor":false,"outdoor":false},"payload":"","submit":"submit","cancel":"cancel","topic":"topic","topicType":"msg","splitLayout":"","x":310,"y":380,"wires":[["24844b21.4e0f24"]]},{"id":"391507dd.567968","type":"debug","z":"d7346a99.716ef8","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","statusVal":"","statusType":"auto","x":810,"y":240,"wires":[]},{"id":"f9edcceb.225e8","type":"function","z":"d7346a99.716ef8","name":"","func":"var atts = global.get(\"attribute_list[\" + msg.payload.id + \"]\" );\nif(atts !== undefined){\nmsg.payload.alias = atts.alias;\nmsg.payload.model = atts.model;\nmsg.payload.indoor = atts.indoor;\nmsg.payload.outdoor = atts.outdoor;\n}\n\nreturn msg;\n","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":660,"y":300,"wires":[["391507dd.567968","6eaba8dd.429e38"]]},{"id":"c27b4f12.71867","type":"debug","z":"d7346a99.716ef8","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":670,"y":380,"wires":[]},{"id":"24844b21.4e0f24","type":"change","z":"d7346a99.716ef8","name":"","rules":[{"t":"set","p":"attribute_list[msg.payload.id].alias","pt":"global","to":"payload.alias","tot":"msg"},{"t":"set","p":"attribute_list[msg.payload.id].model","pt":"global","to":"payload.model","tot":"msg"},{"t":"set","p":"attribute_list[msg.payload.id].indoor","pt":"global","to":"payload.indoor","tot":"msg"},{"t":"set","p":"attribute_list[msg.payload.id].outdoor","pt":"global","to":"payload.outdoor","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":520,"y":380,"wires":[["c27b4f12.71867"]]},{"id":"905dd357.34717","type":"influxdb","hostname":"127.0.0.1","port":"8086","protocol":"http","database":"database","name":"","usetls":false,"tls":"d50d0c9f.31e858","influxdbVersion":"2.0","url":"http://localhost:8086","rejectUnauthorized":true},{"id":"c513f7e9.760658","type":"mqtt-broker","name":"","broker":"localhost","port":"1883","clientid":"","usetls":false,"protocolVersion":"4","keepalive":"60","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","birthMsg":{},"closeTopic":"","closeQos":"0","closePayload":"","closeMsg":{},"willTopic":"","willQos":"0","willPayload":"","willMsg":{},"sessionExpiry":""},{"id":"361e2c11.6c24e4","type":"ui_group","name":"Configuration","tab":"8a89d1e.eb12c3","order":1,"disp":true,"width":"10","collapse":false},{"id":"d50d0c9f.31e858","type":"tls-config","name":"","cert":"","key":"","ca":"","certname":"","keyname":"","caname":"","servername":"","verifyservercert":false},{"id":"8a89d1e.eb12c3","type":"ui_tab","name":"Farm Data Relay System","icon":"dashboard","order":1,"disabled":false,"hidden":false}]
 ```
 
![Basic](/FDRS_Gateway2000/Basic_Setup.png)
![Advanced](/FDRS_Gateway2000/Advanced_Setup.png)
### Sensors
```
typedef struct DataReading {
  float d;
  uint16_t id;
  uint8_t t;
} DataReading;
```
Each sensor in the system sends its data over ESP-NOW as a float 'd' inside of a structure called a DataReading. Its global sensor address is represented by an integer 'id', and each type is represented by a single byte 't'.  If a sensor module needs to send multiple types of readings, then they are sent in an array of DataReadings. A single DataReading.id may have readings of multiple types associated with it.
## Future Plans
 A few things that I intend to add are:
- A way for the sensors to seek out a nearby gateway and pair with it. 
- The ability to send data in reverse, and have nodes to control irrigation, ventilation, and LED illumination. This will be acheived using a similar pairing technique to the above.
- More sensor sketches! If you have designed any open source sensor modules for ESP32 or 8266, please contact me and I will provide a link and/or code for your device in this repo.
- Support for several new devices and protocols: ethernet, nRF24L01, 4G LTE, and the E5 LoRa module from Seeed Studio.
- Some ability to compress data for more efficient LoRa usage and to avoid using floats. Better documentation/development of the DataReading 'type' attribute will come with this. 
 
## Thank you
...very much for checking out my project! I truly appreciate everyone across the net who has reached out with assistance and encouragement. If you have any questions, comments, or issues please feel free to contact me at timmbogner@gmail.com.

If you have any extra money, please consider [supporting me](https://www.buymeacoffee.com/TimmB). I'm a farmer by occupation, and donations would help me to spend more time developing farm gadgets. 

Development of this project would not have been possible without the *gracious* support of my former employer, [**Sola Gratia Farm**](https://www.solagratiacsa.com/) of **Urbana, IL, USA**.  Sola Gratia is a community-based farm dedicated to growing high-quality produce and sharing it with those in need. Thank you!
  
A huge thanks to the ever-instructional [**Andreas Spiess**](https://www.youtube.com/channel/UCu7_D0o48KbfhpEohoP7YSQ).
  
[**Random Nerd Tutorials**](https://randomnerdtutorials.com/) is also an indispensable source of ESP knowledge.
