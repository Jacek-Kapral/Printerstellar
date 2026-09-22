██████╗ ██████╗ ██╗███╗   ██╗████████╗███████╗██████╗ ███████╗████████╗███████╗██╗     ██╗      █████╗ ██████╗
██╔══██╗██╔══██╗██║████╗  ██║╚══██╔══╝██╔════╝██╔══██╗██╔════╝╚══██╔══╝██╔════╝██║     ██║     ██╔══██╗██╔══██╗
██████╔╝██████╔╝██║██╔██╗ ██║   ██║   █████╗  ██████╔╝███████╗   ██║   █████╗  ██║     ██║     ███████║██████╔╝
██╔═══╝ ██╔══██╗██║██║╚██╗██║   ██║   ██╔══╝  ██╔══██╗╚════██║   ██║   ██╔══╝  ██║     ██║     ██╔══██║██╔══██╗
██║     ██║  ██║██║██║ ╚████║   ██║   ███████╗██║  ██║███████║   ██║   ███████╗███████╗███████╗██║  ██║██║  ██║
╚═╝     ╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝   ╚═╝   ╚══════╝╚═╝  ╚═╝╚══════╝   ╚═╝   ╚══════╝╚══════╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝

                         Universal Network Printer Installer

									By Kapral
				
				

		
# PrinterInstaller — Roadmap

A small Windows utility for quickly installing network printers from a driver `.inf` file.

The primary use case is field service: plug in a USB drive, run one executable, enter the printer IP and name, select the driver, and let the application handle the Windows printer setup.

---

## Project Goal

Create a portable, technician-friendly Windows application that reduces a typical network-printer installation to a few simple steps:

1. Enter the printer IP address.
2. Enter the Windows printer name.
3. Select a driver `.inf` file.
4. If the INF contains multiple printer models, select the correct model.
5. Automatically create a Standard TCP/IP printer port using RAW on port `9100`.
6. Enable SNMP on the port.
7. Install the selected printer driver.
8. Create the Windows printer using the requested name.
9. Show a clear success/failure result and useful diagnostic information.

The application should be designed primarily for use from a USB flash drive at customer sites.

---

## Target Platform

- Windows 10/11
- x64 initially
- Administrator privileges required
- .NET / C#
- Prefer a self-contained executable so the tool can be used without installing the .NET runtime.

---

## Proposed Technology

### Language

**C#**

Reasons:

- Excellent Windows integration.
- Direct access to Windows APIs.
- Good support for printer/spooler functionality.
- Easy GUI development.
- Easy creation of a self-contained `.exe`.
- Much better fit for a long-term Windows utility than a collection of PowerShell scripts.

### UI

Start with **WinForms**.

The UI is intentionally simple. This is a technician tool, not a consumer application.

### Windows integration

Prefer Windows APIs and native printer-management mechanisms where practical.

Potential technologies/components:

- Windows Print Spooler APIs
- Windows SetupAPI / driver installation mechanisms
- Standard TCP/IP Port Monitor
- Windows printer management APIs
- PowerShell only where it provides a clear advantage or simplifies compatibility

Avoid making the application depend on PowerShell scripts for its core functionality.

---

# MVP — Version 0.1

The first version should do only the essential job reliably.

## Main Window

Proposed layout:

```text
+------------------------------------------------+
|                 PrinterInstaller               |
+------------------------------------------------+
|                                                |
| IP Address:                                    |
| [ 192.168.1.123                            ]   |
|                                                |
| Printer Name:                                  |
| [ Accounting - Color Printer               ]  |
|                                                |
| Driver (.INF):                                 |
| [ C:\Drivers\Printer\driver.inf           ] |
|                              [ Browse... ]     |
|                                                |
| [ ] Automatically use selected driver model    |
|     as printer name                            |
|                                                |
|                         [ INSTALL PRINTER ]    |
|                                                |
| Status: Ready                                  |
+------------------------------------------------+
```

### Fields

#### IP Address

Required.

Example:

```text
192.168.1.123
```

Validate that the value is a valid IPv4 address.

IPv6 support can be considered later.

