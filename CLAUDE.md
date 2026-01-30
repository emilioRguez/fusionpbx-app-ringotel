# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FusionPBX Ringotel Integration App - a PHP module that integrates FusionPBX with Ringotel cloud PBX service. Enables administrators to manage FusionPBX extensions in Ringotel, monitor status, configure call parking, and integrate with Bandwidth for SMS.

**Version:** 1.1
**License:** Mozilla Public License 1.1

## Installation

```bash
cd /var/www/fusionpbx/app
git clone https://github.com/fusionpbx/fusionpbx-app-ringotel.git ringotel
php /var/www/fusionpbx/core/upgrade/upgrade.php --permissions
php /var/www/fusionpbx/core/upgrade/upgrade.php
```

Configuration requires setting the `ringotel_token` in FusionPBX Default Settings (Advanced > Default Settings > ringotel category).

## Architecture

### Request Flow

```
UI (index.php) → AJAX → service.php → RingotelClass → RingotelApiFunctions → curlRingotel → Ringotel API
```

### Key Files

| File | Purpose |
|------|---------|
| `index.php` | Main UI dashboard with tabs (Connections, Parks, Users, Integration) |
| `service.php` | API dispatcher - routes `?method=<action>` to RingotelClass methods |
| `resources/classes/ringotel.php` | Core business logic (50+ methods for org/branch/user/park management) |
| `resources/classes/ringotelRequests.php` | Builds JSON-RPC 2.0 requests to Ringotel API |
| `resources/classes/curlRingotel.php` | HTTP transport layer |
| `app_config.php` | FusionPBX app metadata, permissions, and default settings |
| `ringotel_extension_list.php` | Adds Ringotel status column to FusionPBX extensions page |

### API Communication

- **Protocol:** JSON-RPC 2.0 over HTTPS POST
- **Auth:** Bearer token in Authorization header
- **Endpoints:**
  - Admin API: `https://shell.ringotel.co/api`
  - Integration API: `https://shell.ringotel.co/integrations`
- **Timeout:** 10 seconds

### Permissions

Defined in `app_config.php`, checked via FusionPBX `permission_exists()`:
- `ringotel` - Basic access (superadmin, admin)
- `ringotel_extensions` - Extension management (superadmin, admin)
- `ringotel_superadmin` - Super admin actions (superadmin only)
- `extension_ringotel` - Show Ringotel status on extension list (superadmin, admin)

### Configuration

Settings stored in FusionPBX `v_default_settings` table under `ringotel` category:
- `ringotel_token` - Bearer token (required)
- `ringotel_organization_port` - Default: 5060
- `ringotel_organization_region` - Default: 2
- `default_connection_protocol` - Default: sip-tcp
- `default_connection_regexpires` - Default: 120

## Development Notes

- No build system, composer, or npm - pure PHP
- Uses FusionPBX core classes (database, permissions, sessions)
- Configuration loaded from `$_SESSION['ringotel']` (populated by `load_config.php`)
- Frontend uses jQuery, Bootstrap, and custom JS libraries (QRCode.js, html-to-image, multiselect-dropdown)
