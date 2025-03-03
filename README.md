# RedSeaHA
Use RedSea devices in Home Assistant

A node-red flow to integrate RedSea Pumps, Skimmers, Dosers, and Wavemakers. 

Lights likely possible using the same flow but need someone with lights to test.

This integration uses local http REST integration allowing you to monitor and control devices remotely using Home Assistant.

INSTALLATION

PREREQUISITES
1. Node-red
   https://nodered.org
2. Node red palatte
   https://flows.nodered.org/node/node-red-contrib-home-assistant-websocket
   Use node-red palatte manager to search for and install: node-red-contrib-home-assistant-websocket
   Follow the documentation to configure the connection to Home Assistant:
   https://zachowj.github.io/node-red-contrib-home-assistant-websocket/
   The key part here is entering the HA URL and Access Token
4. Home Assistant HACS node-red companion integration
   Use HACS to find and install Node-RED Companion
   https://zachowj.github.io/node-red-contrib-home-assistant-websocket/guide/custom_integration/
5. IP addresses or local host names of your RedSea devices
   Note that if using IP addresses, set up the devices as static / reserved IP addresses using your router configuration

INSTALL FLOW


5. Download or copy and paste the flow from this repository into a new node-red flow using the import menu

CONFIGURE FLOW

6. Choose your Home Assistant server node in each of the blue Sensor Nodes -> Entity config -> Edit button -> Server
7. Change the IP addresses / hostnames in each of the http request nodes to the corresponding RedSea devices. Note that no authentication is required on local devices.

DEPLOY FLOW

8. Deploy the node-red changes using the Deploy button

TEST FLOW

8. Use the test buttons to check connectivity and validate flow configuration

CONFIGURE HA

9. The devices and entities will be exposed under the Node-RED Companion integration in HA. Add these to your dashboard as required.


TODO

1. Add overskimming and level sensors for Skimmer
2. Work out payloads for Wavemaker. Currently have on found a mode value that can be used to monitor and change the mode.
3. Add manual advance trigger for ReefMat
4. Lights
5. Figure out what "st" and "pd" is for in setting pump values

PAYLOADS

Pumps

GET http://PumpControllerIP/dashboard
```json
msg.payload = {
   "mode":"auto",
   "is_internet_connected":true,
   "battery_level":"high",
   "time_error":false,
   "restore_settings":true,
   "linked":false,
   "synced":true,
   "ec_sensor_connected":false,
   "pump_1":{
      "name":"Chiller",
      "type":"return",
      "model":"return-6",
      "state":"operational",
      "reconnect_pump":false,
      "sensor_controlled":false,
      "missing_pump":false,
      "missing_sensor":false,
      "schedule_enabled":true,
      "intensity":50,
      "pulse":0,
      "temperature":29.268291473388672},
   "pump_2":{
      "name":"",
      "type":"unknown",
      "model":"unknown",
      "state":"operational",
      "reconnect_pump":false,
      "sensor_controlled":false,
      "missing_pump":false,
      "missing_sensor":false,
      "schedule_enabled":false,
      "intensity":0,
      "pulse":0,
      "temperature":0}
}
```


This PUT changes the pump speed:

PUT http://PumpControllerIP/dashboard
```json

var pumpSpeed = msg.payload
msg.payload = {
    "pump_1": {
        "schedule": [
            {
                "ti": pumpSpeed,
                "st": 0,
                "pd": 0
            }
        ]
    }
}
```

ReefMat

