---
layout: default
title: "ADMINISTRACIÓ DE PROCESSOS I SERVEIS EN WINDOWS"
---
# ADMINISTRACIÓ DE PROCESSOS I SERVEIS EN WINDOWS: CREACIÓ AMB NSSM I POWERSHELL

## Índex

- [1- Fonaments: Processos i Serveis a Windows](#1--fonaments-processos-i-serveis-a-windows)
  - [1.1- Kernel, Aplicació, Servei i Procés a Windows](#11--kernel-aplicació-servei-i-procés-a-windows)
  - [1.2- El Service Control Manager (SCM) i l'Aïllament de Sessió 0](#12--el-service-control-manager-scm-i-laïllament-de-sessió-0)
  - [1.3- Tipus d'inici dels Serveis (Startup Types)](#13--tipus-dinici-dels-serveis-startup-types)
  - [1.4- Per què NSSM (Non-Sucking Service Manager) i PowerShell Admin?](#14--per-què-nssm-non-sucking-service-manager-i-powershell-admin)
- [2- Gestió i Optimització de Serveis del Sistema (Requisit PDF)](#2--gestió-i-optimització-de-serveis-del-sistema-requisit-pdf)
  - [2.1- Identificació de serveis consumidors de recursos a Windows](#21--identificació-de-serveis-consumidors-de-recursos-a-windows)
  - [2.2- Desactivació i Aturada del Servei SysMain (Superfetch)](#22--desactivació-i-aturada-del-servei-sysmain-superfetch)
- **[🏠 TASCA AVALUABLE : Windows Sentinel — Dashboard d'Auditoria, Processos i Seguretat en Temps Real](#tasca-avaluable)**
  - [Objectiu del Projecte i Arquitectura](#objectiu-del-projecte-i-arquitectura)
  - [PAS 0: Preparació de l'entorn i eines a Windows](#pas-0-preparació-de-lentorn-i-eines-a-windows)
  - [PAS 1: Disseny i Programació de l'Script Agent Sentinel (`SentinelServer.ps1`)](#pas-1-disseny-i-programació-de-lscript-agent-sentinel-sentinelserverps1)
  - [PAS 2: Disseny de la Interfície Web Dashboard Premium (`index.html`)](#pas-2-disseny-de-la-interfície-web-dashboard-premium-indexhtml)
  - [PAS 3: Execució i Verificació Manual de l'Script (Requisit Previ)](#pas-3-execució-i-verificació-manual-de-lscript-requisit-previ)
  - [PAS 4: Creació i Configuració del Servei Windows amb NSSM (Requisits del PDF)](#pas-4-creació-i-configuració-del-servei-windows-amb-nssm-requisits-del-pdf)
    - [4.1- Configuració de la pestanya "Application"](#41--configuració-de-la-pestanya-application)
    - [4.2- Configuració de la pestanya "Details"](#42--configuració-de-la-pestanya-details)
    - [4.3- Configuració de "Allow service to interact with desktop" (Pestanya "Log on")](#43--configuració-de-allow-service-to-interact-with-desktop-pestanya-log-on)
    - [4.4- Mètode Alternatiu per Línia de Comandes (PowerShell Admin)](#44--mètode-alternatiu-per-línia-de-comandes-powershell-admin)
  - [PAS 5: Desactivació d'un Servei Consumidor de Recursos (`SysMain`)](#pas-5-desactivació-dun-servei-consumidor-de-recursos-sysmain)
  - [PAS 6: Arrencada i Verificació del Servei amb PowerShell](#pas-6-arrencada-i-verificació-del-servei-amb-powershell)
  - [PAS 7: Reinici del Sistema (Reboot) i Verificació de Persistència](#pas-7-reinici-del-sistema-reboot-i-verificació-de-persistència)
  - [PAS 8: Accés al Web Dashboard i Prova d'Interacció amb l'Escriptori](#pas-8-accés-al-web-dashboard-i-prova-dinteracció-amb-lescriptori)
- [Resum de Compliment dels Requisits de l'Enunciat](#resum-de-compliment-dels-requisits-de-lenunciat)

---

## 1- Fonaments: Processos i Serveis a Windows

### 1.1- Kernel, Aplicació, Servei i Procés a Windows

Dins del sistema operatiu Windows, els rols d'execució es defineixen de la següent manera:

* **Kernel de Windows (`ntoskrnl.exe`)**: El nucli del sistema operatiu encarregat de la gestió de memòria física/virtual, la planificació de fils d'execució (*threads*), el subsistema de seguretat i la comunicació directa amb els controladors de dispositiu (*drivers*).
* **Aplicació d'usuari (User Mode - GUI / CLI)**: Programes executats de forma interactiva en primer pla dins de la sessió d'un usuari concret (ex: Bloc de notes, Microsoft Edge, PowerShell interactiu).
* **Servei de Windows (*Windows Service*)**: Programa de fons (*background process*) gestionat directament pel sistema operatiu. No requereix que cap usuari hagi iniciat sessió gràfica per funcionar, s'inicia normalment durant el procés de càrrega del sistema i roman actiu independentment de qui tanqui la sessió.
* **Procés**: Instància viva d'un executable a la memòria amb identificador únic (PID), espai d'adreçament protegit i taula de recursos (*handles*). Tant les aplicacions convencionals com els serveis generen processos que podem visualitzar i gestionar a l'Administrador de Tasques (*Task Manager*) o via PowerShell amb `Get-Process`.

---

### 1.2- El Service Control Manager (SCM) i l'Aïllament de Sessió 0

Tots els serveis a Windows són administrats pel **Service Control Manager (SCM)**, representat pel procés de sistema `services.exe`:

* El SCM manté la base de dades de serveis al Registre de Windows (`HKLM\SYSTEM\CurrentControlSet\Services`).
* Gestiona l'ordre d'arrencada, les dependències entre serveis i envia senyals de control com `SERVICE_CONTROL_STOP`, `SERVICE_CONTROL_PAUSE` o `SERVICE_CONTROL_CONTINUE`.
* **Aïllament de Sessió 0 (*Session 0 Isolation*)**: Per motius de seguretat estrictes, des de Windows Vista / Server 2008, tots els serveis s'executen aïllats dins de la **Sessió 0**. La sessió gràfica de l'usuari interactiu s'executa a la **Sessió 1** (o superior). Aquest tall impedeix atacs com el *shatter attack*.
* Quan un servei necessita mostrar finestres o alertes visuals directament a l'escriptori de l'usuari connectat, és obligatori habilitar expressament la propietat **"Allow service to interact with desktop"** al gestor de serveis.

---

### 1.3- Tipus d'inici dels Serveis (Startup Types)

A Windows, cada servei té assignada una política d'arrencada:

| Tipus d'Inici (*Startup Type*) | Comportament al sistema | Ús recomanat |
|---|---|---|
| **Automatic (`Automatic`)** | S'arrenca automàticament durant l'arrencada del sistema operatiu, abans de la pantalla d'inici de sessió. | Serveis crítics de xarxa i monitoratge permanent. |
| **Automatic Delayed (`AutomaticDelayedStart`)** | S'inicia pocs minuts després de completar el boot per alleugerir la càrrega inicial del maquinari. | Tasques auxiliars que no són crítiques al segon zero. |
| **Manual (`Manual`)** | Només s'executa quan un usuari, un script o una aplicació en sol·licita explícitament l'engegada. | Serveis sota demanda (ex: servei d'instal·lació de programari). |
| **Disabled (`Disabled`)** | Bloquejat totalment; no es pot arrencar fins que es canviï manualment el seu tipus d'inici. | **Optimització de recursos**: aturar serveis innecessaris o pesats. |

---

### 1.4- Per què NSSM (Non-Sucking Service Manager) i PowerShell Admin?

> ⚠️ **REQUISIT CLAU DE L'ENUNCIAT: CAL FER-HO AMB POWERSHELL COM A ADMIN, NO AMB CMD.**

1. **Per què PowerShell i NO pas CMD?**
   * El símbol del sistema tradicional (`cmd.exe`) està obsolet, només maneja cadenes de text planes i no pot manipular objectes del sistema ni gestionar el cicle de vida de serveis avançats.
   * **PowerShell com a Administrador** permet interactuar directament amb el nucli del sistema mitjançant cmdlets orientats a objectes (`Get-Service`, `Set-Service`, `Start-Service`), accés a consultes WMI/CIM (`Get-CimInstance`), lectura d'esdeveniments del visor de seguretat (`Get-WinEvent`) i manipulació de xarxa (`Get-NetTCPConnection`).

2. **Per què NSSM (Non-Sucking Service Manager)?**
   * Un executable ordinari o un script `.ps1` no conté les trucades a la Win32 API (`ServiceMain`) necessàries per comunicar-se amb el SCM. Si intentéssim registrar directament `powershell.exe script.ps1` com a servei amb `sc.exe`, el SCM donaria l'error de temps d'espera esgotat (*Error 1053*).
   * **NSSM** actua com un encapsulador (*service wrapper*) transparent: gestiona el protocol de comunicació amb Windows, aixeca el nostre script PowerShell en segon pla amb el paràmetre `-ExecutionPolicy Bypass`, redirigeix logs d'error/sortida i permet reiniciar automàticament el servei en cas de caiguda.

---

## 2- Gestió i Optimització de Serveis del Sistema (Requisit PDF)

> 📋 **Requisit de l'enunciat:** *"A més, desactiva un servei que no vulguis que consumeixi recursos."*

### 2.1- Identificació de serveis consumidors de recursos a Windows

A qualsevol equip de proves o màquina virtual, existeixen serveis natius de Windows que consumeixen cicles de CPU, memòria RAM i operacions d'E/S de disc de forma constant. Els principals candidats a desactivar són:

* **`SysMain` (anteriorment Superfetch)**: Servei dissenyat per precarregar aplicacions a la RAM analitzant el patró d'ús. En discs mecànics o discs virtuals en màquines virtuals (VirtualBox / VMware), sol provocar pics constants del **100% d'ús de disc** i alentiment sever del sistema.
* **`WSearch` (Windows Search)**: Indexador de fitxers que indexa el disc constantment en segon pla.
* **`DiagTrack` (Connected User Experiences and Telemetry)**: Servei de recollida de dades i telemetria de Microsoft.

---

### 2.2- Desactivació i Aturada del Servei SysMain (Superfetch)

Seleccionem el servei **`SysMain`** per alliberar recursos de disc i RAM.

Executem la seqüència completa des de **PowerShell com a Administrador**:

```powershell
# 1. Comprovem l'estat inicial del servei SysMain
Get-Service -Name "SysMain" | Select-Object Name, DisplayName, Status, StartType

# 2. Aturem el servei immediatament
Stop-Service -Name "SysMain" -Force

# 3. Canviem el tipus d'arrencada permanent a Desactivat (Disabled)
Set-Service -Name "SysMain" -StartupType Disabled

# 4. Verifiquem que el servei està aturat (Stopped) i desactivat (Disabled)
Get-Service -Name "SysMain" | Select-Object Name, DisplayName, Status, StartType
```

<img width="750" height="423" alt="image" src="https://github.com/user-attachments/assets/22cee562-b99f-4ef9-9523-6867b43d8d6b" />

---

<a id="tasca-avaluable"></a>
# 🏠 TASCA AVALUABLE : Windows Sentinel — Dashboard d'Auditoria, Processos i Seguretat en Temps Real

## Objectiu del Projecte i Arquitectura

Mentre que a l'activitat d'Ubuntu vam desenvolupar un sistema de captures de pantalla passiu (`scrot`), a **Windows** plantegem un projecte autòcton d'auditoria i administració de processos: **PauSerra Windows Sentinel**.

Aquest projecte integra:
1. **Un Servei de Windows natiu** gestionat amb **NSSM**, executant-se a l'arrencada sota el compte del sistema `LocalSystem`.
2. **Un Servidor Web en PowerShell natiu** construït sobre la classe `System.Net.HttpListener` de .NET (sense requerir la instal·lació d'eines externes com Python, Apache o Node.js), escoltant al port `8080`.
3. **Mètriques en Temps Real de Processos (UF1 NF4)**:
   * **Rendiment del maquinari**: Ús en viu de CPU (%) i Memòria RAM utilitzada/total.
   * **Auditoria de Processos (Top 10)**: Processos que més memòria i CPU consumeixen a Windows, amb cerca en viu per nom o PID.
   * **Sockets i Connexions de Xarxa**: Taula de connexions TCP actives i ports en escolta (`Get-NetTCPConnection`).
   * **Visor d'Esdeveniments**: Lectura dels darrers logs de seguretat/sistema de l'Event Viewer (`Get-WinEvent`).
   * **Monitoratge de l'Optimització**: Confirmació en viu de la desactivació del servei consumidor `SysMain`.
4. **Interacció amb l'Escriptori (*Interact with Desktop*)**: Demostració pràctica del permís requerit al PDF, permetent llançar una finestra emergent interactiva de Windows (`MessageBox`) des del servei de fons directament a la pantalla gràfica de l'usuari.

---

## PAS 0: Preparació de l'entorn i eines a Windows

Obre **Windows PowerShell com a Administrador** i crea l'estructura de carpetes necessària per al servei:

```powershell
# Creem la carpeta arrel del projecte i les subcarpetes
New-Item -ItemType Directory -Force -Path "C:\PauSerra-Sentinel\web"
New-Item -ItemType Directory -Force -Path "C:\PauSerra-Sentinel\logs"
New-Item -ItemType Directory -Force -Path "C:\PauSerra-Sentinel\bin"
```

<img width="648" height="494" alt="image" src="https://github.com/user-attachments/assets/ea4f53e2-779e-4e70-be7c-5b1c10dfd64d" />


Copiem l'executable **`nssm.exe`** (versió de 64 bits de la carpeta `nssm-2.24/win64`) dins de la carpeta `C:\PauSerra-Sentinel\bin\` i comprova la seva versió:

```powershell
# Verifiquem l'executable de nssm
& "C:\PauSerra-Sentinel\bin\nssm.exe" version
```

<img width="489" height="86" alt="image" src="https://github.com/user-attachments/assets/baaca53f-1027-4511-ace5-054878de235c" />

---

## PAS 1: Disseny i Programació de l'Script Agent Sentinel (`SentinelServer.ps1`)

Creem l'script principal a `C:\PauSerra-Sentinel\SentinelServer.ps1`. Aquest script aixeca l'HttpListener al port `8080`, recopila dades de processos i esdeveniments, i gestiona les alertes visuals a l'escriptori.

Codi complet de `C:\PauSerra-Sentinel\SentinelServer.ps1`:

```powershell
# ==============================================================================
# SentinelServer.ps1
# PAUSERRA SENTINEL - Servidor de Monitoratge i Auditoria de Processos en Windows
# Assignatura: M6 Administració de Sistemes Operatius (ASIX)
# Autor: Pau Serra
# ==============================================================================

[CmdletBinding()]
param(
    [int]$Port = 8080
)

[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

$ScriptDir = Split-Path -Parent $MyInvocation.MyCommand.Path
if (-not $ScriptDir -or -not (Test-Path $ScriptDir)) {
    $ScriptDir = "C:\PauSerra-Sentinel"
}

$WebFile = Join-Path $ScriptDir "web\index.html"
$LogDir  = Join-Path $ScriptDir "logs"
$LogFile = Join-Path $LogDir "sentinel.log"

if (-not (Test-Path $LogDir)) {
    New-Item -ItemType Directory -Force -Path $LogDir | Out-Null
}

function Write-Log {
    param([string]$Message)
    $timestamp = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
    $logLine = "[$timestamp] $Message"
    Write-Output $logLine
    try {
        $logLine | Out-File -Append -FilePath $LogFile -Encoding utf8
    } catch { }
}

Write-Log "Iniciant PauSerra Sentinel Server al port $Port..."

$listener = New-Object System.Net.HttpListener
$prefixAll = "http://+:$Port/"
$prefixLocal = "http://localhost:$Port/"

try {
    $listener.Prefixes.Add($prefixAll)
    $listener.Start()
    Write-Log "HttpListener engegat escoltant a $prefixAll"
} catch {
    Write-Log "Lligant a localhost..."
    $listener = New-Object System.Net.HttpListener
    $listener.Prefixes.Add($prefixLocal)
    $listener.Start()
    Write-Log "HttpListener engegat escoltant a $prefixLocal"
}

function Get-SystemTelemetry {
    # 1. Mesura de CPU
    $cpuUsage = 0
    try {
        $cpuData = Get-CimInstance -ClassName Win32_PerfFormattedData_PerfOS_Processor -Filter "Name='_Total'" -ErrorAction SilentlyContinue
        if ($cpuData) { $cpuUsage = [int]$cpuData.PercentProcessorTime }
    } catch { $cpuUsage = 0 }

    # 2. Mesura de RAM
    $totalRamGB = 0
    $usedRamGB = 0
    $ramPercent = 0
    try {
        $osData = Get-CimInstance Win32_OperatingSystem -ErrorAction SilentlyContinue
        if ($osData) {
            $totalRamGB = [math]::Round($osData.TotalVisibleMemorySize / 1MB, 2)
            $freeRamGB  = [math]::Round($osData.FreePhysicalMemory / 1MB, 2)
            $usedRamGB  = [math]::Round($totalRamGB - $freeRamGB, 2)
            if ($totalRamGB -gt 0) {
                $ramPercent = [math]::Round(($usedRamGB / $totalRamGB) * 100, 1)
            }
        }
    } catch { }

    # 3. Administració de Processos (Top 10 més consumidors de memòria)
    $topProcesses = @()
    try {
        $topProcesses = Get-Process -ErrorAction SilentlyContinue |
            Sort-Object -Property WorkingSet64 -Descending |
            Select-Object -First 10 Id, ProcessName,
                @{Name='MemoryMB'; Expression={[math]::Round($_.WorkingSet64 / 1MB, 1)}},
                @{Name='CPU'; Expression={ if ($_.CPU) { [math]::Round($_.CPU, 1) } else { 0 } }},
                Responding
    } catch { }

    $totalProcs = 0
    try {
        $totalProcs = (Get-Process -ErrorAction SilentlyContinue).Count
    } catch { }

    # 4. Estat dels Serveis
    $sentinelStatus = "RUNNING"
    $sysmainStatus  = "OPTIMIZED (DISABLED)"
    try {
        $sService = Get-Service -Name "PauSerraSentinel" -ErrorAction SilentlyContinue
        if ($sService) { $sentinelStatus = $sService.Status.ToString() }

        $sysService = Get-Service -Name "SysMain" -ErrorAction SilentlyContinue
        if ($sysService) {
            $sysmainStatus = "$($sysService.Status) / $($sysService.StartType)"
        }
    } catch { }

    # 5. Connexions TCP
    $tcpConnections = @()
    try {
        $tcpConnections = Get-NetTCPConnection -State Listen, Established -ErrorAction SilentlyContinue |
            Select-Object -First 10 LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
    } catch { }

    # 6. Esdeveniments recents del sistema (EventLog)
    $eventsList = @()
    try {
        $eventsList = Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2,3} -MaxEvents 5 -ErrorAction SilentlyContinue |
            Select-Object @{N='Time'; E={$_.TimeCreated.ToString("HH:mm:ss")}},
                          Id,
                          @{N='Level'; E={$_.LevelDisplayName}},
                          @{N='Provider'; E={$_.ProviderName}},
                          @{N='Message'; E={
                              if ($_.Message) {
                                  $line = ($_.Message -split "`r?`n")[0]
                                  if ($line.Length -gt 85) { $line.Substring(0, 85) + "..." } else { $line }
                              } else { "Notificació de sistema" }
                          }}
    } catch { }

    $uptimeStr = "0d 0h 0m"
    $osName = "Windows"
    if ($osData) {
        $uptime = (Get-Date) - $osData.LastBootUpTime
        $uptimeStr = "$($uptime.Days)d $($uptime.Hours)h $($uptime.Minutes)m"
        $osName = $osData.Caption
    }

    [PSCustomObject]@{
        Hostname          = $env:COMPUTERNAME
        OS                = $osName
        Uptime            = $uptimeStr
        CPU_Percent       = $cpuUsage
        RAM_TotalGB       = $totalRamGB
        RAM_UsedGB        = $usedRamGB
        RAM_Percent       = $ramPercent
        TotalProcesses    = $totalProcs
        TopProcesses      = $topProcesses
        SentinelService   = $sentinelStatus
        SysMainService    = $sysmainStatus
        ActiveConnections = $tcpConnections
        RecentEvents      = $eventsList
    }
}

while ($listener.IsListening) {
    try {
        $context = $listener.GetContext()
        $request = $context.Request
        $response = $context.Response
        $path = $request.Url.AbsolutePath

        if ($path -eq "/" -or $path -eq "/index.html") {
            if (Test-Path $WebFile) {
                $bytes = [System.IO.File]::ReadAllBytes($WebFile)
            } else {
                $html = "<html><body><h1>PauSerra Sentinel Actiu</h1><p>Arxiu web no trobat.</p></body></html>"
                $bytes = [System.Text.Encoding]::UTF8.GetBytes($html)
            }
            $response.ContentType = "text/html; charset=utf-8"
            $response.ContentLength64 = $bytes.Length
            $response.OutputStream.Write($bytes, 0, $bytes.Length)
        }
        elseif ($path -eq "/api/telemetry") {
            $telemetryObj = Get-SystemTelemetry
            $json = $telemetryObj | ConvertTo-Json -Depth 4
            $bytes = [System.Text.Encoding]::UTF8.GetBytes($json)

            $response.ContentType = "application/json; charset=utf-8"
            $response.Headers.Add("Access-Control-Allow-Origin", "*")
            $response.ContentLength64 = $bytes.Length
            $response.OutputStream.Write($bytes, 0, $bytes.Length)
        }
        elseif ($path -eq "/api/alert") {
            Write-Log "Petició d'alerta rebuda des del navegador. Llançant finestra d'interacció a l'escriptori..."

            $cmd = @"
Add-Type -AssemblyName System.Windows.Forms
[System.Windows.Forms.MessageBox]::Show(
    `"🚨 ALERTA DEL SERVEI PAUSERRA SENTINEL`n`nDemostració de la funcionalitat NSSM 'Allow service to interact with desktop'.`nEl servei s'executa en segon pla i pot interactuar amb la pantalla de l'usuari interactiu!`",
    `"PauSerra Sentinel - Notificació de Servei Windows`",
    [System.Windows.Forms.MessageBoxButtons]::OK,
    [System.Windows.Forms.MessageBoxIcon]::Information
)
"@
            Start-Process powershell.exe -ArgumentList "-NoProfile", "-WindowStyle", "Hidden", "-Command", $cmd

            $msg = '{"status":"ok","message":"Alerta llançada a l escriptori de Windows"}'
            $bytes = [System.Text.Encoding]::UTF8.GetBytes($msg)
            $response.ContentType = "application/json"
            $response.ContentLength64 = $bytes.Length
            $response.OutputStream.Write($bytes, 0, $bytes.Length)
        }
        else {
            $response.StatusCode = 404
            $err = "404 Not Found"
            $bytes = [System.Text.Encoding]::UTF8.GetBytes($err)
            $response.ContentLength64 = $bytes.Length
            $response.OutputStream.Write($bytes, 0, $bytes.Length)
        }

        $response.Close()
    }
    catch {
        if (-not $listener.IsListening) { break }
    }
}

Write-Log "Sentinel Server aturat."
```


---

## PAS 2: Disseny de la Interfície Web Dashboard Premium (`index.html`)

A la ruta `C:\PauSerra-Sentinel\web\index.html`, desem la plantilla gràfica del panell. 

La interfície inclou:
* **Tema fosc d'alta densitat (*Dark Glassmorphism*)** amb paleta neó (cian `#00e5ff`, maragda `#10b981`, ambre `#f59e0b` i rosa `#f43f5e`).
* **Targetes KPI dinàmiques**:
  * Ús de Processador (CPU) amb barra de càrrega animada en temps real.
  * Memòria RAM (percentatge, GB usats i GB totals).
  * Estat del servei Sentinel (`RUNNING` sota `LocalSystem`).
  * Estat del servei SysMain (`DISABLED / STOPPED`).
* **Explorador de Processos (NF4)**: Taula interactiva ordenada per consum de memòria, amb cerca/filtre en viu per nom de procés o PID.
* **Monitor de Xarxa**: Taula de sockets TCP en escolta i connexions establertes amb les seves adreces i ports.
* **Feed d'Auditoria (Event Viewer)**: Registre dels darrers esdeveniments del sistema.
* **Bàner d'Interacció amb l'Escriptori**: Botó que dispara una crida AJAX cap a `/api/alert` per llançar la notificació emergent nativa a l'escriptori.
* **Auto-refresc dinàmic**: Consulta l'API de telemetria cada 3 segons de forma transparent.


---

## PAS 3: Execució i Verificació Manual de l'Script (Requisit Previ)

Abans de registrar l'script com a servei permanent, comprovem que funciona de manera autònoma des de PowerShell.

A la consola de **PowerShell com a Administrador**, executem:

```powershell
# Execució manual amb política de seguretat Bypass
powershell.exe -ExecutionPolicy Bypass -NoProfile -File "C:\PauSerra-Sentinel\SentinelServer.ps1"
```

La consola mostrarà els missatges d'inici:

<img width="899" height="95" alt="image" src="https://github.com/user-attachments/assets/6e035783-3340-4c65-b96b-bd66679d2246" />


Mentre s'executa, obrim el navegador a:
```text
http://localhost:8080
```
Verifiquem que el panell carrega les dades correctament. 

<img width="937" height="924" alt="image" src="https://github.com/user-attachments/assets/4f503485-b841-48d6-a1c0-c3731da2a259" />





En una altra consola de PowerShell podem comprovar l'API de dades:
```powershell
Invoke-RestMethod -Uri "http://localhost:8080/api/telemetry"
```
<img width="828" height="447" alt="image" src="https://github.com/user-attachments/assets/bae4b3d3-d912-42c2-aa36-75a0330d8dc5" />



---

## PAS 4: Creació i Configuració del Servei Windows amb NSSM (Requisits del PDF)

Ara procedim a registrar el nostre script com a servei del sistema operatiu Windows utilitzant **NSSM**, seguint exactament les indicacions del document de la professora.

Des de **PowerShell com a Administrador**, llancem la interfície gràfica d'instal·lació:

```powershell
# Ens situem a la carpeta on tenim nssm.exe
cd "C:\PauSerra-Sentinel\bin"

# Llancem l'assistent d'instal·lació assignant el nom 'PauSerraSentinel'
.\nssm.exe install PauSerraSentinel
```

---

### 4.1- Configuració de la pestanya "Application"

Segons la indicació literal del PDF:
> *"A les 2 primeres línies afegeixo la ruta a powershell.exe i a la ultima la ruta amb els paràmetres ExecutionPolicy Bypass."*

Configurem els camps següents:

* **Path**:
  ```text
  C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  ```
* **Startup directory**:
  ```text
  C:\PauSerra-Sentinel
  ```
* **Arguments**:
  ```text
  -ExecutionPolicy Bypass -NoProfile -File "C:\PauSerra-Sentinel\SentinelServer.ps1"
  ```

<img width="466" height="298" alt="image" src="https://github.com/user-attachments/assets/32619c6e-035d-43dd-9e28-5cc1e2c296db" />

---

### 4.2- Configuració de la pestanya "Details"

A la pestanya **Details**:
* **Display name**: `PauSerra Sentinel - Monitoratge i Auditoria de Processos`
* **Description**: `Servei de fons per a la telemetria de processos (NF4), connexions TCP i auditoria de seguretat en temps real.`
* **Startup type**: `Automatic`

---
<img width="429" height="232" alt="image" src="https://github.com/user-attachments/assets/ab42cfa7-ac0d-46f1-ac41-1aea8d059d79" />


### 4.3- Configuració de "Allow service to interact with desktop" (Pestanya "Log on")

Segons la indicació literal del PDF:
> *"Activo l’opció de “Allow service to interact with desktop”"*

1. Fem clic a la pestanya **Log on**.
2. Mantenim seleccionat el compte **Local System account**.
3. **Marquem la casella: `Allow service to interact with desktop`**.
4. Cliquem sobre el botó **Install service**.

<img width="431" height="229" alt="image" src="https://github.com/user-attachments/assets/6f0b7be6-be50-4a0d-a86e-93f0432d5d1d" />

Apareixerà el quadre de diàleg confirmant:
```text
Service "PauSerraSentinel" installed successfully!
```
<img width="331" height="143" alt="image" src="https://github.com/user-attachments/assets/33e5bb81-88dd-4747-9c3b-16aa30b544a1" />

---

### 4.4- Mètode Alternatiu per Línia de Comandes (PowerShell Admin)

Si es vol fer la instal·lació directa sense GUI des de PowerShell com a Administrador:

```powershell
$nssm = "C:\PauSerra-Sentinel\bin\nssm.exe"
$serviceName = "PauSerraSentinel"
$powershell = "$env:SystemRoot\System32\WindowsPowerShell\v1.0\powershell.exe"
$scriptPath = "C:\PauSerra-Sentinel\SentinelServer.ps1"
$arguments = '-ExecutionPolicy Bypass -NoProfile -File "{0}"' -f $scriptPath

# Instal·lem el servei
& $nssm install $serviceName $powershell $arguments

# Configurem opcions d'arrencada i directori
& $nssm set $serviceName AppDirectory "C:\PauSerra-Sentinel"
& $nssm set $serviceName Description "PauSerra Sentinel - Monitoratge de Processos i Auditoria"
& $nssm set $serviceName Start SERVICE_AUTO_START

# Activem el flag d'interacció amb l'escriptori
& $nssm set $serviceName Type SERVICE_INTERACTIVE_PROCESS
```

---

## PAS 5: Desactivació d'un Servei Consumidor de Recursos (`SysMain`)

Executem la desactivació del servei pesat `SysMain` des de PowerShell:

```powershell
# Aturem el servei
Stop-Service -Name "SysMain" -Force

# Desactivem l'arrencada
Set-Service -Name "SysMain" -StartupType Disabled

# Comprovem el resultat
Get-Service -Name "SysMain" | Format-List Name, DisplayName, Status, StartType
```

<img width="720" height="165" alt="image" src="https://github.com/user-attachments/assets/b2af9b59-2aba-4f7e-941a-29a524c1392c" />

---

## PAS 6: Arrencada i Verificació del Servei amb PowerShell

Posem en marxa el nostre nou servei de Windows i comprovem que s'executa correctament:

```powershell
# Iniciem el servei
Start-Service -Name "PauSerraSentinel"

# Comprovem el seu estat amb PowerShell
Get-Service -Name "PauSerraSentinel" | Select-Object Name, DisplayName, Status, StartType

# Comprovem l'estat mitjançant NSSM
& "C:\PauSerra-Sentinel\bin\nssm.exe" status PauSerraSentinel
```

<img width="798" height="185" alt="image" src="https://github.com/user-attachments/assets/6867d3d0-6419-41ac-9d90-f6b2914aee31" />


Obrim també la consola oficial de serveis de Windows (`Win + R` -> `services.msc`) per verificar gràficament que el servei està en execució amb inici automàtic.

<img width="928" height="377" alt="image" src="https://github.com/user-attachments/assets/72893fb5-b1ac-4407-a536-2514be67e5dc" />


---

## PAS 7: Reinici del Sistema (Reboot) i Verificació de Persistència

Un servei autèntic de Windows ha de persistir a l'arrencada sense necessitat que ningú iniciï sessió gràfica.

Reiniciem el sistema:

```powershell
Restart-Computer -Force
```

Un cop reiniciat l'equip:
1. Obrim el navegador web des de la màquina o des d'un altre equip de la xarxa:
   ```text
   http://localhost:8080
   # o bé:
   http://<IP-DE-LA-MAQUINA>:8080
   ```
   <img width="937" height="827" alt="image" src="https://github.com/user-attachments/assets/ceaf8786-1230-4d02-808a-e0ec5f031927" />

2. Des de PowerShell verifiquem l'estat del servei:
   ```powershell
   Get-Service -Name "PauSerraSentinel"
   Get-Process -Name powershell
   ```

El servei s'ha iniciat sol a la Sessió 0 immediatament en arrencar Windows.

<img width="531" height="209" alt="image" src="https://github.com/user-attachments/assets/5a799b87-fb6a-49b3-ba4c-4bc55b983fb0" />

---

## PAS 8: Accés al Web Dashboard i Prova d'Interacció amb l'Escriptori

### 8.1- Panell de control en temps real

Obrim `http://localhost:8080` a Microsoft Edge o Chrome:
* **KPIs**: Mètriques de càrrega de CPU i ús de memòria RAM del sistema.
* **Servei SysMain**: Es mostra clarament la targeta vermella confirmant l'estat **DISABLED (Optimitzat)**.
* **Processos (NF4)**: Taula dinàmica amb el Top 10 de processos que més recursos consumeixen, mostrant el nom de l'executable, PID, ús de RAM i temps de CPU.
* **Connexions de Xarxa**: Taula de connexions TCP locals i remotes actives.
* **Esdeveniments**: Històric en viu d'alertes del sistema operatiu.

<img width="951" height="844" alt="image" src="https://github.com/user-attachments/assets/a3144d16-326a-4e86-958e-80f46bb38cf2" />



---



## Resum de Compliment dels Requisits de l'Enunciat

| Requisit del PDF del Professor | Com s'ha resolt i documentat al projecte |
|---|---|
| **1. Crear un servei que executi un script en el mode d'inici habitual** | Hem programat `SentinelServer.ps1` i l'hem configurat com a servei d'arrencada automàtica (`PauSerraSentinel`) amb NSSM (**PASSOS 1, 4, 6 i 7**). |
| **2. Utilitzo nssm per a crear el servei** | Instal·lació i desplegament pas a pas amb `nssm.exe` (x64) versió 2.24 (**PAS 4**). |
| **3. A les 2 primeres línies afegeixo la ruta a powershell.exe i a la darrera la ruta amb els paràmetres ExecutionPolicy Bypass** | Camps configurats a la pestanya *Application* de NSSM: `Path` a `powershell.exe`, `Startup directory` a `C:\PauSerra-Sentinel` i `Arguments` amb `-ExecutionPolicy Bypass -NoProfile -File ...` (**PAS 4.1**). |
| **4. Activo l'opció de “Allow service to interact with desktop”** | Activada la casella a la pestanya *Log on* de NSSM i demostrat en viu mitjançant la notificació emergent interactiva a la pantalla de Windows (**PASSOS 4.3 i 8.2**). |
| **5. Desactiva un servei que no vulguis que consumeixi recursos** | Aturada i desactivació permanent del servei consumidor `SysMain` (Superfetch) mitjançant `Stop-Service -Force` i `Set-Service -StartupType Disabled` (**PASSOS 2.2 i 5**). |
| **6. CAL FER-HO AMB POWERSHELL COM A ADMIN, NO AMB CMD** | Justificat tècnicament a la Secció 1.4 i aplicat amb PowerShell elevat com a Administrador a tots els passos de la pràctica (**SECCIÓ 1.4 i tots els PASSOS**). |
| **7. Després ja veure el servei en execució** | Comprovat amb `Get-Service`, `nssm status`, la consola `services.msc`, el Gestor de Tasques i mitjançant la interfície web accessible a `http://localhost:8080` (**PASSOS 6, 7 i 8**). |
