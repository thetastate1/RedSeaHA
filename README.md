# RedSeaHA
Use RedSea devices in Home Assistant

A node-red flow to integrate RedSea Pumps, Skimmers, Dosers, and Wavemakers. 

Lights likely possible using the same flow but need someone with lights to test.

This integration uses local http REST integration allowing you to monitor and control devices remotely using Home Assistant.

DEVICES AND ENTITIES CREATED FROM NODE-RED
<img width="793" alt="Screenshot 2025-03-04 at 9 48 25 am" src="https://github.com/user-attachments/assets/c0f5b465-22a3-499f-bc8e-d7a6bcb30e81" />

<img width="775" alt="Screenshot 2025-03-04 at 9 49 05 am" src="https://github.com/user-attachments/assets/5a794cd8-0f0e-490a-91e3-eeb3a0e660ac" />

<img width="786" alt="Screenshot 2025-03-04 at 9 49 23 am" src="https://github.com/user-attachments/assets/42d06991-fba9-41ca-a567-131f63726c81" />

<img width="473" alt="Screenshot 2025-03-04 at 9 49 42 am" src="https://github.com/user-attachments/assets/a404e926-d82d-41cc-bde7-61470ca89ae1" />

<img width="783" alt="Screenshot 2025-03-04 at 9 49 58 am" src="https://github.com/user-attachments/assets/bc325b85-100c-416a-880d-25f8dae5eb56" />

<img width="784" alt="Screenshot 2025-03-04 at 9 50 16 am" src="https://github.com/user-attachments/assets/3d850c91-0410-424f-901b-996b92b7a84e" />



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
10. Create automations (e.g. ReefMat roll replacement, Doser suppliment replacement, Feed mode for pumps)


TODO

1. Add overskimming and level sensors for Pump/Skimmer
2. Work out payloads for Wavemaker. Currently have on found a mode value that can be used to monitor and change the mode.
3. Add manual advance trigger for ReefMat
4. Lights
5. Figure out what "st" and "pd" is for in setting pump values
6. Add manual doser capability and additional sensors
7. ATO
8. Perhaps there is a more efficient way to create HA sensors dynamically from a JS function call in node-red rather than creating a node for each sensor

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

Returns:
```json
msg.payload = {
"success":true,
"message":"all pump settings updated with success"
}
```


ReefMat


GET http://ReefMatIP/dashboard

Returns:
```json
msg.payload = {
"mode":"auto",
"is_internet_connected":true,
"is_ec_sensor_connected":true,
"unclean_sensor":false,
"auto_advance":true,
"is_advancing":false,
"last_advance_cause":"ec_sensor",
"roll_level":"full",
"days_till_end_of_roll":9,
"internal_ec_average":0,
"external_ec_average":0,
"setup_date":"2024-11-17T10:02:12Z",
"cumulative_steps":9658,
"device_setup_date":1660259660,
"lifetime_steps":107075,
"today_usage":8.9,
"daily_average_usage":27,
"total_usage":2549.5,
"remaining_length":250.3,
"material":{
   "name":"28 Meter",
   "external_diameter":10.1,
   "thickness":0.0237,
   "is_partial":false}
}
```


Dosers
GET http://DoserIP/head/1/settings

Returns:
```json
msg.payload ={
"slm":true,
"container_volume":545.5129,
"schedule_enabled":true,
"state":"on",
"show":true,
"dc":true,
"last_calibrated":1656594173,
"today_auto_dosed_volume":45.9,
"is_food_head":false,
"food_delay":0,
"vps":0.056061,
"entire_dd":true,
"enable_ratio":true,
"supplement":{
   "uid":"76830db3-a0bd-459a-9974-76a57d026893",
   "name":"KH/Alkalinity (Foundation B)",
   "display_name":"KH",
   "short_name":"KH",
   "brand_name":"Red Sea",
   "type":"null",
   "concentration":0,
   "made_by_redsea":true,
   "is_brand_editable":false,
   "is_supplement_editable":false,
   "is_name_editable":false},
   "schedule":{
      "type":"hourly",
      "dd":100,
      "days":[1,2,3,4,5,6,7],
      "mode":"regular",
      "min":0},
   "recalibration_required":false,
   "manual_dose_scheduled":false
}
```

WaveMaker

GET http://WaveMakerIP/mode

Returns:
```json
msg.payload = {auto}
```

POST http://WaveMakerIP/mode

with:
```json
msg.payload = {
"mode": "auto"
}
```
Returns:
```json
msg.payload = {
"success":true,
"message":"auto mode enabled successfully"
}
```

Keen to collaborate and work out more endpoints, refine flow, optimise install, test lights etc. 
