# SSH Command to MQTT Home Assistant Add-on

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Supports armhf Architecture][armhf-shield]
![Supports armv7 Architecture][armv7-shield]
![Supports i386 Architecture][i386-shield]

A Home Assistant add-on that connects to remote hosts via SSH, executes commands, and publishes the results as auto-discoverable MQTT sensors.

## About

This add-on allows you to:

- 🖥️ Connect to multiple remote hosts via SSH
- ⚡ Execute custom commands on a scheduled basis
- 🏠 Automatically create Home Assistant sensors via MQTT discovery
- 🔐 Support both password and SSH key authentication
- ⏰ Configure different execution intervals for each command
- 📊 Set custom units of measurement and device classes

## Installation

### Method 1: Add Repository to Home Assistant

1. In Home Assistant, go to **Settings** > **Add-ons** > **Add-on Store**
2. Click the three dots in the top right corner
3. Select **Repositories**
4. Add this repository URL: `https://github.com/your-username/ssh-command-addon`
5. Find "SSH Command to MQTT" in the add-on store
6. Click **Install**

### Method 2: Manual Installation

1. Clone this repository to your Home Assistant `addons` directory:
   ```bash
   cd /usr/share/hassio/addons/local/
   git clone https://github.com/your-username/ssh-command-addon.git
   ```
2. Restart Home Assistant
3. Find "SSH Command to MQTT" in the add-on store

## Configuration

### Basic Example

```yaml
ssh_hosts:
  - hostname: "192.168.1.100"
    username: "pi"
    password: "raspberry"
    commands:
      - name: "cpu_temp"
        command: "cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1/1000}'"
        interval: 60
        unit_of_measurement: "°C"
        device_class: "temperature"

mqtt:
  broker: "core-mosquitto"
```

### Advanced Example

```yaml
ssh_hosts:
  # Raspberry Pi with password auth
  - hostname: "192.168.1.100"
    username: "pi"
    password: "raspberry"
    port: 22
    commands:
      - name: "cpu_temperature"
        command: "cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1/1000}'"
        interval: 60
        unit_of_measurement: "°C"
        device_class: "temperature"

      - name: "memory_usage"
        command: "free | grep Mem | awk '{printf \"%.2f\", ($3/$2) * 100.0}'"
        interval: 120
        unit_of_measurement: "%"

      - name: "uptime"
        command: "uptime -p"
        interval: 300

  # Server with SSH key auth
  - hostname: "server.example.com"
    username: "admin"
    private_key: "/ssl/id_rsa"
    port: 22
    commands:
      - name: "nginx_status"
        command: "systemctl is-active nginx"
        interval: 60

      - name: "disk_usage"
        command: "df -h / | tail -1 | awk '{print $5}' | sed 's/%//'"
        interval: 300
        unit_of_measurement: "%"

mqtt:
  broker: "core-mosquitto"
  port: 1883
  username: "mqtt_user"
  password: "mqtt_pass"
  discovery_prefix: "homeassistant"

log_level: "info"
```

## Configuration Options

### SSH Hosts

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `hostname` | string | Yes | SSH host IP address or hostname |
| `username` | string | Yes | SSH username |
| `password` | string | No* | SSH password |
| `private_key` | string | No* | Path to SSH private key file |
| `port` | integer | No | SSH port (default: 22) |
| `commands` | list | Yes | List of commands to execute |

*Either `password` or `private_key` must be provided.

### Commands

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `name` | string | Yes | Unique name for the sensor |
| `command` | string | Yes | Shell command to execute |
| `interval` | integer | No | Execution interval in seconds (default: 300) |
| `unit_of_measurement` | string | No | Unit for the sensor value |
| `device_class` | string | No | Home Assistant device class |

### MQTT Settings

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `broker` | string | No | MQTT broker hostname (default: "core-mosquitto") |
| `port` | integer | No | MQTT broker port (default: 1883) |
| `username` | string | No | MQTT username |
| `password` | string | No | MQTT password |
| `discovery_prefix` | string | No | HA discovery prefix (default: "homeassistant") |

## SSH Key Authentication

To use SSH key authentication:

1. Place your private key file in the `/ssl/` directory of your Home Assistant installation
2. Reference it in the configuration: `private_key: "/ssl/id_rsa"`
3. Ensure the key has proper permissions (600)

## Useful Commands

### System Monitoring

```yaml
commands:
  # CPU Temperature (Raspberry Pi)
  - name: "cpu_temp"
    command: "cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1/1000}'"
    unit_of_measurement: "°C"
    device_class: "temperature"

  # CPU Usage
  - name: "cpu_usage"
    command: "top -bn1 | grep 'Cpu(s)' | awk '{print $2}' | sed 's/%us,//'"
    unit_of_measurement: "%"

  # Memory Usage
  - name: "memory_usage"
    command: "free | grep Mem | awk '{printf \"%.2f\", ($3/$2) * 100.0}'"
    unit_of_measurement: "%"

  # Disk Usage
  - name: "disk_usage"
    command: "df -h / | tail -1 | awk '{print $5}' | sed 's/%//'"
    unit_of_measurement: "%"

  # Load Average
  - name: "load_average"
    command: "uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//'"

  # Network Ping Test
  - name: "ping_google"
    command: "ping -c 1 8.8.8.8 | grep 'time=' | awk -F'time=' '{print $2}' | awk '{print $1}'"
    unit_of_measurement: "ms"
```

### Service Status

```yaml
commands:
  # Check if service is active
  - name: "nginx_status"
    command: "systemctl is-active nginx"

  # Get service status with details
  - name: "docker_status"
    command: "systemctl status docker --no-pager -l | head -3 | tail -1 | awk '{print $2}'"
```

### Custom Scripts

You can execute any script available on the remote host:

```yaml
commands:
  - name: "custom_metric"
    command: "/home/user/scripts/get_metric.sh"
    interval: 300
```

## Troubleshooting

### Common Issues

1. **SSH Connection Failed**
   - Verify hostname, port, and credentials
   - Check if SSH service is running on target host
   - Ensure firewall allows SSH connections

2. **Command Not Found**
   - Verify the command exists on the target system
   - Check the command path and permissions
   - Test the command manually via SSH

3. **MQTT Connection Issues**
   - Verify MQTT broker settings
   - Check MQTT credentials
   - Ensure MQTT integration is configured in Home Assistant

4. **Sensors Not Appearing**
   - Check Home Assistant logs for MQTT discovery messages
   - Verify MQTT broker is receiving messages
   - Ensure discovery prefix matches HA configuration

### Debug Logging

Enable debug logging by setting:

```yaml
log_level: "debug"
```

### Testing Configuration

Use the included test script to validate your configuration:

```bash
docker exec addon_ssh_command_mqtt /usr/bin/test-config
```

## Development

### Building Locally

```bash
docker build -t ssh-command-mqtt .
```

### Testing

```bash
# Test configuration validation
python3 rootfs/usr/bin/test-config example-config.yaml

# Run the addon locally
python3 rootfs/usr/bin/ssh-command-mqtt
```

## Support

- 📋 [Report Issues](https://github.com/your-username/ssh-command-addon/issues)
- 💬 [Discussions](https://github.com/your-username/ssh-command-addon/discussions)
- 📖 [Home Assistant Community](https://community.home-assistant.io/)

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- Home Assistant team for the excellent add-on framework
- The SSH and MQTT communities for their robust protocols

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg