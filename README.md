# Hòa Phú | Báo cáo dòng tiền

Trang báo cáo tiếng Việt theo ảnh mẫu, gồm 6 chỉ số, bộ lọc gói thầu và tháng, biểu đồ kế hoạch/lũy kế, bảng chi tiết và xuất CSV. React + Vite, Express đọc Google Sheets ở phía máy chủ. Không lưu thông tin đăng nhập Google.

## Chạy

Yêu cầu Node.js 22.12+ (môi trường hiện tại: Node 24).

```sh
npm ci --cache /workspace/.npm-cache
npm run dev
```

Máy chủ cổng 3000 (đổi bằng `PORT`). Production: `npm run build` rồi `npm start`. Kiểm thử: `npm test`.

## Nguồn dữ liệu

- Bảng: `1KpADpAKMNwaxr_tELV6ih63p5qc4HVEeTRYX_MilIhk`, tab `113249470`.
- `/api/cashflow` tải CSV và Google Visualization từ Google, không ghi đè bảng tính. CSV giữ tên cột, Visualization cung cấp giá trị số gốc chưa làm tròn. Nếu hai nguồn không khớp cấu trúc, ứng dụng báo lỗi thay vì tính nhầm.
- Cần cho phép `docs.google.com` và các chuyển hướng `*.googleusercontent.com` qua chính sách mạng. Bảng tính phải cho phép đọc CSV không cần đăng nhập; bảng riêng tư có thể tải CSV bằng tài khoản của bạn rồi dùng **Nhập CSV**.
- Dữ liệu minh họa theo ảnh được gắn nhãn rõ ràng khi chưa có số liệu thật. Lỗi làm mới giữ dữ liệu đang xem và thông báo chưa đồng bộ. Không dùng số liệu minh họa để ra quyết định tài chính.
- CSV nhập chỉ tồn tại trong phiên trình duyệt, không lưu trên máy chủ. Làm mới Google sẽ thay thế dữ liệu CSV khi thành công.

### Cấu trúc

Hỗ trợ bảng hai phần `I. KẾ HOẠCH SẢN LƯỢNG` và `II. KẾ HOẠCH DÒNG TIỀN`. Mỗi phần có dòng tiêu đề `Gói thầu`, các cột tháng `07/26`, `08/26`… và (phần sản lượng) `Giá trị hợp đồng`, `IPC lũy kế`. Mỗi dòng là một gói thầu. Nếu bảng gốc dùng tên/cấu trúc khác, cần điều chỉnh parser sau khi đọc được dữ liệu thật.

Hoặc nhập CSV dạng:

```csv
Đơn vị: tỷ đồng
Gói thầu,Tháng,Giá trị hợp đồng,IPC lũy kế,Sản lượng KH,Dòng tiền KH
Kết cấu,07/26,298.5,208.5,18.349,0
Kết cấu,08/26,298.5,208.5,18.349,18.349
```

Mặc định đơn vị nguồn là VNĐ, chuyển sang tỷ đồng để hiển thị. Muốn dùng tỷ đồng, thêm dòng đơn vị như trên. Số thập phân có dấu phẩy cần được bao trong dấu nháy CSV. Giá trị hợp đồng và IPC lặp lại mỗi tháng, không cộng dồn; dòng cuối có giá trị sẽ được dùng. Không đưa dòng tổng vào CSV dạng dài.

### Quy tắc tính

- Giá trị còn lại = hợp đồng − IPC lũy kế.
- Sản lượng/dòng tiền trong kỳ = tổng các tháng và gói thầu đang lọc.
- Chênh lệch phân bổ = sản lượng trong kỳ − giá trị còn lại.
- Dòng tiền theo N+1 = sản lượng của tháng lịch trước, bao gồm tháng trước ngoài bộ lọc nếu có trong nguồn.
- Chênh lệch theo tháng = dòng tiền KH − dòng tiền theo N+1.
- Lũy kế bắt đầu từ tháng đang lọc, không bao gồm IPC đầu kỳ.
- Tỷ trọng = dòng tiền tháng / tổng dòng tiền trong kỳ; bằng 0 nếu tổng bằng 0.

## Xác minh nguồn

Đã đọc được bảng thật sau khi cập nhật cấu hình mạng. Bảng có 3 gói thầu và 6 tháng từ 07/26 đến 12/26. Hợp đồng: 539.024322434 tỷ; IPC: 386.296200404 tỷ; sản lượng và dòng tiền toàn kỳ: 152.72812203 tỷ. Đây là số liệu tại thời điểm thiết lập, không cố định trong ứng dụng. Ảnh mẫu có số làm tròn khác; ứng dụng dùng giá trị gốc hiện tại của Google Sheets.

CSV tải thủ công chỉ chứa số đã định dạng (ví dụ “0,9 tỷ”), nên có thể lệch với giá trị gốc. Để giữ độ chính xác khi nhập, xuất số VNĐ đầy đủ hoặc CSV dạng dài với đủ chữ số. Dữ liệu live dùng giá trị gốc, hiển thị hai chữ số thập phân sau khi tính.
