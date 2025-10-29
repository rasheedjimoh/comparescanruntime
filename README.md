# 🔬 Port Scanner Runtime & Results Comparator (Bash)

## Overview
A compact benchmarking script that runs **three popular port scanners** (`nmap`, `masscan`, and `recon-ng`) against a single target and reports:
- how long each tool took to complete (wall-clock seconds), and  
- which open ports each tool detected.

This is a practical utility for comparing tool runtimes and quick result parity checks during lab work, tool evaluations, or when tuning scan parameters.

---

## Key Features
- ⏱️ **Runtime measurement** for each scanner (simple wall-clock seconds).  
- 🔍 **Result aggregation** — extracts “open” ports and prints them as comma-separated lists.  
- ⚖️ **Quick parity check** — helps you see differences between scanners (coverage vs speed trade-offs).  
- 🧰 **Lightweight & shell-native** — single-file Bash script that orchestrates external tools.

---

## Dependencies
Make sure these tools are installed and properly configured on the host that runs the script:
- `nmap` (privileged operations for SYN scan may require sudo)  
- `masscan` (typically requires root privileges and proper rate tuning)  
- `recon-ng` with the `scanner/portscan/tcp` module available  
- Common text utilities: `grep`, `awk`, `tr`, `sed`, `date`

---

## Why this script is useful
- Useful for research and benchmarking: compare default behaviour and performance across scanners.  
- Helps evaluate trade-offs: `masscan` (extreme speed, asynchronous), `nmap` (balanced accuracy & features), `recon-ng` (framework convenience).  
- Great for lab notes when documenting which scanner detected which services during an engagement or a lab exercise.

---

## Usage
```bash
chmod +x compare_scanners.sh
./compare_scanners.sh <target_host>
# Example:
./compare_scanners.sh 192.168.1.10
````

**Output example**

```
Nmap took 12 seconds to run and found the following open ports: 22/tcp,80/tcp,443/tcp
Masscan took 2 seconds to run and found the following open ports: 80,443
Recon-ng took 18 seconds to run and found the following open ports: 22,8080
```

---

## Caveats & Measurement Notes (Read before using)

* **Different scan techniques**: `nmap -sS` (SYN) behaves differently than `masscan` (asynchronous raw packets) and `recon-ng` modules; results are not 1:1 comparable by default.
* **Privilege requirements**: `masscan` and SYN scans often require root/administrator privileges to send raw packets. Run with caution.
* **Parsing fragility**: Output parsing relies on `grep`/`awk` and may break if scanner output formats differ by version or locale.
* **Timing granularity**: The script uses `date +%s` (whole-second granularity). For more precise benchmarking, use `date +%s%N` or the `time` command.
* **Impact on networks**: High-speed scanning (especially masscan) can trigger IDS/IPS alerts or degrade network performance. Only scan targets you own or have explicit permission to test.

---

## Safety & Ethics (must-read)

* **Only scan targets you own or are authorised to test.** Unauthorized scanning may be illegal and can disrupt networks.
* Use conservative settings on production networks, and notify defenders/stakeholders where appropriate.

---

## Author & Context

**Author:** Rasheed Jimoh

**Purpose:** Tool evaluation and lab benchmarking for offensive security learning and tool selection.

**Intended use:** Controlled lab environments and authorised penetration tests only.

---

## Script
```
#!/bin/bash

# Check if the required parameters are provided
if [ $# -ne 1 ]; then
 echo "Usage: $0 <target_host>"
 exit 1
fi

# Define the target host
target_host=$1

# Define the current date and time
current_time=$(date +%s)

# Run Nmap
start_nmap=$(date +%s)
nmap_ports=$(nmap -sS $target_host | grep open | awk '{print $1}' | tr
'\n' ',' | sed 's/,$//')
end_nmap=$(date +%s)
nmap_time=$((end_nmap - start_nmap))

# Run Masscan
start_masscan=$(date +%s)
masscan_ports=$(masscan $target_host | grep open | awk '{print $3}' |
tr '\n' ',' | sed 's/,$//')
end_masscan=$(date +%s)
masscan_time=$((end_masscan - start_masscan))

# Run Recon-ng
start_recon=$(date +%s)
recon_ports=$(recon-ng --no-check -m scanner/portscan/tcp --workspace
default -e RHOSTS=$target_host | grep open | awk '{print $2}' | tr '\n'
',' | sed 's/,$//')
end_recon=$(date +%s)
recon_time=$((end_recon - start_recon))

# Print the results
echo "Nmap took $nmap_time seconds to run and found the following open
ports: $nmap_ports"
echo "Masscan took $masscan_time seconds to run and found the following
open ports: $masscan_ports"
echo "Recon-ng took $recon_time seconds to run and found the following
open ports: $recon_ports"
