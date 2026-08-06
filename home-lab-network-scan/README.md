#Home Lab Network Reconnaissance

Date(s): 08/02/2026 –08/05/2026
Status: Complete
Environment:Home Lab (VirtualBox - Kali Linux attacker VM, Metasploitable2 target VM, Host-Only network)
Tools used: Nmap

---

## 1. Objective

Goal: Perform network reconnaissance against a target host to identify open ports, running services, and service versions, then research known vulnerabilities associated with those services.

Why this matters: Identifying exposed services and their associated risks is the foundational first step of a security assessment. You can't evaluate or remediate risk you haven't first discovered. This project demonstrates that discovery-and-analysis process in a controlled environment.

---

## 2. Scope

- In scope: 
  - Scanning Metasploitable2 (target) from Kali Linux (attacker) via Nmap for open ports and service/version detection.
  - Researching publicly known vulnerabilities (CVEs) associated with identified services
- Out of scope:
  - Active exploitation of any identified vulnerabilities (no attempts to gain access, execute code, or otherwise act on findings — identification and risk assessment only)
- Assumptions / constraints:
  - Network isolated via Host-Only Adapter, eliminating connectivity to the production/home network
  - Metasploitable2 is an intentionally vulnerable practice target — findings reflect deliberately unpatched services, not typical real-world conditions
- Authorization:
  - Self-authorized — all systems involved (Kali, Metasploitable2) are personally owned and operated within an isolated home lab environment

---

## 3. Method

| Step | Action | Tool/Command | Result / Observation |
|------|--------|--------------|----------------------|
| 1 |Changed network adapter on both VMs from NAT Network to Host-Only Adapter to isolate lab from production network |VirtualBox Network Settings |Both VMs configured on isolated Host-Only network |
| 2 |Verified connectivity between attacker and target machines |ifconfig (metasploitable2), ping -c 4 192.168.56.101 (Kali) |Metasploitable2 IP confirmed as 192.168.56.101; 4/4 ping replies received, 0% packet loss |
| 3 |Performed port scan with service/version detection against target |nmap -sV 192.168.56.101 |23 open TCP ports identified with associated services and versions (see Findings for full list and analysis) |

---

## 4. Findings

- Finding 1: Port 21 (vsftpd 2.3.4) — Backdoored FTP Service
  - Severity: Critical
  - Details: Nmap identified vsftpd version 2.3.4 running on port 21. This exact version is
documented as a compromised build, distributed with a backdoor inserted into the source code
by an unknown intruder between June 30 and July 3, 2011, before being redistributed through
official download channels. The trigger for this backdoor is unusually specific: sending an
FTP username ending in a smiley face (":)") causes the compromised binary to open a shell on
TCP port 6200. This finding is confirmed through documentation rather than direct triggering,
consistent with this project's scope.

  - CVE research note: CVE-2011-2523 confirms this vulnerability and matches the exact version
identified by Nmap. Rapid7's own Metasploitable2 exploitability guide independently confirms
this backdoor is one of the intentional vulnerabilities included in the target machine, which
is a strong source given Rapid7 maintains Metasploitable2 itself.

  - Verification note: The original Nmap scan was reviewed for port 6200 to check whether the
backdoor shell had already been triggered. It was not present in the scan results, consistent
with the backdoor not having been triggered, since the connection was not attempted.

  - Risk: If triggered, this backdoor grants an unauthenticated remote shell, equivalent in
severity to the port 1524 finding. Because the trigger only requires a specifically crafted FTP username, no valid credentials or complex exploitation technique are required, making this
finding trivially exploitable if the FTP service is reachable.

  - References:
    - CVE-2011-2523: https://www.cve.org/CVERecord?id=CVE-2011-2523
    - Rapid7 Metasploitable2 Exploitability Guide: https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/

- Finding 2: Telnet (Port 23) - Unencrypted Remote Access Protocol
  - Severity: High
  - Details: Nmap identified Linux telnetd (netkit-telnet 0.17-35ubuntu1) running on port 23. Telnet transmits all data, including login credentials, in unencrypted plaintext. Any party able to observe network traffic between client and server can capture credentials or session data directly, with no need to break encryption since none is used. This is a fundamental design limitation of the protocol, not a version-specific bug; SSH has been the standard secure replacement for over two decades.

  - CVE research note: Recent 2026 CVEs affecting telnetd implementations (e.g. CVE-2026-24061, CVE-2026-32746) were investigated but confirmed not applicable. Those affect GNU Inetutils telnetd, while this host runs netkit-telnet 0.17-35ubuntu1, a different codebase.

  - References:
    - CVE-2026-24061: https://www.cve.org/CVERecord?id=CVE-2026-24061 (researched and ruled out — affects GNU Inetutils telnetd, not the netkit-telnet package confirmed on this target)
    - CVE-2026-32746: https://www.cve.org/CVERecord?id=CVE-2026-32746 (researched and ruled out — affects GNU Inetutils telnetd, not the netkit-telnet package confirmed on this target)

