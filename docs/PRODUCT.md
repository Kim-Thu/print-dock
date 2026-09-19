# Product Analysis

## 1. Product statement

PrintDock là ứng dụng Windows giúp người dùng và kỹ thuật viên xác định vì sao máy in chia sẻ trong LAN không hoạt động, thực hiện repair có kiểm soát và xác nhận hệ thống đã trở lại trạng thái mong muốn.

## 2. Người dùng chính

### Người dùng phổ thông
Biết máy in nào cần dùng nhưng không biết SMB, RPC, Spooler, driver hay registry là gì.

Nhu cầu:
- biết lỗi nằm ở đâu;
- có thể sửa bằng vài thao tác;
- không phải chạy lệnh thủ công;
- được báo rõ PrintDock sắp thay đổi gì.

### Kỹ thuật viên / IT support
Cần:
- bằng chứng diagnostic;
- system error code;
- before/after state;
- log export;
- thao tác repair riêng lẻ;
- tránh phải nhớ nhiều lệnh Windows.

## 3. Bài toán lõi

"Không in được" chỉ là triệu chứng. Chuỗi phụ thuộc thực tế có nhiều tầng:

```text
User action
  -> local printer configuration
  -> driver / port / queue
  -> Print Spooler
  -> Windows RPC / printing subsystem
  -> SMB / printer share
  -> hostname / DNS / IP
  -> LAN / firewall
  -> print server
  -> physical printer
```

Một kết luận hợp lệ phải dựa trên nhiều tín hiệu. Ví dụ:
- ping fail không đủ để kết luận server down;
- TCP/445 open không đủ để kết luận printer connection sẽ thành công;
- share tồn tại không đủ để kết luận client có quyền add;
- Spooler running không đủ để kết luận queue/driver bình thường.

## 4. Jobs to be done

### JTBD-01 - Tôi không in được
PrintDock phải giúp user phân biệt:
- lỗi local;
- lỗi kết nối tới server;
- lỗi share;
- lỗi permission;
- lỗi driver;
- lỗi queue/spooler;
- lỗi policy/RPC.

### JTBD-02 - Tôi không add được máy in share
PrintDock phải xác định bước nào fail:
- resolve server;
- reach service;
- enumerate share;
- connect printer;
- install/use driver;
- create local connection.

### JTBD-03 - Tôi nhận mã lỗi Windows
Mã lỗi như `0x0000011b` hoặc `0x00000709` chỉ là input bổ sung, không phải nguyên nhân mặc định. Engine phải kiểm chứng tình trạng máy trước khi repair.

### JTBD-04 - Tôi cần gửi log cho người hỗ trợ
Report phải đủ để người khác biết:
- môi trường;
- diagnostic nào pass/fail;
- system error;
- repair nào đã chạy;
- verify cuối cùng ra sao.

## 5. Phạm vi MVP

### In scope
- Windows client.
- Windows machine acting as print server trong LAN.
- Shared printer qua Windows printer sharing.
- Discovery/validation thủ công bằng hostname/IP/share name.
- Read-only diagnostics.
- Limited safe repairs.
- Structured logs và export report.

### Out of scope
- Internet printing.
- Enterprise print server management ở quy mô domain lớn.
- Printer fleet monitoring.
- Cloud telemetry.
- Remote admin tự động.
- Driver download không kiểm soát.
- Registry "tweak pack".
- Bypass security controls.

## 6. Functional requirements

### FR-01 Environment discovery
Thu thập:
- Windows edition/version/build;
- x64/ARM64 nếu có;
- hostname;
- current user;
- elevation state;
- network interfaces liên quan.

### FR-02 Target input
Cho phép nhập:
- hostname hoặc IP server;
- printer share name;
- local printer display name tùy chọn.

Input phải validate trước khi dùng.

### FR-03 Diagnostic execution
Mỗi probe trả về một object chuẩn:
- id;
- status;
- summary;
- evidence;
- error code;
- duration;
- remediation hints;
- privilege requirement.

### FR-04 Root-cause classification
Không chỉ liệt kê pass/fail. Engine phải gom evidence thành hypothesis:
- target unreachable;
- SMB unavailable;
- spooler unhealthy;
- share missing;
- connection denied;
- existing stale connection;
- driver problem;
- unknown/insufficient evidence.

Hypothesis phải có confidence/evidence, không khẳng định quá mức.

### FR-05 Repair plan
Trước khi sửa phải hiển thị:
- action;
- reason;
- privilege;
- expected impact;
- rollback nếu có.

### FR-06 Verification
Mỗi repair phải có verify tương ứng.

### FR-07 Logging
Log gồm:
- timestamp;
- correlation/session id;
- diagnostic/repair id;
- status;
- system code;
- safe metadata;
- elapsed time.

## 7. Non-functional requirements

### Safety
Không có repair mù.

### Security
Không lưu credential. Không shell concatenate input.

### Reliability
Một diagnostic fail không được làm toàn bộ session crash.

### Explainability
User phải hiểu vì sao tool đề xuất repair.

### Maintainability
Diagnostic và repair phải là module độc lập, test được.

### Performance
Các probe độc lập có thể chạy song song sau khi xác định không gây side effect.

### Compatibility
Ban đầu ưu tiên Windows 10/11 còn được hỗ trợ; hỗ trợ Windows Server phải được xác nhận bằng test matrix, không mặc định.

## 8. Success criteria cho MVP

MVP được xem là hữu dụng khi:
1. chạy diagnostic mà không yêu cầu admin nếu chưa cần;
2. xác định được tầng fail thay vì chỉ in raw command output;
3. restart spooler/remove/reconnect printer có verify;
4. mọi repair được ghi log;
5. input độc hại/không hợp lệ không thể biến thành shell command;
6. diagnostic engine có unit test và adapter Windows có integration test cơ bản.

## 9. UX principle

UI không bắt người dùng phải biết trước lỗi là 0x0000011b hay 0x00000709.

Luồng mặc định:
```text
Chọn/nhập server + printer
        |
        v
      Kiểm tra
        |
        v
Kết quả theo từng tầng
        |
        v
Repair plan phù hợp
        |
        v
User xác nhận
        |
        v
Repair -> Verify
```

Advanced mode mới hiển thị raw technical detail.