#### Printer Name

Optional if the "use driver model as printer name" option is enabled.

Example:

```text
Accounting - Color Printer
```

This is the **Windows printer name**, not the TCP/IP port name.

#### Driver INF

Required.

Use a standard Windows file picker.

The user selects an `.inf` file from the local disk or USB drive.

---

# Driver Model Selection

An INF can contain multiple printer models.

Example:

```text
HP LaserJet Pro M404
HP LaserJet Pro M405
HP LaserJet Enterprise M406
```

If multiple applicable printer models are found, show a separate selection dialog:

```text
+----------------------------------------------+
|             Select Printer Model             |
+----------------------------------------------+
|                                              |
| Driver contains multiple models:             |
|                                              |
| ( ) HP LaserJet Pro M404                     |
| ( ) HP LaserJet Pro M405                     |
| ( ) HP LaserJet Enterprise M406              |
|                                              |
|                         [ Cancel ] [ OK ]     |
+----------------------------------------------+
```

If only one applicable model is found, skip the dialog and select it automatically.

The selected model must be clearly displayed in the main window before installation.

---

# Printer Name Behavior

The printer name should be independent from the driver model.

Example:

```text
IP Address:
192.168.1.123

Driver Model:
Ricoh IM C3000

Printer Name:
Accounting - Main Printer
```

The Windows printer should therefore appear as:

```text
Accounting - Main Printer
```

while using:

```text
Ricoh IM C3000
```

as the selected driver.

## Convenience Checkbox

Provide:

```text
[ ] Use selected driver model as printer name
```

When enabled:

```text
Printer Name = Ricoh IM C3000
```

This is useful when the technician does not care about a custom printer name.

The checkbox should automatically populate/update the printer-name field, while still allowing the user to disable the option and enter a custom name.

---

# Network Port

The application should automatically create a Standard TCP/IP printer port.

Required configuration:

```text
Protocol: RAW
Port:     9100
SNMP:     Enabled
```

Suggested automatically generated port name:

```text
IP_192.168.1.123
```

The port name is an implementation detail and should not normally be exposed as the printer name.

## SNMP

Initial MVP assumption:

```text
SNMP: Enabled
Community: public
```

SNMP community configuration should become configurable in a later version.

---

# Installation Flow

The intended sequence is:

```text
Start
  |
  v
Validate IP
  |
  v
Validate INF
  |
  v
Read available printer models
  |
  +---- one model ----> select automatically
  |
  +---- multiple -----> show model selection dialog
  |
  v
Validate printer name
  |
  v
Check whether printer/port already exists
  |
  v
Install driver
  |
  v
Create TCP/IP port
  |
  v
Configure RAW 9100 + SNMP
  |
  v
Create Windows printer
  |
  v
Verify installation
  |
  v
Show result
```

---

# Existing Printer / Port Handling

The application should not blindly fail if something already exists.

Before installation, check:

- Does a printer with the requested name already exist?
- Does a TCP/IP port for the specified IP already exist?
- Does the specified printer already use that port?
- Is the selected driver already installed?

Possible behavior:

```text
Printer "Accounting - Color Printer" already exists.

[ Cancel ]
[ Reinstall ]
```

For an existing port:

```text
TCP/IP port for 192.168.1.123 already exists.

[ Use Existing Port ]
[ Recreate Port ]
[ Cancel ]
```

Exact behavior can be refined after the MVP.

---

# Status / Logging

The main window should show human-readable progress.

Example:

```text
Installing driver...
OK

Creating TCP/IP port...
OK

Configuring RAW 9100...
OK

Enabling SNMP...
OK

Creating printer...
OK

Testing configuration...
OK

Installation completed successfully.
```

Failures should explain what failed whenever possible.

Example:

```text
ERROR: Failed to install printer driver.

Windows error:
0x00000002

The specified file was not found.
```

---

# Logging

Create a simple local log file.

Possible location:

```text
%LOCALAPPDATA%\PrinterInstaller\Logs\
```

Example:

