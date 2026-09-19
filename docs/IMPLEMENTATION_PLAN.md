# Implementation Plan

Tài liệu này nối backlog với thứ tự xây dựng. Mục tiêu là tránh bắt đầu từ UI hoặc các registry workaround trước khi có diagnostic core.

## Dependency graph

```text
#2 Foundation
   |
   +--> #3 Core contracts
   |      |
   |      +--> #4 Input validation
   |      +--> #5 Environment
   |      +--> #6 Network
   |      +--> #7 Local printing
   |      +--> #8 Remote share
   |      +--> #10 Logging
   |
   +--> #15 Test strategy

#4 + #5 + #6 + #7 + #8
             |
             v
            #9 Hypothesis engine
             |
             +--> #11 UI
             |
             +--> #12 Repair planning/elevation
                        |
                        +--> #13 Spooler repair
                        +--> #14 Reconnect repair

#16 Error research
   depends on evidence model/probes above
   and should create later implementation issues,
   not bypass them.
```

## Phase A - Foundation

1. #2 solution/projects/CI
2. #3 domain contracts
3. #15 QA strategy and fixtures baseline
4. #10 structured logging

Exit:
- build/test green;
- core models stable enough for probes;
- no Windows mutation yet.

## Phase B - Read-only diagnostics

1. #4 input validation
2. #5 environment
3. #6 network
4. #7 local print subsystem
5. #8 remote share
6. #9 hypothesis engine

Exit:
- a real Windows session can explain which layer is failing;
- a failed probe does not crash the session;
- no repair is required to demonstrate value.

## Phase C - Desktop UX

#11 can begin with fake/fixture sessions once #3 is stable, but production wiring waits until read-only pipeline exists.

Exit:
- simple mode;
- technical details;
- cancel;
- report/export;
- no direct Windows calls from UI.

## Phase D - Safe repair infrastructure

1. #12 plan/confirm/elevation
2. #13 Spooler restart
3. #14 shared-printer reconnect

Exit:
- every mutation has precondition + confirmation + verify;
- healthy resources are not modified without reason.

## Phase E - Error-specific expansion

#16 researches 0x0000011b and 0x00000709 using the evidence pipeline.

Only after research:
- create separate implementation issue per supported remediation;
- document affected Windows versions/builds;
- add security impact and rollback tests.

## Pull request rule

Một PR implementation nên:
- link issue;
- state diagnostic/repair IDs added;
- describe side effects;
- include tests;
- update docs if behavior changed;
- never silently expand privilege or registry scope.
