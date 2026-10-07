# Common Home PC Fixes for Windows

## A Practical Guide to Cleaning Up and Maintaining Your Personal Windows PC

Especially useful after removing work accounts, software, or policies — or just for regular home maintenance.

---

## **1. Remove Work Accounts & Profiles**

- **Settings → Accounts → Access work or school** → Disconnect any remaining organizational accounts
- **Settings → Accounts → Other users** → Remove old work user profiles
- **Control Panel → User Accounts → Manage another account** → Delete work profiles (back up data first!)

---

## **2. Clean Up Work Software**

- **Settings → Apps → Installed apps** → Uninstall:
  - VPN clients (Cisco AnyConnect, GlobalProtect, Pulse Secure, etc.)
  - Endpoint agents (CrowdStrike, SentinelOne, Microsoft Defender for Endpoint, Tanium, etc.)
  - MDM/Intune Company Portal
  - Remote desktop tools (TeamViewer, AnyDesk, LogMeIn if installed by work)
  - Certificate-based WiFi profiles (eduroam, corporate WiFi)
- **Task Manager → Startup** → Disable any lingering work applications

---

## **3. Reset Network & Certificates**

- **Settings → Network & Internet → Advanced network settings → Network reset** (reboots and reinstalls network adapters)
- **certmgr.msc** → Personal → Certificates → Delete old work/client certificates
- **Control Panel → Internet Options → Content → Certificates** → Clear work-related certificates
- **WiFi Settings** → Manage known networks → Forget corporate SSIDs (eduroam, company WiFi, etc.)

---

## **4. Reclaim Storage & Clean Temporary Files**

- **Settings → System → Storage → Temporary files** → Select all categories → Remove files
- **Disk Cleanup (cleanmgr)** → Run as administrator → Check:
  - Windows Update Cleanup
  - Delivery Optimization Files
  - Recycle Bin
  - Temporary Internet Files
  - Thumbnails
  - Previous Windows installations
- **Enable Storage Sense** → Settings → System → Storage → Storage Sense → Configure automatic cleanup schedules

---

## **5. Reset Default Apps & File Associations**

- **Settings → Apps → Default apps** → Click "Reset" to restore Microsoft recommended defaults
- **Settings → Apps → Default apps → Choose defaults by file type** → Verify associations for:
  - .pdf (should be Edge or your preferred reader)
  - .docx/.xlsx (should be Word/Excel or your preferred office suite)
  - .jpg/.png (should be Photos or your preferred image viewer)
  - .html/.htm (should be your preferred browser)

---

## **6. Privacy & Telemetry Reset**

- **Settings → Privacy & security → General** → Turn off:
  - "Let apps show me personalized ads by using my advertising ID"
  - "Let websites show me locally relevant content by accessing my language list"
- **Settings → Privacy & security → Diagnostics & feedback** → 
  - Diagnostic data → Basic (or Required only)
  - Click "Delete" under "Delete diagnostic data"
- **Settings → Privacy & security → Activity history** → 
  - Uncheck "Store my activity history on this device"
  - Uncheck "Send my activity history to Microsoft"
  - Click "Clear"

---

## **7. Windows Update & Drivers**

- **Settings → Windows Update → Check for updates** → Install all available updates
- **Advanced options → Optional updates** → Install driver updates (especially for graphics, network, chipset)
- **Device Manager** → Look for any devices with yellow warning icons → Right-click → Update driver
- **Optional:** Use manufacturer-specific tools (Dell Update, HP Support Assistant, Lenovo Vantage) for driver updates

---

## **8. Security Baseline Verification**

- **Windows Security → Virus & threat protection → Manage settings** → Ensure:
  - Real-time protection: ON
  - Cloud-delivered protection: ON
  - Automatic sample submission: ON
- **Windows Security → Firewall & network protection** → Ensure firewall is ON for all network profiles (Domain, Private, Public)
- **Windows Security → App & browser control** → 
  - Reputation-based protection: ON
  - Exploit protection: ON (system defaults)
  - SmartScreen for Microsoft Edge: ON

---

## **9. Remove Lingering Work Policies (PowerShell)**

Run **PowerShell as Administrator** to check and remove leftover MDM/work policies:

