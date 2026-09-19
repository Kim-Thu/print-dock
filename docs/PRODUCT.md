# Product Analysis

## 1. Product statement

PrintDock là ứng dụng desktop đa nền tảng giúp người dùng và IT support xác định vì sao máy in local/network/shared không hoạt động trên Windows hoặc Linux, thực hiện repair có kiểm soát và verify kết quả.

## 2. Bối cảnh doanh nghiệp

Do chi phí bản quyền, chuẩn hóa hạ tầng hoặc nhu cầu vận hành, một doanh nghiệp có thể có:

- Windows desktop + Windows print host;
- Linux desktop + CUPS server;
- Linux desktop + Windows printer share qua Samba;
- Windows desktop + printer IPP/network;
- môi trường hỗn hợp Windows/Linux.

Vì vậy sản phẩm không được coi "máy in mạng = Windows SMB printer share".

## 3. Người dùng chính

### Người dùng phổ thông
Chỉ biết "không in được", không cần biết Spooler/CUPS/SMB/IPP là gì.

### Kỹ thuật viên
Cần evidence, native error, before/after, export log và repair chọn lọc theo OS.

## 4. Bài toán lõi

```text
User action
  -> local printer/queue
  -> print service
       Windows: Print Spooler
       Linux: CUPS
  -> protocol
       SMB / IPP / IPPS / others
  -> hostname / IP / network
  -> remote queue/share
  -> permissions/auth
  -> driver or driverless capability
  -> physical printer
```

## 5. Jobs to be done

### JTBD-01 - Tôi không in được
Tool xác định tầng lỗi bất kể Windows/Linux.

### JTBD-02 - Tôi không add được printer
Tool xác định protocol và bước fail: discovery, addressability, authentication, driver/queue creation.

### JTBD-03 - Tôi dùng mixed environment
Tool phải hỗ trợ Linux client tới Windows share và ngược lại khi protocol phù hợp.

### JTBD-04 - Tôi cần gửi support bundle
Report phải ghi platform, print stack, protocol, probes, repairs và verify.

## 6. Phạm vi MVP

### Cross-platform in scope
- Windows 10/11.
- Linux desktop/server phổ biến có CUPS.
- Hostname/IP.
- SMB diagnostics.
- IPP/IPPS diagnostics.
- Printer/queue inventory.
- Print-service status.
- Structured logs/report.
- Safe repairs theo capability.

### Windows-specific
- Print Spooler.
- Windows SMB printer shares.
- Windows connection state.
- Win32/RPC-specific error research.

### Linux-specific
- CUPS scheduler/service.
- CUPS queues.
- IPP/IPPS.
- Samba/SMB printer connections khi dùng Windows share.
- queue enable/disable/restart/reconnect ở mức an toàn.

### Out of scope ban đầu
- fleet management;
- cloud telemetry;
- remote arbitrary command;
- auto-download driver;
- bypass security;
- every Linux distro/package manager;
- printer vendor proprietary management.

## 7. Functional requirements

### FR-01 Platform discovery
Thu thập OS/platform, version, architecture, privilege/elevation và available capabilities.

### FR-02 Target model
Không chỉ "server + share". Target phải biểu diễn được:
- SMB share;
- IPP/IPPS URI;
- local queue;
- raw network printer mở rộng sau.

### FR-03 Capability discovery
App biết adapter hiện tại hỗ trợ gì. Unsupported phải khác Failed.

### FR-04 Diagnostic execution
Mỗi probe trả structured result với status/evidence/error/duration/platform/capability.

### FR-05 Root-cause classification
Rule engine dùng evidence trung lập OS khi có thể; rule OS-specific tách riêng.

### FR-06 Repair plan
Hiển thị action, platform, required privilege, impact, rollback/recovery và verify.

### FR-07 Logging
Ghi session/platform/protocol/probe/action/native error an toàn.

## 8. Non-functional requirements

### Portability
Core, Application, rules, contracts và phần lớn tests phải chạy được Windows/Linux.

### Safety
Không repair mù.

### Security
Không lưu credential; không shell concatenate input; không chạy app full-admin/root mặc định.

### Reliability
Một adapter/probe fail không crash toàn app.

### Explainability
Người dùng thấy nguyên nhân theo ngôn ngữ đời thường; kỹ thuật viên xem raw evidence.

### Compatibility
Hỗ trợ distro phải được định nghĩa theo print stack/capability, không quảng cáo "mọi Linux".

## 9. Success criteria MVP

1. cùng một Core chạy trên Windows và Linux;
2. detect đúng platform/capability;
3. Windows diagnostic dùng Spooler;
4. Linux diagnostic dùng CUPS;
5. cả hai dùng chung network/hypothesis/logging model;
6. Linux có thể kiểm tra SMB share của Windows khi Samba client capability tồn tại;
7. unsupported operation không bị báo failed;
8. repair có verify và không yêu cầu quyền cao hơn cần thiết.
