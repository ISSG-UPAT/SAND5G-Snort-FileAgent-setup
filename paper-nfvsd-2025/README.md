# Technical Overview of the Paper: "A Practical Assessment of IDS and IPS Capabilities in a 5G Network Testbed"

## Abstract

This paper explores the deployment and evaluation of Snort3, an open-source Intrusion Detection System (IDS) and Intrusion Prevention System (IPS), within a virtualized 5G network testbed. The study focuses on real-time threat detection, dynamic rule updates, and cross-operator threat coordination, addressing the growing cybersecurity challenges in 5G networks.

## Key Components

### 1. **Snort3 Integration**

- **Purpose**: Real-time traffic analysis, packet filtering, and alert generation.
- **Deployment**: Containerized environment with custom rules for detecting malicious traffic patterns.
- **Capabilities**:
  - Alert generation for suspicious traffic.
  - Packet dropping/blocking based on predefined rules.
  - Dynamic rule updates via API integration.

### 2. **5G Testbed Architecture**

- **Components**:
  - 5G Core and simulated Radio Access Network (RAN).
  - User equipment emulator (UERANsim) for realistic traffic generation.
- **Objective**: Mimic real-world 5G environments to evaluate Snort3's performance.

### 3. **Central Coordination Platform**

- **Role**: Aggregates alerts from multiple Mobile Network Operators (MNOs), analyzes threats, and distributes mitigation actions.
- **Features**:
  - Alert sharing across independent MNOs.
  - Collaborative threat mitigation while preserving MNO autonomy.

### 4. **FileAgent Module**

- **Functionality**:
  - API-based rule injection into Snort3.
  - Dynamic adaptation of detection capabilities based on central platform recommendations.

## Snort3 Rules Overview

### 1. **Alert Rule**

This rule generates an alert when ICMP traffic is detected targeting a specific IP address (`10.1.6.137`). It is useful for monitoring suspicious ping activity.

```lua
alert icmp any any -> 10.1.6.137 any (msg:"[ALERT] ICMP to 10.1.6.137"; sid:10000; rev:1; priority:1;)
```

- Purpose: Detect ICMP traffic targeting 10.1.6.137.
- Action: Logs an alert with metadata (timestamp, source/destination IPs, protocol).
- Use Case: Monitoring network activity for potential reconnaissance attempts.

### 2. Drop Rule

This rule silently drops ICMP traffic targeting 10.1.6.207. It prevents communication without notifying the sender.

```lua
drop icmp any any -> 10.1.6.207 any (msg:"Drop ICMP to 10.1.6.207";  sid:10001; rev:1; priority:1;)
```

- Purpose: Block ICMP traffic targeting 10.1.6.207.
- Action: Drops packets without generating alerts.
- Use Case: Mitigating harmful ICMP traffic while minimizing exposure.

### 3. Block Rule

This rule blocks TLS traffic with a specific Server Name Indication (SNI) (**training.testerver.gr**) during the Client Hello phase of the handshake.

```lua
block ssl any any -> any any (
   msg:" Blocked TLS Client Hello with SNI: training.testerver.gr";
   ssl_state:client_hello;
   content:"|74 72 61 69 6e 69 6e 67 2e 74 65 73 74 73 65 72 76 65 72 2e 67 72|";
   sid:10002;
   rev:1;
)
```

- Purpose: Block connections to **training.testerver.gr**.
- Action: Prevents TLS handshake and logs the event.
- Use Case: Blocking access to known malicious domains.

### 4. Alert Rule for TLS Traffic

This rule generates an alert when TLS traffic with SNI issg.gr is detected during the Client Hello phase.

```lua
alert ssl any any -> any any (
    msg:"Alert TLS Client Hello with SNI: issg.gr";
    sid: 10003;
    rev: 1;
    ssl_state: client_hello;
    content:"|69 73 73 67 2e 67 72|";
)
```

- Purpose: Detect TLS traffic targeting **issg.gr**.
- Action: Logs an alert with metadata.
- Use Case: Monitoring connections to specific domains for potential threats.

---

## Evaluation Results

- **Alert Rules**: Successfully detected malicious traffic and forwarded alerts to the central platform.
- **Drop Rules**: Silently discarded harmful packets while logging events for analysis.
- **Dynamic Rule Updates**: Demonstrated seamless integration of new rules via FileAgent API.

Running the command curl

```bash
curl --interface uesimtun0 https://training.testserver.gr
```

will never return a response, indicating that the Snort3 rules are effectively blocking malicious traffic. One look of the alert_json.txt and we can confirm it.

```json
{
  "timestamp": "07/21-19:07:30.923922",
  "rule": "1:10002:1",
  "action": "block",
  "msg": " Blocked TLS Client Hello with SNI: training.testerver.gr",
  "dst_ap": "150.140.195.229:443",
  "src_ap": "10.45.0.2:54036"
}
```

## Contributions

- **Collaborative Security Model**: Enables cross-operator threat coordination without compromising individual MNO independence.
- **Real-Time Detection**: Validates Snort3's effectiveness in 5G-specific scenarios.
- **Dynamic Rule Management**: Supports rapid response to evolving threats.

## References

For detailed implementation and source code, visit:

- [FileAgent-SAND5G Repository](https://github.com/ISSG-UPAT/FileAgent-SAND5G)
- [Snort Docker Repository](https://github.com/ISSG-UPAT/Snort-Docker-SAND5G)
- [FileAgent Docker Repository](https://github.com/ISSG-UPAT/FileAgent-Docker-SAND5G)
