# PrintDock

**PrintDock** là ứng dụng Windows mã nguồn mở dùng để **chẩn đoán, sửa lỗi và quản lý kết nối máy in chia sẻ trong mạng LAN**.

Mục tiêu của dự án không phải gom các "mẹo sửa lỗi" vào một nút bấm. PrintDock phải xác định nguyên nhân trước, chỉ thực hiện thay đổi phù hợp, ghi lại mọi thay đổi và kiểm tra lại sau khi sửa.

> Trạng thái: đang phân tích và xây dựng MVP.

## Bài toán

Trong môi trường Windows, lỗi máy in chia sẻ thường bị biểu hiện giống nhau: "không in được", "không add được máy in", "không thấy máy in", hoặc các mã như `0x0000011b`, `0x00000709`.

Nhưng nguyên nhân có thể nằm ở nhiều lớp khác nhau:

```text
Client
  -> DNS / hostname / IP
  -> LAN / firewall
  -> SMB
  -> RPC
  -> Print Spooler
  -> printer share
  -> credentials / permissions
  -> driver
  -> local printer connection
  -> print queue
  -> Windows policy / registry
  -> physical printer
```

Nếu sửa theo kiểu "thử hết registry, restart hết service" thì có thể làm thay đổi máy không cần thiết, khó rollback và khó biết nguyên nhân thật.

PrintDock xử lý theo mô hình:

```text
Collect -> Diagnose -> Explain -> Plan -> Repair -> Verify -> Report
```

## Nguyên tắc sản phẩm

1. **Diagnostic trước, repair sau.**
2. Không sửa thành phần đang hoạt động bình thường.
3. Mọi thay đổi có rủi ro phải có thông tin before/after.
4. Repair phải idempotent khi có thể: chạy lại không làm trạng thái xấu hơn.
5. Sau mỗi repair phải chạy verification tương ứng.
6. Không yêu cầu quyền Administrator cho các phép kiểm tra chỉ đọc.
7. Chỉ nâng quyền khi thao tác thật sự cần.
8. Không lưu mật khẩu người dùng.
9. Không thực thi chuỗi lệnh ghép trực tiếp từ input người dùng.
10. Log phải đủ cho người dùng và kỹ thuật viên hiểu "đã kiểm tra gì, thấy gì, sửa gì".

## MVP

MVP tập trung vào luồng máy khách Windows kết nối tới máy in được share từ một máy Windows khác trong cùng LAN.

### Diagnostic

- Thu thập Windows version/build, architecture và quyền hiện tại.
- Validate hostname, IPv4/IPv6 và tên printer/share.
- Resolve hostname/IP.
- Ping khi phù hợp nhưng không dùng ping làm điều kiện duy nhất.
- Kiểm tra SMB TCP/445.
- Kiểm tra Print Spooler local.
- Phát hiện printer local / printer connection hiện có.
- Liệt kê printer share trên server.
- Kiểm tra khả năng resolve đường dẫn printer share.
- Thu thập driver, port, queue và trạng thái liên quan.
- Trả về kết quả có cấu trúc thay vì chỉ chuỗi log.

### Repair

- Restart Print Spooler an toàn.
- Xóa printer connection cũ theo lựa chọn.
- Kết nối lại shared printer.
- Verify printer đã xuất hiện ở client.
- Gửi test page khi người dùng chủ động yêu cầu.
- Ghi nhận trạng thái `SUCCESS`, `FAILED`, `PARTIAL`, `SKIPPED`.

### Chưa thuộc MVP

- Tự động sửa tất cả registry/policy liên quan PrintNightmare.
- Tự tải driver từ Internet.
- Remote execution trên máy chủ.
- Quản trị nhiều máy hàng loạt.
- Cloud backend / telemetry.
- Auto-update.
- Thay đổi firewall diện rộng.

Các phần này chỉ được thêm sau khi có diagnostic rule và test matrix rõ ràng.

## Kiến trúc dự kiến

```text
PrintDock.App
  UI / ViewModels
        |
        v
PrintDock.Application
  Use cases / orchestration
        |
        +--> PrintDock.Diagnostics
        |      network
        |      smb
        |      spooler
        |      printers
        |      drivers
        |      queue
        |
        +--> PrintDock.Repairs
        |      spooler repair
        |      remove connection
        |      connect printer
        |
        +--> PrintDock.Windows
        |      Windows API / PowerShell adapters
        |
        +--> PrintDock.Core
               models
               rules
               result types
               logging contracts
```

Mục tiêu là tách UI khỏi logic hệ thống. UI không được tự gọi `cmd.exe`, sửa registry hay restart service.

## Luồng người dùng

### Chế độ đơn giản

Người dùng nhập/chọn máy chủ và máy in, sau đó bấm **Kiểm tra**.

PrintDock hiển thị:

- cái gì đang hoạt động;
- cái gì đang lỗi;
- nguyên nhân có khả năng phù hợp với bằng chứng nào;
- thao tác nào sẽ được thực hiện trước khi người dùng bấm **Sửa**.

### Chế độ kỹ thuật

Hiển thị thêm:

- từng diagnostic probe;
- command/API đã sử dụng ở mức an toàn;
- mã lỗi hệ thống;
- before/after state;
- thời gian thực thi;
- raw log có thể export.

## Safety

PrintDock có khả năng thay đổi service, printer connection, driver/policy trong các phiên bản sau nên safety là yêu cầu lõi:

- input phải được validate và truyền dưới dạng argument, không nối chuỗi command;
- repair có scope rõ;
- thao tác destructive phải có confirm;
- registry/policy phải snapshot trước khi sửa;
- log phải che dữ liệu nhạy cảm;
- không lưu credential;
- chức năng elevated phải tách khỏi phần read-only khi có thể.

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

Backlog được quản lý bằng GitHub Issues. Mỗi issue implementation phải có:

- vấn đề cần giải quyết;
- phạm vi;
- phụ thuộc;
- thiết kế kỹ thuật;
- edge cases;
- acceptance criteria;
- test cases;
- điều kiện không được làm.

## License

Chưa chọn license. Không mặc định coi repository này là public-domain cho tới khi license được bổ sung.
