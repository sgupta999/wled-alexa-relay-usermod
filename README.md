# wled-alexa-relay-usermod

## Controllable by Alexa - Multi Relay 
### (Please still consider this a beta - only user will tell how well it works)

So I really wanted the same esp32 to control both my SR Leds and my normal AC lights through alexa without having multiple contollers. I also wanted additional relay to control the DC PSU as that sits in the attic. Nothing quite matched my needs so I built excellent work done by many others.

This usermod is a modificaiton of the original Multi-Relay usermod built by blazoncek.
Since relays will now be exposed as additioanl devices to Alexa the base WLED build which uses Virtual Alexa devices to execute presets
has been replaced with presets now being enabled by calling brightness events from 1-9.  Total brightness range is (1-100). For e.g.
"Alexa turn device brightness to 1" will execute preset 1
"Alexa turn device brightness to 2" will execute preset 2   and so on.

The assumption here being for brightness from 1-10 most people will not care much, as rarely will you call brightness below 10.
Number of presets that can be mapped to each brightness value is controlled by (ESPALEXA_MAXDEVICES - 1) and is set to 9 by default as the original

Original relay modification allowed the connection of multiple relays, each with individual delay and on/off mode.
This enhancement makes each of the relays cotrollable from the WLED info setup as well as enabling the relay controls through Alexa.

None of the original capabilities of the nulti-relay usermod have been modified or diminished in any way and still as usable as before.
the below text is from that orginal mod modfied for alexa controls

Currently alexa can send state changes to wled relays and they are reflected in  all the wled interfaces. However, I currently do not propagate the
state changes that happen outside of alexa back to alexa - just havent figure out what the right UDP packet to send it as yet.
However, alexa constantly polls the system for updated state to it updates itself - a slight delay but alexa eventually synchs up the state

![image](https://github.com/sgupta999/wled-alexa-relay-usermod/assets/35128032/32b7d704-bff5-4914-b1a6-ca6cd17881f0)
![image](https://github.com/sgupta999/wled-alexa-relay-usermod/assets/35128032/d3286dcb-c3b5-4741-8ee0-d8eb3d015c7e)
![image](https://github.com/sgupta999/wled-alexa-relay-usermod/assets/35128032/7f6ef617-d682-4a69-a54f-0694a9a66199)


## HTTP API
### I have not modified or used the original HTTP API or MQTT topics in any way.
<=========================>

All responses are returned in JSON format. 

* Status Request: `http://[device-ip]/relays`
* Switch Command: `http://[device-ip]/relays?switch=1,0,1,1`

The number of values behind the switch parameter must correspond to the number of relays. The value 1 switches the relay on, 0 switches it off. 

* Toggle Command: `http://[device-ip]/relays?toggle=1,0,1,1`

The number of values behind the parameter switch must correspond to the number of relays. The value 1 causes the relay to toggle, 0 leaves its state unchanged.

Examples:
1. total of 4 relays, relay 2 will be toggled: `http://[device-ip]/relays?toggle=0,1,0,0`
2. total of 3 relays, relay 1&3 will be switched on: `http://[device-ip]/relays?switch=1,0,1`

## JSON API
You can toggle the relay state by sending the following JSON object to: `http://[device-ip]/json`

Switch relay 0 on: `{"MultiRelay":{"relay":0,"on":true}}`

Switch relay 3 and 4 off: `{"MultiRelay":[{"relay":2,"on":false},{"relay":3,"on":false}]}`


## MQTT API

* `wled`/_deviceMAC_/`relay`/`0`/`command` `on`|`off`|`toggle`
* `wled`/_deviceMAC_/`relay`/`1`/`command` `on`|`off`|`toggle`

When a relay is switched, a message is published:

* `wled`/_deviceMAC_/`relay`/`0` `on`|`off`


## Usermod installation

1. Register the usermod by adding `#include "../usermods/multi_relay/usermod_multi_relay.h"` at the top and `usermods.add(new MultiRelay());` at the bottom of `usermods_list.cpp`.
or
2. Use `#define USERMOD_MULTI_RELAY` in wled.h or `-D USERMOD_MULTI_RELAY` in your platformio.ini

You can override the default maximum number of relays (which is 4) by defining MULTI_RELAY_MAX_RELAYS.

Example **usermods_list.cpp**:

```cpp
#include "wled.h"
/*
 * Register your v2 usermods here!
 *   (for v1 usermods using just usermod.cpp, you can ignore this file)
 */

/*
 * Add/uncomment your usermod filename here (and once more below)
 * || || ||
 * \/ \/ \/
 */
//#include "usermod_v2_example.h"
//#include "usermod_temperature.h"
#include "../usermods/usermod_multi_relay.h"

void registerUsermods()
{
  /*
   * Add your usermod class name here
   * || || ||
   * \/ \/ \/
   */
  //usermods.add(new MyExampleUsermod());
  //usermods.add(new UsermodTemperature());
  usermods.add(new MultiRelay());

}
```

### I have modfied the alexa.cpp file signficantly and the fcn_declare.h to declare an additioanl alexa callback routine.


## Configuration

Usermod can be configured via the Usermods settings page.

* `enabled` - enable/disable usermod
* `pin` - ESP GPIO pin the relay is connected to (can be configured at compile time `-D MULTI_RELAY_PINS=xx,xx,...`)
* `delay-s` - delay in seconds after on/off command is received
* `active-high` - assign high/low activation of relay (can be used to reverse relay states)
* `external` - if enabled, WLED does not control relay, it can only be triggered by an external command (MQTT, HTTP, JSON or button)
* `alexa-enabled` - if enabled here and Alexa Control is eabled in sync interfaces, the relays will be automatically disoverable by Alexa
* `alexa-invocation-name` - device name that will show up in WLED info page and during Alexa discovery/add device workflow
* `button` - button (from LED Settings) that controls this relay
* `broadcast`- time in seconds between MQTT relay-state broadcasts
* `HA-discovery`- enable Home Assistant auto discovery

If there is no MultiRelay section, just save current configuration and re-open Usermods settings page. 
