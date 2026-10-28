# Evidence Collection: Methods for Documenting AI Data Access Violations

## Overview

This document provides detailed methods for collecting, preserving, and presenting evidence of AI model data access violations for legal proceedings and regulatory complaints.

## Types of Evidence

### 1. Visual Evidence

#### Screenshots
**Purpose**: Document permission dialogs, consent flows, and UI elements

**Best Practices**:
- Capture full screen, not just portions
- Include timestamp (system clock)
- Include browser URL or app window title
- Capture before and after states
- Use high-resolution screenshots

**Tools**:
- macOS: Cmd+Shift+4 (screenshot), Cmd+Shift+5 (screen recording)
- Windows: Win+Shift+S (screenshot), Win+G (screen recording)
- Linux: gnome-screenshot, kazam

**Documentation Template**:
```
Screenshot ID: SS-001
Date/Time: [ISO 8601 timestamp]
Capture Method: [tool used]
Content: [description of what's shown]
Context: [what action led to this state]
Relevance: [why this is important to the case]
```

#### Screen Recordings
**Purpose**: Document dynamic consent flows, loading states, and transitions

**Best Practices**:
- Record entire consent flow from start to finish
- Include mouse cursor and clicks
- Record with system audio if applicable
- Include system clock in recording
- Keep recordings unedited (or document edits)

**Tools**:
- macOS: QuickTime Player, OBS Studio
- Windows: OBS Studio, Game Bar (Win+G)
- Linux: OBS Studio, SimpleScreenRecorder

**Documentation Template**:
```
Recording ID: REC-001
Date/Time: [ISO 8601 timestamp]
Duration: [length of recording]
Capture Method: [tool used]
Content: [description of recorded sequence]
Context: [what actions were performed]
Relevance: [why this is important]
```

### 2. Network Evidence

#### Packet Captures
**Purpose**: Document network traffic, API calls, and data transmission

**Collection Method**:
```bash
# Start capture
tcpdump -i any -w evidence.pcap host ai-service.com

# During capture, perform actions to be documented
# Stop capture when complete

# Verify capture
tcpdump -r evidence.pcap -v
```

**Metadata to Record**:
- Capture start and end times
- Network interface used
- Filter criteria applied
- System configuration
- IP addresses involved

**Analysis**:
```bash
# Analyze with Wireshark
wireshark evidence.pcap

# Extract HTTP objects
tshark -r evidence.pcap --export-objects http,./extracted

# Generate statistics
tshark -r evidence.pcap -q -z io,phs
```

#### HTTPS Traffic (MITM)
**Purpose**: Capture decrypted HTTPS traffic

**Setup**:
```bash
# Install mitmproxy
pip install mitmproxy

# Start proxy
mitmproxy --listen-port 8080 -w mitm-traffic.mitm

# Configure system/browser to use proxy
# Install CA certificate

# During capture, perform actions
# Stop capture
```

**Considerations**:
- Certificate installation required
- Some apps detect and block MITM
- Legal considerations for intercepting traffic
- Document proxy configuration

### 3. API Evidence

#### OAuth Token Analysis
**Purpose**: Document granted permissions and scopes

**Collection**:
```javascript
// Browser console
// Get OAuth token from storage
const token = localStorage.getItem('auth_token');

// Decode JWT
const decoded = JSON.parse(atob(token.split('.')[1]));
console.log('Scopes:', decoded.scope);
console.log('Permissions:', decoded.permissions);
```

**Documentation**:
- Token type (access, refresh, etc.)
- Issuer and audience
- Granted scopes
- Expiration time
- How token was obtained

#### API Call Logs
**Purpose**: Document specific API calls and data accessed

**Collection**:
```python
import requests
import json

# Log API calls
class APILogger:
    def __init__(self):
        self.log = []

    def log_call(self, method, url, headers, body, response):
        self.log.append({
            'timestamp': datetime.now().isoformat(),
            'method': method,
            'url': url,
            'headers': headers,
            'body': body,
            'response': response
        })

# Use wrapper
logged_requests = APILogger()
response = logged_requests.log_call('GET', url, headers, None, response)
```