```powershell
# Check for leftover MDM policies in registry
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\PolicyManager\AdmxDefault" -ErrorAction SilentlyContinue

# Common policies to remove if present (use with caution!)
# Remove-Item "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" -Recurse -Force -ErrorAction SilentlyContinue
# Remove-Item "HKLM:\SOFTWARE\Policies\Microsoft\Edge" -Recurse -Force -ErrorAction SilentlyContinue
# Remove-Item "HKCU:\SOFTWARE\Policies\Microsoft\Edge" -Recurse -Force -ErrorAction SilentlyContinue

# Verify Group Policy results (should show mostly Not Configured for user policies)
gpresult /h gpreport.html /f
# Open gpreport.html to review applied policies
```

> **Warning:** Only remove registry keys if you are certain they are work-related and not needed. When in doubt, leave them alone or create a system restore point first.

---

## **10. Create a System Restore Point**

- Search for "Create a restore point" in the Start menu
- Select your system drive (usually C:)
- Click "Configure..." → Ensure "Turn on system protection" is selected
- Click "Create..." → Name it: `Post-work-cleanup` or `Home-PC-Baseline-[date]`
- Click OK to create the restore point

---

## **Quick Verification Checklist**

After completing the above steps, verify:

- [ ] No work or school accounts remain in **Settings → Accounts → Access work or school**
- [ ] No work-related user profiles in **Settings → Accounts → Other users** or **Control Panel → User Accounts**
- [ ] Your personal Microsoft account (or local account) is the primary account
- [ ] OneDrive is syncing your personal files (not a work/school OneDrive)
- [ ] Windows Hello / PIN / Fingerprint is yours and not managed by an organization
- [ ] No banners saying "Some settings are managed by your organization" in Settings app
- [ ] Task Manager → Startup shows only personal applications
- [ ] Default apps are set to your preferences (browser, editor, media player)
- [ ] Windows Security shows all protections enabled (no red warnings)

---

## **Optional Maintenance Scripts**

You can save the following PowerShell script as `Reset-HomePC.ps1` and run it as Administrator for routine maintenance:

```powershell
# Reset-HomePC.ps1
# Common home PC maintenance script for Windows 10/11

Write-Host "=== Starting Home PC Maintenance ===" -ForegroundColor Green

# 1. Clear temporary files
Write-Host "Cleaning temporary files..." -ForegroundColor Yellow
Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "$env:WINDIR\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue

# 2. Clear Windows Update cache (stop service first)
Write-Host "Clearing Windows Update cache..." -ForegroundColor Yellow
net stop wuauserv
net stop cryptSvc
net stop bits
net stop msiserver
Remove-Item "$env:WINDIR\SoftwareDistribution\Download\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "$env:WINDIR\System32\catroot2\*" -Recurse -Force -ErrorAction SilentlyContinue
net start wuauserv
net start cryptSvc
net start bits
net start msiserver

# 3. Reset network adapters
Write-Host "Resetting network adapters..." -ForegroundColor Yellow
netsh int ip reset
netsh winsock reset
ipconfig /flushdns
ipconfig /registerdns

# 4. Check for system file integrity
Write-Host "Checking system file integrity..." -ForegroundColor Yellow
sfc /scannow

# 5. Check Windows Update
Write-Host "Checking for Windows Update..." -ForegroundColor Yellow
# This will just report status; actual updates need manual approval via Settings

Write-Host "=== Maintenance Complete ===" -ForegroundColor Green
Write-Host "Consider creating a restore point and reviewing installed apps." -ForegroundColor Yellow
```

> **Note:** Review any script before running it. This script is provided as-is and should be tested in a safe environment first.

---

## **When to Consider a Fresh Install**

If after cleanup you still experience:
- Persistent performance issues
- Unexplained errors or crashes
- Suspicious malware or unwanted software
- Difficulty removing deep-seated work policies

Consider backing up your personal data and performing a clean Windows 10/11 installation using the [Microsoft Media Creation Tool](https://www.microsoft.com/software-download/windows10).

---

## **Resources for Further Learning**

- [Microsoft Support: Reset your PC](https://support.microsoft.com/windows/how-to-reset-your-pc-in-windows-66f1345c-9632-4e8e-a9c1-0993322635b0)
- [The PC Decrapifier](https://www.thepcdecrapifier.com/) (for removing bloatware on new PCs)
- [BleachBit](https://www.bleachbit.org/) (advanced open-source system cleaner)
- [Autoruns](https://learn.microsoft.com/sysinternals/downloads/autoruns) (to see what really runs at startup)
- [Process Explorer](https://learn.microsoft.com/sysinternals/downloads/process-explorer) (advanced task manager)

---

*This guide is designed for typical home Windows 10/11 PCs. Always back up important data before making significant system changes.*

*Last updated: October 2026*