# tfrec-mqtt
Docker image to receive wireless sensor data and publish them via MQTT.

## Usage
Start a container to publish sensor data via MQTT.
### Docker Compose
1. Create a run configuration `docker-compose.yml`, e.g.
    ```yaml
    version: 2.2
    services:
      tfrec-mqtt:
        image: ckware/tfrec-mqtt
        container_name: tfrec-mqtt
        devices:
        - "/dev/bus/usb:/dev/bus/usb"
        environment:
        - MQTT_OPTIONS: "-h broker"
        command: [ "tfrec", "-q", "-e",  "mqtt-publish" ]
        init: "true"
        restart: unless-stopped
    ```
2. Start a container:
    ```bash
    $ docker-compose up -d
    ```

### Docker
  ```bash
  $ docker run --device /dev/bus/usb -e "MQTT_OPTIONS=-h broker" --init -d ckware/tfrec-mqtt tfrec -q -e mqtt-publish
  ```

### Result
```bash
$ mosquitto_sub -v -h broker -t tfrec/#

tfrec/1a2b/json { "sensor_id":"1a2b", "temperature":9.1, "humidity":78, "seq":0, "lowbat":0, "rssi":82, "flags":0, "timestamp":1588303401 }

tfrec/c3d4/json { "sensor_id":"c3d4", "temperature":15.6, "humidity":69, "seq":0, "lowbat":0, "rssi":79, "flags":0, "timestamp":1588303402 }
```

## Requirements
### Hardware
* RTL2832-based USB-Adapter
* One or more thermo/hygro sensors made by or compatible to TFA Dostmann, Technoline or LaCrosse (see [sensors.txt](https://github.com/baycom/tfrec/blob/master/sensors.txt) for a list)

### Software
* Required: Docker
* Recommended: Docker Compose

The Docker Compose [documentation](https://docs.docker.com/compose/install/)
contains a comprehensive guide explaining several install options. On recent debian-based systems, Docker Compose may be installed by calling
  ```shell
  $ sudo apt install docker-compose
  ```

## Configuration
The configuration is based on environment variables.

|Variable|Description|Allowed values|Default|Example
|--------|-----------|-----|-------|-------
|`MQTT_OPTIONS`|MQTT options|All options [supported by `mosquitto_pub`](https://mosquitto.org/man/mosquitto_pub-1.html)|_none_|`-v -h broker`
|`MQTT_TOPIC`|MQTT topic for publishing sensor data|[Topic names](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718106)|`tfrec`|`devices/sensors`
|`MQTT_TOPIC_APPEND_ID`|Append sensor ID to topic?|`true` / `false`|`true`|`true`
|`MQTT_TOPIC_APPEND_FORMAT`|Append format (one of: `json`, `raw`) to topic?|`true` / `false`|`true`|`true`
|`FORMAT_JSON`|Publish sensor data in JSON format?|`true` / `false`|`true`|`true`
|`FORMAT_JSON_LOWBAT_AS_BOOLEAN`|Convert native `lowbat` value (`0` or `1`)  to JSON `boolean` type?|`true` / `false`|`false`|`false`
|`FORMAT_RAW`|Publish sensor data in raw format?|`true` / `false`|`false`|`false`
|`FORMAT_RAW_SEPARATOR`|Field separator for raw format|String|Whitespace (`\u0020`)|`,`

## FHEM-Integration
Example configuration to use the sensor values within [FHEM](https://fhem.de/):
```
define mosquitto MQTT localhost:1883

define mqtt_klima_wohnzimmer MQTT_DEVICE
attr   mqtt_klima_wohnzimmer subscribeReading_json tfrec/1a2b/json
attr   mqtt_klima_wohnzimmer stateFormat T: temperature H: humidity

define expandjson_mqtt_klima expandJSON mqtt_klima_.+.json:.\{.*\}
```

## References
* This project is an integration of
  * [tfrec](https://github.com/baycom/tfrec) - A SDR tool for receiving wireless sensor data
  * [Mosquitto](https://mosquitto.org/) - An Open Source MQTT Broker
  * The [OCI image](https://github.com/opencontainers/image-spec) format 
  * [Docker](https://www.docker.com)
* History and details (in German): [Temperatur/Feuchte-Sender mit tfrec und MQTT](https://github.com/git-developer/fhem-examples/wiki/Temperatur-Feuchte-Sender-mit-tfrec-und-MQTT)