### 4. File System Evidence

#### File Access Logs
**Purpose**: Document which files AI applications access

**Collection**:
```bash
# Linux
auditctl -w /home/user/Documents -p rwxa -k ai-access
ausearch -k ai-access > file-access.log

# macOS
sudo log show --predicate 'process == "ai-service"' --info
sudo log show --predicate 'eventMessage contains "Documents"' --info

# Windows
# Use Process Monitor with filters
# Export to CSV
```

**Documentation**:
- File paths accessed
- Timestamp of access
- Read/write operations
- Process responsible
- User context

#### Cloud Storage Logs
**Google Drive**:
- Export Drive Activity Log
- Filter by application
- Filter by date range
- Export to CSV

**OneDrive**:
- Microsoft 365 Audit Log
- Filter by file access
- Export logs

**Dropbox**:
- Event Log export
- Filter by app activity
- Download logs

### 5. System Evidence

#### Process Activity
**Purpose**: Document what processes are running and what they do

**Collection**:
```bash
# Linux
ps aux > processes.log
top -b -n 1 > top.log
lsof -p <pid> > lsof.log

# macOS
ps aux > processes.log
sample <pid> 10 > sample.log

# Windows
tasklist > processes.log
tasklist /v > detailed-processes.log
```

#### System Configuration
**Purpose**: Document system state at time of audit

**Collection**:
```bash
# Linux
uname -a > system-info.log
df -h > disk-usage.log
free -h > memory-usage.log

# macOS
system_profiler SPSoftwareDataType > system-info.log
df -h > disk-usage.log

# Windows
systeminfo > system-info.log
wmic logicaldisk get > disk-usage.log
```

### 6. Documentary Evidence

#### Terms of Service
**Purpose**: Document legal agreements at time of consent

**Collection**:
- Archive using Internet Archive (web.archive.org)
- Save PDF copies
- Include version dates
- Document access date

**Documentation**:
```
Document ID: TOS-001
Document Type: Terms of Service
Source: [URL]
Access Date: [date]
Version Date: [date if known]
Archived Copy: [URL to archive]
Key Clauses: [relevant sections]
```

#### Privacy Policies
**Purpose**: Document privacy commitments

**Collection**:
- Archive using Internet Archive
- Save PDF copies
- Note version dates
- Document changes over time

#### Permission Descriptions
**Purpose**: Document how permissions are described to users

**Collection**:
- Screenshots of permission dialogs
- Text of permission descriptions
- Context in which permissions are requested
- UI/UX analysis

## Evidence Preservation

### Hashing
**Purpose**: Prove evidence hasn't been altered

**Method**:
```bash
# Create hash file
sha256sum evidence.pcap > evidence.pcap.sha256
sha256sum screenshot.png > screenshot.png.sha256
sha256sum recording.mp4 > recording.mp4.sha256

# Verify hash
sha256sum -c evidence.pcap.sha256
```

**Documentation**:
```
Evidence ID: EVID-001
File: evidence.pcap
SHA-256: [hash]
Creation Date: [date]
Verification Date: [date]
Status: [verified/altered]
```

### Timestamping
**Purpose**: Establish when evidence was created

**Method**:
```bash
# Use trusted timestamping authority
openssl ts -query -data evidence.pcap -out evidence.pcap.tsq
curl -H "Content-Type: application/timestamp-query" \
     --data-binary @evidence.pcap.tsq \
     https://freetsa.org/tsr > evidence.pcap.tsr
```

### Secure Storage
**Best Practices**:
- Store in multiple locations
- Use encrypted storage
- Maintain read-only copies
- Document storage locations
- Regular integrity checks

