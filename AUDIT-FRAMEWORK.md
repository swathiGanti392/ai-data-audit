# Audit Framework: Technical Methods for Monitoring AI Data Access

## Overview

This document provides a comprehensive technical framework for auditing AI model data access, including methods for monitoring, logging, and proving unauthorized or excessive data access to personal information.

## Audit Objectives

1. **Identify** what data AI models access
2. **Document** when and how data is accessed
3. **Prove** authorization or lack thereof
4. **Quantify** the scope of data access
5. **Preserve** evidence for legal proceedings

## Technical Approaches

### 1. Network Traffic Analysis

#### Method 1: Packet Capture
**Tools**: Wireshark, tcpdump, tshark

**Setup**:
```bash
# Install Wireshark
brew install wireshark  # macOS
sudo apt install wireshark  # Linux

# Capture network traffic
sudo tcpdump -i any -w ai-traffic.pcap host ai-service.com
```

**What to Capture**:
- API endpoints called
- Data payloads (if unencrypted)
- Request/response headers
- Timestamps
- IP addresses

**Limitations**:
- HTTPS encryption obscures content
- Need to decrypt SSL/TLS (if possible)
- May require root/admin privileges

#### Method 2: MITM Proxy
**Tools**: mitmproxy, Charles Proxy, Fiddler

**Setup**:
```bash
# Install mitmproxy
pip install mitmproxy

# Start proxy
mitmproxy --listen-port 8080

# Configure browser/system to use proxy
# Install CA certificate
```

**What to Capture**:
- Full HTTPS traffic (decrypted)
- Request/response bodies
- Cookies and tokens
- API calls and data

**Considerations**:
- Requires certificate installation
- Some apps pin certificates (bypass difficult)
- AI services may detect and block

#### Method 3: Browser DevTools
**Tools**: Chrome DevTools, Firefox Developer Tools

**Setup**:
- Open browser DevTools (F12)
- Navigate to Network tab
- Use AI service
- Capture network activity

**What to Capture**:
- Network requests
- Response data
- Headers
- Timing information

**Limitations**:
- Browser-based only
- Doesn't capture desktop app traffic
- Session-based

### 2. Application-Level Monitoring

#### Method 1: Browser Extensions
**Tools**: Custom extension, Request Monitor

**Architecture**:
```javascript
// manifest.json
{
  "manifest_version": 3,
  "permissions": ["webRequest", "storage"],
  "host_permissions": ["<all_urls>"],
  "background": {
    "service_worker": "background.js"
  }
}

// background.js
chrome.webRequest.onBeforeRequest.addListener(
  (details) => {
    if (isAIService(details.url)) {
      logRequest(details);
    }
  },
  {urls: ["<all_urls>"]},
  ["requestBody"]
);
```

**What to Monitor**:
- API calls to AI services
- Data sent in requests
- Response data
- Permission checks

**Deployment**:
- Load unpacked extension
- Monitor AI service usage
- Export logs for analysis

#### Method 2: Mobile App Monitoring
**Tools**: mitmproxy with mobile device, Charles Proxy, Frida

**iOS Setup**:
```bash
# Install mitmproxy
pip install mitmproxy

# Configure iOS device proxy
# Install mitmproxy CA certificate
# Enable full trust for certificate
```

**Android Setup**:
```bash
# Install mitmproxy
pip install mitmproxy

# Configure Android device proxy
# Install mitmproxy CA certificate
# Enable SSL inspection
```

**What to Monitor**:
- App network traffic
- API calls
- Data transmission
- Third-party SDKs

#### Method 3: Desktop Application Monitoring
**Tools**: Process Monitor (ProcMon), Wireshark, API Monitor

**Windows**:
```bash
# Download Process Monitor
# Run ProcMon
# Filter by AI application
# Monitor file, registry, network access
```

**macOS**:
```bash
# Use Instruments
# Monitor process activity
# Track file and network access
```

