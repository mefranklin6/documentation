# OpenAV unified endpoint definitions

This is a working document to collect and agree on URIs for OpenAV microservices

For all of these, :address is the hostname or IP address of the device (e.g. m210-video-switcher.thayer.dartmouth.edu).

## PTZAbsolute (was PantiltAbsolute)

Endpoint \- /:address/ptzabsolute

GET returns the camera’s absolute pan/tilt and zoom coordinates in the format: {“pan”: value, “tilt”: value, “zoom”: value}  
PUT accepts coordinates in the format: {“pan”: value, “tilt”: value, “zoom”: value}  
(If zoom is not included, the microservice will not move the camera in that parameter. Both pan and tilt have to be included for the microservice to move in either direction)  
Valid formats:  
{“pan”: value, “tilt”: value, “zoom”: value}  
{“pan”: value, “tilt”: value}  
{“zoom”: value}

## PTZDrive

Endpoint \- /:address/ptzdrive

PUT accepts “left”, “right”, “up”, “down”, “in”, “out”, “pan stop”, or “zoom stop” in the “action” value in the body. The directional commands move the camera until a stop command is sent.  
Speed for zoom and pan/tilt can also be specified. The range for zoom is 0-7. For pan/tilt, it is 1-14 (defined by Visca). Normalizing it to 100 seemed odd since the gaps between values would be large.  
If no speed is passed, or if it is outside of the acceptable ranges, a default will be used. 2 for zoom and 5 for pan/tilt.  
Valid formats:
{“action”: value (string),  
“zoom\_speed”: value (int),  
“pan\_tilt\_speed”: value(int)}

{“action”: value (string)}

## Focus

Endpoint \- /:address/focus

GET returns “manual” or “auto” to indicate the camera’s current focus mode.  
PUT accepts “manual”, “auto”, or “trigger” in the body to change the camera’s focus mode or to trigger the one-push autofocus adjustment.

## Preset

Endpoint \-:/address/preset

GET returns the preset number that was last called for the camera.  
PUT accepts in the body the preset number that the camera should be set to move to Ex: “1”, “2” and commands the camera to move to that preset.

## Calibrate

Endpoint \-:/address/calibrate

PUT causes the camera to calibrate its pan/tilt function.

## Power

Gets or sets power on or off for the device.

Endpoint \- /:address/power

GET returns “on” or “off” to indicate the power state of the device.  
PUT accepts “on” or “off”, makes a power state change if needed, and returns “ok” or an error.

## Volume

Gets or sets the volume for the specified input or output.   Microservice implementations all scale from 0 to 100 into whatever range the device uses for volume.

Endpoint \- /:address/volume/:name

GET returns a number in quotes that is the volume setting for the specified input or output (e.g. “65”).  If the device is unavailable to query for volume, then return 0\.  
PUT accepts "0" to "100", makes the settings, and returns "ok" or an error.  
ASK ??  

:name specifies the input or output for which we want to get or set volume.  It is a number from a device-specific list.

Note: The “name” value is ignored by some microservices such as FPDs or projectors that have only one output.

Kramer volume values:

If you specify name \>= 1000 these functions will set or get the output stage channel calculated as (name \- 1000\) instead of setting the input channel specified.  So for the VP-440H2 output with its single analog output, the output name is always 1000\.

Input and output values for VP-440H2:  
  output – 1 (HDMI OUT)  
  input – 0 (HDMI IN 1), 1 (HDMI IN 2), 2 (HDMI IN 3), 3 (HDBT IN), 4 (PC IN)

