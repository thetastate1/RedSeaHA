# RedSeaHA
Use RedSea ReefBeat devices in Home Assistant.

A node-red flow to integrate RedSea Pumps, Skimmers, Dosers, and Wavemakers. 

Lights should be straight forward but I don't have any to test.

This integration uses local and cloud http REST integration allowing you to monitor and control devices remotely using Home Assistant.

The cloud API is used to get information about the devices associated with the aquarium to create sensors and controls.

HA can control:
- Aquarium feed mode can be started and stopped
- ReefMat can be manually advanced
- ReefWave change forward and reverse intensity/time and pulse time
- ReefRun pumps change speed


## DEVICES AND ENTITIES CREATED FROM NODE-RED

<img width="825" alt="Screenshot 2025-03-07 at 1 51 19 pm" src="https://github.com/user-attachments/assets/e10d09ee-84e2-4bbe-9615-649858f34ddb" />

<img width="506" alt="Screenshot 2025-03-07 at 1 52 21 pm" src="https://github.com/user-attachments/assets/28528ddf-ebd5-42c2-86fd-a428cacfeef2" />

<img width="512" alt="Screenshot 2025-03-07 at 1 52 39 pm" src="https://github.com/user-attachments/assets/64a37831-4201-401c-b36f-84e8e60f5120" />

<img width="508" alt="Screenshot 2025-03-07 at 1 52 57 pm" src="https://github.com/user-attachments/assets/f7397f68-3007-4336-a027-0111ef82748b" />

<img width="511" alt="Screenshot 2025-03-07 at 1 53 17 pm" src="https://github.com/user-attachments/assets/52b7808e-50c9-4e41-ab0c-ba8cd2256b87" />

<img width="515" alt="Screenshot 2025-03-07 at 1 53 29 pm" src="https://github.com/user-attachments/assets/c39139e4-f506-4cc9-b4dd-817fdc72a5f6" />



## INSTALLATION

### PREREQUISITES
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

### INSTALL FLOW

5. Download or copy and paste the flow from this repository into a new node-red flow using the import menu

### CONFIGURE FLOW

6. Choose your Home Assistant server node in each of the blue Sensor Nodes -> Entity config -> Edit button -> Server
7. Enter your ReefBeat credentials in the function: *"Do this First: Enter ReefBeat Credentials Here Then Deploy Flow Then Trigger"*

### DEPLOY FLOW

8. Deploy the node-red changes using the Deploy button

### INITIATE CLOUD FLOW

8. Use the trigger: *"Trigger This Second"* to initiate connection for the first time
9. Use the trigger: *"This Third"* to populate aquarium info

### CONFIGURE HA

9. The devices and entities will be exposed under the Node-RED Companion integration in HA. Add these to your dashboard as required.
10. Create automations (e.g. ReefMat roll replacement, Doser suppliment replacement, Feed mode for pumps)


## TODO

1. Add overskimming and level sensors for Pump/Skimmer
2. Lights
3. Figure out what "st" and "pd" is for in setting pump values
6. Add manual doser capability and additional sensors
7. ATO
8. Perhaps there is a more efficient way to create HA sensors dynamically from a JS function call in node-red rather than creating a node for each sensor

## PAYLOADS

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
More payloads are included in the comment nodes of the flow

Keen to collaborate and work out more endpoints, refine flow, optimise install, test lights etc. 
