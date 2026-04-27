# Device Microservice Unified Endpoint Definitions

This is a working document to collect and agree on URIs for OpenAV device microservices

## Usage

### Using a device microservice directly

You can call an exposed microservice endpoint directly using the following format:

`http://<microserviceAddr>/<protocol>|<user>:<pw>@<deviceAddr>:<devicePort>`

#### Params

- `microserviceAddr`: Mandatory IP or hostname of the exposed microservice

- `protocol`: New in [microservice-framework](https://github.com/dartmouth-openav/microservice-framework) version 1.4.10, specify an optional protocol such as SSH or Telnet. See [Pull Request 5](https://github.com/Dartmouth-OpenAV/microservice-framework/pull/5) for complete details.

- `user`: optional username for the device

- `pw`: optional password for the device

- `deviceAddr`: Mandatory device IP address or hostname

- `devicePort`: Optional port for device communication

Full Example: `http://192.168.1.5/telnet|user:secretpw@192.168.1.10:23/`

Minimal Example: `http://192.168.1.5/:@192.168.1.10`

### Using an endpoint in an OpenAV JSON configuration

For all of the below, `:address` is the hostname or IP address of the device (e.g. `m210-video-switcher.thayer.dartmouth.edu`).

```json
"get": [
  "dartmouth-openav/microservice-pjlink:current/user:pw@192.168.254.3/power"
],
```

## PTZAbsolute

**Endpoint**  
`/:address/ptzabsolute`

**GET**  
Returns the camera’s absolute pan/tilt and zoom coordinates in the format: `{"pan": value, "tilt": value, "zoom": value}`

**PUT**  
Accepts coordinates in the format: `{"pan": value, "tilt": value, "zoom": value}`

**Notes**  
(If zoom is not included, the microservice will not move the camera in that parameter. Both pan and tilt have to be included for the microservice to move in either direction)

Valid formats:

```text
{"pan": value, "tilt": value, "zoom": value}
{"pan": value, "tilt": value}
{"zoom": value}
```

## PTZDrive

**Endpoint**  
`/:address/ptzdrive`

**PUT**  
Accepts `"left"`, `"right"`, `"up"`, `"down"`, `"in"`, `"out"`, `"pan stop"`, or `"zoom stop"` in the `"action"` value in the body. The directional commands move the camera until a stop command is sent.

**Notes**  
Speed for zoom and pan/tilt can also be specified. The range for zoom is `0-7`. For pan/tilt, it is `1-14` (defined by Visca). Normalizing it to `100` seemed odd since the gaps between values would be large.  
If no speed is passed, or if it is outside of the acceptable ranges, a default will be used. `2` for zoom and `5` for pan/tilt.

Valid formats:

```text
{"action": value (string),
"zoom_speed": value (int),
"pan_tilt_speed": value(int)}

{"action": value (string)}
```

## Focus

**Endpoint**  
`/:address/focus`

**GET**  
Returns `"manual"` or `"auto"` to indicate the camera’s current focus mode.

**PUT**  
Accepts `"manual"`, `"auto"`, or `"trigger"` in the body to change the camera’s focus mode or to trigger the one-push autofocus adjustment.

## Preset

**Endpoint**  
`:/address/preset`

**GET**  
Returns the preset number that was last called for the camera.

**PUT**  
Accepts in the body the preset number that the camera should be set to move to Ex: `"1"`, `"2"` and commands the camera to move to that preset.

## Calibrate

**Endpoint**  
`:/address/calibrate`

**PUT**  
Causes the camera to calibrate its pan/tilt function.

## Power

**Description**  
Gets or sets power on or off for the device.

**Endpoint**  
`/:address/power`

**GET**  
Returns `"on"` or `"off"` to indicate the power state of the device.

**PUT**  
Accepts `"on"` or `"off"`, makes a power state change if needed, and returns `"ok"` or an error.

## Volume

**Description**  
Gets or sets the volume for the specified input or output. Microservice implementations all scale from `0` to `100` into whatever range the device uses for volume.

**Endpoint**  
`/:address/volume/:name`

**GET**  
Returns a number in quotes that is the volume setting for the specified input or output (e.g. `"65"`). If the device is unavailable to query for volume, then return `0`.

**PUT**  
Accepts `"0"` to `"100"`, makes the settings, and returns `"ok"` or an error.

**Notes**  
`:name` specifies the input or output for which we want to get or set volume. It is a number from a device-specific list.

Note: The `"name"` value is ignored by some microservices such as FPDs or projectors that have only one output.

## Video route

**Description**  
Gets or sets what input is routed to the specified output.

**Endpoint**  
`/:address/videoroute/:output`

**GET**  
Returns a number in quotes that is the input setting for the specified output (e.g. `"3"`).

**PUT**  
Accepts a number, routes that input number to the specified output, and returns `"ok"` or an error.

**Notes**  
`:output` specifies the output for which we want to get or set the input that is routed to it. Numbering is the same as Volume.

## Video route for QSC Core

**Description**  
QSC Core video switching is special because it supports numerous peripheral devices, each device having different types of I/O connectors, i.e. HDMI, AV over IP, and traditional AV. As such, the endpoint’s output and input are not single numbers like the typical videoroute endpoint described in the previous section. Instead the output and input are derived from the Q-Sys component control name, which takes the following form:

**Component control format**  
`<inputName>.select.<outputName>`

**Example component control name**  
`hdmi.out.1.select.muxed.hdmi.2.led`

where `"hdmi.out.1"` is `<inputName>` and `"muxed.hdmi.2.led"` is `<outputName>`.

**Endpoint**  
The endpoint for controlling this component will look something like this:

`/vail-811d-core.openav.dartmouth.edu/videoroute/<deviceName>_<outputName>`

**PUT input**  
The input, which is the data in the PUT request, will be:

`<inputName>`

**Notes**  
`<deviceName>` is a label chosen by the AV engineer, `<outputName>` is everything before `".select."` and `<inputName>` is everything after `".select."` in the component control name.

**Example**  
Here is an example SET parameter that would appear in the Vail-811d JSON config file, given that the component control name is `"hdmi.out.1.select.muxed.hdmi.2.led"` and the device name is `"decoder-1"`:

```bash
/vail-811d-core.openav.dartmouth.edu/videoroute/decoder-1_hdmi.out.1 -d "muxed.hdmi.2.led"
```

**Example**  
Another example SET given a component control name `"nv21.input.select.hdmi"` and a device name `"decoder-2"`:

```bash
/vail-811d-core.openav.dartmouth.edu/videoroute/decoder-2_nv21.input -d "hdmi"
```

**Reference**  
For more information about components, see [QRC Commands](https://q-syshelp.qsc.com/Content/External_Control_APIs/QRC/QRC_Commands.htm).

## Audio and Video route

**Description**  
Gets or sets what audio and video inputs are routed to the specified output (audio follows video).

**Endpoint**  
`/:address/audioandvideoroute/:output`

**GET**  
Returns a number in quotes that is the input setting for the specified output (e.g. `"3"`).

**PUT**  
Accepts a number, routes that input number to the specified output, and returns `"ok"` or an error.

**Notes**  
`:output` specifies the output for which we want to set the input that is routed to it. Numbering is the same as Volume.

## Audio and video mute

**Description**  
Gets or sets audio or video mute state for the specified output.

**Endpoints**  
Audio endpoint - `/:address/audiomute/:output`  
Video endpoint - `/:address/videomute/:output`  
Both endpoint - `/:address/audioandvideomute/:output`  
New for Extron - Video sync endpoint - `/:address/videosyncmute/:output`

**GET**  
Returns the mute status for audio or video: `"true"` for muted, `"false"` for not muted.

**PUT**  
Accepts `"true"` or `"false"` in the body, makes that setting for audio or video on the specified output, and returns `"ok"` or an error.

**Notes**  
The Sony Bravia microservice, NEC LCD screen panel, and maybe others in the future also accept `"toggle"` which will flip the mute setting from `"true"` to `"false"` or vice versa.

## Input status

**Description**  
Reports whether the specified input has a signal synced on it or not. Currently used only in the context of video inputs.

**Endpoint**  
`/:address/videoinputstatus/:input`

**GET**  
Returns the signal status for the specified input (`"true"` or `"false"`).

## Occupancy status

**Description**  
Reports (in the OpenAV `occupancy_detected` format) based on the status of a single input. If there is a connection synced to the specified input, it returns true. Otherwise it returns false. The idea is that the system can decide not to auto shutdown if certain inputs (e.g. HDMI) have a synced connection.

**Endpoint**  
`/:address/occupancystatus/:input`

**GET**  
Returns `{"occupancy_detected": "false"}`, or an error.

## Matrix mute

**Description**  
Mutes or unmutes a connection in the input x output matrix. Devices start with all points in the matrix muted.

**Endpoint**  
`/:address/matrixmute/:input/:output`

**GET**  
Returns the mute value set at the specified location in the matrix.

**PUT**  
Accepts either `"true"` or `"false"` in the PUT body, attempts to set the mute value accordingly at the specified location in the matrix, and returns `"ok"` or an error.

## Matrix volume

**Description**  
Sets the audio gain at a point in the input x output matrix.

**Endpoint**  
`/:address/matrixvolume/:input/:output`

**GET**  
Returns the gain set at the specified location in the matrix.

**PUT**  
Accepts a value between `"0"` and `"100"` in the PUT body, attempts to set the volume at the specified location in the matrix to that value, and returns `"ok"` or an error.

## Set state (e.g. for relay)

**Description**  
Sets the specified state to `"on"` or `"off"`.

**Endpoint**  
`/:address/state/:output`

**GET**  
Returns the state value for the specified output.

**PUT**  
Accepts either `"on"` or `"off"` in the PUT body, attempts to set the specified output state, and returns `"ok"` or an error.

## Trigger state (e.g. for relay)

**Description**  
Toggles the specified output `"on"`, sleeps the specified amount of time, and then toggles the specified state `"off"`.

**Endpoint**  
`/:address/trigger/:output`

**PUT**  
Accepts the amount of time to sleep in the PUT body (an integer in a string), attempts to toggle the output, and returns `"ok"` or an error.

## Timed trigger state (e.g. for relay)

**Description**  
Toggles output1 `"on"`, sleeps for `1`, toggles output1 `"off"`, sleeps for time specified in the `"body"` of the PUT, toggles specified outputs on at the same time, sleeps `1`, toggles outputs off.

**Endpoint**  
`/:address/timedtrigger/:output1/:output2`

**PUT**  
Accepts the amount of time to sleep in the PUT body (an integer in a string), attempts to toggle the output, and returns `"ok"` or an error

## Power Cycle (e.g. for switched PDU outlets)

**Description**  
Toggles the specified output `"off"`, sleeps the specified amount of time, and then toggles the specified output `"on"`.

**Endpoint**  
`/:address/powercycle/:output`

**PUT**  
Accepts the amount of time to sleep in the PUT body (an integer in a string), attempts to toggle the output, and returns `"ok"` or an error

## Errors

**Description**  
Reports any errors the microservice has encountered since the last time this endpoint was called and clears the error cache.

**Endpoint**  
`/:address/errors`

**GET**  
Returns any errors and a code of `StatusInternalServerError` if there were any errors.

## Possible Future Endpoints

### Passcode

**Description**  
Sets the passcode for a device (e.g. Auracast transmitter).

**Endpoint**  
`/:address/passcode`

**GET**  
Returns the current passcode value.

**PUT**  
Accepts the value of the passcode to set and returns `"ok"` or an error.

### Calendar Passcode

**Description**  
Sets auto-generated passcodes in the device based on calendar data. If the microservice is in this mode, GET on the `/:address/passcode` endpoint will return the latest passcode that it automatically set. This automation queries the calendar data when started and then updates it every hour thereafter. The microservice generates a new passcode at the beginning and end of each calendar entry.

**Notes**  
This feature generates easy to type passcodes because it is intended for rapidly-changing codes that don’t need to be extremely secure.

**Endpoint**  
`/:address/calendarpasscode`

**PUT**  
Accepts the URL to query for calendar data and returns `"ok"` or an error. Specifying an empty URL stops the automatic passcode setting.

**GET**  
Returns the calendar URL currently being used for this device.

**Example response**  
The microservice currently parses calendar data responses in this form:

```json
{
  "success": true,
  "message": "ok",
  "data": {
    "room_name": "c118",
    "room_id": 4637,
    "events": [
      {
        "name": "Thayer 197.1 - Professional Development",
        "type": "Lecture",
        "start_time": "2025-12-18T17:00:00",
        "end_time": "2025-12-18T18:30:00",
        "status_id": 9,
        "booking_id": 2937425,
        "organizers": [
          "<redacted@dartmouth.edu>"
        ]
      },
      {
        "name": "Thayer 197.1 - Professional Development",
        "type": "Lecture",
        "start_time": "2025-12-22T17:00:00",
        "end_time": "2025-12-22T18:30:00",
        "status_id": 9,
        "booking_id": 2937426,
        "organizers": [
          "<redacted@dartmouth.edu>"
        ]
      },
      {
        "name": "Thayer 197.1 - Professional Development",
        "type": "Lecture",
        "start_time": "2025-12-23T17:00:00",
        "end_time": "2025-12-23T18:30:00",
        "status_id": 9,
        "booking_id": 2937427,
        "organizers": [
          "<redacted@dartmouth.edu>"
        ]
      }
    ]
  }
}
```

**Notes**  
The endpoint `"calendarpasscode"` only uses `start_time` and `end_time`. It could be extended to recognize different JSON responses and parse those too (pull requests are welcome).

New endpoints could also share the implementation and extend it to recognize other fields and act accordingly (e.g. a `"calendarrecording"` endpoint could start recordings for events with `"type": "Lecture"`).
