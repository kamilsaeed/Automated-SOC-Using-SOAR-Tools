# Code Folder - Wazuh to TheHive Integration

## Overview

This folder contains custom integration scripts that bridge **Wazuh** (an open-source security monitoring platform) with **TheHive** (a scalable, open-source incident response platform). The integration enables automated alert forwarding from Wazuh to TheHive for centralized incident management.

## Files

### `custom-w2thive.py`
**Purpose:** Main integration script that processes Wazuh alerts and creates incidents in TheHive.

**Key Functionality:**
- Parses Wazuh alert JSON payloads
- Extracts and formats alert data into readable markdown
- Detects artifacts (IPs, URLs, domains) from alert content
- Applies severity thresholds for filtering:
  - **Wazuh alerts:** Threshold level = 0 (configurable via `lvl_threshold`)
  - **Suricata alerts:** Threshold level = 3 (configurable via `suricata_lvl_threshold`)
- Creates TheHive alerts with:
  - Title from Wazuh rule description
  - Formatted description with alert details
  - Artifact detection for IOCs (Indicators of Compromise)
  - Tags for agent info, rule ID, and alert source
  - TLP (Traffic Light Protocol) level = 2 (Amber)

**Alert Tags Created:**
- `wazuh` - Source platform
- `rule=<rule_id>` - Wazuh rule ID
- `agent_name=<name>` - Source agent name
- `agent_id=<id>` - Source agent ID
- `agent_ip=<ip>` - Source agent IP address

**Configuration:**
Located at the beginning of the script, adjustable parameters:
```python
lvl_threshold = 0              # Wazuh rule level threshold
suricata_lvl_threshold = 3     # Suricata rule level threshold
debug_enabled = False          # Debug logging toggle
info_enabled = True            # Info logging toggle
```

**Dependencies:**
- `thehive4py` - TheHive Python API library
- Python 3.x with standard libraries: `json`, `sys`, `os`, `re`, `logging`, `uuid`

**Logging:**
- Log file location: `{WAZUH_HOME}/logs/integrations.log`
- Supports DEBUG, INFO, and WARNING levels based on configuration

### `ossec.conf`
**Purpose:** Example Wazuh integration configuration file.

**Contents:**
```xml
<integration>
  <name>custom-w2thive</name>
  <hook_url>http://hostip:9000</hook_url>
  <api_key>REDACTED_FOR_SECURITY</api_key>
  <alert_format>json</alert_format>
</integration>
```

**Configuration Details:**
- **name:** Integration script reference (`custom-w2thive`)
- **hook_url:** TheHive API endpoint (update `hostip` with actual TheHive server IP/hostname)
- **api_key:** TheHive API authentication key (obtain from TheHive admin panel)
- **alert_format:** Format of alerts sent to the script (JSON)

## Setup Instructions

### Prerequisites
1. **Wazuh Manager** - Fully operational Wazuh installation
2. **TheHive Server** - Running with API access enabled
3. **Python 3.x** - With pip package manager
4. **thehive4py** library - Install via: `pip install thehive4py`

### Installation Steps

1. **Copy the integration script to Wazuh:**
   ```bash
   cp custom-w2thive.py /var/ossec/integrations/
   chmod +x /var/ossec/integrations/custom-w2thive.py
   ```

2. **Update Wazuh configuration:**
   - Locate Wazuh configuration file: `/var/ossec/etc/ossec.conf`
   - Add the integration block (modify from `ossec.conf` template)
   - Replace `hostip` with actual TheHive server address
   - Insert valid TheHive API key

3. **Restart Wazuh Manager:**
   ```bash
   systemctl restart wazuh-manager
   ```

4. **Verify integration:**
   - Check logs: `tail -f /var/ossec/logs/integrations.log`
   - Generate test alert in Wazuh and confirm it appears in TheHive

## Usage Flow

```
┌─────────────────────┐
│   Wazuh Manager     │
│  (Alert triggered)  │
└──────────┬──────────┘
           │
           ▼
┌──────────────────────────┐
│  ossec.conf integration  │
│  (triggers script)       │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ custom-w2thive.py        │
│ • Parse JSON alert       │
│ • Extract artifacts      │
│ • Apply thresholds       │
│ • Format description     │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│   TheHive API (v1)       │
│   POST /api/alert        │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ TheHive Dashboard        │
│ (Alert created & visible)│
└──────────────────────────┘
```

## Troubleshooting

### Alerts not appearing in TheHive
- Check TheHive API key validity
- Verify TheHive server is accessible: `curl http://hostip:9000/api/alert`
- Review logs: `tail -f /var/ossec/logs/integrations.log`
- Ensure alert severity meets configured thresholds

### Connection errors
- Verify network connectivity: `ping <thehive_server>`
- Check firewall rules allow port 9000
- Ensure `thehive4py` library is installed in Wazuh's Python environment

### Script not executing
- Verify execute permissions: `ls -l /var/ossec/integrations/custom-w2thive.py`
- Check script path in ossec.conf matches actual file location
- Review Wazuh manager logs for integration errors

## Future Enhancements

Potential improvements for this integration:
- Multi-rule threshold configuration per rule ID
- Dynamic artifact type detection (CVEs, email addresses, file hashes)
- Case-sensitive filtering options
- Batch alert processing for high-volume scenarios
- Webhook/REST API endpoint for reverse notifications
- Support for custom metadata fields

## License & References

This integration is part of the Automated SOC project using Wazuh, TheHive, MISP, and Cortex.

Related Documentation:
- Wazuh Integration Documentation: https://documentation.wazuh.com/current/user-manual/capabilities/integration/
- TheHive4py GitHub: https://github.com/TheHive-Project/TheHive4py
- TheHive API Documentation: https://docs.thehive-project.org/api/