### Chain of Custody
**Documentation Template**:
```
Evidence ID: EVID-001
Item: [description]

Custody Log:
- [Date/Time] Collected by [name] from [location]
- [Date/Time] Transferred to [name] at [location]
- [Date/Time] Stored at [location] by [name]
- [Date/Time] Accessed by [name] for [purpose]
- [Date/Time] Transferred to [legal counsel] at [location]

Current Custodian: [name]
Current Location: [location]
Last Verification: [date]
```

## Evidence Organization

### Directory Structure
```
EVIDENCE/
├── SCREENSHOTS/
│   ├── permission-dialogs/
│   ├── consent-flows/
│   └── ui-elements/
├── RECORDINGS/
│   ├── consent-flows/
│   └── demos/
├── NETWORK/
│   ├── packet-captures/
│   ├── mitm-traffic/
│   └── api-logs/
├── FILES/
│   ├── file-access-logs/
│   └── cloud-storage-logs/
├── SYSTEM/
│   ├── process-logs/
│   └── system-config/
├── DOCUMENTS/
│   ├── terms-of-service/
│   ├── privacy-policies/
│   └── permission-descriptions/
└── METADATA/
    ├── hashes/
    ├── timestamps/
    └── chain-of-custody/
```

### Naming Convention
- Use descriptive names
- Include date/timestamp
- Use consistent format
- Example: `SS-001-permission-dialog-2024-01-15-143022.png`

### Indexing
Create a master index:
```
Evidence Index
===============

ID          Type          Description                    Date          Location
----------- ------------- ------------------------------- ------------- ----------------------
SS-001      Screenshot    Gmail permission dialog         2024-01-15    SCREENSHOTS/permission-dialogs/
REC-001     Recording     Full consent flow                2024-01-15    RECORDINGS/consent-flows/
PCAP-001    Packet Cap    Network traffic to AI service   2024-01-15    NETWORK/packet-captures/
```

## Legal Presentation

### Evidence Reports
**Structure**:
1. Executive Summary
2. Methodology
3. Findings
4. Evidence Catalog
5. Analysis
6. Conclusions
7. Appendices (raw evidence)

### Expert Reports
**Components**:
1. Expert qualifications
2. Scope of examination
3. Methodology used
4. Findings
5. Technical analysis
6. Conclusions
7. Attachments

### Exhibit Preparation
- Label each exhibit clearly
- Provide context for each exhibit
- Explain technical terms
- Include authentication statements
- Prepare witness testimony

## Privacy and Ethics

### Redaction
- Redact personal information not relevant to case
- Redact third-party information
- Redact sensitive credentials
- Document what was redacted and why

### Anonymization
- Anonymize user data where possible
- Use pseudonyms for individuals
- Anonymize IP addresses if not needed
- Document anonymization process

### Legal Review
- Have legal counsel review evidence
- Ensure evidence collection was legal
- Verify chain of custody is complete
- Confirm evidence is admissible

## Tools

### Screen Capture
- macOS: Built-in screenshot tools
- Windows: Snipping Tool, Game Bar
- Linux: gnome-screenshot, flameshot

### Network Analysis
- Wireshark: https://www.wireshark.org
- tcpdump: https://www.tcpdump.org
- mitmproxy: https://mitmproxy.org

### File Analysis
- Hex editors: HxD, hexdump
- Log analysis: grep, awk, sed
- Timeline analysis: Plaso

### Hashing
- sha256sum: Built-in on Linux/macOS
- CertUtil: Windows built-in
- OpenSSL: Cross-platform

## Next Steps

1. Review [LEGAL-ANALYSIS.md](./LEGAL-ANALYSIS.md) for legal context
2. Review [AUDIT-FRAMEWORK.md](./AUDIT-FRAMEWORK.md) for technical methods
3. Explore [TEMPLATES/](./TEMPLATES/) for legal document templates
4. See [CASE-STUDIES.md](./CASE-STUDIES.md) for examples

## Disclaimer

This document provides guidance on evidence collection. Ensure all evidence collection activities comply with applicable laws and regulations. Consult legal counsel before collecting evidence for legal proceedings.
