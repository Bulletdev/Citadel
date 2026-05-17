 
<div align="center">

[![Rust](https://img.shields.io/badge/rust-1.80+-orange?logo=rust)](https://www.rust-lang.org/)
[![Tauri](https://img.shields.io/badge/tauri-2.x-blue?logo=tauri)](https://tauri.app/)
[![SQLCipher](https://img.shields.io/badge/SQLCipher-4.5+-green)](https://www.zetetic.net/sqlcipher/)
[![AES-256-GCM](https://img.shields.io/badge/AES--256--GCM-AEAD-darkblue)]()
[![Argon2id](https://img.shields.io/badge/KDF-Argon2id-purple)](https://datatracker.ietf.org/doc/html/rfc9106)
[![License](https://img.shields.io/badge/license-RESTRICTED-red)]()

</div>

---
<p align="center">
  <img src="https://i.ibb.co/NMfkdNv/banner.png" alt="profile banner" border="0">

```
╔════════════════════════════════════════════════════════════════════════════╗
║  PROJECT CITADEL - Rust / Tauri 2.x / SQLite+SQLCipher / AES-256-GCM       ║
╠════════════════════════════════════════════════════════════════════════════╣
║  Local-first inventory management for high-risk operational environments.  ║
║  Air-gapped · Plausible Deniability · Dead Man's Switch · Atomic Wipe      ║
╚════════════════════════════════════════════════════════════════════════════╝
```

---

<details>
<summary><kbd>▶ Security Capabilities (click to expand)</kbd></summary>

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  [■] AES-256-GCM Encryption    - All data encrypted at rest since byte 0    │
│  [■] Argon2id KDF              - Memory-hard key derivation (RFC 9106)      │
│  [■] Dual-Database Auth        - Real + Decoy with identical timing         │
│  [■] Plausible Deniability     - Duress Password exposes believable dataset │
│  [■] Dead Man's Switch         - Auto secure-shutdown on inactivity (180s)  │
│  [■] Panic Button              - 3-pass disk wipe in < 30s(Ctrl+Shift+Alt+W)│
│  [■] BLAKE3 Hash Chain         - Tamper-evident append-only ledger          │
│  [■] Zeroization               - Keys erased from RAM before any exit path  │
│  [■] Zero Network              - Air-gapped by design, no I/O syscalls      │
│  [■] Single Static Binary      - No runtime dependencies on the host OS     │
│  [■] journal_mode=DELETE       - No WAL/SHM forensic artifacts on disk      │
│  [■] secure_delete=ON          - SQLCipher overwrites deleted pages         │
│  [■] Decoy DB Isolation        - Real DB untouched during Duress sessions   │
│  [■] Inventory Ledger          - ENTRY / EXIT / TRANSFER / ADJUSTMENT       │
│  [■] Node Balance Triggers     - Atomic SQLite triggers maintain balance    │
│  [■] Balance Reconciliation    - Ledger vs materialized view consistency    │
│  [■] Encrypted CSV Export      - Ad-hoc key, separate from master key       │
│  [■] POSIX Signal Wipe         - kill -SIGUSR1 triggers full wipe           │
└─────────────────────────────────────────────────────────────────────────────┘
```

</details>

---

## Table of Contents

```
┌──────────────────────────────────────────────────────┐
│  01 · Quick Start                                    │
│  02 · Technology Stack                               │
│  03 · Architecture                                   │
│  04 · Data Model                                     │
│  05 · Cryptographic Design                           │
│  06 · Security Mechanisms                            │
│  07 · Threat Model                                   │
│  08 · Building                                       │
│  09 · Testing & Acceptance Criteria                  │
│  10 · Roadmap                                        │
│  11 · Known Limitations                              │
│  12 · License                                        │
└──────────────────────────────────────────────────────┘
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Quick Start


```
[✓] Rust (stable >= 1.80)
[✓] Tauri CLI       cargo install tauri-cli
[✓] Node (optional) only for tauri dev tooling
```

<details>
<summary><kbd>▶ Build and Run (click to expand)</kbd></summary>

**1. Clone the repository:**
```bash
git clone <repository-url>
cd citadel
```

**2. First-time setup - generate databases:**
```bash
# Requires interactive setup to define real and duress passwords
cargo tauri dev
# On first launch: Setup Mode creates real.db and decoy.db
```

**3. Development build:**
```bash
cargo tauri dev
```

**4. Production build (current platform):**
```bash
cargo tauri build
```

**5. Fully static Linux binary (recommended for deployment):**
```bash
rustup target add x86_64-unknown-linux-musl
cargo tauri build --target x86_64-unknown-linux-musl
ldd target/x86_64-unknown-linux-musl/release/citadel
# Expected: not a dynamic executable
```

**6. Verify security posture:**
```bash
cargo audit
strings target/release/citadel | grep -iE 'key|pass|secret'
file citadel_real.db    # Expected: data
strace -e trace=network ./citadel  # Expected: no network syscalls
```

</details>

```
  Binary:     ./target/release/citadel  (or .exe / .app)
  Real DB:    ./citadel_real.db
  Decoy DB:   ./citadel_decoy.db
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Technology Stack

```
╔══════════════════════╦═════════════════════════════════════════════════╗
║  LAYER               ║  TECHNOLOGY                                     ║
╠══════════════════════╬═════════════════════════════════════════════════╣
║  Core Language       ║  Rust (stable >= 1.80)                          ║
║  Desktop Framework   ║  Tauri 2.x (>= 2.1.0)                           ║
║  Frontend            ║  HTML5 / CSS3 / Vanilla JS (ES2022)             ║
║  Database            ║  SQLite 3.x + SQLCipher >= 4.5                  ║
║  Primary Cipher      ║  AES-256-GCM (aes-gcm crate)                    ║
║  Key Derivation      ║  Argon2id (argon2 crate >= 0.5)                 ║
║  Hashing             ║  BLAKE3 (blake3 crate)                          ║
║  Entropy             ║  OsRng / getrandom (OS CSPRNG)                  ║
║  Memory Safety       ║  zeroize crate >= 1.7 (ZeroizeOnDrop)           ║
║  Async Runtime       ║  Tokio (Dead Man's Switch timer)                ║
║  Distribution        ║  Single static binary (musl for Linux)          ║
╚══════════════════════╩═════════════════════════════════════════════════╝
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        SINGLE AUTONOMOUS BINARY                         │
│  (citadel.exe / citadel / citadel.app)                                  │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                  PRESENTATION LAYER                             │    │
│  │        HTML / CSS / Vanilla JS  (Local WebView)                 │    │
│  │  [Login] [Dashboard] [Inventory] [Transactions] [Panic]         │    │
│  └───────────────────────┬─────────────────────────────────────────┘    │
│                          │  Tauri IPC (invoke / emit)                   │
│  ┌───────────────────────▼─────────────────────────────────────────┐    │
│  │                    CORE LAYER (RUST)                            │    │
│  │                                                                 │    │
│  │  ┌──────────────┐  ┌──────────────┐ ┌───────────────────┐       │    │
│  │  │ AuthManager  │  │ CryptoEngine │ │  WipeOrchestrator │       │    │
│  │  │  Argon2id KDF│  │  AES-256-GCM │ │   Zeroize RAM     │       │    │
│  │  │  Dual-DB Auth│  │  BLAKE3 Hash │ │   3-pass Overwrite│       │    │
│  │  │  Session Mgmt│  │  Key Deriv   │ │   Process Kill    │       │    │
│  │  └──────┬───────┘  └──────┬───────┘ └────────┬──────────┘       │    │
│  │         └─────────────────┬──────────────────┘                  │    │
│  │  ┌────────────────────────▼──────────────┐                      │    │
│  │  │           InventoryService            │                      │    │
│  │  │  CRUD Items / Nodes                   │                      │    │
│  │  │  Transaction Ledger (append-only)     │                      │    │
│  │  │  BLAKE3 Hash Chain Verification       │                      │    │
│  │  │  Balance Reconciliation               │                      │    │
│  │  └──────────────────────┬────────────────┘                      │    │
│  │  ┌──────────────────────▼────────────────────────────────────┐  │    │
│  │  │        DatabaseLayer (SQLite + SQLCipher)                 │  │    │
│  │  │  citadel_real.db [AES-256]  |  citadel_decoy.db [AES-256] │  │    │
│  │  └───────────────────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
                              │ Filesystem (Local Disk)
                    ┌─────────▼──────────┐
                    │  citadel_real.db   │  <- AES-256 encrypted
                    │  citadel_decoy.db  │  <- AES-256 encrypted
                    └────────────────────┘
```

### Design Principles (Non-Negotiable)

|               Principle             |                                          Description                                                   |
|-------------------------------------|--------------------------------------------------------------------------------------------------------|
| **P1 - Privacy by Default**         | Every byte is encrypted from first write. No plaintext mode exists.                                    |
| **P2 - Zero External Dependencies** | The final binary is fully autonomous. No runtime, driver, or library required on the host.             |
| **P3 - Plausible Deniability**      | Duress Password exposes a structurally identical fake dataset.                                         |
| **P4 - Minimal Forensic Footprint** | Keys exist only in RAM during the session and are zeroed deterministically on exit. No key ever touches disk. |
| **P5 - Atomic Wipe Capability**     | The operator can irrevocably destroy all real data in seconds with a single command. |

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Data Model

<details>
<summary><kbd>▶ Schema (click to expand)</kbd></summary>

### items - Asset Registry

```sql
CREATE TABLE items (
    item_id         TEXT PRIMARY KEY,   -- UUID v4, locally generated
    sku             TEXT NOT NULL,
    name            TEXT NOT NULL,
    category        TEXT,               -- e.g. 'MED-SUPPLY', 'COMMS'
    unit            TEXT NOT NULL,      -- 'un', 'kg', 'L', 'cx'
    batch_id        TEXT,
    weight_kg       REAL,
    notes_encrypted BLOB,               -- sub-encrypted with AES-256-GCM
    created_at      INTEGER NOT NULL,   -- Unix timestamp UTC
    is_active       INTEGER DEFAULT 1   -- soft-delete flag
);
```

### nodes - Distribution Nodes

```sql
CREATE TABLE nodes (
    node_id     TEXT PRIMARY KEY,
    node_code   TEXT UNIQUE,            -- e.g. 'POST-7A', 'VEHICLE-03'
    description TEXT,
    geo_hint    TEXT,                   -- OPTIONAL: code name only, never GPS
    is_virtual  INTEGER DEFAULT 0,      -- 1 = virtual node (e.g. 'In Transit')
    created_at  INTEGER NOT NULL,
    is_active   INTEGER DEFAULT 1
);
```

### node_balances - Materialized Balance

```sql
CREATE TABLE node_balances (
    node_id         TEXT NOT NULL REFERENCES nodes(node_id),
    item_id         TEXT NOT NULL REFERENCES items(item_id),
    quantity        REAL NOT NULL DEFAULT 0,
    last_updated_at INTEGER NOT NULL,
    PRIMARY KEY (node_id, item_id),
    CHECK (quantity >= 0)               -- balance NEVER goes negative
);
-- Updated automatically by SQLite triggers on every transaction INSERT
```

### transactions - Immutable Append-Only Ledger

```sql
CREATE TABLE transactions (
    tx_id           TEXT PRIMARY KEY,
    tx_type         TEXT NOT NULL CHECK (tx_type IN ('ENTRY','EXIT','TRANSFER','ADJUSTMENT')),
    item_id         TEXT NOT NULL REFERENCES items(item_id),
    from_node_id    TEXT REFERENCES nodes(node_id),   -- NULL for ENTRY
    to_node_id      TEXT REFERENCES nodes(node_id),   -- NULL for EXIT
    quantity        REAL NOT NULL CHECK (quantity > 0),
    reference_code  TEXT,
    operator_hash   TEXT,               -- BLAKE3 of operator ID (non-reversible)
    timestamp_utc   INTEGER NOT NULL,
    notes_encrypted BLOB,
    hash_chain      TEXT NOT NULL       -- BLAKE3(prev_hash || tx_canonical_bytes)
);
-- UPDATE and DELETE blocked by triggers - append-only enforced at DB level
```

### Transaction Types

| Type       | from_node | to_node  | Typical Trigger                      |
|------------|-----------|----------|--------------------------------------|
| ENTRY      | NULL      | Dest     | External supply received at a node   |
| EXIT       | Origin    | NULL     | Final delivery to beneficiary        |
| TRANSFER   | Origin    | Dest     | Redistribution between nodes         |
| ADJUSTMENT | Affected  | NULL     | Correction after physical inventory  |

</details>

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Cryptographic Design

### Key Hierarchy

```
LEVEL 1: Master Password (operator input)
    |
    v  Argon2id (memcost=64MB, timecost=4, parallelism=2, salt=16B CSPRNG)
LEVEL 2: 64-byte key material  ->  NEVER leaves RAM
    |                    |
    v                    v
  first 32 bytes      last 32 bytes
LEVEL 3a: DB Key (REAL)  LEVEL 3b: DB Key (DECOY)
    |  SQLCipher PRAGMA key = hex(db_key_real)
    |  SQLCipher PRAGMA key = hex(db_key_decoy)
    v
  AES-256-GCM at SQLite page level

NOTE: A single Argon2id call produces both keys. Both databases are
      attempted in parallel - the KDF dominates execution time equally
      for all three outcomes (Real, Decoy, wrong password).

DESTRUCTION: on logout, panic, or dead-man's switch trigger
    -> zeroize(key_material_64_bytes)
    -> zeroize(db_key_real)
    -> zeroize(db_key_decoy)
    -> close database connections
    -> process::exit(0)
```

### Argon2id Parameters

| Parameter    | PoC Value | Production Value | Notes                          |
|--------------|-----------|------------------|--------------------------------|
| Memory Cost  | 64 MB     | 256 MB           | Scale with available RAM       |
| Time Cost    | 4         | 8                | Target: login latency < 3s     |
| Parallelism  | 2         | 4                | Half of available cores        |
| Salt         | 16 bytes  | 16 bytes         | OsRng, stored in DB header     |
| Output       | 32 bytes  | 32 bytes         | AES-256 key                    |

### SQLCipher Configuration

```sql
-- Executed immediately after opening the connection, before any query
PRAGMA key = "x'<256bit_hex_derived_key>'";
PRAGMA cipher_page_size = 4096;
PRAGMA kdf_iter = 256000;
PRAGMA cipher_hmac_algorithm = HMAC_SHA512;
PRAGMA cipher_kdf_algorithm = PBKDF2_HMAC_SHA512;
PRAGMA cipher_memory_security = ON;
PRAGMA foreign_keys = ON;
PRAGMA journal_mode = DELETE;   -- WAL PROHIBITED: creates -wal/-shm forensic artifacts
PRAGMA secure_delete = ON;      -- Deleted pages overwritten with zeros
```

### BLAKE3 Hash Chain (Tamper-Evident Ledger)

```rust
fn compute_hash_chain(prev_hash: &str, tx_data: &TransactionData) -> String {
    let mut hasher = blake3::Hasher::new();
    hasher.update(prev_hash.as_bytes());
    hasher.update(&tx_data.serialize_canonical()); // deterministic serialization
    hasher.finalize().to_hex().to_string()
}
// Genesis hash: 64 zeros
// Each record chains the hash of all prior records - retroactive tampering is detectable
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Security Mechanisms

### Dual-Database Authentication (Plausible Deniability)

```
PASSWORD ENTERED
    |
    v  Argon2id KDF (~2-3s) -> 64-byte key material
    |                              |
    |              +---------------+---------------+
    |              |                               |
    |              v                               v
    |   first 32B: try open REAL.db    last 32B: try open DECOY.db
    |              |                               |
    |              +----------- tokio::join! ------+
    |                    (both attempts run in parallel)
    |
    v  timing barrier: result withheld until >= 2500ms from start
    |  (prevents side-channel leak regardless of which branch resolved first)
    |
    +-- REAL opened   -> Session Mode: Real  (frontend receives: success)
    +-- DECOY opened  -> Session Mode: Decoy (frontend receives: success)
    +-- both failed   -> generic error + backoff (frontend receives: error)

CRITICAL: The frontend never receives 'real' or 'decoy' - only success/error.
          Timing is indistinguishable across all three outcomes.
          Sequential open (real first, then decoy) is a prohibited pattern:
          it leaks via file-open latency delta even without measuring KDF time.
```

### Dead Man's Switch

```
Default timeout: 180 seconds (configurable: 30s – 600s)

  t=0s     Operator logs in. DMS timer starts.
  t=120s   Visual warning banner appears.
  t=150s   Red pulsing banner (critical warning).
  t=180s   Secure Shutdown:
             -> zeroize all keys in RAM
             -> close SQLite connections
             -> process::exit(0)
           (No disk wipe - keys are gone, DB remains encrypted on disk)

Every keyboard/mouse event resets the timer to 0.
```

### Panic Button - 3-Pass Wipe Protocol

```
TRIGGER: Ctrl+Shift+Alt+W | UI red button | login wipe code | kill -SIGUSR1

PHASE 1 - RAM ZEROIZATION          (< 1ms)
  1.1  zeroize(master_key)          32 bytes
  1.2  zeroize(db_key_real)         32 bytes
  1.3  zeroize(db_key_decoy)        32 bytes
  1.4  zeroize(password buffers)
  1.5  Close SQLite connections

PHASE 2 - DISK OVERWRITE           (time proportional to file size)
  2.1  Open citadel_real.db with O_DIRECT|O_SYNC (Linux) — bypasses
       OS page cache, writes go directly to storage controller
  2.2  Pass 1: overwrite with 0x00  (aligned 4096-byte blocks)
  2.3  fsync() — barrier before next pass
  2.4  Pass 2: overwrite with 0xFF
  2.5  fsync()
  2.6  Pass 3: overwrite with CSPRNG random bytes
  2.7  fsync() — final physical flush confirmation
  2.8  unlink(citadel_real.db)

  NOTE: PHASE 1 (RAM zeroization) is the PRIMARY defense. If the master
        key is gone, ciphertext on disk is unrecoverable regardless of
        whether overwrite reached physical blocks. PHASE 2 is defense-
        in-depth, most effective on HDDs. SSDs with wear-leveling may
        retain ciphertext in spare blocks until GC; physical destruction
        is required for ultra-high-risk scenarios.

PHASE 3 - AUXILIARY FILES
  3.1  citadel_real.db-journal -> same 3-pass + delete
  3.2  Clear temporary export directory

PHASE 4 - TERMINATION
  4.1  Black screen for 500ms (visual confirmation for operator)
  4.2  std::process::exit(0)

RESULT: Device contains only citadel_decoy.db (encrypted, fake data)
        and the binary. Re-running the binary shows the login screen,
        which authenticates only against the decoy database.
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Threat Model

|  ID  |             Vector             |            Adversary           |              Defense               |
|------|--------------------------------|--------------------------------|------------------------------------|
| T-01 | Physical seizure (app closed)  | Hostile state, armed faction   | SQLCipher AES-256 at rest          |
| T-02 | Seizure with active session    | Same                           | Dead Man's Switch + Secure Shutdown|
| T-03 | Direct operator coercion       | Same + criminal actors         | Duress Password + identical timing |
| T-04 | Post-wipe forensic analysis    | Government forensics lab       | 3-pass CSPRNG overwrite            |
| T-05 | Cold boot RAM dump             | Physical hardware access       | Deterministic zeroization          |
| T-06 | Network interception           | MITM adversary                 | Air-gapped, zero network I/O       |
| T-07 | Binary metadata analysis       | OS-level access                | Static binary, stripped symbols    |
| T-08 | Hardware/software keylogger    | Prior device compromise        | Out of scope for PoC               |

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Building

### Cross-Platform Matrix

```
╔════════════════════════════╦══════════════════════════════════════╗
║  TARGET                    ║  COMMAND                             ║
╠════════════════════════════╬══════════════════════════════════════╣
║  Linux x86_64 (static)     ║  cargo tauri build                   ║
║                            ║    --target x86_64-unknown-linux-musl║
║  Windows x86_64            ║  cargo tauri build                   ║
║                            ║    --target x86_64-pc-windows-msvc   ║
║  macOS Apple Silicon       ║  cargo tauri build                   ║
║                            ║    --target aarch64-apple-darwin     ║
║  macOS Intel               ║  cargo tauri build                   ║
║                            ║    --target x86_64-apple-darwin      ║
╚════════════════════════════╩══════════════════════════════════════╝
```

### Binary Hardening (Cargo.toml)

```toml
[profile.release]
opt-level      = 3
strip          = "symbols"    # remove debug symbols from final binary
lto            = true         # link-time optimization
codegen-units  = 1            # single codegen unit for maximum optimization
panic          = "abort"      # panics terminate immediately, no stack unwinding
```

### Verification Checklist

```bash
# Linux: confirm fully static
ldd target/x86_64-unknown-linux-musl/release/citadel
# Expected: not a dynamic executable

# Database opacity
file citadel_real.db
# Expected: data
# Not acceptable: SQLite 3.x database

# No sensitive strings in binary
strings target/release/citadel | grep -iE 'key|pass|secret|master|argon|aes'

# No network syscalls during runtime
strace -e trace=network ./citadel 2>&1 | grep -v '^---'

# Dependency audit
cargo audit
# Expected: 0 vulnerabilities known
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Testing & Acceptance Criteria

### Functional Criteria

|   ID   |                             Criterion                              |             Method             | Priority  |
|--------|--------------------------------------------------------------------|--------------------------------|-----------|
| AC-F01 | Login with real password opens real.db with real data              | Manual functional  - CRITICAL  | CRITICAL  |
| AC-F02 | Duress password opens decoy.db - UI visually identical             | Functional+visual  - CRITICAL  | CRITICAL  |
| AC-F03 | Wrong password returns generic error after backoff                 | Manual functional  - CRITICAL  | CRITICAL  |
| AC-F04 | ENTRY, EXIT, TRANSFER correctly update node_balances               | Unit + Integration - CRITICAL  | CRITICAL  |
| AC-F05 | Hash chain detects retroactive tampering of any record             | Injection test     - HIGH      | HIGH      |
| AC-F06 | Dead Man's Switch terminates session after configured timeout      | Timed test         - CRITICAL  | CRITICAL  |
| AC-F07 | Panic Button destroys real.db in < 30s for a 100MB file            | Timed + forensic   - CRITICAL  | CRITICAL  |
| AC-F08 | Binary runs without installation on Win10+, Ubuntu 22.04+,macOS 13+ | Clean VM tests     - HIGH      | HIGH      |
| AC-F09 | No cryptographic key found in RAM dump after logout/wipe           | Volatility 3       - CRITICAL  | CRITICAL  |
| AC-F10 | real.db remains intact after 100 consecutive decoy login cycles    | Automated stress   - HIGH      | HIGH      |
---

### Security Criteria

|   ID   | Requirement                                                      | Verification Tool          |
|--------|------------------------------------------------------------------|----------------------------| 
| AC-S01 | Binary compiled with: relro=full, opt-level=3, strip=symbols     | `readelf -l` / `objdump`   |
| AC-S02 | Zero key or password strings identifiable in the binary          | `strings citadel \| grep`  |
| AC-S03 | .db file returns `data` from file(1), not `SQLite 3.x database`  | `file citadel_real.db`     |
| AC-S04 | No network communication during normal execution                 | `strace -e network`        |
| AC-S05 | `cargo audit` returns 0 known vulnerabilities                    | `cargo audit`              |
| AC-S06 | No temporary files created outside the binary directory          | `inotifywait -r /tmp`      |
| AC-S07 | Wipe produces high-entropy output verifiable before unlink       | `xxd` + entropy analysis   |

### Running Tests

```bash
# Full test suite
cargo test

# Functional acceptance criteria
cargo test ac_f

# Security acceptance criteria (manual - see security checklist above)
cargo audit
strings target/release/citadel | grep -iE 'key|pass|secret'

# Stress test: 100 decoy cycles (AC-F10)
cargo test ac_f10 -- --nocapture

# Wipe timing benchmark for 100MB (AC-F07)
cargo test ac_f07 -- --nocapture
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Roadmap

```
╔════════════════════════╦════════════╦══════════════════════════════════════════╗
║  PHASE                 ║  DURATION  ║  DELIVERABLES                            ║
╠════════════════════════╬════════════╬══════════════════════════════════════════╣
║  PHASE 0 - Foundation  ║  2 weeks   ║  Tauri 2.x boilerplate + SQLCipher       ║
║                        ║            ║  binding, basic login screen             ║
╠════════════════════════╬════════════╬══════════════════════════════════════════╣
║  PHASE 1 - Crypto Core ║  3 weeks   ║  AuthManager (Argon2id), dual-DB auth,   ║
║                        ║            ║  SessionKeys (ZeroizeOnDrop), DMS timer  ║
╠════════════════════════╬════════════╬══════════════════════════════════════════╣
║  PHASE 2 - Inventory   ║  3 weeks   ║  Full CRUD, hash chain ledger, SQLite    ║
║                        ║            ║  triggers, balance reconciliation        ║
╠════════════════════════╬════════════╬══════════════════════════════════════════╣
║  PHASE 3 - Security    ║  2 weeks   ║  Dead Man's Switch UI, Panic Button,     ║
║  Mechanisms            ║            ║  3-pass wipe, SIGUSR1 handler            ║
╠════════════════════════╬════════════╬══════════════════════════════════════════╣
║  PHASE 4 - UI + Decoy  ║  2 weeks   ║  Full dashboard, transaction forms,      ║
║                        ║            ║  decoy setup workflow                    ║
╠════════════════════════╬════════════╬══════════════════════════════════════════╣
║  PHASE 5 - Testing     ║  2 weeks   ║  All AC-F and AC-S criteria validated,   ║
║  & Hardening           ║            ║  forensic analysis, binary hardening     ║
╚════════════════════════╩════════════╩══════════════════════════════════════════╝

```

### Out of Scope (Phase 2 - Production)

```
[ ] Peer-to-peer sync between nodes (Bluetooth, WiFi Direct, USB)
[ ] Native mobile interface (iOS/Android via Tauri Mobile)
[ ] HSM/TPM integration for master key protection
[ ] Encrypted backup for disaster recovery
[ ] Biometric multi-factor authentication
[ ] Auto-wipe after N failed login attempts with exponential backoff
```

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> Known Limitations

**SSD Wear Leveling:**
The 3-pass wipe operates at the OS file level. SSDs with wear-leveling
algorithms may retain copies of overwritten data in spare blocks not
addressable by the OS. For ultra-high-risk scenarios, physical destruction
of the storage device is recommended after software wipe. Mechanical hard
drives respond better to software overwrite.

**Cold Boot Attacks (T-05):**
Zeroization reduces but does not fully eliminate RAM remanence risk under
optimal conditions (low temperature, fast RAM extraction). The key exposure
window is minimized by zeroizing immediately on all exit paths (logout, DMS,
panic). Physical protection of the device during active sessions remains
required.

**Keyloggers (T-08):**
A compromised operating system (hardware or software keylogger) is outside
the scope of the PoC. Citadel assumes a clean OS. For hostile environments,
a verified-boot operating system (e.g., Tails, Qubes OS) is recommended.

**Decoy Plausibility:**
The decoy database must be pre-populated with believable data during Setup
Mode before field deployment. An empty or obviously synthetic decoy database
weakens the plausible deniability guarantee.

---

## <img alt="screenshot" src="public/citadelLOGO.png" width="90" height="90"> License

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  PROJECT CITADEL                                                             ║
║  Classification: RESTRICTED - Controlled Distribution                        ║
║                                                                              ║
║  See LICENSE file for terms.                                                 ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## References

```
RFC 9106          - Argon2 Memory-Hard Function (IETF, 2021)
NIST SP 800-38D   - GCM and GMAC Block Cipher Modes
NIST SP 800-88    - Guidelines for Media Sanitization (Rev. 1)
SQLCipher Design  - https://www.zetetic.net/sqlcipher/design/
Tauri v2 Security - https://tauri.app/security/
zeroize crate     - https://docs.rs/zeroize
Volatility 3      - https://www.volatilityfoundation.org/
Halderman et al.  - "Lest We Remember: Cold Boot Attacks on Encryption Keys"
                    USENIX Security 2008
```

---

<div align="center">

```
▓▒░ · PROJECT CITADEL · ░▒▓
```

</div>