Input and output values for VP-558:  
  output: 1 (Output1),  2 (Output2), 3 (Output3) 4 (Output4)  
  input: 1 (HDMI1) 2 (HDMI2) 3 (HDMI3) 4 (HDMI4) 5 (HDMI5) 6 (HDMI6) 7 (HDBT1) 8 (HDBT2) 9 (HDBT3) 10 (HDBT4 11 (PC)

## Video route

Gets or sets what input is routed to the specified output.

Endpoint \- /:address/videoroute/:output

GET returns a number in quotes that is the input setting for the specified output (e.g. “3”).  *(Note: this does away with the ugly "x0AVx1" type return values we inherited from Atlona and BYU.)*  
PUT accepts a number, routes that input number to the specified output, and returns "ok" or an error.

:output specifies the output for which we want to get or set the input that is routed to it.  Numbering is the same as Volume, but don’t add 1000 for Kramer.

## Video route for QSC Core

QSC Core video switching is special because it supports numerous peripheral devices, each device having different types of I/O connectors, i.e. HDMI, AV over IP, and traditional AV.  As such, the endpoint’s output and input are not single numbers like the typical videoroute endpoint described in the previous section.  Instead the output and input are derived from the Q-Sys component control name, which takes the following form:  
\<*inputName*\>.select.\<*outputName*\>

An example component control name is:  
  hdmi.out.1.select.muxed.hdmi.2.led  
where “hdmi.out.1” is \<*inputName*\> and “muxed.hdmi.2.led” is \<*outputName*\>.

The endpoint for controlling this component will look something like this:  
  /vail-811d-core.openav.dartmouth.edu/videoroute/\<*deviceName*\>\_\<*outputName*\>  
and the input, which is the data in the PUT request, will be:  
  *\<inputName\>*  
where \<*deviceName\>* is a label chosen by the AV engineer, \<*outputName*\> is everything before ".select." and \<*inputName*\> is everything after ".select." in the component control name.

Here is an example SET parameter that would appear in the Vail-811d JSON config file, given that the component control name is “hdmi.out.1.select.muxed.hdmi.2.led” and the device name is “decoder-1”:  
  /vail-811d-core.openav.dartmouth.edu/videoroute/decoder-1\_hdmi.out.1 \-d "muxed.hdmi.2.led"

Another example SET given a component control name “nv21.input.select.hdmi” and a device name “decoder-2”:  
  /vail-811d-core.openav.dartmouth.edu/videoroute/decoder-2\_nv21.input \-d "hdmi"

For more information about components, see [QRC Commands](https://q-syshelp.qsc.com/Content/External_Control_APIs/QRC/QRC_Commands.htm).

## Audio and Video route

Gets or sets what audio and video inputs are routed to the specified output (audio follows video).

Endpoint \- /:address/audioandvideoroute/:output

GET returns a number in quotes that is the input setting for the specified output (e.g. “3”).

PUT accepts a number, routes that input number to the specified output, and returns "ok" or an error.

:output specifies the output for which we want to set the input that is routed to it.  Numbering is the same as Volume, but don’t add 1000 for Kramer.

## Audio and video mute

Gets or sets audio or video mute state for the specified output.

Audio endpoint \- /:address/audiomute/:output  
Video endpoint \- /:address/videomute/:output  
Both endpoint \- /:address/audioandvideomute/:output  
New for Extron DA \- Video sync endpoint \- /:address/videosyncmute/:output

GET returns the mute status for audio or video: “true” for muted, “false” for not muted.  
PUT accepts “true" or “false” in the body, makes that setting for audio or video on the specified output, and returns "ok" or an error.  The Sony Bravia microservice, NEC LCD screen panel, and maybe others in the future also accept “toggle” which will flip the mute setting from “true” to “false” or vice versa.

BUG: The OpenAV PJLink microservice uses “on” and “off” instead of “true” and “false” for the \*mute endpoints.  This is a holdover from previous endpoints that never got changed.  If we change this, we probably want to add “true” and “false” and leave “on” and “off” so we don’t have to change all the config files in sync with changing the microservice.

NOTE 1:  audiomute is not implemented for VP-558 (would be a lot of work to implement it), but we do use it for VP-440H2 in rooms that don’t have a DSP.  VP-558 is an old product, and we expect that a room with a video switcher that big will also have a DSP for audio mute control.

NOTE 2: videoandaudiomute is our response to the ugly fact that some PJLink devices throw an error when trying to mute audio or video independently because they can only mute both at once.

## Input status

Reports whether the specified input has a signal synced on it or not.  Currently used only in the context of video inputs.

Endpoint \- /:address/videoinputstatus/:input

GET returns the signal status for the specified input ("true" or "false").

## Occupancy status

Reports (in the OpenAV occupancy\_detected format) based on the status of a single input.  If there is a connection synced to the specified input, it returns true.  Otherwise it returns false.  The idea is that the system can decide not to auto shutdown if certain inputs (e.g. HDMI) have a synced connection.  *(Note: For Kramer, at least this is very similar to videoinputstatus, but it formats the response differently.)*

Endpoint \- /:address/occupancystatus/:input

GET returns {"occupancy\_detected":"false"}, or an error.

## Matrix mute

Mutes or unmutes a connection in the input x output matrix.  Initially developed for the Shure DSP.  Devices start with all points in the matrix muted.

Endpoint \- /:address/matrixmute/:input/:output

GET returns the mute value set at the specified location in the matrix.  
PUT accepts either “true” or “false” in the PUT body, attempts to set the mute value accordingly at the specified location in the matrix, and returns “ok” or an error.

## Matrix volume

Sets the audio gain at a point in the input x output matrix.  Initially developed for the Shure DSP.  

Endpoint \- /:address/matrixvolume/:input/:output

GET returns the gain set at the specified location in the matrix.  
PUT accepts a value between “0” and “100” in the PUT body, attempts to set the volume at the specified location in the matrix to that value, and returns “ok” or an error.

## Set state (e.g. for relay)

Sets the specified state to “on” or “off”.

Endpoint \- /:address/state/:output

GET returns the state value for the specified output.  
PUT accepts either “on” or “off” in the PUT body, attempts to set the specified output state, and returns “ok” or an error.

## Trigger state (e.g. for relay)

Toggles the specified output “on”, sleeps the specified amount of time, and then toggles the specified state  “off”.

Endpoint \- /:address/trigger/:output

PUT accepts the amount of time to sleep in the PUT body (an integer in a string), attempts to toggle the output, and returns “ok” or an error.

## Timed trigger state (e.g. for relay)

Toggles output1 “on”, sleeps for 1, toggles output1 “off”, sleeps for time specified in the “body” of the PUT, toggles specified outputs on at the same time, sleeps 1, toggles outputs off.

Endpoint \- /:address/timedtrigger/:output1/:output2

PUT accepts the amount of time to sleep in the PUT body (an integer in a string), attempts to toggle the output, and returns “ok” or an error

## Power Cycle (e.g. for switched PDU outlets)

Toggles the specified output “off”, sleeps the specified amount of time, and then toggles the specified output “on”.

Endpoint \- /:address/powercycle/:output

PUT accepts the amount of time to sleep in the PUT body (an integer in a string), attempts to toggle the output, and returns “ok” or an error

## Errors

Reports any errors the microservice has encountered since the last time this endpoint was called and clears the error cache.

Endpoint \- /:address/errors

GET returns any errors and a code of StatusInternalServerErrror if there were any errors.

## Possible Future Endpoints

### Passcode

Sets the passcode for a device (e.g. Auracast transmitter).

Endpoint \- /:address/passcode

GET returns the current passcode value.  
PUT accepts the value of the passcode to set and returns “ok” or an error.

### Calendar Passcode

Sets auto-generated passcodes in the device based on calendar data.  If the microservice is in this mode, GET on the /:address/passcode endpoint will return the latest passcode that it automatically set.  This automation queries the calendar data when started and then updates it every hour thereafter.  The microservice generates a new passcode at the beginning and end of each calendar entry.

This feature generates easy to type passcodes because it is intended for rapidly-changing codes that don’t need to be extremely secure.

Endpoint \- /:address/calendarpasscode

PUT accepts the URL to query for calendar data and returns “ok” or an error.  Specifying an empty URL stops the automatic passcode setting.
GET returns the calendar URL currently being used for this device.

The microservice currently parses calendar data responses in this form:

```json
{  
  "success": true,  
  "message": "ok",  
  "data": {  
    "room\_name": "c118",  
    "room\_id": 4637,  
    "events": \[  
      {  
        "name": "Thayer 197.1 \- Professional Development",  
        "type": "Lecture",  
        "start\_time": "2025-12-18T17:00:00",  
        "end\_time": "2025-12-18T18:30:00",  
        "status\_id": 9,  
        "booking\_id": 2937425,  
        "organizers": \[  
          "<redacted@dartmouth.edu>"  
        \]  
      },  
      {  
        "name": "Thayer 197.1 \- Professional Development",  
        "type": "Lecture",  
        "start\_time": "2025-12-22T17:00:00",  
        "end\_time": "2025-12-22T18:30:00",  
        "status\_id": 9,  
        "booking\_id": 2937426,  
        "organizers": \[  
          "<redacted@dartmouth.edu>"  
        \]  
      },  
      {  
        "name": "Thayer 197.1 \- Professional Development",  
        "type": "Lecture",  
        "start\_time": "2025-12-23T17:00:00",  
        "end\_time": "2025-12-23T18:30:00",  
        "status\_id": 9,  
        "booking\_id": 2937427,  
        "organizers": \[  
          "<redacted@dartmouth.edu>"  
        \]  
      }  
    \]  
  }  
}
```

The endpoint “calendarpasscode” only uses start\_time and stop\_time.  It could be extended to recognize different JSON responses and parse those too (pull requests are welcome).  

New endpoints could also share the implementation and extend it to recognize other fields and act accordingly (e.g. a “calendarrecording” endpoint could start recordings for events  with “type”: “Lecture”).  