```text
2026-09-22 21:14:03
IP: 192.168.1.123
Printer: Accounting - Color Printer
Driver: Ricoh IM C3000
Port: IP_192.168.1.123
Protocol: RAW
Port: 9100
SNMP: Enabled
Result: SUCCESS
```

A later version may provide a "Copy log" button.

---

# Validation

Before changing the system, validate as much as possible.

## IP

- Valid IPv4 address.
- Reject empty input.
- Reject malformed addresses.

## INF

- File exists.
- Extension is `.inf`.
- File can be accessed.
- Contains at least one applicable printer driver/model.

## Printer name

- Not empty unless automatic naming is enabled.
- Validate Windows printer-name restrictions.

---

# Verification

After installation, verify:

1. Driver exists.
2. TCP/IP port exists.
3. Port points to the requested IP.
4. RAW protocol is configured.
5. Port `9100` is configured.
6. SNMP is enabled.
7. Printer exists.
8. Printer uses the expected driver.
9. Printer uses the expected port.

A real print-test page is useful but should probably be an optional action rather than automatic MVP behavior.

---

# Suggested Project Structure

```text
PrinterInstaller/
|
+-- PrinterInstaller.sln
|
+-- src/
|   |
|   +-- PrinterInstaller/
|       |
|       +-- Program.cs
|       +-- MainForm.cs
|       +-- ModelSelectionForm.cs
|       |
|       +-- Models/
|       |   +-- PrinterDriverModel.cs
|       |   +-- PrinterConfiguration.cs
|       |
|       +-- Services/
|       |   +-- DriverService.cs
|       |   +-- PrinterService.cs
|       |   +-- TcpIpPortService.cs
|       |   +-- InfParserService.cs
|       |   +-- PrinterVerificationService.cs
|       |   +-- LoggingService.cs
|       |
|       +-- Native/
|       |   +-- PrintSpoolerNative.cs
|       |   +-- SetupApiNative.cs
|       |
|       +-- Validation/
|           +-- IpAddressValidator.cs
|           +-- PrinterNameValidator.cs
|
+-- tests/
|   +-- PrinterInstaller.Tests/
|
+-- docs/
|   +-- architecture.md
|   +-- troubleshooting.md
|
+-- README.md
+-- ROADMAP.md
+-- LICENSE
```

This structure is intentionally more modular than necessary for v0.1 so that the project does not become a single giant `MainForm.cs`.

---

# Development Milestones

## v0.1 — Basic Installer

- [ ] Create C# WinForms project.
- [ ] Create main window.
- [ ] IP address input.
- [ ] Printer name input.
- [ ] INF file picker.
- [ ] Read printer models from INF.
- [ ] Model selection dialog.
- [ ] Driver installation.
- [ ] Create Standard TCP/IP port.
- [ ] Configure RAW 9100.
- [ ] Enable SNMP.
- [ ] Create Windows printer.
- [ ] Basic verification.
- [ ] Human-readable status output.
- [ ] Basic error handling.

**Goal:** Reliably install a normal RAW/9100 SNMP network printer from an INF.

---

# v0.2 — Technician Quality of Life

- [ ] Existing printer detection.
- [ ] Existing port detection.
- [ ] Reuse existing port.
- [ ] Reinstall printer option.
- [ ] Better error messages.
- [ ] Log files.
- [ ] Copy log to clipboard.
- [ ] Remember last-used directory.
- [ ] Remember last-used SNMP settings.
- [ ] Optional print-test button.
- [ ] Self-contained single-file build.

---

# v0.3 — Automatic Network Detection

Investigate SNMP-based device discovery.

Possible workflow:

```text
IP entered
   |
   v
SNMP query
   |
   v
Device responds
   |
   v
Read sysName / sysDescr / printer information
   |
   v
Display:

Device detected:
Ricoh IM C3000
```

Potentially use this information to assist with driver-model selection.

Important: automatic detection should remain an assistive feature. The technician should always be able to override it manually.

---

# v0.4 — Driver Library / USB Workflow

Optional support for a predictable USB layout:

```text
PrinterInstaller.exe

Drivers/
    Canon/
    HP/
    Kyocera/
    KonicaMinolta/
    Ricoh/
    Xerox/
```

