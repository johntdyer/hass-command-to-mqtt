# SSH Command to MQTT Add-on

![Supports aarch64 Architecture][aarch64-shield] ![Supports amd64 Architecture][amd64-shield] ![Supports armhf Architecture][armhf-shield] ![Supports armv7 Architecture][armv7-shield] ![Supports i386 Architecture][i386-shield]

This add-on connects to remote hosts via SSH, executes commands, and publishes the results as auto-discoverable sensors in Home Assistant through MQTT.

## About

The SSH Command to MQTT add-on allows you to:

- Connect to multiple remote hosts via SSH
- Execute custom commands on a scheduled basis
- Automatically create Home Assistant sensors via MQTT discovery
- Support both password and private key authentication
- Configure different execution intervals for each command

## Installation

1. Add this repository to your Home Assistant add-on store
2. Install the "SSH Command to MQTT" add-on
3. Configure the add-on with your SSH hosts and commands
4. Start the add-on

## Configuration

### SSH Hosts Configuration

```yaml
ssh_hosts:
  - hostname: "192.168.1.100"
    username: "pi"
    password: "mypassword"
    port: 22
    commands:
      - name: "cpu_temperature"
        command: "cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1/1000}'"
        interval: 60
        unit_of_measurement: "°C"
        device_class: "temperature"
      - name: "uptime"
        command: "uptime -p"
        interval: 300
```

### MQTT Configuration

```yaml
mqtt:
  broker: "core-mosquitto"
  port: 1883
  username: ""
  password: ""
  discovery_prefix: "homeassistant"
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `ssh_hosts` | List of SSH hosts and commands to execute | Required |
| `mqtt.broker` | MQTT broker hostname | `core-mosquitto` |
| `mqtt.port` | MQTT broker port | `1883` |
| `mqtt.username` | MQTT username | Empty |
| `mqtt.password` | MQTT password | Empty |
| `mqtt.discovery_prefix` | Home Assistant discovery prefix | `homeassistant` |
| `log_level` | Logging level | `info` |

### SSH Host Options

| Option | Description | Required |
|--------|-------------|----------|
| `hostname` | SSH host IP or hostname | Yes |
| `username` | SSH username | Yes |
| `password` | SSH password | No* |
| `private_key` | Path to private key file | No* |
| `port` | SSH port | No (default: 22) |
| `commands` | List of commands to execute | Yes |

*Either password or private_key must be provided

### Command Options

| Option | Description | Required |
|--------|-------------|----------|
| `name` | Unique name for the sensor | Yes |
| `command` | Shell command to execute | Yes |
| `interval` | Execution interval in seconds | No (default: 300) |
| `unit_of_measurement` | Unit for the sensor value | No |
| `device_class` | Home Assistant device class | No |

## Example Commands

### System Monitoring

```yaml
commands:
  - name: "cpu_usage"
    command: "top -bn1 | grep 'Cpu(s)' | awk '{print $2}' | sed 's/%us,//'"
    interval: 60
    unit_of_measurement: "%"

  - name: "memory_usage"
    command: "free | grep Mem | awk '{printf \"%.2f\", ($3/$2) * 100.0}'"
    interval: 120
    unit_of_measurement: "%"

  - name: "disk_usage_root"
    command: "df -h / | tail -1 | awk '{print $5}' | sed 's/%//'"
    interval: 300
    unit_of_measurement: "%"
```

### Network Monitoring

```yaml
commands:
  - name: "ping_google"
    command: "ping -c 1 8.8.8.8 | grep 'time=' | awk -F'time=' '{print $2}' | awk '{print $1}'"
    interval: 60
    unit_of_measurement: "ms"
```

## Troubleshooting

### SSH Connection Issues

1. Verify the hostname/IP and port are correct
2. Check SSH credentials (username/password or private key)
3. Ensure SSH service is running on the target host
4. Check firewall settings on both hosts

### MQTT Issues

1. Verify MQTT broker settings
2. Check if MQTT integration is enabled in Home Assistant
3. Ensure MQTT credentials are correct

## Support

For issues and feature requests, please use the GitHub repository.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg