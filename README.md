# ha-bridge
Emulates philips hue api to other home automation gateways.  The Amazon echo now supports wemo and philip hue.
Build
-----

To customize and build it yourself, build a new jar with maven:
```
mvn install
```
Then locate the jar and start the server with:
```
java -jar -Dvera.address=192.168.X.Y ha-bridge-0.X.Y.jar
```
The argument for the vera address should be given as it the system does not have a way to find the address. Supply -Dvera.address=X.Y.Z.A on the command line to provide it.

The server defaults to the first available address on the host. Replace the -Dupnp.config.address=<ip address> value with the server ipv4 address you would like to use. 

The server defaults to running on port 8080. If you're already running a server (like openHAB) on 8080, -Dserver.port=<port> on the command line.

The default location for the db to contain the devices as they are added is "data/devices.db". If you would like a different filename or directory, specify -Dupnp.devices.db=<directory>/<filename> or <filename> if it is the same directory.

The default upnp response port will be 50000 otherwise it can be set with -Dupnp.response.port=<port>.

Then configure by going to the url for the host you are running on or localhost: 
```
http://192.168.1.240:8080
```
or Register a device, via REST by binding some sort of on/off (vera style) url
```
POST http://host:8080/api/devices
{
"name" : "bedroom light",
"deviceType" : "switch",
  "onUrl" : "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=1&DeviceNum=41",
  "offUrl" : "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=0&DeviceNum=41"
}
Dimming
----
Dimming is also supported by using the expessions ${intensity.percent} or ${intensity.byte} for 0-100 and 0-255 respectively.  
e.g.
```
{
    "name": "entry light",
    "deviceType": "switch",
    "offUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=0&DeviceNum=31",
    "onUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&DeviceNum=31&serviceId=urn:upnp-org:serviceId:Dimming1&action=SetLoadLevelTarget&newLoadlevelTarget=${intensity.percent}"
}
```
See the echo's documentation for the dimming phrase.

POST/PUT support
-----
added optional fields
 * contentType (currently un-validated)
 * httpVerb (POST/PUT/GET only supported)
 * contentBody your post/put body here

This will allow control of any other application that may need mroe then GET.
e.g: 
```
{
    "name": "test device",
    "deviceType": "switch",
    "offUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=0&DeviceNum=31",
    "onUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&DeviceNum=31&serviceId=urn:upnp-org:serviceId:Dimming1&action=SetLoadLevelTarget&newLoadlevelTarget=${intensity.percent}",
  "contentType" : "application/json",
  "httpVerb":"POST",
  "contentBody" : "{\"fooBar\":\"baz\"}"
}
```
Anything that takes an action as a result of an HTTP request will probably work - like putting Vera in and out of night mode:
```
{
  "name": "night mode",
  "deviceType": "switch",
  "offUrl": "http://192.168.1.201:3480/data_request?id=lu_action&serviceId=urn:micasaverde-com:serviceId:HomeAutomationGateway1&action=SetHouseMode&Mode=1",
  "onUrl": "http://192.168.1.201:3480/data_request?id=lu_action&serviceId=urn:micasaverde-com:serviceId:HomeAutomationGateway1&action=SetHouseMode&Mode=3"
}
```
Ask Alexa
----
After this Tell Alexa: "Alexa, discover my devices"

Then you can say "Alexa, Turn on the office light" or whatever name you have given your configured devices.

To view or remove devices that Alexa knows about, you can use the mobile app Menu / Settings / Connected Home