Potential features:

- Search a selected folder for INF files.
- Display manufacturer/model information.
- Remember recently used drivers.
- Allow the technician to keep a personal driver collection on the same USB drive.

Do not make the application depend on this folder structure.

The normal workflow should continue to support selecting any INF manually.

---

# v0.5 — Advanced Printer Options

Potential future options:

- LPR instead of RAW.
- Custom RAW port.
- Configurable SNMP community.
- SNMP v1/v2 settings where supported.
- Disable SNMP.
- Custom TCP/IP port name.
- WSD avoidance.
- Printer location/comment.
- Default printer.
- Color/mono defaults.
- Duplex defaults.
- Paper size.
- Tray configuration.

These should not complicate the basic installation screen.

A possible "Advanced options" section can hide them.

---

# v1.0 — Stable Field Tool

The 1.0 release should prioritize:

- Reliability.
- Clear error messages.
- Safe handling of existing printers and ports.
- Compatibility with common printer manufacturers.
- Portable deployment.
- No unnecessary dependencies.
- Detailed logging.
- Predictable behavior.

The application should remain fast to use:

> **IP → Name → INF → Model (if necessary) → Install**

---

# Design Principles

## 1. Technician first

The application should minimize clicks and typing.

## 2. No unnecessary automation

Automation should help the technician, not make decisions that are difficult to override.

## 3. Never hide important configuration

The tool should make it clear which:

- IP address
- printer name
- driver
- driver model
- port
- protocol
- SNMP configuration

will be used.

## 4. Fail safely

Do not silently overwrite existing printers or ports.

## 5. Useful errors

"Installation failed" is not enough.

Whenever Windows provides an error code or useful diagnostic information, display it and put it in the log.

## 6. Portable

The ideal deployment is:

```text
USB drive
    |
    +-- PrinterInstaller.exe
    +-- Drivers/
```

No installer should be required for the tool itself.

---

# Future Idea: Quick Install Mode

Once the standard workflow is reliable, consider a compact mode:

```text
IP:
[ 192.168.1.123 ]

Name:
[ Office Printer ]

Driver:
[ Browse... ]

[ INSTALL ]
```

If the selected INF contains exactly one model, everything else happens automatically.

The technician's common workflow becomes only:

```text
Paste IP
Type name
Select INF
Click Install
```

This should be the ultimate usability goal.

---

# Non-Goals

The project is **not** intended to be:

- A complete enterprise print-management system.
- A print server management suite.
- A replacement for vendor-specific configuration utilities.
- A universal solution for every exotic printer protocol.
- A driver repository containing proprietary vendor drivers.

The application should focus on one job and do it very well:

> **Quickly install a Windows network printer from a supplied driver INF.**

---

# Initial GitHub TODO

Before writing the installer logic:

- [ ] Create repository.
- [ ] Add this roadmap.
- [ ] Create `.gitignore` for Visual Studio/.NET.
- [ ] Create solution and WinForms project.
- [ ] Decide minimum supported Windows version.
- [ ] Implement basic UI.
- [ ] Research the Windows APIs required for:
  - [ ] INF driver/model enumeration.
  - [ ] Driver installation.
  - [ ] TCP/IP port creation.
  - [ ] RAW 9100 configuration.
  - [ ] SNMP configuration.
  - [ ] Printer creation.
  - [ ] Printer/port verification.
- [ ] Build a first proof of concept before polishing the UI.

---

# Definition of Done — MVP

The MVP is considered successful when a technician can take a Windows PC with administrator access and:

1. Start `PrinterInstaller.exe` from a USB drive.
2. Enter an IP address.
3. Enter a printer name or select automatic naming.
4. Select an INF file.
5. Select a driver model if necessary.
6. Click **Install Printer**.
7. Wait for the process to complete.
8. See a clear success/failure result.
9. Find the correctly configured printer in Windows.
10. Have the printer use RAW `9100` with SNMP enabled.

The entire process should be significantly faster and less error-prone than manually navigating Windows printer installation dialogs.
