# Architecture

## Mục tiêu kiến trúc

PrintDock cần tách ba việc thường bị trộn lẫn trong các tiện ích Windows nhỏ:

1. UI;
2. quyết định diagnostic/repair;
3. tương tác trực tiếp với Windows.

Nếu UI trực tiếp gọi `cmd.exe`, sửa registry và restart service thì hệ thống khó test, khó audit và khó rollback.

## Logical architecture

```text
+---------------------------+
| PrintDock.App             |
| WPF/WinUI UI + ViewModels |
+-------------+-------------+
              |
              v
+---------------------------+
| PrintDock.Application     |
| use cases / orchestration |
| repair planning           |
+-----+---------------+-----+
      |               |
      v               v
+-----------+   +-----------+
|Diagnostics|   | Repairs   |
+-----+-----+   +-----+-----+
      |               |
      +-------+-------+
              v
+---------------------------+
| PrintDock.Windows         |
| OS adapters               |
| services / printers /     |
| network / registry / APIs |
+-------------+-------------+
              |
              v
+---------------------------+
| Windows                   |
+---------------------------+

Cross-cutting:
PrintDock.Core
- result contracts
- domain models
- rule engine
- logging abstractions
- validation
```

## Suggested solution layout

```text
src/
  PrintDock.App/
  PrintDock.Application/
  PrintDock.Core/
  PrintDock.Diagnostics/
  PrintDock.Repairs/
  PrintDock.Windows/

tests/
  PrintDock.Core.Tests/
  PrintDock.Diagnostics.Tests/
  PrintDock.Repairs.Tests/
  PrintDock.Windows.IntegrationTests/
```

## Diagnostic contract

Ví dụ khái niệm:

```csharp
public record DiagnosticResult(
    string Id,
    DiagnosticStatus Status,
    string Summary,
    IReadOnlyList<Evidence> Evidence,
    SystemError? Error,
    TimeSpan Duration
);
```

Không trả về duy nhất string log.

Status tối thiểu:
- Pass
- Fail
- Warning
- Skipped
- Unknown

## Repair contract

Một repair phải tách ba phase:

```text
CanApply(context)
    -> Plan(context)
        -> Execute(plan)
            -> Verify(result)
```

Plan lưu:
- preconditions;
- changes;
- privilege;
- backup/snapshot;
- rollback strategy;
- verification steps.

## Rule engine

Diagnostic không nên hard-code toàn bộ quyết định trong UI.

Ví dụ:

```text
serverResolved = PASS
tcp445         = PASS
shareExists    = PASS
connectPrinter = ACCESS_DENIED

=> hypothesis: PrinterConnectionPermission
=> do NOT repair DNS/SMB
=> collect RPC/policy/credential evidence
```

Rule engine có thể bắt đầu bằng code thuần C# và immutable rules; chưa cần DSL.

## Windows interaction strategy

Ưu tiên theo thứ tự:

1. .NET / Windows API có kiểu dữ liệu rõ ràng.
2. WMI/CIM hoặc PowerShell API khi Windows API quá phức tạp.
3. Process invocation chỉ khi cần.
4. Tránh parse localized CLI text nếu có structured API thay thế.

Nếu phải gọi process:
- executable cố định;
- arguments tách riêng;
- timeout;
- capture stdout/stderr;
- không `cmd /c "<user input>"`.

## Privilege model

Read-only diagnostics chạy non-admin khi có thể.

Elevation chỉ xảy ra khi:
- restart service;
- sửa cấu hình protected;
- thao tác khác thực sự cần quyền.

Không chạy toàn bộ app elevated mặc định nếu không cần.

## Concurrency

Có thể parallel:
- OS info;
- hostname resolution;
- local printer inventory;
- local spooler status.

Phải sequence:
- repair -> verify;
- remove connection -> reconnect;
- snapshot -> mutation.

## Failure isolation

Mỗi probe phải:
- timeout;
- catch OS-specific exception;
- map error;
- không crash whole diagnostic session.

## Extensibility

Mỗi diagnostic/repair có ID ổn định, ví dụ:
- `NET.RESOLVE_HOST`
- `NET.TCP_445`
- `PRINT.LOCAL_SPOOLER`
- `PRINT.SHARE_EXISTS`
- `PRINT.CONNECTION_EXISTS`
- `REPAIR.SPOOLER_RESTART`

ID dùng trong log, test và rule engine.