- Finding 3: R-Services (Ports 512, 513, 514) — Unencrypted Remote Access Suite
  - Severity: High
  - Details: Nmap identified rexecd (port 512), rlogind (port 513), and rshd (port 514) running on the target. The package providing all three services was confirmed directly on the Metasploitable2 host using `dpkg -l | grep rsh`, which identified rsh-server version 
  0.17-14ubuntu1. These are the Berkeley r-services, predecessors to SSH from the early 1980s. All three transmit authentication credentials and session data in unencrypted plaintext. In addition, rsh supports a trust-based authentication model using a file called .rhosts, which can be configured to allow login from specific remote hosts with no password at all, an even weaker authentication method than a plaintext password.

  - CVE research note: Searched cve.org for rsh-server. No CVE was found that directly targets rexecd, rlogind, or rshd. CVE-2019-7283 affects rcp in NetKit through version 0.17, the same package family and a matching version number, but it concerns a separate tool (rcp, used for remote file copying) rather than the three services scanned here. Included for context, not as the basis of this finding. The primary risk documented above is the inherent lack of encryption and weak authentication model shared by all three services, independent of any specific CVE.

  References:
    - CVE-2019-7283: https://www.cve.org/CVERecord?id=CVE-2019-7283 (related context — affects rcp in NetKit through 0.17, matching package family and version, but a different tool than rexecd/rlogind/rshd)
    - CVE-2018-19518: https://www.cve.org/CVERecord?id=CVE-2018-19518 (researched and ruled out — affects the University of Washington IMAP Toolkit's use of rsh as a helper process, not the rshd service itself)

- Finding 4: Port 1524 (ingreslock/bindshell) — Unauthenticated Root Shell Backdoor
  - Severity: Critical
  - Details: Nmap identified a service on port 1524 labeled "Metasploitable root shell." 
Investigation confirmed the port is served via xinetd (PID 4450), mapped to the service name 
ingreslock (/etc/services). The backdoor definition was located in /etc/inetd.conf:

    ingreslock stream tcp nowait root /bin/bash bash -i

  This configures xinetd to launch an interactive root shell with no authentication whenever a connection is made to this port.

  - Scope deviation: To verify the finding directly rather than rely solely on public documentation, a single confirmatory connection was made from Kali to the target on port 1524 using netcat, followed by one command (whoami). This constitutes active exploitation, which falls outside this project's stated Out of Scope boundary. The exception was made deliberately and narrowly, to confirm a specific finding rather than to explore further. The connection immediately returned a root shell with no authentication prompt, confirmed by whoami returning root.

  - Risk: This represents the most severe possible finding: complete, unauthenticated administrative access to the host, requiring no credentials, no vulnerability chaining, and no prior access. Any party able to reach this port on the network can gain full control of the system.

  - CVE research note: No CVE applies to this finding. Rapid7's own Metasploitable2 exploitability guide confirms this behavior is a deliberately planted training backdoor, not a flaw in real-world software with a vendor or patch cycle, so there is nothing for a CVE to be assigned to. This was also confirmed independently by locating the exact backdoor definition in /etc/inetd.conf rather than assuming its absence from search results meant incomplete research.

  - References:
    - Rapid7 Metasploitable2 Exploitability Guide: https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/ (confirms the ingreslock backdoor is an intentionally included vulnerability in Metasploitable2)

- Finding 5: Port 3306 (MySQL 5.0.51a-3ubuntu5) — Blank Root Password
  - Severity: Critical
  - Details: Nmap identified MySQL version 5.0.51a-3ubuntu5 running on port 3306. A direct cve.org search for this version returned no results, which is consistent with this being a configuration weakness rather than a code-level vulnerability, configuration issues are typically not assigned CVEs. Rapid7's own Metasploitable2 exploitability guide confirms the machine is documented as having weak password security across both system and database accounts, specifically noting the MySQL root account is configured with a blank password. This finding is documented based on this source rather than direct connection, consistent with this project's scope.

  - Risk: If accurate, a blank root password on the database service would allow any party able to reach port 3306 to authenticate as the MySQL administrative account with no credentials at all, granting full read and write access to all databases on the host. This is comparable in severity to the port 1524 finding, since it represents a complete authentication bypass, though scoped to the database service rather than the operating system shell.

  - References:
    - Rapid7 Metasploitable2 Exploitability Guide: https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/ (documents weak/blank password configuration for MySQL and other services on Metasploitable2)
---

## 5. Impact / Risk

Taken together, the five findings in this assessment do not represent five isolated issues, they represent multiple independent and mutually reinforcing paths to full compromise of the target host.

Three findings (the port 1524 bindshell, the port 21 vsftpd backdoor, and the port 3306 blank 
MySQL root password) each independently grant complete, unauthenticated access on their own, with no chaining or prior access required. An attacker does not need to find a single "correct" vulnerability, any one of these three is sufficient alone, meaning remediation of only one or two would leave the host fully compromised.

