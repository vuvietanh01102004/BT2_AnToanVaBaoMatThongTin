# BÀI TẬP 2: CHỮ KÝ SỐ TRONG FILE PDF  
## Môn: An toàn và Bảo mật Thông tin  
## Giảng viên: Đỗ Duy Cốp  
## Lớp: K58KTP  
## Sinh viên: Nguyễn Như Khiêm  
## Hạn nộp: 31/10/2025

# Bài làm
## 1. Cấu trúc PDF liên quan chữ ký số
- Mô tả ngắn gọn:
  + Catalog: Đối tượng gốc của tài liệu PDF, liên kết đến các phần tử khác như cây trang (Pages tree).
  + Pages tree: Cấu trúc phân cấp chứa thông tin về tất cả các trang trong tài liệu.
  + Page object: Mỗi trang là một đối tượng riêng, gồm nội dung, kích thước, và tài nguyên liên quan.
  + Resources: Tập hợp các tài nguyên (font, hình ảnh, màu, v.v.) được dùng trong trang.
  + Content streams: Dòng lệnh mô tả nội dung hiển thị của trang (văn bản, hình ảnh, đồ họa).
  + XObject: Đối tượng có thể tái sử dụng, thường dùng cho hình ảnh hoặc biểu mẫu.
  + AcroForm: Cấu trúc cho phép tạo và quản lý các biểu mẫu tương tác trong PDF.
  + Signature field (widget): Trường chứa vùng hiển thị của chữ ký số trên trang.
  + Signature dictionary (/Sig): Chứa dữ liệu chữ ký, thuật toán, và chứng chỉ liên quan.
  + /ByteRange: Xác định các đoạn byte trong tệp được ký và bỏ qua phần chứa chữ ký.
  + /Contents: Chứa giá trị chữ ký số (chuỗi mã hóa).
  + Incremental updates: Cơ chế cho phép thêm chữ ký mới mà không cần ghi đè lên phần cũ.
  + DSS (Document Security Store): Lưu trữ chứng chỉ, CRL, và OCSP để xác thực lâu dài (theo chuẩn PAdES).

```
Catalog
 ├── Pages (Pages tree)
 │    ├── Page object 1
 │    │    ├── Resources
 │    │    ├── Content streams
 │    │    └── XObject
 │    └── Page object n
 ├── AcroForm
 │    └── Signature field (widget)
 │         └── Signature dictionary (/Sig)
 │              ├── /ByteRange
 │              ├── /Contents
 │              └── DSS (theo PAdES)
 └── Incremental updates (các phiên bản thêm)
```

- Object refs quan trọng và vai trò của từng object trong lưu/truy xuất chữ ký:
  + Catalog: Gốc của tài liệu PDF — trỏ tới AcroForm và DSS. Là điểm bắt đầu để truy xuất chữ ký.
  + AcroForm: Chứa danh sách các trường biểu mẫu, trong đó có trường chữ ký (Signature field).
  + Signature field (widget): Xác định vị trí chữ ký trong tài liệu và liên kết tới Signature dictionary (/Sig).
  + Signature dictionary (/Sig): Nơi lưu dữ liệu chữ ký số, thuật toán, chứng chỉ và thông tin xác thực.
  + /ByteRange: Chỉ ra các vùng byte trong file được ký (phần nào được bảo vệ bởi chữ ký).
  + /Contents: Chứa giá trị chữ ký số đã mã hóa (chuỗi hex).
  + DSS (Document Security Store): Lưu trữ thông tin xác thực lâu dài như chứng chỉ, OCSP, CRL theo chuẩn PAdES.
  + Incremental updates: Cơ chế thêm chữ ký mới mà không ghi đè nội dung cũ, hỗ trợ nhiều chữ ký trong cùng tài liệu.

## 2. Thời gian ký được lưu ở đâu?
### Các vị trí có thể chứa thông tin thời gian ký trong PDF
- /M trong Signature Dictionary:
  + Lưu thời gian ký ở dạng chuỗi text
  + Do người ký hoặc phần mềm ký chèn vào.
  + Không có giá trị pháp lý vì có thể bị thay đổi, không được xác thực bằng chữ ký số.
- Timestamp token (RFC 3161) trong PKCS#7:
  + Là tem thời gian điện tử được cấp bởi TSA (Time Stamping Authority).
  + Lưu trong thuộc tính "timeStampToken" của chữ ký.
  + Có giá trị pháp lý vì được ký số bởi TSA, đảm bảo tính toàn vẹn và xác thực thời gian ký.
- Document Timestamp Object (PAdES)
  + Là đối tượng tem thời gian toàn tài liệu, áp dụng cho toàn bộ file PDF (theo chuẩn PAdES).
  + Dùng để chứng minh thời điểm tồn tại của tài liệu ngay cả khi không có chữ ký cá nhân.
- DSS (Document Security Store): Nếu có chứa timestamp và dữ liệu xác minh (OCSP, CRL, chứng chỉ TSA),
thì DSS giúp duy trì khả năng xác thực lâu dài cho chữ ký và thời gian ký.
### Khác biệt giữa thông tin thời gian ?M và timestamp RFC3161
- Thông tin thời gian /M chỉ là chuỗi văn bản do phần mềm ký chèn vào, có thể sửa đổi nên không có giá trị pháp lý.
- Ngược lại, timestamp RFC 3161 là tem thời gian điện tử được ký số bởi TSA (Tổ chức cung cấp dịch vụ thời gian tin cậy), đảm bảo tính xác thực và toàn vẹn, do đó có giá trị pháp lý để chứng minh thời điểm ký.


