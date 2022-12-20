# User Manual AhoyDTU (on ESP8266)
Version #{VERSION}#
## Introduction
See the repository [README.md](Getting_Started.md)

## Setup
Assuming you have a running ahoy-dtu and you can access the setup page.
In the initial case or after click "erase settings" the fields for the inverter setup are empty.

Set at least the serial number and a name for each inverter, check "reboot after save" and click the "Save" button.

## MQTT Output
The AhoyDTU will publish on the following topics
`<TOPIC>/<INVERTER_NAME_FROM_SETUP>/ch0/#`

| Topic | Example Value | Remarks |
|---|---|---|
|U_AC | 233.300|actual AC Voltage in Volt|
|I_AC | 0.300 | actual AC Current in Ampere|
|P_AC | 71.000| actual AC Power in Watt|
|Q_AC | 21.200| actual AC reactive power in var|
|F_AC | 49.990| actual AC Frequency in Hz|
|PF_AC | 95.800| actual AC Power factor|
|Temp | 19.800|Temperature of inverter in Celsius|
|EVT | 9.000|Last Event/Alarm Message Index|
|YieldDay | 51.000|Energy converted to AC per day in Watt hours (measured on DC)|
|YieldTotal | 465.294|Energy converted to AC since reset Watt hours (measured on DC)|
|P_DC | 74.600|actual DC Power in Watt|
|Efficiency | 95.174|actual ration AC Power over DC Power in percent|
|FWVersion | 10012.000| Firmware version eg. 1.00.12|
|FWBuildYear | 2020.000| Firmware build date|
|FWBuildMonthDay | 624.000| Firmware build month and day eg. 24th of june|
|HWPartId | 100.000| Hardware Id|
|PowerLimit | 80.000|actual set point for power limit control AC active power in percent|
|LastAlarmCode | 1.000| Last Alarm Code eg. "inverter start"|

`<TOPIC>/<INVERTER_NAME_FROM_SETUP>/ch<CHANNEL_NUMBER>/#`

`<CHANNEL_NUMBER>` is in the range 1 to 4 depending on the inverter type

| Topic | Example Value | Remarks |
|---|---|---|
|U_DC | 38.900| actual DC Voltage in Volt|
|I_DC | 0.640 | actual DC current in Ampere|
|P_DC | 25.000 | actual DC power in Watt|
|YieldDay | 17.000 | Energy converted to AC per day Watt hours per module/channel (measured on DC) |
|YieldTotal | 110.819 | Energy converted to AC since reset Watt hours per module/channel (measured on DC) |
|Irradiation |5.65 | ratio DC Power over set maximum power per module/channel in percent |

## Active Power Limit via Serial / Control Page
URL: `/serial`
If you leave the field "Active Power Limit" empty during the setup and reboot the ahoy-dtu will set a value of 65535 in the setup.
That is the value you have to fill in case you want to operate the inverter without a active power limit.
If the value is 65535 or -1 after another reboot the value will be set automatically to "100" and in the drop-down menu "relative in percent persistent" will be set. Of course you can do this also by your self.

You can change the setting in the following manner.
Decide if you want to set

- an absolute value in Watt
- an relative value in percent based on the maximum Power capabilities of the inverter

and if this settings shall be

- persistent
- not persistent

after a power cycle of the inverter (P_DC=0 and P_AC=0 for at least 10 seconds)

The user has to ensure correct settings. Remember that for the inverters of 3rd generation the relative active power limit is in the range of 2% up to 100%.
Also an absolute active power limit below approx. 30 Watt seems to be not meanful because of the control capabilities and reactive power load.

## Control via MQTT

### Generic Information

The AhoyDTU subscribes on three topics `<TOPIC>/ctrl/#`, `<TOPIC>/setup` and `<TOPIC>/status`.

👆 `<TOPIC>` can be set on setup page, default is `inverter`.

👆 `<INVERTER_ID>` is the number of the specific inverter in the setup page.


### Inverter Power (On / Off)
```mqtt
<TOPIC>/ctrl/power/<INVERTER_ID>
```
with payload `1` = `ON` and `0` = `OFF`

Example:
```mqtt
inverter/ctrl/power/0     1
```

### Inverter restart
```mqtt
<TOPIC>/ctrl/restart/<INVERTER_ID>
```
Example:
```mqtt
inverter/ctrl/restart/0
```

### Power Limit relative persistent [%]

```mqtt
<TOPIC>/ctrl/limit_persistent_relative/<INVERTER_ID>
```
with a payload `[2 .. 100]`

Example:
```mqtt
inverter/ctrl/limit_persistent_relative/0     70
```

### Power Limit absolute persistent [Watts]
```mqtt
<TOPIC>/ctrl/limit_persistent_absolute/<INVERTER_ID>
```
with a payload `[0 .. 65535]`

