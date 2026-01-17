# AstCTI

AstCTI is a simple Computer Telephony Integration (CTI) solution built around **Asterisk**, composed of:

- **AstCTIServer** – backend service that connects to Asterisk (AMI) and MySQL
- **AstCTIClient** – desktop client that reacts to inbound/outbound calls

This document explains how to set up the full environment.

---

## Table of Contents

- [Requirements](#requirements)
- [Database Setup](#database-setup)
- [AstCTIServer Setup](#astctiserver-setup)
- [Asterisk Dialplan](#asterisk-dialplan)
- [AstCTIClient Setup](#astcticlient-setup)
- [Testing](#testing)

---

## Requirements

- Asterisk (with AMI enabled)
- MySQL 5.0.x or compatible
- .NET Framework 2.0 or Mono
- Windows (client) or Linux (server)

---

## Database Setup

You need a working **MySQL** instance.  
In the examples below, the predefined `test` database is used.

### Create the `cti` Table

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

> ℹ️ It is recommended to create **two MySQL users**:
> - one for the server
> - one for the clients

MySQL user creation is out of scope for this document.

---

## AstCTIServer Setup

AstCTIServer requires a **.NET 2.0** compatible runtime.

### Tested Platforms

- Windows XP SP2 + .NET Framework 2.0
- Fedora Core 7, Kernel 2.6.22 + Mono 1.2.5
- Gentoo Linux, Kernel 2.6.22 + Mono 1.2.4

### Configuration

AstCTIServer is configured via an **XML configuration file**, divided into four sections:

- `database` – MySQL connection settings
- `logging` – file logging configuration
- `manager` – Asterisk Manager Interface (AMI) settings
- `ctiserver` – server socket configuration

Ensure that an AMI user is correctly configured in Asterisk (`manager.conf`).

### Starting the Server

AstCTIServer does **not** run as a service or daemon.

#### Windows

- Double-click `AstCTIServer.exe`
- Or run it from a command prompt

#### Linux

```bash
mono AstCTIServer.exe
```

If everything is configured correctly:

- Logs appear in the `logs/` directory
- A manager connection is visible in the Asterisk console

---

## Asterisk Dialplan

A demo dialplan is available in `Docs/Demo`:

- `asterisk/extensions.conf`
- `asterisk/sip.conf`
- `asterisk/queues.conf`
- `asterisk/sounds/astctidemo/enter_five_digits.wav`

### Installation Steps

1. Copy demo contexts from `extensions.conf` into your dialplan
2. Merge SIP agents into `sip.conf`
3. Merge queue definitions into `queues.conf`
4. Copy sound files to:

```
/var/lib/asterisk/sounds/astctidemo
```

5. Reload the dialplan:

```bash
asterisk -rx "extensions reload"
```

### Demo Extensions

#### Extension 100

- IVR asks for 5 digits
- Digits are stored in `calldata`
- Call is sent to `SIP/201`

#### Extension 101

- Same IVR logic
- Call is sent to a queue (single agent: `SIP/201`)

Key line:

```asterisk
exten => 101,3,Set(calldata=${cdata})
```

> ⚠️ Always set a variable named `calldata` **before** routing calls to agents.

---

## AstCTIClient Setup

AstCTIClient is a **Win32 C# (.NET 2.0)** application.

After startup, click **Settings** to configure the client.

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

## CTI Contextes

Inbound contexts determine which application is launched on incoming calls.

Example configuration:

- Context: `astctidemo`
- Application: `iexplorer.exe`
- Parameters:

```
http://centralino-voip.brunosalzano.com/demo_page.php?callerid={CALLERID}&calldata={CALLDATA}&channel={CHANNEL}&uniqueid={UNIQUEID}
```

When a call is answered on `SIP/201`, the browser is automatically launched.

---

## CTI Outbound Contextes

Outbound contexts allow calls to be originated directly from AstCTIClient.

Example:

- Context: `astctidemo`
- Allowed extensions: `200–299`

---

## Testing

1. Start **AstCTIServer**
2. Verify AMI connection to Asterisk
3. Login from **AstCTIClient**
4. Call extension **100** or **101** from another phone

✅ If everything is set up correctly, the client application will react to the call.

---

## License

This project is provided as-is, without warranty.
