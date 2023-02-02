# ESPHome component for NSPanel Lovelace UI

This repository contains ESPHome component that can be used as an alternative to the official
Tasmota driver provided in [NSPanel Lovelace UI](https://github.com/joBr99/nspanel-lovelace-ui)
repository. It still uses the same AppDaemon backend and TFT driver but the MQTT commands
from the AppDaemon are forwarded to a custom component that translates them to the protocol
used by the display's Nextion firmware, providing the same UX for users that prefer ESPHome
over Tasmota.

Please note that this code is currently in a very early stage of development, based on code
~~stolen from~~ based on the abandoned code from joBr99's repo (which is actually based on an
[outstanding PR for the stock NSPanel FW](https://github.com/esphome/esphome/pull/2702)) and
official [ESPHome Nextion component](https://github.com/esphome/esphome/tree/dev/esphome/components/nextion)
(TFT FW upload). Once the code stabilizes (no ETA), properly semver-tagged releases will likely
appear.

## Usage

Add reference to this repository to the `external_components` definitions. Make sure that `ref` is a valid tag
from the [Releases](https://github.com/sairon/esphome-nspanel-lovelace-ui/tags) page or an existing
[branch](https://github.com/sairon/esphome-nspanel-lovelace-ui/branches).
You can use `dev` for the latest bleeding-edge version but be aware that things may break from time to time.

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/sairon/esphome-nspanel-lovelace-ui
      ref: release/v0.2.x
    components: [nspanel_lovelace]
```

To handle messages received via MQTT from the AppDaemon, first create the `mqtt` component with the address of your
MQTT broker:

```yaml
mqtt:
  broker: 192.168.1.1
```

Then initialize the `nspanel_lovelace` component by adding the following line:

```yaml
nspanel_lovelace:
```

There are several options you can use:

- `id`: ID of the component used in lambdas
- `mqtt_recv_topic`: change if you are using different `panelRecvTopic` in your AppDaemon config
- `mqtt_send_topic`: change if you are using different `panelSendTopic` in your AppDaemon config
- `on_incoming_message`: action called when a message is received from the NSPanel firmware (e.g. on a touch event)
- `berry_driver_version`: version of the official Tasmota Berry driver version reported to AppDaemon backend. If set to `0`, automatic updates or notifications about them are effectively disabled. Defaults to `999`.
- `use_missed_updates_workaround`: use workaround for [missed status updates](https://github.com/sairon/esphome-nspanel-lovelace-ui/issues/8) - introduce short delay in internal ESPHome-NSPanel communication. Defaults to `true`, usually there should be no need to disable this option.
- `update_baud_rate`: baud rate to use when updating the TFT firmware, defaults to `921600`. Set to `115200` if you encounter any issues during the updates.

### TFT firmware update

Firmware of the TFT can be updated using the `upload_tft` function - for that you can create a service that can
be triggered from the Home Assistant when needed (make sure you have defined an ID for the NSPanel component):

```yaml
api:
  - service: upload_tft
    variables:
      url: string
    then:
      - lambda: |-
          id(nspanel).upload_tft(url);
```

### Other functions

There are currently few more public functions which can be useful for debugging or writing custom automations
for controlling the display:

- `send_custom_command(command)`: send NSPanel Lovelace UI custom command (`0x55 0xBB` + len prefix, CRC suffix)
- `send_nextion_command(command)`: send Nextion command (suffixed with `0xFF 0xFF 0xFF`)
- `start_reparse_mode()`: start Nextion reparse mode (component will also stop handling data from the display)
- `exit_reparse_mode()`: exit Nextion reparse mode

### Example configuration

See the [examples](examples) folder for configuration examples.

## License

Code in this repository is licensed under the [ESPHome License](LICENSE) (MIT for Python sources, GPL for C++).
