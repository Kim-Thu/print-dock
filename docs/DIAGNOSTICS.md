# Diagnostics Model

## Tư duy

PrintDock không coi mã lỗi Windows là root cause. Mỗi kết luận phải được hình thành từ evidence.

## Diagnostic pipeline

```text
Environment
   |
Target validation
   |
Network identity
   |
Transport/services
   |
Print subsystem
   |
Share discovery
   |
Connection state
   |
Driver/queue
   |
Classification
```

## Probe matrix ban đầu

| ID | Probe | Mục đích | Side effect | Admin |
|---|---|---|---|---|
| ENV.OS | OS/build | compatibility context | Không | Không |
| ENV.ELEVATION | Current privilege | biết repair nào khả dụng | Không | Không |
| INPUT.TARGET | Validate target | chống input sai/injection | Không | Không |
| NET.RESOLVE_HOST | Resolve hostname | hostname -> address | Không | Không |
| NET.REACHABILITY | Reachability hints | network evidence | Không | Không |
| NET.TCP_445 | TCP 445 | SMB transport evidence | Không | Không |
| PRINT.LOCAL_SPOOLER | Spooler status | local print subsystem | Không | Thường không |
| PRINT.LOCAL_INVENTORY | Installed printers | local state | Không | Không |
| PRINT.SHARE_ENUM | Enumerate shares | server advertises printer? | Không | Tùy môi trường |
| PRINT.SHARE_RESOLVE | Resolve printer path | client can address share? | Không | Không |
| PRINT.CONNECTION | Existing connection | stale/duplicate state | Không | Không |
| PRINT.DRIVER | Driver state | compatibility/evidence | Không | Không |
| PRINT.QUEUE | Queue state | stuck/error jobs | Không | Không |

## Không suy luận quá mức

### Ping fail
Không kết luận:
> Server offline.

Phải nói:
> ICMP probe không nhận phản hồi. Tiếp tục kiểm tra hostname/TCP vì ICMP có thể bị chặn.

### TCP 445 open
Không kết luận:
> Printer share hoạt động.

Chỉ kết luận:
> Có service nhận TCP/445 tại target.

### Share exists
Không kết luận:
> Client add được printer.

Cần kiểm tra connection attempt và error cụ thể.

## Error normalization

Windows có thể trả:
- Win32 error;
- HRESULT;
- PowerShell exception;
- service error;
- printer-specific status.

Core nên normalize thành:

```text
SystemError
  source
  nativeCode
  hexCode
  symbol
  message
  operation
```

Không chỉ giữ message localized.

## Hypothesis model

Ví dụ categories ban đầu:

- InvalidInput
- NameResolutionFailure
- NetworkPathUnavailable
- SmbUnavailable
- LocalSpoolerStopped
- PrinterShareNotFound
- PrinterConnectionPermissionDenied
- ExistingConnectionConflict
- DriverUnavailableOrInvalid
- QueueProblem
- PolicyOrRpcRestriction
- Unknown

Mỗi hypothesis cần:
- supporting evidence;
- contradicting evidence;
- confidence;
- next probe.

## Session result

Một session nên có:

```text
DiagnosticSession
  sessionId
  startedAt
  environment
  target
  results[]
  hypotheses[]
  recommendedActions[]
  rawEvents[]
```

## Verification

Không dùng cùng một tín hiệu để vừa repair vừa "chứng minh" repair thành công nếu có kiểm tra mạnh hơn.

Ví dụ restart spooler:
1. capture status trước;
2. restart;
3. query service status;
4. optional printer API sanity check.

Reconnect printer:
1. existing connection state;
2. remove nếu cần;
3. connect;
4. query installed connection;
5. optional test print do user chủ động.
