# Roadmap

## Phase 0 - Foundation

Mục tiêu: repository có kiến trúc, contracts, logging và test foundation trước khi đụng repair thực tế.

- solution/project structure;
- domain result types;
- diagnostic runner;
- structured logger;
- validation;
- Windows adapter boundaries;
- CI build/test.

## Phase 1 - Read-only MVP diagnostics

- environment discovery;
- target validation;
- hostname/IP resolution;
- TCP/445 probe;
- local Spooler status;
- local printer inventory;
- remote printer share discovery;
- existing connection detection;
- structured diagnostic session;
- basic hypothesis engine;
- simple UI.

Exit criteria:
- diagnostic không cần admin ở luồng bình thường;
- không có mutation;
- report giải thích được failure layer.

## Phase 2 - Safe repairs

- restart Spooler;
- remove stale printer connection;
- connect shared printer;
- verify after each action;
- elevation flow;
- user confirmation;
- result states SUCCESS/FAILED/PARTIAL/SKIPPED.

## Phase 3 - Error-specific diagnostics

- `0x0000011b`;
- `0x00000709`;
- RPC/policy-related restrictions;
- driver mismatch;
- stuck queue;
- credential/permission scenarios.

Không implement bằng fixed "registry tweak". Mỗi case cần evidence + compatibility matrix.

## Phase 4 - Technician mode

- advanced details;
- raw system codes;
- export support bundle;
- selective probe execution;
- manual repair actions;
- before/after comparison.

## Phase 5 - Hardening & release

- Windows compatibility matrix;
- integration test lab;
- packaging;
- portable build if feasible;
- signing/checksum;
- privacy/security review;
- documentation;
- release process.

## Future candidates

Chỉ xem xét sau MVP:
- print server-side diagnostics;
- driver package management;
- firewall-specific checks;
- fleet mode;
- enterprise policy inspection;
- remote support bundle;
- auto-update.
