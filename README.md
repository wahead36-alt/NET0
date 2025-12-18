# ======================================================
# 🛡️ ANTI DEBUG / DUMP
# ======================================================

$debuggers = @(
    "dnSpy","x64dbg","x32dbg","ollydbg",
    "ida","ida64","cheatengine",
    "processhacker","procmon","procexp"
)

$running = Get-Process | Select-Object -ExpandProperty ProcessName

foreach ($dbg in $debuggers) {
    if ($running -contains $dbg) {
        Write-Host "❌ Debugger Detected" -ForegroundColor Red
        Start-Sleep 2
        exit
    }
}

# ======================================================
# 🔑 KEYAUTH CONFIGURATION
# ======================================================
$name    = "Extreme"
$ownerid = "UpQAc4Dv5a"
$secret  = "4753823f0a9c94eeef3657d6db4ef7f7e8770493067fab3297cd82502d636f04"
$version = "1.0"

Clear-Host

# ======================================================
# 🔐 KEYAUTH AUTHENTICATION
# ======================================================
Write-Host "=== EXTREME AUTH SYSTEM ===" -ForegroundColor Cyan
$key = Read-Host "Enter License Key"

# =========================
# ✅ INIT (เพิ่มเข้ามา)
# =========================
$initBody = @{
    type    = "init"
    name    = $name
    ownerid = $ownerid
    secret  = $secret
    version = $version
}

try {
    $init = Invoke-RestMethod -Uri "https://keyauth.win/api/1.2/" `
        -Method POST `
        -Body $initBody `
        -ContentType "application/x-www-form-urlencoded"
}
catch {
    Write-Host "❌ INIT FAILED: Cannot connect to KeyAuth" -ForegroundColor Red
    Pause
    exit
}

if ($init.success -ne $true) {
    Write-Host "❌ INIT FAILED: $($init.message)" -ForegroundColor Red
    Pause
    exit
}

$sessionid = $init.sessionid

# =========================
# ✅ LICENSE (ของเดิม + sessionid)
# =========================
$body = @{
    type      = "license"
    key       = $key
    sessionid = $sessionid
    name      = $name
    ownerid   = $ownerid
}

try {
    $response = Invoke-RestMethod -Uri "https://keyauth.win/api/1.2/" `
        -Method POST `
        -Body $body `
        -ContentType "application/x-www-form-urlencoded"
}
catch {
    Write-Host "❌ Cannot connect to KeyAuth server" -ForegroundColor Red
    Pause
    exit
}

if ($response.success -ne $true) {
    Write-Host "❌ AUTH FAILED: $($response.message)" -ForegroundColor Red
    Pause
    exit
}

Write-Host "✅ AUTH SUCCESS - Welcome $($response.info.username)" -ForegroundColor Green
Start-Sleep 1
Clear-Host

# ======================================================
# 🔥 ORIGINAL SCRIPT (ห้ามลบ / ไม่แก้)
# ======================================================

# 1. Disable Network Throttling
Write-Host "--- Disabling Network Throttling ---" -ForegroundColor Red
$NTI_Path = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile"
Set-ItemProperty -Path $NTI_Path -Name "NetworkThrottlingIndex" -Value 0xFFFFFFFF
Set-ItemProperty -Path $NTI_Path -Name "SystemResponsiveness" -Value 0

# 2. Advanced TCP Tweaks
Write-Host "--- Applying Advanced TCP Registry Tweaks ---" -ForegroundColor Red
$TCP_Path = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
Set-ItemProperty -Path $TCP_Path -Name "MaxUserPort" -Value 65534
Set-ItemProperty -Path $TCP_Path -Name "TcpTimedWaitDelay" -Value 30
Set-ItemProperty -Path $TCP_Path -Name "MaxFreeTcbs" -Value 65536
Set-ItemProperty -Path $TCP_Path -Name "MaxHashTableSize" -Value 65536

# 3. Disable HPET & Dynamic Tick
Write-Host "--- Disabling HPET and Dynamic Ticks ---" -ForegroundColor Red
bcdedit /set disabledynamictick yes
bcdedit /deletevalue useplatformclock
bcdedit /set useplatformtick yes

# 4. Disable VBS / Memory Integrity
Write-Host "--- Disabling Virtualization-Based Security (VBS) ---" -ForegroundColor Red
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" `
    -Name "EnableVirtualizationBasedSecurity" -Value 0

# 5. Disable Windows Firewall & Defender
Write-Host "--- Disabling Real-time Network Monitoring ---" -ForegroundColor Red
Set-MpPreference -DisableRealtimeMonitoring $true
netsh advfirewall set allprofiles state off

# 6. Optimize Network Adapter
Write-Host "--- Configuring Network Adapter for Low Latency ---" -ForegroundColor Red
Get-NetAdapterLso -Name "*" | Disable-NetAdapterLso
Get-NetAdapterChecksumOffload -Name "*" | Disable-NetAdapterChecksumOffload

# ======================================================
# 🧠 POST-CHECK STATUS LOG
# ======================================================
Write-Host ""
Write-Host "===== SYSTEM STATUS CHECK =====" -ForegroundColor Cyan

# 1️⃣ Network Throttling
$nti = Get-ItemProperty `
    -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile"

if ($nti.NetworkThrottlingIndex -eq 0xFFFFFFFF) {
    Write-Host "[OK] Network Throttling : DISABLED" -ForegroundColor Green
} else {
    Write-Host "[WARN] Network Throttling : NOT FULLY DISABLED" -ForegroundColor Yellow
}

# 2️⃣ TCP Parameters
$tcp = Get-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"

Write-Host "[INFO] MaxUserPort        : $($tcp.MaxUserPort)"
Write-Host "[INFO] TcpTimedWaitDelay  : $($tcp.TcpTimedWaitDelay)"

# 3️⃣ BCD / Timer Status
$bcd = bcdedit

if ($bcd -match "disabledynamictick\s+Yes") {
    Write-Host "[OK] Dynamic Tick : DISABLED" -ForegroundColor Green
} else {
    Write-Host "[WARN] Dynamic Tick : ENABLED" -ForegroundColor Yellow
}

if ($bcd -match "useplatformtick\s+Yes") {
    Write-Host "[OK] Platform Tick : ENABLED" -ForegroundColor Green
} else {
    Write-Host "[INFO] Platform Tick : DEFAULT" -ForegroundColor Gray
}

if ($bcd -match "useplatformclock") {
    Write-Host "[WARN] HPET : FORCED" -ForegroundColor Yellow
} else {
    Write-Host "[OK] HPET : NOT FORCED" -ForegroundColor Green
}

# 4️⃣ VBS Status
$vbs = Get-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard" `
    -ErrorAction SilentlyContinue

if ($vbs.EnableVirtualizationBasedSecurity -eq 0) {
    Write-Host "[OK] VBS / Memory Integrity : DISABLED" -ForegroundColor Green
} else {
    Write-Host "[WARN] VBS : ENABLED (Reboot required)" -ForegroundColor Yellow
}

# 5️⃣ Defender Realtime
$def = Get-MpComputerStatus

if ($def.RealTimeProtectionEnabled -eq $false) {
    Write-Host "[OK] Defender Realtime : DISABLED" -ForegroundColor Green
} else {
    Write-Host "[WARN] Defender Realtime : ENABLED" -ForegroundColor Yellow
}

Write-Host "================================" -ForegroundColor Cyan
Write-Host "⚠️ Some settings require RESTART to take effect" -ForegroundColor Yellow