**Linux**:
```bash
# Use strace
strace -f -e trace=file,network -o ai-service.log ai-service-app

# Use lsof
lsof -p <pid> > ai-service-lsof.log
```

### 3. API Access Logging

#### Method 1: OAuth Token Analysis
**Tools**: jwt.io, Custom scripts

**Setup**:
```python
import jwt
import base64

# Decode OAuth token
token = "your-oauth-token"
decoded = jwt.decode(token, options={"verify_signature": False})

# Analyze scopes
print("Scopes:", decoded.get("scope"))
print("Permissions:", decoded.get("permissions"))
```

**What to Analyze**:
- OAuth scopes granted
- Permission claims
- Token expiration
- Issuer information

#### Method 2: Google API Audit
**Tools**: Google Cloud Console, API Usage Dashboard

**Setup**:
1. Go to Google Cloud Console
2. Navigate to API & Services
3. View API usage
4. Export activity logs

**What to Audit**:
- Which APIs are called
- How often
- What data is accessed
- Authentication method

#### Method 3: Microsoft Graph API Audit
**Tools**: Microsoft 365 Admin Center, Azure AD

**Setup**:
1. Go to Microsoft 365 Admin Center
2. Navigate to Audit Log Search
3. Filter by API activity
4. Export logs

**What to Audit**:
- Graph API calls
- Mail access
- Calendar access
- Contact access

### 4. File System Monitoring

#### Method 1: File Access Monitoring
**Tools**: inotify (Linux), FSEvents (macOS), DirectoryWatcher (Windows)

**Linux**:
```bash
# Install inotify-tools
sudo apt install inotify-tools

# Monitor directory
inotifywait -m -r /home/user/Documents --format '%w%f %e'
```

**macOS**:
```bash
# Use fswatch
brew install fswatch

# Monitor directory
fswatch -r /home/user/Documents
```

**Windows**:
```powershell
# Use PowerShell
Get-ChildItem -Path C:\Users\user\Documents -Recurse |
    ForEach-Object { Write-Host $_.FullName }
```

**What to Monitor**:
- File reads by AI applications
- File modifications
- File deletions
- Metadata access

#### Method 2: Cloud Storage Monitoring
**Google Drive**:
- Use Google Drive Activity Log
- Export activity reports
- Filter by application access

**OneDrive**:
- Use Microsoft 365 Audit Log
- Filter by file access
- Export logs

**Dropbox**:
- Use Dropbox Event Log
- Monitor file access
- Export activity

### 5. Database Access Monitoring

#### Method 1: Application Database Monitoring
**Tools**: Database logs, Query monitoring

**Setup**:
```sql
-- Enable query logging (PostgreSQL)
ALTER SYSTEM SET log_statement = 'all';
SELECT pg_reload_conf();

-- Enable slow query log (MySQL)
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
```

**What to Monitor**:
- SQL queries executed
- Tables accessed
- Data retrieved
- Query frequency

#### Method 2: Cloud Database Monitoring
**AWS RDS**:
- Enable Performance Insights
- Enable audit logging
- Export CloudWatch logs

**Google Cloud SQL**:
- Enable audit logs
- Use Cloud Logging
- Export to BigQuery

**Azure SQL**:
- Enable audit logging
- Use Azure Monitor
- Export logs

### 6. System-Level Auditing

#### Method 1: System Call Tracing
**Linux**:
```bash
# Use strace
strace -f -e trace=file,network,process -o ai-service.log ./ai-service

# Use auditd
sudo auditctl -w /etc/passwd -p wa -k ai-service
sudo ausearch -k ai-service
```

**macOS**:
```bash
# Use dtrace
sudo dtrace -n 'syscall:::entry /execname == "ai-service"/ { printf("%s", probefunc); }'
```

**Windows**:
```powershell
# Use Process Monitor
# Download and run ProcMon
# Filter by process name
# Monitor system calls
```

#### Method 2: Registry/Config Monitoring
**Windows**:
```bash
# Use Process Monitor
# Monitor registry access
# Filter by registry keys
```

