# Windows Event Viewer : RDP BruteForce Attack Detection & Log Analysis.

## Project Overview ~
This homelab project demonstrates how to simulate a Red Team RDP brute-force attack using **Hydra** via Kali Linux and subsequently analyze the defensive security posture using Windows **Event Viewer**. The goal is to understand how security auditing catches malicious login attempts, how to isolate attack indicators.

### Technologies & Tools Used ~
* **Target Machine :** Windows Machine
* **Attacker Machine :** Kali Linux
* **Attack Tool :** Hydra
* **Wordlist :** RockYou.txt
* **Defensive Tool :** Windows Event Viewer
---

## Step-by-Step Implementation ~

### Phase 1 : Target Environment Setup (Windows)
To allow the attack simulation, Remote Desktop Protocol (RDP) must be enabled on the target system :
1. Open **System Properties** by pressing `Win + R`, typing `sysdm.cpl`, and hitting Enter.
2. Navigate to the **Remote** tab.
3. Select **"Allow remote connections to this computer"**.
4. Click **Select Users** to add specific test accounts under the Remote Desktop Users group (administrator)
5. Ensure that the **Windows Defender Firewall** permits incoming RDP traffic over **TCP port 3389**.


### Phase 2 : Network Verification & Attacker Setup (Kali Linux)
1. Ensure both the Windows and Kali Linux machines are configured on the same network interface (NAT network / Shared network).
2. Open the command prompt (cmd) on the Windows machine and run `ipconfig` to verify its IP address & Ping the IP to confirm connectivity.

#### Download and Unzip the Wordlist :
```bash
# Download wordlist
sudo apt update && sudo apt install wordlists -y

# Extract the rockyou.txt.gz file
cd /usr/share/wordlists/
sudo gunzip rockyou.txt.gz
```
### Phase 3 : Executing the BruteForce Attack
1. From the Kali Linux terminal, launch Hydra
against the target Windows machine using the
targeted administrator username and the
unzipped wordlist :

```bash
hydra -t 4-V-l Administrator -P /usr/share/
wordlists/rockyou.txt rdp://192.168.232.131
```
• -t 4 : Sets parallel tasks to 4.
• -V : Enables verbose mode to display username & password combinations in real
time.
• -l Administrator : Targets the "Administrator" user account.
• -P : Specifies the path to the password wordlist (rockyou.txt)


### Phase 4 : Blue Team Log Analysis (Event Viewer)
• Identifying the Attack Indicators :
1. Open Event Viewer on the Windows machine.
2. Expand Windows Logs and select the "Security log".
3. Clear past logs to isolate the attack session,
then refresh to look for a rapid influx of Audit Failure events occurring within seconds of each other.

• Filtering and Sorting Logs because  bruteforce attack generates thousands of logs, filtering can narrow downs the investigation :
1. Click "Filter Current Log" in the right hand
Actions pane.
2. Filter by Event ID = 4625 (An account failed
to log on).
3. We can see log Property to know more.

---

▪︎ How to Export Logs for External Analysis To preserve evidence for a security report. 
1. Open Event Viewer and navigate to the Security log
2. Select the specific range of attack events or
the entire log module.
3. Click "Save All Events As" in the Actions pane.
4. Name the file (e.g. bruteforce_simulation.evtx) and select a
destination folder to export.

▪︎ How to Review Exported Logs on Another Device
1. Open Event Viewer on any windows machine.
2. In the right-hand panel, click on Open "Saved Log".
3. Browse and select your exported .evtx file.
4. The logs will load natively, allowing you to
filter, sort, and perform full forensics exactly
as if you were on the target machine.
