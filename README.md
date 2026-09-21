# Applied Cyber Security — Practical Lab Manual
### Department of Computer Science and Engineering (Cyber Security) | Session 2026–27 | Semester VII
### Subject: Applied Cyber Security (23BTCYP401)

**Lab Environment:** 1 × Kali Linux VM (Attacker/Analyst machine) + 1 × Windows Server 2022 VM (Target/Defender machine)

---

## ⚠️ Safety & Ethics Disclaimer (Read First)

- **All practicals in this manual must be performed only within your own isolated lab environment** — the two VMs described above, connected on a **Host-only / Internal virtual network** with no exposure to the public internet or any third-party system.
- **Never** run any tool, scan, exploit, phishing page, or credential harvester described here against a real person, real company, or any system you do not own or have explicit written authorization to test.
- Use only **dummy/fake data** for usernames, passwords, and personal information during demonstrations (e.g., `lab-user` / `DummyPass123`).
- Unauthorized use of these techniques against real targets may violate the **IT Act 2000 (India)**, the **Computer Misuse Act**, the **CFAA**, or equivalent laws in your jurisdiction.
- This manual is for **educational purposes only**, to teach detection, defense, and incident-response skills.

---

## Table of Contents

1. [Lab Setup Prerequisites](#lab-setup-prerequisites)
2. [Practical 1 — OSINT: Maltego, Shodan, theHarvester](#practical-1--collect-intelligence-about-a-target-using-open-source-tools-maltego-shodan-theharvester)
3. [Practical 2 — Phishing & Credential Harvesting](#practical-2--demonstrate-how-phishing-emails-trick-users-and-collect-credentials)
4. [Practical 3 — Detect Suspicious Activity via Log Analysis](#practical-3--detect-suspicious-activities-by-analyzing-logs)
5. [Practical 4 — Firewall Rules: Configure & Test](#practical-4--configure-and-test-firewall-rules-for-blocking-unauthorized-access)
6. [Practical 5 — Packet Capture & Analysis](#practical-5--capture-and-analyze-network-packets-for-suspicious-behavior)
7. [Practical 6 — Automated Backup & Restore](#practical-6--configure-automated-backups-and-restore-data-after-a-failure)
8. [Practical 7 — Recovery Procedures During Outage](#practical-7--test-recovery-procedures-during-system-or-network-outage)
9. [Practical 8 — Ransomware Simulation & Restore](#practical-8--simulate-a-ransomware-attack-and-restore-systems-without-paying-ransom)
10. [Practical 9 — Threat Monitoring Dashboards & Alerts (Wazuh SIEM)](#practical-9--create-dashboards-and-alerts-for-threat-monitoring)
11. [S10 — Open-Source Practical: Vulnerability Assessment + SCA + File Integrity Monitoring](#s10--open-source-practical-vulnerability-detection--sca--file-integrity-monitoring-fim)
12. [Suggested Time Allocation](#suggested-time-allocation)
13. [References](#references)

---

## Lab Setup Prerequisites

| Requirement | Detail |
|---|---|
| VM 1 | Kali Linux (latest rolling release recommended) |
| VM 2 | Windows Server 2022 (Desktop Experience) |
| Network | Both VMs on the same **Host-only** or **Internal** virtual switch/network |
| Privileges | Administrator access on Windows Server; root/sudo access on Kali |
| Find Kali's IP | `ip a` |
| Find Windows IP | `ipconfig` |
| Recommended | Take a VM snapshot/checkpoint of both VMs **before** starting, so you can roll back cleanly after each class |

> 💡 Throughout this manual, replace `<Kali-IP>` and `<Windows-IP>` with the actual IP addresses of your VMs.

---

## Practical 1 — Collect Intelligence About a Target Using Open-Source Tools (Maltego, Shodan, theHarvester)

**Course Outcome Mapping:** CO-1
**Objective:** Demonstrate passive reconnaissance (OSINT) — gathering publicly available information about a target without touching its network.

**Safe target for practice:** `zonetransfer.me` (a domain purpose-built for OSINT practice by DigiNinja) — or your own owned domain. **Never** target a real organization without written authorization.

### A) theHarvester

**Step 1 — Verify installation** (pre-installed on Kali):
```bash
theHarvester -h
```

**Step 2 — Run a basic scan:**
```bash
theHarvester -d zonetransfer.me -l 200 -b crtsh,duckduckgo -f /home/kali/osint_report
```
| Flag | Meaning |
|---|---|
| `-d` | Target domain |
| `-l` | Result limit per source |
| `-b` | Data source(s) — use `all` to query every source |
| `-f` | Filename prefix for saved report |

**Step 3 — Advanced scan with DNS resolution + subdomain brute force:**
```bash
theHarvester -d zonetransfer.me -b crtsh -n -c -f /home/kali/osint_report2
```

**Output/Report Location:**
| Item | Location |
|---|---|
| Terminal output | Live in the terminal |
| HTML report | `/home/kali/osint_report.html` |
| XML report | `/home/kali/osint_report.xml` |
| View HTML report | `firefox /home/kali/osint_report.html` |

**Cleanup:**
```bash
rm /home/kali/osint_report.html /home/kali/osint_report.xml
```
*(Read-only tool — no target-side changes to revert.)*

---

### B) Shodan

**Step 1:** Go to `https://shodan.io` in a browser → create a free account.

**Step 2 — Demo search queries:**
```
port:3389 country:"IN"
webcam
apache city:"Nagpur"
```

**Step 3:** Click any result to inspect the raw **banner**, open ports, service versions, and geolocation.

**Output Location:** Results remain in your Shodan web dashboard (bulk export/download requires a paid API plan).

**Cleanup:** None required — purely a web search, no local or target changes.

---

### C) Maltego

**Step 1 — Launch:**
```bash
maltego
```

**Step 2:** Select **Maltego CE (Free)** → accept license → register/log in with a free Community Edition account.

**Step 3 — Install Transforms:** Transform Hub → install `Shodan`, `WhoisXML`, and built-in DNS transforms (some require a free API key from the respective service, pasted into Maltego's transform settings).

**Step 4 — Build a graph:**
1. New Graph → drag a **Domain** entity onto canvas → set value to `zonetransfer.me`.
2. Right-click → run transforms: **"To DNS Name – NS"**, **"To Email address"**, **"To IP Address [DNS]"**.
3. Watch the graph expand, visually connecting discovered subdomains, IPs, and emails.

**Output Location:**
| Item | Location |
|---|---|
| Maltego config/cache | `~/.maltego/` |
| Saved graph file | Wherever you choose, e.g., `~/Desktop/OSINT_Graph.mtgx` |
| Exported image | Right-click canvas → Export → Image → path you choose |

**Cleanup:**
```bash
rm -rf ~/Desktop/OSINT_Graph.mtgx
rm -rf ~/.maltego     # optional full reset for next class
```

### Teaching Notes
- All three tools are **passive** — no packets beyond normal DNS/web queries are sent to the target, so nothing needs to be reverted on the target side.
- Discussion idea: have students search their own college domain (with permission) to see what's publicly discoverable.

---

## Practical 2 — Demonstrate How Phishing Emails Trick Users and Collect Credentials

**Course Outcome Mapping:** CO-1, CO-2
**Objective:** Simulate a credential-harvesting phishing attack in a fully isolated lab to teach the mechanics of phishing and how to spot it.

### ⚠️ Rules for this practical
- Stay entirely within your Kali ↔ Windows Server lab network.
- Never point this at the public internet or a real login page/email address.
- Use only dummy credentials, e.g. `lab-user` / `DummyPass123`.

### Steps

**Step 1 — Launch the Social-Engineer Toolkit (SET) on Kali:**
```bash
sudo setoolkit
```

**Step 2 — Navigate the menu:**
```
1) Social-Engineering Attacks
2) Website Attack Vectors
3) Credential Harvester Attack Method
2) Site Cloner
```

**Step 3:** When prompted for the **IP for POST back**, enter Kali's IP (e.g., `192.168.56.101`).

**Step 4:** When prompted for the **URL to clone**, use a simple dummy login page you built yourself (do not clone a real company's live site).

**Step 5:** SET clones the page and starts a Python web server (default port 80) with a listener.

**Step 6 — On the Windows Server 2022 VM:** open a browser → go to `http://<Kali-IP>` (simulating a user clicking a phishing link).

**Step 7:** Enter dummy credentials (`lab-user` / `DummyPass123`) → click Login. The page typically redirects to the real cloned site afterward — this is exactly what makes real phishing convincing.

### Where to View Captured Credentials/Logs
| Item | Location |
|---|---|
| Live terminal output | Appears instantly in the SET terminal on Kali |
| Saved harvester report | `/root/.set/reports/` (timestamped `.txt`/`.xml` files) |
| View the log | `sudo cat /root/.set/reports/<latest-report-file>` |
| SET config file | `/etc/setoolkit/set.config` |

### Optional: Spear-Phishing Email Vector
```
1) Social-Engineering Attacks
1) Spear-Phishing Attack Vectors
```
Send only to a lab mailbox you control — never a real external address.

### Cleanup / Revert
```bash
# Stop the listener: Ctrl+C in the SET terminal, then exit via menu (99 repeatedly)

# Delete captured credential logs
sudo rm -rf /root/.set/reports/*

# Remove cloned site files
sudo rm -rf /root/.set/web_clone/

# Confirm port 80 is free again
sudo netstat -tulnp | grep :80
```
On the Windows VM: clear browser history/cache (`Ctrl+Shift+Delete`) so dummy credentials aren't cached.

### Teaching Wrap-Up — Red Flags to Discuss
- Mismatched/suspicious URL (not the real domain)
- No HTTPS padlock or a certificate warning
- Urgent/alarming language
- Generic greeting instead of the user's real name
- Credential request via link instead of navigating directly to the known site

---

## Practical 3 — Detect Suspicious Activities by Analyzing Logs

**Course Outcome Mapping:** CO-2
**Objective:** Generate and analyze Windows Security logs to detect brute-force/suspicious login patterns.

### Prerequisite
Enable RDP on Windows Server 2022: Server Manager → Local Server → Remote Desktop → Enable.

### Steps

**Step 1 — On Kali, generate failed login attempts:**
```bash
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://<Windows-IP>
```
Let it run for 10–20 attempts, then `Ctrl+C` (no need to exhaust the wordlist).

**Step 2 — On Windows Server 2022, open Event Viewer:**
```
Win + R → eventvwr.msc → Enter
Navigate: Windows Logs → Security
```

**Step 3 — Filter for failed logons:**
Right-click **Security** → **Filter Current Log** → Event ID: `4625` → OK

**Step 4:** Inspect an entry's **General** tab: Account Name, Source Network Address (Kali's IP), Logon Type, timestamp.

**Step 5 — PowerShell alternative (scriptable):**
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} |
  Select-Object TimeCreated, @{n='SourceIP';e={$_.Properties[19].Value}}, @{n='Account';e={$_.Properties[5].Value}} |
  Format-Table -AutoSize
```

### Where to View Logs
| Item | Location |
|---|---|
| Live log file on disk | `C:\Windows\System32\winevt\Logs\Security.evtx` |
| GUI viewer | Event Viewer → Windows Logs → Security |
| Export filtered results | Event Viewer → right-click Security → **Save Filtered Log File As...** (`.evtx`, `.csv`, `.xml`) |
| PowerShell CSV export | `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} \| Export-Csv C:\Temp\FailedLogons.csv -NoTypeInformation` |

### Cleanup / Revert
```powershell
# Clear the Security log (explain to students why this is itself suspicious in real production!)
wevtutil cl Security

# Delete exported files
Remove-Item C:\Temp\FailedLogons.csv -Force
```

### Troubleshooting
If Event ID 4625 doesn't appear, confirm Audit Policy has logon-failure auditing enabled:
`secpol.msc → Local Policies → Audit Policy → Audit logon events → Failure`

### Teaching Point
Multiple 4625 events from the same source IP within seconds/minutes = classic brute-force signature. It's the **pattern**, not a single event, that a SOC analyst flags.

---

## Practical 4 — Configure and Test Firewall Rules for Blocking Unauthorized Access

**Course Outcome Mapping:** CO-2
**Objective:** Create, test, and verify a Windows Firewall rule that blocks a specific IP, with logging enabled to prove the block worked.

### Steps

**Step A — Baseline test (before blocking), on Kali:**
```bash
ping <Windows-IP>
nmap -p 3389 <Windows-IP>
```

**Step B — Enable firewall logging on Windows Server:**
1. `Win + R` → `wf.msc` → Enter.
2. Right-click root node → **Properties** → select network profile tab (check active profile via `Get-NetConnectionProfile`) → **Logging → Customize**.
3. Set **Log dropped packets: Yes**, size limit 20480 KB, note log path → OK → OK.

PowerShell equivalent:
```powershell
Set-NetFirewallProfile -Profile Domain,Private,Public -LogBlocked True -LogFileName "%systemroot%\system32\LogFiles\Firewall\pfirewall.log" -LogMaxSizeKilobytes 20480
```

**Step C — Create the block rule (GUI):**
1. `wf.msc` → **Inbound Rules** → **New Rule**.
2. **Custom** → Next → **All programs** → Next.
3. Protocol: **Any** (or TCP port 3389 for RDP only) → Next.
4. **Scope** → under "remote IP addresses" click **Add** → enter Kali's IP → Next.
5. Action: **Block the connection** → Next → leave all profiles checked → Next.
6. Name: `Block-Kali-Demo` → Finish.

**Step D — PowerShell alternative:**
```powershell
New-NetFirewallRule -DisplayName "Block-Kali-Demo" -Direction Inbound -RemoteAddress <Kali-IP> -Action Block -Protocol Any
```

**Step E — Re-test from Kali:**
```bash
ping <Windows-IP>          # should now time out
nmap -p 3389 <Windows-IP>  # port shows filtered
```

### Where to View Logs
| Item | Location |
|---|---|
| Firewall log (blocked packets) | `C:\Windows\System32\LogFiles\Firewall\pfirewall.log` |
| Quick path shortcut | `wf.msc` → **Monitoring** (left pane) → clickable hyperlink to log path |
| View contents | `notepad C:\Windows\System32\LogFiles\Firewall\pfirewall.log` |
| Log fields | date, time, action (ALLOW/DROP), protocol, src-ip, dst-ip, src-port, dst-port, size, tcpflags, path |

### Cleanup / Revert
```powershell
# Disable or delete the rule
Remove-NetFirewallRule -DisplayName "Block-Kali-Demo"

# Confirm access restored
# (from Kali) ping <Windows-IP>

# Turn off firewall logging (optional)
Set-NetFirewallProfile -Profile Domain,Private,Public -LogBlocked False

# Clear the log file (optional)
Remove-Item "C:\Windows\System32\LogFiles\Firewall\pfirewall.log" -Force
```

### Troubleshooting Tip
The #1 reason a rule looks correct but doesn't block anything: the IP was placed under **"local IP addresses"** instead of **"remote IP addresses"** on the Scope tab. Always double-check this.

---

## Practical 5 — Capture and Analyze Network Packets for Suspicious Behavior

**Course Outcome Mapping:** CO-3
**Objective:** Capture live traffic (an Nmap scan) with Wireshark and analyze it for reconnaissance signatures.

### Steps

**Step 1 — Start Wireshark on Kali:**
```bash
sudo wireshark
```
Select the correct interface (shared subnet with Windows VM) → click the blue shark-fin **Start Capture** icon.

**Step 2 — Generate traffic (separate Kali terminal):**
```bash
ping -c 4 <Windows-IP>
nmap -sS -p 1-1000 <Windows-IP>
```

**Step 3 — Stop the capture** (red square icon) once the scan completes.

**Step 4 — Apply display filters:**
```
icmp                                        # view ping requests/replies
tcp.flags.syn==1 && tcp.flags.ack==0        # isolate Nmap's SYN-scan packets
```
Right-click a packet → **Follow → TCP Stream** → view full conversation.

**Step 5 — Save the capture:** File → Save As → choose filename/location.

**CLI alternative (tshark):**
```bash
sudo tshark -i eth0 -w /home/kali/capture_demo.pcap -a duration:30
```

### Where to View Output
| Item | Location |
|---|---|
| Live capture (unsaved) | In memory only until saved |
| Saved capture file | Your chosen path, e.g., `/home/kali/capture_demo.pcapng` |
| Reopen later | `wireshark /home/kali/capture_demo.pcapng` |

### Cleanup / Revert
```bash
# Stop any active capture: Ctrl+E in Wireshark

# Delete demo capture files if not needed
rm /home/kali/capture_demo.pcapng
```
No configuration changes are made to either VM — packet capture is passive.

### Teaching Point
Many SYN packets to sequential ports from one source IP in a short time window = the classic fingerprint of port-scanning reconnaissance, exactly what an IDS/IPS or SOC analyst flags.

---

## Practical 6 — Configure Automated Backups and Restore Data After a Failure

**Course Outcome Mapping:** CO-3
**Objective:** Install Windows Server Backup, run/schedule a backup, simulate data loss, and restore.

### Steps

**Step A — Install the feature:**
```powershell
Install-WindowsFeature -Name Windows-Server-Backup
```

**Step B — Create test data:**
```powershell
New-Item -Path "C:\DemoData" -ItemType Directory
"This is important test data" | Out-File "C:\DemoData\important.txt"
```

**Step C — Run a one-time backup (GUI):**
1. Server Manager → Tools → **Windows Server Backup**.
2. **Backup Once** → **Different options** → Next.
3. **Custom** → Next → Add Items → select `C:\DemoData` → OK → Next.
4. Destination: separate volume (e.g., attached virtual disk `E:`) or network share → Next → **Backup**.

**Step D — Command-line backup:**
```powershell
wbadmin start backup -backupTarget:E: -include:C:\DemoData -quiet
```
For a **recurring scheduled** backup:
```powershell
wbadmin enable backup -addtarget:E: -schedule:22:00 -include:C:\DemoData -quiet
```

**Step E — Simulate a failure:**
```powershell
Remove-Item "C:\DemoData\important.txt" -Force
```

**Step F — Check available backup versions:**
```powershell
wbadmin get versions
```
Note the exact **VersionIdentifier** printed (e.g., `03/11/2026-09:07`).

**Step G — Restore (GUI):**
1. Windows Server Backup → **Recover** → **This server** → select backup date/version → Next.
2. **Files and folders** → Next → browse to `C:\DemoData` → select `important.txt` → Next.
3. Destination: **Original location** (or "Another location" to compare safely) → Next → **Recover**.

**Step H — Command-line restore:**
```powershell
wbadmin start recovery -version:<VersionIdentifier> -itemtype:File -items:C:\DemoData\important.txt -recoverytarget:C:\DemoData
```

### Where to View Logs/Output
| Item | Location |
|---|---|
| Backup catalog (what exists) | Query via `wbadmin get versions` |
| Actual backup data | `E:\WindowsImageBackup\<ComputerName>\` |
| Job status | Windows Server Backup GUI → Local Backup node |
| Live progress | `wbadmin get status` (blocks until job finishes) |
| App/system event logs | Event Viewer → `Applications and Services Logs → Microsoft → Windows → Backup` |

### Cleanup / Revert
```powershell
# Delete test data
Remove-Item "C:\DemoData" -Recurse -Force

# Disable scheduled backup if configured
wbadmin disable backup

# Delete a specific old backup version
wbadmin delete backup -version:<VersionIdentifier> -backupTarget:E:

# If catalog gets corrupted during testing (rare, good to teach)
wbadmin delete catalog -quiet
wbadmin restore catalog -backupTarget:E: -quiet

# Full removal (optional, for a clean baseline)
Uninstall-WindowsFeature -Name Windows-Server-Backup
```

### Important Note
`wbadmin get versions` (no `-backupTarget`) reads the local **catalog** — the server's record of what it believes it backed up, not the actual media. If catalog and media drift apart, query the media directly with `-backupTarget:E:`.

### Teaching Point
Reinforces **RPO** (how much data you can afford to lose) and **RTO** (how fast you can restore) — core business continuity concepts.

---

## Practical 7 — Test Recovery Procedures During System or Network Outage

**Course Outcome Mapping:** CO-4
**Objective:** Simulate a service/network failure and prove recovery works.

> **Note:** A true multi-node Windows Failover Cluster test requires 2+ Windows Server nodes with shared storage — not achievable with a single Windows Server VM. The drill below is the standard single-server substitute used in academic labs and fully covers the learning outcome. If you want a real cluster failover demo, add a second Windows Server VM temporarily.

### Part A — Simulate a Network Outage

**Step 1 — Baseline (from Kali):**
```bash
ping <Windows-IP>
```

**Step 2 — Simulate outage (on Windows Server):**
```powershell
Get-NetAdapter                     # note adapter name, e.g. "Ethernet"
Disable-NetAdapter -Name "Ethernet" -Confirm:$false
```

**Step 3 — Confirm outage (from Kali):**
```bash
ping <Windows-IP>   # should time out
```

**Step 4 — Recover:**
```powershell
Enable-NetAdapter -Name "Ethernet" -Confirm:$false
```

**Step 5 — Verify recovery (from Kali):**
```bash
ping <Windows-IP>   # should succeed
```

### Part B — Simulate a Critical Service Crash & Auto-Recovery

**Step 1:** `services.msc` → pick a test service (e.g., Print Spooler) → **Properties → Recovery** tab:
- First failure: Restart the Service
- Second failure: Restart the Service
- Restart service after: 1 minute

**Step 2 — Kill it manually:**
```powershell
Stop-Process -Name spoolsv -Force
```

**Step 3 — Check auto-recovery after ~60 seconds:**
```powershell
Get-Service Spooler
```

### Part C — Full VM Crash & Restore (ties into Practical 6)

1. Take a checkpoint/snapshot of the Windows VM **before** simulating failure (Hyper-V: right-click VM → Checkpoint; VMware: Snapshot).
2. Simulate disaster: delete test data or force power-off (`Stop-VM -Name <VMName> -TurnOff`).
3. Restore via the **VM checkpoint** (fast demo) or via the **Windows Server Backup restore** from Practical 6.

### Where to View Logs
| Item | Location |
|---|---|
| Network adapter state history | Event Viewer → Windows Logs → System, Source `Tcpip`/`NETIO` |
| Service crash/restart events | Event Viewer → Windows Logs → System, Event ID **7031** (crash) / **7036** (state change) |
| Cluster validation report (multi-node only) | HTML file at path specified in `Test-Cluster -ReportName` |
| Cluster debug logs (multi-node only) | `C:\ClusterLogs\<NodeName>_cluster.log` via `Get-ClusterLog -Destination C:\ClusterLogs` |

### Cleanup / Revert
```powershell
Enable-NetAdapter -Name "Ethernet" -Confirm:$false
Get-Service Spooler | Start-Service
```
Delete the VM checkpoint once done (Hyper-V Manager → right-click checkpoint → Delete Checkpoint) to reclaim disk space. Reset the service Recovery tab to default ("Take No Action") if desired.

---

## Practical 8 — Simulate a Ransomware Attack and Restore Systems Without Paying Ransom

**Course Outcome Mapping:** CO-4
**Objective:** Safely simulate real ransomware-like file encryption, then restore using Practical 6's backup — reinforcing "restore, don't pay."

### ⚠️ Safety Note
Use **only** a known, purpose-built, harmless simulation tool such as **RanSim** (open-source PowerShell script by lawndoc) — it encrypts only a folder you specify, doesn't spread, doesn't contact a C2 server, and includes a decrypt mode. **Never** use real ransomware samples, even in isolated VMs.

### Steps

**Step 1 — Get the script (on Windows Server 2022):**
```powershell
git clone https://github.com/lawndoc/RanSim.git C:\RanSim
```
*(Or copy `RanSim.ps1` and `FileCryptography.psm1` manually via USB/shared folder if the VM has no internet access.)*

**Step 2 — Create test "victim" files:**
```powershell
New-Item -Path "C:\RanSim\TargetData" -ItemType Directory
"Confidential project report" | Out-File "C:\RanSim\TargetData\report.docx"
"Employee salary data" | Out-File "C:\RanSim\TargetData\salary.xls"
```

**Step 3 — Back up this data first:**
```powershell
wbadmin start backup -backupTarget:E: -include:C:\RanSim\TargetData -quiet
```

**Step 4 — Simulate the attack (encrypt):**
```powershell
cd C:\RanSim
.\RanSim.ps1 -Mode encrypt -TargetPath C:\RanSim\TargetData
```
This recursively encrypts files (`.docx`, `.xls`, `.pdf`, `.txt`, images, etc.) with 256-bit AES, appending `.encrypted` to filenames. Try opening `report.docx.encrypted` — it's now unreadable, just like real ransomware.

**Step 5 — The "ransom note" teaching moment:** Explain that real ransomware would demand payment here. Emphasize: **do not pay** — restore from backup instead.

**Step 6 — Restore from Practical 6's backup:**
```powershell
wbadmin get versions
wbadmin start recovery -version:<VersionIdentifier> -itemtype:File -items:C:\RanSim\TargetData -recoverytarget:C:\RanSim\TargetData
```
Verify `report.docx` opens normally — clean data restored without any attacker involvement.

**Step 7 — Alternative: use RanSim's own decrypt mode** (to contrast "having the key" vs. independent backup restore):
```powershell
.\RanSim.ps1 -Mode decrypt -TargetPath C:\RanSim\TargetData
```

### Where to View Output
| Item | Location |
|---|---|
| Encrypted files (proof of "attack") | `C:\RanSim\TargetData\*.encrypted` |
| RanSim script + module | `C:\RanSim\RanSim.ps1`, `C:\RanSim\FileCryptography.psm1` |
| Backup data | `E:\WindowsImageBackup\<ComputerName>\` |
| Windows Defender detection logs (if flagged) | Event Viewer → Applications and Services Logs → Microsoft → Windows → Windows Defender → Operational |
| File access audit events (if enabled) | Event Viewer → Security log, Event ID 4663 |

### Cleanup / Revert
```powershell
Remove-Item "C:\RanSim" -Recurse -Force

# Delete the demo backup if not needed
wbadmin delete backup -version:<VersionIdentifier> -backupTarget:E:

# If Defender quarantined/flagged the script, temporarily exclude the lab folder, then remove exclusion after
Add-MpPreference -ExclusionPath "C:\RanSim"      # only during the lab
Remove-MpPreference -ExclusionPath "C:\RanSim"   # revert after
```

### Teaching Point
This is the payoff demo for Practical 6: *"The only difference between paying a ransom and not paying is whether you have a clean, tested backup."*

---

## Practical 9 — Create Dashboards and Alerts for Threat Monitoring

**Course Outcome Mapping:** CO-5
**Objective:** Deploy a real open-source SIEM (Wazuh) with Kali as the manager and Windows Server 2022 as the monitored agent — giving live dashboards and real-time alerts.

### Steps

**Step A — Install Wazuh Manager on Kali:**
```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash wazuh-install.sh -a
```
This installs the manager, indexer, and dashboard together (all-in-one). If Kali shows an unsupported-OS warning:
```bash
sudo bash wazuh-install.sh --ignore-check
```
**Important:** Copy the auto-generated dashboard admin password shown at the end of installation — it's displayed only once.

**Step B — Open the Dashboard:**
```bash
ip a   # note Kali's IP
```
Browse to `https://<Kali-IP>` → log in with `admin` and the password from Step A.

**Step C — Register the Windows Server 2022 as an Agent:**
1. Dashboard → **Agents → Deploy new agent** → select **Windows**.
2. Enter Kali's IP as the manager address → copy the generated deployment command.
3. On Windows Server 2022:
   ```powershell
   Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.7-1.msi -OutFile wazuh-agent.msi
   msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER="<Kali-IP>"
   ```
4. Start the agent:
   ```powershell
   NET START WazuhSvc
   ```

**Step D — Verify Connection:** Dashboard → **Agents** list should show the Windows server as **Active**.

**Step E — Trigger Alerts for the Demo:**
1. Repeat the Hydra RDP brute-force test from Practical 3 (or manually enter wrong passwords a few times).
2. Dashboard → **Security Events** module → real-time alerts correlating to the failed logon attempts appear.
3. Explore built-in dashboards: File Integrity Monitoring, Vulnerability Detection, Security Configuration Assessment — these populate automatically once the agent is active (see S10 below for deep dives).

### Where to View Output/Logs
| Item | Location |
|---|---|
| Wazuh dashboard (web UI) | `https://<Kali-IP>` |
| Manager alerts (raw JSON) | `/var/ossec/logs/alerts/alerts.json` (on Kali) |
| Manager logs | `/var/ossec/logs/ossec.log` (on Kali) |
| Windows agent install path | `C:\Program Files (x86)\ossec-agent` |
| Windows agent local log | `C:\Program Files (x86)\ossec-agent\ossec-agent.log` |

### Cleanup / Revert
```powershell
# Stop the Windows agent
NET STOP WazuhSvc
# Then uninstall via Settings → Apps → "Wazuh Agent" → Uninstall
```
```bash
# Stop Wazuh services on Kali (if not needed for future classes)
sudo systemctl stop wazuh-manager wazuh-indexer wazuh-dashboard

# Full removal on Kali (only if you want a clean baseline)
sudo bash wazuh-install.sh -u
```

### Notes
- Wazuh is completely free and open source (GPLv2/Apache 2.0).
- **Resource tip:** the all-in-one stack recommends 4GB+ RAM for the dashboard component alone — ensure your Kali VM has adequate allocation.

---

## S10 — Open-Source Practical: Vulnerability Detection + SCA + File Integrity Monitoring (FIM)

**Course Outcome Mapping:** CO-5
**Objective:** Combine two open-source practical options into one session, both built directly on top of the Wazuh setup from Practical 9 — no new infrastructure required.

**Prerequisite check:**
```bash
# On Kali
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard
```
```powershell
# On Windows Server 2022
Get-Service WazuhSvc
```

---

### Part A — Vulnerability Detection & Security Configuration Assessment (SCA)

#### A1) Verify/Enable Vulnerability Detection on the Manager
**On Kali:**
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Confirm/add:
```xml
<vulnerability-detection>
  <enabled>yes</enabled>
  <index-status>yes</index-status>
  <feed-update-interval>60m</feed-update-interval>
</vulnerability-detection>
```
Confirm the indexer connector block is present:
```xml
<indexer>
  <enabled>yes</enabled>
  <hosts>
    <host>https://0.0.0.0:9200</host>
  </hosts>
  <ssl>
    <certificate_authorities><ca>/etc/filebeat/certs/root-ca.pem</ca></certificate_authorities>
    <certificate>/etc/filebeat/certs/filebeat.pem</certificate>
    <key>/etc/filebeat/certs/filebeat-key.pem</key>
  </ssl>
</indexer>
```
Restart the manager:
```bash
sudo systemctl restart wazuh-manager
```

#### A2) Enable Software Inventory Collection (Syscollector) on the Windows Agent
**On Windows Server 2022:**
```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```
Confirm/add:
```xml
<wodle name="syscollector">
  <disabled>no</disabled>
  <interval>1h</interval>
  <scan_on_start>yes</scan_on_start>
  <hardware>yes</hardware>
  <os>yes</os>
  <network>yes</network>
  <packages>yes</packages>
  <hotfixes>yes</hotfixes>
  <ports all="no">yes</ports>
  <processes>yes</processes>
</wodle>
```
Restart the agent:
```powershell
Restart-Service -Name WazuhSvc
```

#### A3) View the Vulnerability Dashboard
Dashboard (`https://<Kali-IP>`) → **Threat Intelligence → Vulnerability Detection** → select the Windows agent → view CVEs matched against installed software/hotfixes with severity, CVE ID, affected package, and reference link.

#### A4) Run a Security Configuration Assessment (SCA) Scan
SCA is enabled by default; Windows policy files ship with the agent install automatically.

Confirm the SCA block in the same `ossec.conf`:
```xml
<sca>
  <enabled>yes</enabled>
  <scan_on_start>yes</scan_on_start>
  <interval>12h</interval>
</sca>
```
Relevant CIS-benchmark policy file location:
```
C:\Program Files (x86)\ossec-agent\ruleset\sca\cis_win2019.yml
```
Restart the agent to trigger an immediate scan:
```powershell
Restart-Service -Name WazuhSvc
```
Dashboard → **Endpoint Security → Configuration Assessment** → select the Windows agent → view compliance score (%) plus a checklist of individual CIS checks (Passed/Failed, with description, rationale, and remediation).

#### Where to View Output/Logs (Part A)
| Item | Location |
|---|---|
| Vulnerability dashboard | Wazuh Dashboard → Threat Intelligence → Vulnerability Detection |
| SCA dashboard | Wazuh Dashboard → Endpoint Security → Configuration Assessment |
| Raw alerts (manager) | `/var/ossec/logs/alerts/alerts.json` (Kali) |
| Manager logs | `/var/ossec/logs/ossec.log` (Kali) |
| Windows SCA policy files | `C:\Program Files (x86)\ossec-agent\ruleset\sca\` |
| Windows agent local log | `C:\Program Files (x86)\ossec-agent\ossec-agent.log` |

#### Cleanup / Revert (Part A)
```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
# set <enabled>no</enabled> under <sca> and <wodle name="syscollector"> if desired
Restart-Service -Name WazuhSvc
```
*(These are read-only assessment modules — nothing on the Windows Server needs to be "undone" beyond disabling the modules if desired.)*

---

### Part B — File Integrity Monitoring (FIM)

#### B1) Create a Test Folder to Monitor
**On Windows Server 2022:**
```powershell
New-Item -Path "C:\WazuhTest" -ItemType Directory
```
**Important:** The folder must exist **before** restarting the agent, or Wazuh ignores it until the next restart.

#### B2) Configure Real-Time FIM on the Windows Agent
```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```
Inside `<syscheck>`, add:
```xml
<syscheck>
  <directories check_all="yes" realtime="yes" report_changes="yes">C:\WazuhTest</directories>
</syscheck>
```
| Attribute | Purpose |
|---|---|
| `realtime="yes"` | Continuous monitoring instead of waiting for scheduled scan |
| `report_changes="yes"` | Captures a diff of what changed inside modified files |
| `check_all="yes"` | Records full metadata (hashes, size, permissions, owner) |

Restart the agent:
```powershell
Restart-Service -Name WazuhSvc
```

#### B3) (Optional) Add Custom Alert Rules on the Manager
**On Kali:**
```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```
```xml
<group name="Win-syscheck,">
  <rule id="100303" level="7">
    <if_sid>550</if_sid>
    <field name="file" type="pcre2">(?i)C:\\WazuhTest</field>
    <description>File modified in the WazuhTest monitored folder</description>
  </rule>
  <rule id="100304" level="7">
    <if_sid>554</if_sid>
    <field name="file" type="pcre2">(?i)C:\\WazuhTest</field>
    <description>File added in the WazuhTest monitored folder</description>
  </rule>
</group>
```
```bash
sudo systemctl restart wazuh-manager
```

#### B4) Trigger a Live Alert
**On Windows Server 2022:**
```powershell
"Test file for FIM demo" | Out-File "C:\WazuhTest\testfile.txt"
Start-Sleep -Seconds 5
"Modified content" | Out-File "C:\WazuhTest\testfile.txt"
Start-Sleep -Seconds 5
Remove-Item "C:\WazuhTest\testfile.txt"
```
Each action (create → modify → delete) generates a separate real-time alert.

#### B5) View the Alerts Live
Dashboard → **Endpoint Security → File Integrity Monitoring** → select the Windows agent → watch alerts populate within seconds, showing event type, full file path, timestamp, and (if `report_changes` is on) a content diff.

**Bonus tie-in with Practical 8:** Add `C:\RanSim\TargetData` as an additional monitored directory, then re-run the RanSim encryption from Practical 8 — students will see a **flood of "file modified" alerts** within seconds, exactly how real EDR/FIM tools detect ransomware activity in production.

#### Where to View Output/Logs (Part B)
| Item | Location |
|---|---|
| FIM dashboard | Wazuh Dashboard → Endpoint Security → File Integrity Monitoring |
| Custom alert rules file | `/var/ossec/etc/rules/local_rules.xml` (Kali) |
| Raw FIM alerts | `/var/ossec/logs/alerts/alerts.json` (Kali) |
| Windows agent config | `C:\Program Files (x86)\ossec-agent\ossec.conf` |

#### Cleanup / Revert (Part B)
```powershell
Remove-Item "C:\WazuhTest" -Recurse -Force

notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
# remove the <directories> line for C:\WazuhTest
Restart-Service -Name WazuhSvc
```
```bash
# On Kali, optionally remove the custom rules if added only for the demo
sudo nano /var/ossec/etc/rules/local_rules.xml   # delete the <group name="Win-syscheck,"> block
sudo systemctl restart wazuh-manager
```

### Combined S10 Session Flow (~30–35 min)
| Step | Activity | Time |
|---|---|---|
| 1 | Verify Wazuh manager + agent active | 3 min |
| 2 | Enable/confirm Vulnerability Detection + Syscollector, restart agent | 5 min |
| 3 | Walk through Vulnerability Detection dashboard (CVEs found) | 7 min |
| 4 | Confirm SCA enabled, restart agent, walk through Configuration Assessment score | 7 min |
| 5 | Configure FIM on `C:\WazuhTest`, restart agent | 5 min |
| 6 | Live demo: create/modify/delete file → show real-time alerts | 5 min |
| 7 (bonus) | Point FIM at RanSim folder, re-trigger ransomware sim → show alert flood | 5 min |

**Key teaching takeaway:** Vulnerability scanning tells you *what could be exploited*, SCA tells you *what's misconfigured*, and FIM tells you *when something actually changed*. Together they form the detection layer behind every real SOC alert.

---

## Suggested Time Allocation

| # | Practical | Approx. Time |
|---|---|---|
| 1 | OSINT (theHarvester, Shodan, Maltego) | 20 min |
| 2 | Phishing / Credential Harvester | 20 min |
| 3 | Log analysis (failed logons) | 20 min |
| 4 | Firewall rules | 15 min |
| 5 | Packet capture & analysis | 15 min |
| 6 | Backup & restore | 20 min |
| 7 | Outage/recovery drill | 15 min |
| 8 | Ransomware simulation & restore | 20 min |
| 9 | Wazuh SIEM dashboard & alerts | 25–30 min |
| S10 | Vulnerability + SCA + FIM (combined) | 30–35 min |

**Total: ~3.5–4 hours** — recommended to split across 2 sessions if your workshop slot is limited to 2 hours per sitting.

**Time-saving combo tip:** Practicals 2 and 3's generated traffic (phishing + brute-force) can both be captured live during Practical 5's Wireshark session if you want to merge demos.

---

## References

- Wazuh Documentation — [documentation.wazuh.com](https://documentation.wazuh.com)
- Microsoft Learn — `wbadmin` command reference
- Microsoft Learn — Windows Firewall with Advanced Security
- TrustedSec — Social-Engineer Toolkit (SET)
- lawndoc/RanSim — [github.com/lawndoc/RanSim](https://github.com/lawndoc/RanSim)
- theHarvester — [github.com/laramies/theHarvester](https://github.com/laramies/theHarvester)
- Maltego — [maltego.com](https://www.maltego.com)
- Shodan — [shodan.io](https://www.shodan.io)

---

*This lab manual was prepared for the Applied Cyber Security (23BTCYP401) course, Semester VII, Department of Computer Science and Engineering (Cyber Security). For educational use only.*