**macOS**:
```bash
# Use opensnoop
sudo opensnoop -n ai-service
```

**Linux**:
```bash
# Use auditd
sudo auditctl -w /etc/ -p r -k config-access
```

## Evidence Collection Process

### Step 1: Preparation
1. Identify AI services to audit
2. Choose appropriate monitoring tools
3. Set up logging infrastructure
4. Configure data storage for logs
5. Ensure proper timezone/time synchronization

### Step 2: Baseline Establishment
1. Capture normal system activity
2. Document current permissions
3. Record initial system state
4. Establish baseline network traffic
5. Document AI service configuration

### Step 3: Active Monitoring
1. Start monitoring tools
2. Use AI service normally
3. Capture all relevant data
4. Monitor for anomalies
5. Document specific actions

### Step 4: Evidence Preservation
1. Export logs in immutable format
2. Create cryptographic hashes
3. Store in secure location
4. Document chain of custody
5. Backup evidence

### Step 5: Analysis
1. Parse and analyze logs
2. Identify data access patterns
3. Correlate with user actions
4. Identify unauthorized access
5. Generate reports

## Evidence Integrity

### Hashing
```bash
# Create SHA-256 hash
sha256sum ai-traffic.pcap > ai-traffic.pcap.sha256

# Verify hash
sha256sum -c ai-traffic.pcap.sha256
```

### Timestamping
```bash
# Use trusted timestamping
openssl ts -query -data ai-traffic.pcap -out ai-traffic.pcap.tsq
openssl ts -reply -in ai-traffic.pcap.tsq -out ai-traffic.pcap.tsr
```

### Chain of Custody Documentation
- Document who collected evidence
- Document when evidence was collected
- Document how evidence was collected
- Document storage locations
- Document any changes to evidence

## Legal Admissibility

### Authentication
- Prove logs are accurate
- Document logging methodology
- Show logs haven't been altered
- Expert testimony on logging system

### Chain of Custody
- Document evidence handling
- Show continuous control
- Document storage and access
- No gaps in custody

### Expert Testimony
- Technical experts to explain evidence
- Explain logging methodology
- Interpret technical data
- Establish reliability

## Privacy Considerations

### Ethical Auditing
- Only audit your own data
- Don't expose others' personal data
- Follow ethical guidelines
- Respect terms of service

### Data Minimization
- Collect only necessary evidence
- Redact sensitive information
- Anonymize when possible
- Secure storage of evidence

### Legal Compliance
- Ensure auditing is legal
- Don't violate computer fraud laws
- Don't intercept encrypted communications illegally
- Consult legal counsel if unsure

## Tools and Utilities

### Network Monitoring
- Wireshark: https://www.wireshark.org
- tcpdump: https://www.tcpdump.org
- mitmproxy: https://mitmproxy.org
- Charles Proxy: https://www.charlesproxy.com

### Application Monitoring
- Chrome DevTools: Built into Chrome
- Process Monitor: https://learn.microsoft.com/sysinternals
- strace: Linux built-in
- dtrace: macOS built-in

### Log Analysis
- ELK Stack: https://www.elastic.co
- Splunk: https://www.splunk.com
- Graylog: https://www.graylog.org

### API Monitoring
- Postman: https://www.postman.com
- Insomnia: https://insomnia.rest
- API Gateway logging

## Next Steps

1. Review [EVIDENCE-COLLECTION.md](./EVIDENCE-COLLECTION.md) for detailed evidence methods
2. Explore [TOOLS/](./TOOLS/) for specific auditing utilities
3. Review [LEGAL-ANALYSIS.md](./LEGAL-ANALYSIS.md) for legal context
4. See [CASE-STUDIES.md](./CASE-STUDIES.md) for examples

## Disclaimer

This document provides technical guidance for auditing AI data access. Ensure all auditing activities comply with applicable laws and regulations. Consult legal counsel if unsure about legality of specific methods.
