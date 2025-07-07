# 🚀 Out-of-Band Management (OBM) Project Overview

## 📌 Project Summary
- **Project Name:** ILO Project (renamed to OBM)
- **Timeline:** 2012–2013
- **Goal:** Enable full remote server lifecycle management after relocating hardware to North Carolina while admins remained in Arizona.

### 🔍 Why It Mattered
Managing servers from thousands of miles away isn't just about remote logins. Admins needed:
- Post-event diagnostics during boot-up
- BIOS/firmware updates and reboots
- Mounting of ISO/USB devices remotely
- Full power control and network provisioning

These actions couldn’t be reliably executed with traditional tools like KVM, leading to the birth of a dedicated Out-of-Band strategy.

---

## 🔐 SSL and Security Challenges

### ❌ Initial Roadblock
The project ground to a halt early—SSL issues:
- **Context:** Some OOB-capable devices required SSL ≤ 1.2 for AD authentication.
- **Policy:** Org-wide ban on SSL below v1.2 due to vulnerabilities like man-in-the-middle attacks.

### 🧠 The Breakthrough
- Reviewed policy line-by-line to scope exceptions
- Found an **F5 device** on the management network
- Proposed it serve as a secure proxy for AD auth—eliminating direct server-to-AD communication

> 📎 **Security Insight:**
> Policy exceptions were precisely worded to allow SSL 1.2 *only* on the management network. All tunnel endpoints were authenticated and logged via the F5.

---

## 🛠️ Technical Execution

### ✅ Hardware Compatibility
OBM cards on HP, Dell, IBM (ILO, IMM, DRAC)

#### 🔁 BIOS/Firmware Update Strategy
| Scenario | Action |
|---------|--------|
| Version below AuthReq, no reboot required | Upgrade to highest non-reboot version |
| Version below AuthReq, reboot required | Upgrade as far as possible, preconfigure, log incomplete |
| Stepped upgrades | Applied where version skips were unsupported (e.g., v1 → v3 required v2 first) |

```bash
# Sample logic pseudo-code for update decisions
if [ "$requires_reboot" = false ] && [ "$meets_authreq" = false ]; then
  apply_stepped_update
elif [ "$requires_reboot" = true ]; then
  partial_update
  configure_hostname_and_ip
  log_incomplete
fi
```
## 🔐 Authentication Design

### 🎯 Objectives
- Use AD groups for access management
- Apply read-only by default
- Existing AD groups could elevate privileges with temporary admin access

> 🛡️ **Security Detail:** Authentication was layered using existing group privileges rather than expanding them. This ensured tight RBAC compliance while allowing controlled elevation when needed.

---

## 💻 Supported Platforms

| OS Family | Versions                              |
|-----------|----------------------------------------|
| Windows   | NT4, 2000, 2008, 2012                  |
| VMware    | 4.1, 5.0 (Bash & remote PowerShell)    |
| Linux     | Red Hat (Korn/Bash tooling for config) |

> 🎓 **Historical Note:** Bash was more primitive in 2012, especially for XML or memory-based configs. Korn proved better for structured blobs.

---

## 🧰 Tooling
- Vendor-specific ILO/IMM/DRAC tools
- Enterprise run tools
- Notepad++ for script edits and diffing

---

## 📝 Policies & Governance

Included full documentation, handoff, and revision of:
- DNS and Firewall policy
- Server deployment & decommission procedures
- F5 communication flow
- Management network segmentation
- Change control documentation

---

## 👥 Stakeholders Involved

| Group            | Role                                          |
|------------------|-----------------------------------------------|
| F5 Admins        | Secure proxy configuration                    |
| DNS Admins       | Assisted in build processes via time-sharing  |
| Security Groups  | Firewall & SSL policy approval                |
| Deployment Managers | Approved procedural changes                |
| Datacenter Ops   | Ran scripts for existing server upgrades      |
| VP-Level Execs   | Final approval on high-impact change orders   |

---

## 🔁 Execution & Maintenance

### New Servers
- OBM cards configured during deployment

### Existing Servers
- Updated via scripted rollout coordinated by Datacenter Ops

---

## 🧠 Programming Language Usage

| Language    | Purpose                                                          |
|-------------|------------------------------------------------------------------|
| Perl        | Used for script launching; full stack limited by module policies |
| Batch       | Legacy NT4 scripting                                             |
| PowerShell  | Versioned scripts tailored to OS specs                           |
| Korn        | XML blob management on Linux systems                             |
| Bash        | Host info, navigation, binary execution, Korn launcher           |

```Ksh
# Korn example: XML generation
echo "<config>" > server_config.xml
echo "<hostname>$HOSTNAME</hostname>" >> server_config.xml
echo "</config>" >> server_config.xml
```
###📋 Error Handling & Logging
Try/Catch Logic: Captures BIOS/FW states, IPs, hostnames
Error Codes: Matched against known matrix; unknown codes flagged
Logs: Separated into success/failure logs with detailed tracebacks

```bash # Bash snippet for exit logging
log_exit() {
  echo "$HOSTNAME | $IP | BIOS:$BIOS_VERSION | FW:$FW_VERSION" >> success.log
}
```