Example:
```mqtt
inverter/ctrl/limit_persistent_absolute/0     600
```

### Power Limit relative non persistent [%]
```mqtt
<TOPIC>/ctrl/limit_nonpersistent_relative/<INVERTER_ID>
```
with a payload `[2 .. 100]`

Example:
```mqtt
inverter/ctrl/limit_nonpersistent_relative/0     70
```

### Power Limit absolute non persistent [Watts]
```mqtt
<TOPIC>/ctrl/limit_nonpersistent_absolute/<INVERTER_ID>
```
with a payload `[0 .. 65535]`

Example:
```mqtt
inverter/ctrl/limit_nonpersistent_absolute/0     600
```

## Control via REST API

### Generic Information

The rest API works with *JSON* POST requests. All the following instructions must be sent to the `/api` endpoint of the AhoyDTU.

👆 `<INVERTER_ID>` is the number of the specific inverter in the setup page.

### Inverter Power (On / Off)

```json
{
    "id": <INVERTER_ID>,
    "cmd": "power",
    "val": <VALUE>
}
```
The `<VALUE>` should be set to `1` = `ON` and `0` = `OFF`


### Inverter restart

```json
{
    "id": <INVERTER_ID>,
    "cmd": "restart"
}
```


### Power Limit relative persistent [%]

```json
{
    "id": <INVERTER_ID>,
    "cmd": "limit_persistent_relative",
    "val": <VALUE>
}
```
The `VALUE` represents a percent number in a range of `[2 .. 100]`


### Power Limit absolute persistent [Watts]

```json
{
    "id": <INVERTER_ID>,
    "cmd": "limit_persistent_absolute",
    "val": <VALUE>
}
```
The `VALUE` represents watts in a range of `[0 .. 65535]`


### Power Limit relative non persistent [%]

```json
{
    "id": <INVERTER_ID>,
    "cmd": "limit_nonpersistent_relative",
    "val": <VALUE>
}
```
The `VALUE` represents a percent number in a range of `[2 .. 100]`


### Power Limit absolute non persistent [Watts]

```json
{
    "id": <INVERTER_ID>,
    "cmd": "limit_nonpersistent_absolute",
    "val": <VALUE>
}
```
The `VALUE` represents watts in a range of `[0 .. 65535]`



### Developer Information REST API (obsolete)
In the same approach as for MQTT any other SubCmd and also MainCmd can be applied and the response payload can be observed in the serial logs. Eg. request the Alarm-Data from the Alarm-Index 5 from inverter 0 will look like this:
```json
{
    "inverter":0,
    "tx_request": 21,
    "cmd": 17,
    "payload": 5,
    "payload2": 0
}
```

## Zero Export Control (needs rework)
* You can use the mqtt topic `<TOPIC>/devcontrol/<INVERTER_ID>/11` with a number as payload (eg. 300 -> 300 Watt) to set the power limit to the published number in Watt. (In regular cases the inverter will use the new set point within one intervall period; to verify this see next bullet)
* You can check the inverter set point for the power limit control on the topic `<TOPIC>/<INVERTER_NAME_FROM_SETUP>/ch0/PowerLimit` 👆 This value is ALWAYS in percent of the maximum power limit of the inverter. In regular cases this value will be updated within approx. 15 seconds. (depends on request intervall)
* You can monitor the actual AC power by subscribing to the topic `<TOPIC>/<INVERTER_NAME_FROM_SETUP>/ch0/P_AC` 👆 This value is ALWAYS in Watt


## Firmware Version collection
Gather user inverter information here to understand what differs between some inverters.
To get the information open the URL `/api/record/info` on your AhoyDTU. The information will only be present once the AhoyDTU was able to communicate with an inverter.

