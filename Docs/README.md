# AstCTI – Full Setup Guide

AstCTI is a lightweight **Computer Telephony Integration (CTI)** solution for **Asterisk**, composed of:

- **AstCTIServer** – backend service (AMI + MySQL)
- **AstCTIClient** – desktop client reacting to call events

This README documents the complete setup using the provided demo configuration.

---

## Table of Contents

- [Requirements](#requirements)
- [Database Setup](#database-setup)
- [AstCTIServer Setup](#astctiserver-setup)
- [Asterisk Configuration](#asterisk-configuration)
- [AstCTIClient Setup](#astcticlient-setup)
- [Inbound & Outbound Contexts](#inbound--outbound-contexts)
- [Testing](#testing)

---

## Requirements

- Asterisk with AMI enabled
- MySQL 5.0.x or compatible
- .NET Framework 2.0 or Mono
- Windows (client) / Linux or Windows (server)

---

## Database Setup

A working **MySQL** installation is required.  
The examples below assume the use of the predefined `test` database.

### Create Table

```sql
CREATE TABLE `cti` (
  `USERNAME` varchar(255) default NULL,
  `SECRET` varchar(255) default NULL,
  `HOST` varchar(255) default NULL,
  `EXT` varchar(255) default NULL,
  UNIQUE KEY `USERNAME` (`USERNAME`),
  UNIQUE KEY `EXT` (`EXT`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

This table stores credentials used by **AstCTIServer** to authenticate clients.  
Passwords are stored as **MD5 hashes**.

### Create a User

```sql
INSERT INTO cti VALUES ('test', MD5('test'), '0.0.0.0', '');
```

> 🔐 Recommended:
> - one MySQL account for **AstCTIServer**
> - one MySQL account for **AstCTIClient**

MySQL user creation is not covered here.

---

## AstCTIServer Setup

AstCTIServer requires a **.NET 2.0–compatible runtime**.

### Tested Platforms

- Windows XP SP2 + .NET Framework 2.0
- Fedora Core 7, Kernel 2.6.22 + Mono 1.2.5
- Gentoo Linux, Kernel 2.6.22 + Mono 1.2.4

### Configuration File

AstCTIServer is configured via an **XML file** with four sections:

- `database` – MySQL connection
- `logging` – log file settings
- `manager` – Asterisk AMI configuration
- `ctiserver` – socket and server parameters

Ensure an AMI user exists in `asterisk/manager.conf` matching this configuration.

### Starting the Server

AstCTIServer does not run as a daemon/service.

#### Windows

- Double-click `AstCTIServer.exe`
- Or start it from a console

#### Linux

```bash
mono AstCTIServer.exe
```

If configured correctly:

- Logs are written to `logs/`
- Asterisk shows an active AMI connection

---

## Asterisk Configuration

Demo configuration files are available under `Docs/Demo`:

- `asterisk/manager.conf`
- `asterisk/extensions.conf`
- `asterisk/sip.conf`
- `asterisk/queues.conf`
- `asterisk/sounds/astctidemo/enter_five_digits.wav`

### AMI Setup

Ensure `manager.conf` allows connections from **AstCTIServer**.  
Refer to the demo file for an example user.

### Dialplan Setup

1. Copy demo contexts from `extensions.conf`
2. Merge SIP agents from `sip.conf`
3. Merge queue definitions from `queues.conf`
4. Copy sounds to:

```
/var/lib/asterisk/sounds/astctidemo
```

5. Reload dialplan:

```bash
asterisk -rx "extensions reload"
```

### Demo Extensions

#### Extension 100

- IVR asks for 5 digits
- Stored in variable `calldata`
- Call routed to `SIP/201`

#### Extension 101

- Same IVR logic
- Call routed to a queue (single agent: `SIP/201`)

Required dialplan line:

```asterisk
exten => 101,3,Set(calldata=${cdata})
```

> ⚠️ Always define `calldata` before routing calls to agents.

---

## AstCTIClient Setup

AstCTIClient is a **Win32 C# application** targeting **.NET 2.0**.

Click **Settings** to open the configuration window.

### Configuration Sections

#### 1. CTI Application

- `CalleridFadeSpeed`
- `CalleridTimeout`
- `TriggerCallerid`
- `CTIContextes`
- `CTIOutboundContextes`

#### 2. CTI Server

- `Host`
- `Port`
- `Username`
- `Password`
- `PhoneExt` (e.g. `SIP/201`)
- `SocketTimeout`

#### 3. Database

- `MySQLHost`
- `MySQLUsername`
- `MySQLPassword`
- `MySQLDatabase`

#### 4. Interface

- `MinimizeOnStart` – minimize to tray after login

#### 5. Misc

- `SaveOnClose` – persist settings on close

---

## Inbound & Outbound Contexts

### Inbound (CTIContextes)

Inbound contexts define which application is launched when a call is answered.

Example:

- Context: `astctidemo`
- Application: `c:\program files\Internet Explorer\iexplorer.exe`
- Parameters:

```
http://centralino-voip.brunosalzano.com/demo_page.php?callerid={CALLERID}&calldata={CALLDATA}&channel={CHANNEL}&uniqueid={UNIQUEID}
```

When `SIP/201` answers a call, the browser is launched automatically.

### Outbound (CTIOutboundContextes)

Outbound contexts allow call origination directly from AstCTIClient.

Example:

- Context: `astctidemo`
- Allowed extensions: `200–299`

---

## Testing

1. Start **AstCTIServer**
2. Verify AMI connection in Asterisk
3. Login from **AstCTIClient**
4. Call extension **100** or **101**

If configured correctly, the client reacts to the call event.

---

## License

Provided as-is, without warranty.
