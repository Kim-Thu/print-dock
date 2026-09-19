# Security & Safety

PrintDock thao tác với printing subsystem của Windows nên security và reversibility là yêu cầu thiết kế, không phải phần bổ sung sau.

## Threat model ban đầu

### 1. Command injection
Người dùng có thể nhập hostname/share name chứa ký tự đặc biệt.

Không được:
```text
cmd.exe /c "some-command " + userInput
```

Phải:
- validate input;
- dùng typed API;
- hoặc truyền argument riêng;
- không cho input quyết định executable.

### 2. Privilege abuse
Không chạy mọi diagnostic dưới Administrator.

Elevation phải:
- có lý do;
- chỉ dùng cho action cần thiết;
- hiển thị action trước khi UAC.

### 3. Destructive repair
Các hành động như remove printer, clear queue, registry mutation có thể làm mất trạng thái.

Yêu cầu:
- explicit scope;
- precondition;
- snapshot khi có thể;
- verify;
- rollback hoặc recovery instruction.

### 4. Credential exposure
Không:
- lưu password;
- ghi password/token vào log;
- export credential;
- tự động gửi dữ liệu ra ngoài.

### 5. Malicious/unsafe driver
MVP không tự download/install driver từ nguồn Internet.

Driver management sau này phải xác định trust/source/signature policy riêng.

## Repair safety levels

### Level 0 - Read only
Không thay đổi máy.

### Level 1 - Reversible low risk
Ví dụ restart Spooler.

### Level 2 - State mutation
Ví dụ remove/reconnect printer.

### Level 3 - System configuration
Registry/policy/firewall/driver.

MVP chủ yếu Level 0-2.

Level 3 bắt buộc:
- issue riêng;
- threat analysis;
- rollback;
- version compatibility evidence;
- integration test.

## Logging policy

Được log:
- OS/build;
- generic hostname khi user cho phép export;
- printer/share name;
- error code;
- action/status/duration.

Không log:
- password;
- auth token;
- credential material.

Export report nên có redaction mode.

## Network boundary

PrintDock không được biến thành remote execution tool ngoài scope.

MVP:
- đọc/truy cập resource cần thiết để chẩn đoán printer share;
- không chạy command trên remote server.

## Release trust

Trước public release cần:
- reproducible documented build ở mức hợp lý;
- checksum;
- code signing khi có điều kiện;
- release notes;
- source tag tương ứng binary;
- không nhúng updater/telemetry không được mô tả.