The remaining two findings (telnet and the r-services suite) do not grant direct access on their own, but instead expose any credentials used over those protocols in plaintext to anyone able to observe network traffic. Since credentials are frequently reused across multiple services on the same host, a single captured password has the potential to compromise additional systems beyond the service it was captured from, extending the impact of this finding well past the two protocols themselves.

In combination, this target has no single point of failure to remediate. It has several, and 
addressing any one in isolation would leave the host critically exposed through the others.

---

## 6. Remediation

| # |Finding | Recommended Fix | Status | Owner (if applicable) |
|---|--------|-----------------|--------|-----------------------|
| 1 | vsftpd 2.3.4 backdoor (port 21) | Uninstall the compromised vsftpd binary and reinstall a current, verified version of vsftpd from an official, trusted source. Verify the integrity of the downloaded package (e.g. checksum or package manager signature verification) before deployment, since this vulnerability specifically resulted from a compromised distribution in the past. | Not remediated (documentation exercise, no changes applied to target) | N/A |
| 2 | Telnet (port 23) | Disable the telnet service. SSH is already available on port 22 and provides an encrypted alternative for remote administrative access, so telnet serves no purpose that isn't already covered more securely. | Not remediated (documentation exercise, no changes applied to target) | N/A |
| 3 | R-services (ports 512, 513, 514) | Remove the rsh-server package (`sudo apt remove rsh-server`), which provides rexecd, rlogind, and rshd. This is a separate package from telnet and requires its own explicit removal. SSH already covers the legitimate remote login and remote command execution use cases these services were originally designed for. | Not remediated (documentation exercise, no changes applied to target) | N/A |
| 4 | ingreslock/bindshell backdoor (port 1524) | Remove the ingreslock entry from /etc/inetd.conf and restart the xinetd/inetd service. This entry launches an unauthenticated root shell and serves no legitimate purpose; SSH already provides secure remote access. | Not remediated (documentation exercise, no changes applied to target) | N/A |
| 5 | MySQL blank root password (port 3306) | Set a strong, unique password for the MySQL root account (e.g. via mysqladmin -u root password) rather than leaving it blank. Additionally, restrict which hosts are permitted to connect to the database service, rather than allowing connections from any network location. | Not remediated (documentation exercise, no changes applied to target) | N/A |

---

## 7. Evidence
All evidence is stored in the `evidence/` subfolder, organized into `before/`, `after/`, and 
`logs/`.

Baseline setup (evidence/before/):
- `network-adapter-metasploitable2.png` — Host-Only Adapter configuration
- `network-adapter-kali.png` — Host-Only Adapter configuration
- `ip-and-ping-confirmation.png` — Metasploitable2 IP address and successful connectivity test from Kali

Full scan (evidence/logs/):
- `nmap-scan-full-output.png` — full Nmap scan results (referenced by Findings 1, 3, and 5)

Finding 3 — R-services (evidence/logs/):
- `rsh-server-package-confirmation.png` — dpkg confirmation of rsh-server 0.17-14ubuntu1

Finding 4 — ingreslock/bindshell (evidence/logs/):
- `port1524-xinetd-process.png` — netstat output confirming xinetd on port 1524
- `port1524-service-name-mapping.png` — /etc/services mapping to ingreslock
- `port1524-inetd-conf-backdoor-definition.png` — backdoor definition found in /etc/inetd.conf
- `port1524-netcat-root-shell-proof.png` — confirmed unauthenticated root access via netcat

---

## 8. Lessons Learned

- Defining scope before starting work is only half the discipline, holding to it once you 
encounter something you're eager to verify hands-on is the harder half. This project included a deliberate exception (the port 1524 finding) where the temptation to directly confirm a finding conflicted with the stated Out of Scope boundary. Documenting the deviation honestly, rather than quietly treating it as if it were always permitted, preserved the credibility of the rest of the document.

- Evidence should be captured as work happens, not reconstructed afterward. Early in this project, a network configuration change and connectivity test were performed before any documentation had begun, requiring the evidence to be recreated rather than captured in the moment. No harm resulted here since the steps were easily reproducible, but in a more complex or time-sensitive project, that gap could mean losing evidence entirely.

- A CVE or vulnerability report matching a service name or protocol is not the same as it matching the actual software running on the target. Several points in this project (GNU Inetutils telnetd versus the netkit-telnet package actually installed, and a NetKit rcp CVE versus the rsh-server package actually running) initially looked applicable but were ruled out only after confirming the exact package and version on the host directly, rather than relying on the CVE's description alone.

- Filtered searches can silently hide the answer you're looking for. A grep-filtered search of /etc/inetd.conf for "1524" returned no results, while a plain, unfiltered read of the same file revealed the entry clearly. When a targeted search comes up empty, it's worth trying a broader, unfiltered look before concluding the information isn't there.

---

