# PrintDock

**PrintDock** là ứng dụng desktop mã nguồn mở dùng để **chẩn đoán, sửa lỗi và quản lý kết nối máy in trong mạng LAN trên Windows và Linux**.

Mục tiêu của dự án không phải gom các "mẹo sửa lỗi" vào một nút bấm. PrintDock phải xác định nguyên nhân trước, chỉ thực hiện thay đổi phù hợp với hệ điều hành hiện tại, ghi lại mọi thay đổi và kiểm tra lại sau khi sửa.

> Trạng thái: đang phân tích và xây dựng MVP đa nền tảng.

## Vì sao phải hỗ trợ Windows và Linux

Nhiều doanh nghiệp dùng Windows ở máy người dùng nhưng Linux ở máy chủ, hoặc dùng Linux desktop để giảm chi phí bản quyền. Vì vậy PrintDock không được gắn chặt vào Print Spooler, Registry hay PowerShell.

PrintDock phải hiểu hai hệ sinh thái in chính:

```text
Windows
  -> Print Spooler
  -> Windows printer sharing
  -> SMB / RPC
  -> Windows printer drivers
  -> Registry / Group Policy

Linux
  -> CUPS
  -> IPP / IPPS
  -> Samba / SMB khi kết nối printer share kiểu Windows
  -> lpadmin / lpstat / libcups
  -> PPD / driverless printing
  -> systemd/service permissions
```

Core của PrintDock phải dùng chung. Phần phụ thuộc hệ điều hành nằm trong adapter riêng.

## Bài toán

"Không in được" chỉ là triệu chứng. Nguyên nhân có thể nằm ở:

```text
Client
  -> hostname / IP
  -> LAN / firewall
  -> SMB hoặc IPP
  -> print service
       Windows: Print Spooler
       Linux: CUPS
  -> printer share / print queue
  -> credentials / permissions
  -> driver / driverless configuration
  -> local printer connection
  -> physical printer
```

PrintDock xử lý theo mô hình:

```text
Collect -> Diagnose -> Explain -> Plan -> Repair -> Verify -> Report
```

## Nguyên tắc sản phẩm

1. **Diagnostic trước, repair sau.**
2. Core không phụ thuộc Windows hoặc Linux.
3. Không sửa thành phần đang hoạt động bình thường.
4. Mọi thay đổi có rủi ro phải có before/after.
5. Sau mỗi repair phải verify.
6. Chỉ nâng quyền khi action thực sự cần.
7. Không lưu mật khẩu.
8. Không ghép trực tiếp input người dùng thành shell command.
9. Mỗi platform adapter phải có capability rõ ràng.
10. Nếu một chức năng không hỗ trợ trên OS hiện tại, UI phải nói rõ thay vì giả vờ chạy.

## Phạm vi MVP

MVP có hai track song song:

### Shared diagnostics dùng chung

- Validate hostname/IP/printer target.
- Resolve hostname/IP.
- Kiểm tra network reachability.
- Kiểm tra TCP service theo protocol.
- Printer target model.
- Structured diagnostic result.
- Hypothesis engine.
- Logging/report export.

### Windows adapter

- Print Spooler.
- Windows printer inventory.
- SMB printer share.
- Windows printer connection.
- Windows-specific repair.

### Linux adapter

- Phát hiện distro/runtime cơ bản.
- CUPS service.
- Printer/queue inventory qua CUPS.
- IPP/IPPS endpoint.
- Samba/SMB printer share khi cần.
- Driverless printer capability khi có.
- Linux-specific repair qua CUPS/system service.

## Kiến trúc dự kiến

```text
PrintDock.App
        |
        v
PrintDock.Application
        |
        +--> PrintDock.Core
        |      models
        |      contracts
        |      rule engine
        |      validation
        |
        +--> PrintDock.Diagnostics
        |
        +--> PrintDock.Repairs
        |
        +--> PrintDock.Platform
               |
               +--> PrintDock.Platform.Windows
               |      Win32 / Service / SMB / RPC
               |
               +--> PrintDock.Platform.Linux
                      CUPS / IPP / Samba / systemd
```

UI không được gọi trực tiếp `cmd.exe`, PowerShell, bash, `lpadmin`, Registry hay systemctl.

## Cross-platform capability

Không phải mọi lỗi đều tồn tại trên cả hai OS.

Ví dụ:

| Capability | Windows | Linux |
|---|---:|---:|
| Hostname/IP diagnostics | ✓ | ✓ |
| TCP/445 SMB | ✓ | ✓ |
| IPP/IPPS | Có thể | ✓ |
| Print service | Spooler | CUPS |
| Shared printer discovery | SMB/RPC | CUPS/IPP hoặc Samba |
| Registry repair | ✓ | Không áp dụng |
| CUPS queue repair | Không áp dụng | ✓ |
| Driverless IPP Everywhere | Tùy printer | ✓ |

Core chỉ yêu cầu capability; adapter quyết định implementation thực tế.

## Safety

- validate input trước platform adapter;
- không shell-concatenate;
- repair có scope rõ;
- destructive action cần confirm;
- privilege escalation theo action;
- không lưu credential;
- export report có redaction;
- Windows-only workaround không bao giờ chạy trên Linux và ngược lại.

Chi tiết: [docs/SECURITY.md](docs/SECURITY.md)

## Tài liệu

- [Problem & Product Scope](docs/PRODUCT.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Diagnostics Model](docs/DIAGNOSTICS.md)
- [Security & Safety](docs/SECURITY.md)
- [Roadmap](docs/ROADMAP.md)
- [Implementation Plan](docs/IMPLEMENTATION_PLAN.md)
- [MVP Epic](https://github.com/Kim-Thu/print-dock/issues/1)

## Development status

Backlog được quản lý bằng GitHub Issues. Mỗi implementation issue phải nêu rõ:
- platform nào áp dụng;
- capability nào cần;
- side effects;
- privilege;
- acceptance criteria;
- test cases;
- hành vi khi capability không tồn tại.

## License

Chưa chọn license. Không mặc định coi repository này là public-domain cho tới khi license được bổ sung.