| Name       | Inverter Typ | Bootloader V. | FWVersion | FWBuild [YYYY] | FWBuild [MM-DD] | HWPartId  |          |           |
| ---------- | ------------ | ------------- | --------- | -------------- | --------------- | --------- | -------- | --------- |
| DanielR92  | HM-1500      |               | 1.0.16    | 2021           | 10-12           | 100       |          |           |
| isdor      | HM-300       |               | 1.0.14    | 2021           | 12-09           | 102       |          |           |
| aschiffler | HM-1500      |               | 1.0.12    | 2020           | 06-24           | 100       |          |           |
| klahus1    | HM-300       |               | 1.0.10    | 2020           | 07-07           | 102       |          |           |
| roku133    | HM-400       |               | 1.0.10    | 2020           | 07-07           | 102       |          |           |
| eeprom23   | HM-1200      | 0.1.0         | 1.0.18    | 2021           | 12-24           | 269619201 | 18:21:00 | HWRev 256 |
| eeprom23   | HM-1200 2t   | 0.1.0         | 1.0.16    | 2021           | 10-12           | 269619207 | 17:06:00 | HWRev 256 |
| fila612    | HM-700       |               | 1.0.10    | 2021           | 11-01           | 104       |          |           |
| tfhcm      | TSUN-350     |               | 1.0.14    | 2021           | 12-09           | 102       |          |           |
| Groobi     | TSOL-M400    |               | 1.0.14    | 2021           | 12-09           | 102       |          |           |
| setje      | HM-600       |               | 1.0.08    | 2020           | 07-10           | 104       |          |           |
| madmartin  | HM-600       | 0.1.4         | 1.0.10    | 2021           | 11-01           | 104       |          |           |
| lumapu     | HM-1200      | 0.1.0         | 1.0.12    | 2020           | 06-24           |           |          |           |
| chehrlic   | HM-600       |               | 1.0.10    | 2021           | 11-01           | 104       |          |           |
| chehrlic   | TSOL-M800de  |               | 1.0.10    | 2021           | 11-01           | 104       |          |           |
| B5r1oJ0A9G | HM-800       |               | 1.0.10    | 2021           |                 | 104       |          |           |
|            |              |               |           |                |                 |           |          |           |
|            |              |               |           |                |                 |           |          |           |

## Developer Information about Command Queue
After reboot or startup the ahoy firmware it will enque three commands in the following sequence:

1. Get active power limit in percent (`SystemConfigPara = 5 // 0x05`)
2. Get firmware version (`InverterDevInform_All = 1 // 0x01`)
3. Get data (`RealTimeRunData_Debug = 11 // 0x0b`)

With the command get data (`RealTimeRunData_Debug = 11 // 0x0b`) the alarm message counter will be updated. In the initial case then aonther command is queued to get the alarm code (`AlarmData = 17 // 0x11`).

This command (`AlarmData = 17 // 0x11`) will enqued in any operation phase if alarm message counter is raised by one or greater compared to the last request with command get data (`RealTimeRunData_Debug = 11 // 0x0b`)

In case all commands are processed (`_commandQueue.empty() == true`) then as a default command the get data (`RealTimeRunData_Debug = 11 // 0x0b`) will be enqueued.

In case a Device Control command (Power Limit, Off, On) is requested via MQTT or REST API this request will be send before any other enqueued command.
In case of a accepted change in power limit the command  get active power limit in percent (`SystemConfigPara = 5 // 0x05`) will be enqueued. The acceptance is checked by the reponse packets on the devive control commands (tx id 0x51 --> rx id 0xD1) if in byte 12 the requested sub-command (eg. power limit) is present.

## How To
### Sunrise & Sunset
In order to display the sunrise and sunset on the start page, the location coordinates (latitude and longitude) must be set in decimal in the setup under Sunrise & Sunset. If the coordinates are set, the current sunrise and sunset are calculated and displayed daily.
If this is set, you can also tick "disable night communication", then sending to the inverter is switched off outside of the day (i.e. at night), this avoids unnecessary communication attempts and thus also the incrementing of "RX no anwser".
Here you can get easy your GeoLocation: [https://www.mapsdirections.info/en/gps-coordinates.html](https://www.mapsdirections.info/en/gps-coordinates.html)
### Commands and informations
Turn On - turns on the inverter/feeder (LED flashes green if there is no error)
Turn Off - switches off the inverter/feeder (LED flashes fast red), can be switched on again with Turn On or by disconnecting and reconnecting the DC voltage
Restart - restarts the microcontroller in the inverter, which deletes the error memory and the YieldDay values, feed-in stops briefly and starts with the last persistent limit
Send Power Limit:
- A limitation of the AC power can be sent in relative (in %) or in absolute (Watt).
- It can be set to a different value non-persistently (temporarily) at any time (regardless of what you have set for persistent), this should be normal in order to limit the power (zero feed/battery control) and does not damage the EEPROM in the WR either.
- With persistent you send a saving limit (only use it seldom, otherwise the EEPROM in the HM can break!), This is then used as the next switch-on limit when DC comes on, i.e. when the sun comes up early or the WR on batteries is switched on the limit is applied immediately when sending, like any other, but it is stored in the EEPROM of the WR.
- A persistent limit is only needed if you want to throttle your inverter permanently or you can use it to set a start value on the battery, which is then always the switch-on limit when switching on, otherwise it would ramp up to 100% without regulation, which is continuous load is not healthy.
- You can set a new limit in the turn-off state, which is then used for on (switching on again), otherwise the last limit from before the turn-off is used, but of course this only applies if DC voltage is applied the whole time.
- If the DC voltage is missing for a few seconds, the microcontroller in the inverter goes off and forgets everything that was temporary/non-persistent in the RAM: YieldDay, error memory, non-persistent limit.