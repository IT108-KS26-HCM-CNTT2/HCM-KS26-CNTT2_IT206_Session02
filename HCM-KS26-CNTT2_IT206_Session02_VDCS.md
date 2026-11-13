
## BƯỚC 1: THIẾT KẾ GIẢI PHÁP (INPUT / OUTPUT)

### 1. Bối cảnh áp dụng
Nhân viên cửa hàng bán đồ công nghệ thường xuyên nhận tin nhắn đặt hàng lộn xộn qua Zalo/Facebook. Thay vì ngồi nhập tay từng đơn vào phần mềm quản lý, ta dùng AI để tự động đọc tin nhắn và xuất ra dữ liệu chuẩn JSON.

### 2. Xác định Input (Đầu vào)
Đoạn tin nhắn văn bản thô do khách hàng nhắn tới (thường chứa tên sản phẩm, số lượng, số điện thoại, địa chỉ, nhưng đôi khi khách nhắn thiếu hoặc ghi sai định dạng).

### 3. Xác định Output (Đầu ra mong muốn)
Một chuỗi dữ liệu định dạng **JSON** duy nhất (không kèm lời chào hay giải thích thừa) gồm các trường cố định:
* `ten_san_pham` (Chuỗi)
* `so_luong` (Số nguyên)
* `so_dien_thoai` (Chuỗi 10 số)
* `dia_chi` (Chuỗi)
* `trang_thai_loi` (Chuỗi - báo lỗi nếu thiếu hoặc sai thông tin quan trọng)

### 4. Bẫy dữ liệu & Cách xử lý lỗi
* Nếu khách quên nhắn số điện thoại hoặc địa chỉ: AI không được tự bịa ra mà phải điền giá trị `null` và ghi rõ thiếu gì vào mục `trang_thai_loi`.
* Nếu khách không ghi số lượng: Mặc định hiểu là `1`.

---

## BƯỚC 2: VIẾT PROMPT HOÀN CHỈNH & CHẠY THỬ

### 1. Prompt thực thi (Áp dụng CREATE + Few-shot + Ép JSON)

> **[C - Vai trò]**
> Đóng vai hệ thống trích xuất dữ liệu đơn hàng tự động cho cửa hàng phụ kiện công nghệ.
> 
> **[R - Yêu cầu]**
> Hãy đọc tin nhắn đặt hàng của khách ở cuối và chuyển đổi thông tin thành định dạng JSON chuẩn.
> 
> **[E - Ví dụ mẫu (Few-shot)]**
> *Ví dụ 1 (Đủ thông tin):*
> Input: "Shop ơi ship cho mình 2 cái chuột không dây Logitech tới 12 Nguyễn Trãi, Q1 nha. SĐT 0909123456"
> Output:
> ```json
> {
>   "ten_san_pham": "Chuột không dây Logitech",
>   "so_luong": 2,
>   "so_dien_thoai": "0909123456",
>   "dia_chi": "12 Nguyễn Trãi, Q1",
>   "trang_thai_loi": "Hop_le"
> }
> ```
> *Ví dụ 2 (Thiếu thông tin & SĐT sai):*
> Input: "Lấy anh bàn phím cơ màu đen, gọi số 0912345"
> Output:
> ```json
> {
>   "ten_san_pham": "Bàn phím cơ màu đen",
>   "so_luong": 1,
>   "so_dien_thoai": null,
>   "dia_chi": null,
>   "trang_thai_loi": "Thieu_dia_chi, So_dien_thoai_khong_du_10_so"
> }
> ```
> 
> **[A & E - Điều chỉnh & Ràng buộc xử lý lỗi]**
> * Nếu khách không ghi số lượng, mặc định điền `1`.
> * Nếu thiếu địa chỉ hoặc số điện thoại (không đủ 10 chữ số), bắt buộc điền `null` vào trường đó và ghi chú lỗi vào `trang_thai_loi`, tuyệt đối không tự đoán mò.
> 
> **[T - Định dạng đầu ra]**
> Chỉ trả về duy nhất khối mã JSON, không viết thêm bất kỳ câu mở đầu hay giải thích nào bên ngoài.
> 
> **[Dữ liệu cần xử lý]**
> Input: "Em ơi gửi gấp cho chị 3 dây sạc iPhone loại 2m về địa chỉ Chung cư Hoàng Anh Gia Lai, Quận 7 nhé, chị quên mất chưa để lại số rồi."

### 2. Kết quả phản hồi chạy thử từ AI

```json
{
  "ten_san_pham": "Dây sạc iPhone loại 2m",
  "so_luong": 3,
  "so_dien_thoai": null,
  "dia_chi": "Chung cư Hoàng Anh Gia Lai, Quận 7",
  "trang_thai_loi": "Thieu_so_dien_thoai"
}
```