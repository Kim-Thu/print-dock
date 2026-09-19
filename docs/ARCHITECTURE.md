# Architecture

## Mục tiêu kiến trúc

PrintDock là ứng dụng đa nền tảng. Kiến trúc phải tránh việc business logic bị khóa vào Windows API hoặc Linux command.

Tách bốn lớp:

1. UI;
2. orchestration và rule engine;
3. capability contracts;
4. OS-specific adapters.

## Logical architecture

```text
+-----------------------------+
| PrintDock.App               |
| Cross-platform desktop UI   |
+--------------+--------------+
               |
               v
+-----------------------------+
| PrintDock.Application       |
| orchestration / repair plan |
+--------------+--------------+
               |
       +-------+-------+
       |               |
       v               v
+-------------+   +-------------+
| Diagnostics |   | Repairs     |
+------+------+   +------+------+
       |                 |
       +--------+--------+
                v
+-----------------------------+
| PrintDock.Core              |
| domain / contracts / rules  |
| capability abstractions     |
+--------------+--------------+
               |
       +-------+--------+
       |                |
       v                v
+----------------+  +----------------+
|Platform.Windows|  |Platform.Linux  |
|Win32/SMB/RPC   |  |CUPS/IPP/Samba  |
|Spooler/Registry|  |systemd/services|
+----------------+  +----------------+
```

## Suggested solution layout

```text
src/
  PrintDock.App/
  PrintDock.Application/
  PrintDock.Core/
  PrintDock.Diagnostics/
  PrintDock.Repairs/
  PrintDock.Platform/
  PrintDock.Platform.Windows/
  PrintDock.Platform.Linux/

tests/
  PrintDock.Core.Tests/
  PrintDock.Diagnostics.Tests/
  PrintDock.Repairs.Tests/
  PrintDock.Platform.Windows.Tests/
  PrintDock.Platform.Linux.Tests/
```

## Capability model

Không gọi trực tiếp "WindowsSpoolerService" từ Application.

Ví dụ abstraction:

```csharp
public interface IPrintServiceAdapter
{
    Task<PrintServiceStatus> GetStatusAsync(CancellationToken ct);
    Task<RepairResult> RestartAsync(CancellationToken ct);
}

public interface IPrinterInventoryAdapter
{
    Task<IReadOnlyList<PrinterInfo>> GetPrintersAsync(CancellationToken ct);
}

public interface IPrinterConnectionAdapter
{
    Task<ConnectionProbeResult> ProbeAsync(PrinterTarget target, CancellationToken ct);
    Task<RepairResult> ConnectAsync(PrinterTarget target, CancellationToken ct);
}
```

Windows có thể map `IPrintServiceAdapter` sang Print Spooler.

Linux map cùng contract sang CUPS.

## Protocol model

Protocol không đồng nghĩa OS.

PrintDock phải tách:

- SMB
- IPP
- IPPS
- local USB/queue
- network socket/LPR nếu mở rộng sau

Ví dụ Linux client vẫn có thể kết nối printer được share từ Windows qua Samba/SMB.

Windows client cũng có thể kết nối printer hỗ trợ IPP.

## Diagnostic contract

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

Status:
- Pass
- Fail
- Warning
- Skipped
- Unknown
- Unsupported

`Unsupported` rất quan trọng cho đa nền tảng: khác với Fail.

## Repair contract

```text
CanApply(context)
    -> CheckCapability(platform)
        -> Plan(context)
            -> Execute(plan)
                -> Verify(result)
```

Repair plan chứa:
- platform;
- required capability;
- preconditions;
- privilege;
- changes;
- snapshot;
- rollback/recovery;
- verification.

## Windows interaction strategy

Ưu tiên:
1. .NET/Win32 API;
2. WMI/CIM hoặc PowerShell API nếu cần;
3. process invocation cuối cùng.

Không parse localized command output nếu có structured API.

## Linux interaction strategy

Ưu tiên:
1. libcups / IPP client library có cấu trúc;
2. D-Bus/system APIs khi thích hợp;
3. `lpstat`, `lpadmin`, `systemctl` chỉ qua process adapter an toàn khi chưa có API phù hợp.

Không dùng:
```text
bash -c "<user input>"
```

Command phải có executable cố định và argument tách riêng.

## Privilege model

Windows:
- non-admin diagnostics;
- UAC khi repair cần Administrator.

Linux:
- non-root diagnostics;
- polkit/sudo/system permission chỉ khi action cần;
- không yêu cầu chạy cả app bằng root.

## UI framework

UI phải cross-platform từ đầu. Không dùng WPF hoặc WinUI cho shell chính nếu mục tiêu Linux là first-class.

Các hướng phù hợp cần đánh giá trong issue riêng, ví dụ:
- Avalonia UI với .NET;
- hoặc web-based desktop shell nếu có lý do mạnh.

Quyết định UI framework phải dựa trên:
- Windows + Linux support;
- packaging;
- native integration;
- accessibility;
- footprint;
- maintenance.

## Failure isolation

Mỗi probe:
- timeout;
- cancellation;
- map OS/native error;
- không crash diagnostic session.

## Extensibility

Stable IDs nên trung lập OS khi có thể:

- `NET.RESOLVE_HOST`
- `NET.TCP_445`
- `PRINT.SERVICE_STATUS`
- `PRINT.INVENTORY`
- `PRINT.CONNECTION_EXISTS`

OS-specific IDs chỉ khi thật sự khác:

- `WIN.PRINT.RPC_POLICY`
- `LINUX.CUPS.SCHEDULER`
