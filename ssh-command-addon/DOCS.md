# Configuration

The SSH Command to MQTT add-on allows you to execute commands on remote hosts via SSH and publish the results as sensors in Home Assistant.

## SSH Hosts

Configure one or more SSH hosts with their credentials and commands:

```yaml
ssh_hosts:
  - hostname: "192.168.1.100"
    username: "pi"
    password: "mypassword"
    port: 22
    commands:
      - name: "system_uptime"
        command: "uptime -p"
        interval: 300
```

## MQTT Settings

Configure the MQTT broker connection:

```yaml
mqtt:
  broker: "core-mosquitto"
  port: 1883
  username: ""
  password: ""
```

## Authentication

You can use either password or private key authentication:

### Password Authentication
```yaml
ssh_hosts:
  - hostname: "example.com"
    username: "user"
    password: "your_password"
```

### Private Key Authentication
```yaml
ssh_hosts:
  - hostname: "example.com"
    username: "user"
    private_key: "/ssl/id_rsa"
```

Note: Place private key files in the `/ssl/` directory of your Home Assistant installation.

## Command Examples

### System Monitoring
- CPU Temperature: `cat /sys/class/thermal/thermal_zone0/temp | awk '{print $1/1000}'`
- Memory Usage: `free | grep Mem | awk '{printf "%.2f", ($3/$2) * 100.0}'`
- Disk Usage: `df -h / | tail -1 | awk '{print $5}' | sed 's/%//'`

### Service Status
- Check if service is running: `systemctl is-active nginx`
- Get service status: `systemctl status nginx --no-pager -l`

### Custom Scripts
You can execute any custom script or command available on the remote host.