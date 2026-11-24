
# University Bus & Point System (UBMS)

An offline-first Windows desktop application built for university transport offices to manage student bus/point enrollments, routes and ordered stops, vehicles, drivers, academic periods, transport fee plans, payments and printable receipts, daily boarding attendance, operational/financial reports, audit trails, and verified database backups.

---

## Features & Daily Workflows

- **Quick Student Lookup (`Ctrl+K`)**: Global search bar in the top bar searches by student ID/number, name, department, or phone number and opens the **Student Detail Workspace** (`Profile`, `Transport`, `Fees & Payments`, `Attendance`, `History`).
- **Route & Stop Enrollment**: Enroll students onto a route/point and active pickup stop for an academic period with live capacity checks, fee plan previews, route transfers (preserving history), and audited capacity overrides for authorized roles.
- **Transport Fees, Payments & Receipts**:
  - Fee plans and immutable posted charges using integer minor-unit ledger math (`Balance Due = (Posted Charges + Debit Adjustments) - (Credit/Waiver Adjustments + Allocated Posted Payments)`).
  - Transactional payment recording with automatic or explicit allocation to open charges, duplicate-receipt and overpayment protection, printable HTML receipts, and audited reversals/waivers.
- **Daily Boarding Roster & Attendance**: Mark passengers `Present`, `Absent`, or `Excused` per route and trip run (unmarked riders start explicitly as `Not Recorded`), with confirmation-protected `Mark Remaining Absent` and undo support.
- **CSV Import & Exports**: Guided UTF-8 CSV student import (preview, column mapping, validation, `all_or_nothing` / `valid_only` modes) plus formula-injection-safe CSV exports and printable HTML reports across 7 operational and financial report types.
- **Role-Based Access Control & Audit Trail**: Enforced in application services as well as UI navigation, with salted `PBKDF2-HMAC-SHA256` password hashing and audit logging across auth, assignments, finance, settings, and backup actions.
- **Verified Backup & Restore**: Create `.bak` snapshots verified with `PRAGMA integrity_check`, with automatic pre-restore safety snapshots and atomic replacement.
- **Fictional `[DEMO]` Dataset**: Optional one-click demo dataset during first-run setup (strictly labeled `[DEMO]`) that can be removed cleanly from **Admin & Backup → Demo Data**.

---

## Technology Stack

| Layer | Technology | Details |
|---|---|---|
| **Desktop UI** | Python + **PySide6 (Qt 6.12)** | Windows 11 Light `QPalette` + QSS theme, dark slate sidebar, compact top bar, crisp in-memory vector SVG icons (`QSvgRenderer`), non-blocking toast notifications |
| **Domain & Services** | Modular Python (`ubms/domain/`, `ubms/services/`) | Integer minor-unit currency handling, inclusive ISO `YYYY-MM-DD` date-effective rules, centralized permission checks |
| **Database** | Embedded **SQLite 3** (`%LOCALAPPDATA%\UBMS\ubms.db`) | WAL mode, `PRAGMA foreign_keys = ON`, parameterized SQL, versioned `schema_migrations` |
| **Packaging** | **PyInstaller** + **Inno Setup 6** | Standalone Windows `.exe` bundle and Windows setup installer (`v1.1.0`) |

---

## Installation (Windows)

### Option 1: Windows Setup Installer
1. Download or run `dist_installer\UniversityBusManagementSystem_Setup_v1.1.0.exe` (built via [installer/ubms.iss](installer/ubms.iss)).
   > **SmartScreen Note**: The installer is currently **unsigned** (no commercial code-signing certificate). If Windows Defender SmartScreen shows *"Windows protected your PC"*, click **More info** → **Run anyway**.
2. Complete the setup wizard.
3. On first launch, complete the **First-Run Setup Wizard** (institution name, initial academic period, currency/timezone, primary Administrator account, and optional `[DEMO]` dataset).

User data is stored in `%LOCALAPPDATA%\UBMS\ubms.db` (overrideable with the `UBMS_DATA_DIR` environment variable) and is preserved across upgrades and uninstalls.

### Option 2: Run from Source
Requires **Python 3** on Windows (developed and tested on Python 3.14) and `PySide6`:

```powershell
git clone https://github.com/<your-username>/<your-repo>.git
cd ubms

# Install UI dependency
python -m pip install PySide6

# Launch the desktop application
python run_ubms.py
```

---

## Roles & Permissions

Permissions are enforced at the service layer across the following roles:

| Role | Code | Summary of Access |
|---|---|---|
| **Administrator** | `administrator` | Full access (`*`): user/role management, settings, periods, audit log, backup/restore, financial & capacity overrides |
| **Transport Manager** | `transport_manager` | Students (read/write/import/archive), routes & stops, vehicles, drivers, assignments (with capacity override), attendance, notices, operational reports |
| **Accountant** | `accountant` | Fee plans, charges, payments, receipts, adjustments/reversals, financial reports, read-only student/route/assignment lookup |
| **Transport Operator** | `transport_operator` | Trip rosters, daily boarding attendance recording/correction, service notices |
| **Read-only Viewer** | `viewer` | Read-only dashboard, student/route/assignment lookup, permitted operational reports |
| **Student (Portal)** | `student` | Read-only personal transport portal and active notices |

*(Legacy role aliases `staff` and `driver` remain supported for backward compatibility.)*

---

## Project Structure

```text
ubms/
├── run_ubms.py                          # Application entry point
├── UniversityBusManagementSystem.spec   # PyInstaller build specification
├── installer/
│   ├── ubms.iss                         # Inno Setup 6 installer script (v1.1.0)
│   ├── generate_icon.py                 # App icon generator script
│   └── app_icon.ico / app_icon.png      # Application icons
├── docs/
│   ├── ASSUMPTIONS.md                   # Architecture, role matrix & domain conventions
│   ├── USER_GUIDE.md                    # Staff workflows, backup/restore & troubleshooting
│   └── TECHNICAL_HANDOFF.md             # Build verification & handoff notes
├── tests/
│   ├── test_all_services.py             # Core service integration tests
│   ├── test_fees_roles_migrations.py    # Finance, RBAC, and DB migration tests
│   └── test_gui_smoke.py                # PySide6 GUI smoke check across all views
└── ubms/
    ├── app.py                           # App bootstrap & window lifecycle
    ├── config.py                        # Paths, constants, roles & permission matrix
    ├── domain/                          # Money minor-unit & date-range helpers
    ├── database/                        # SQLite connection, schema, migrations & demo seed
    ├── services/                        # Auth, students, routes, fleet, fees, attendance, reports, backup, audit
    └── ui/
        ├── shell.py                     # Sidebar, top bar, Ctrl+K search & navigation shell
        ├── styles.py                    # Explicit Light QPalette & Windows 11 QSS theme
        ├── icons.py                     # Vector SVG icon renderer
        ├── components/                  # Data table, dialogs, finance dialogs, CSV import, toast banner
        └── views/                       # 15 PySide6 desktop views
```

---

