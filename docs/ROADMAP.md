# Roadmap

## Phase 0 - Cross-platform foundation

- .NET cross-platform core;
- capability abstractions;
- diagnostic/repair result contracts;
- structured logging;
- validation;
- platform detection;
- CI matrix Windows + Linux;
- chọn cross-platform UI framework.

Exit:
- Core/Application build và test trên Windows + Linux.
- Không có WPF/WinUI dependency trong Core/Application.

## Phase 1 - Shared read-only diagnostics

- hostname/IP;
- DNS resolution;
- reachability hints;
- TCP probes;
- SMB TCP/445;
- IPP/IPPS endpoint probes;
- target/protocol model;
- diagnostic session;
- hypothesis engine;
- report export.

## Phase 2A - Windows adapter

- environment;
- Print Spooler;
- local printer inventory;
- Windows printer connections;
- SMB printer share discovery;
- driver/queue evidence.

## Phase 2B - Linux adapter

- distro/runtime info;
- CUPS availability/scheduler;
- CUPS queues;
- IPP/IPPS printer discovery/probe;
- Samba/SMB capability;
- Linux queue/driverless evidence.

## Phase 3 - Cross-platform desktop UI

- simple diagnostics;
- technical view;
- capability-aware controls;
- unsupported state;
- export.

## Phase 4A - Windows safe repairs

- Spooler restart;
- remove/reconnect exact target;
- verify;
- least-privilege elevation.

## Phase 4B - Linux safe repairs

- restart CUPS when evidence supports;
- enable/recreate exact CUPS queue;
- reconnect IPP/SMB target;
- verify;
- least-privilege via platform mechanism.

## Phase 5 - OS-specific error research

Windows:
- 0x0000011b;
- 0x00000709;
- RPC/policy.

Linux:
- CUPS scheduler unavailable;
- stopped/disabled queue;
- IPP auth/TLS;
- Samba auth/share issues;
- driverless capability mismatch.

## Phase 6 - Hardening & release

- CI/test matrix;
- Windows packaging;
- Linux AppImage/deb/rpm strategy evaluation;
- signing/checksums;
- privacy/security review;
- support matrix documentation.

## Future

- print server-side diagnostics;
- enterprise policies;
- vendor-specific extensions;
- fleet mode;
- remote support bundle;
- auto-update.
