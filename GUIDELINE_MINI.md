# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Nguyễn Thành Đạt
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                              | Không gán                                                     |
| -------------------------------- | ------------------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải    | người đi bộ                                                   |
| van, minivan                     | xe đạp                                                        |
| xe buýt, minibus                 | **xe máy / mô tô**                                            |
| xe tải, xe đầu kéo               | xe trong ảnh quảng cáo, trong gương, dưới bóng nước           |

Bổ sung của nhóm (nếu có): `Không gán các phương tiện thô sơ hoặc xe đẩy hàng tự chế không có động cơ ô tô.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống                          | Luật của nhóm                                                                                                | Vì sao |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------ | ------- |
| Xe bị che một phần rồi hiện lại     | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps)                  | Tránh đứt gãy track khi xe chạy qua vật cản ngắn hạn |
| Xe bị che lâu hơn ngưỡng trên       | Mở `track mới` với ID hoàn toàn khác                                                                         | Đảm bảo tính nhất quán định danh khi đối tượng biến mất quá lâu |
| Xe rời khung hình rồi quay lại      | Mặc định: **track mới**                                                                                      | Đã ra khỏi khung hình coi như kết thúc một chu kỳ xuất hiện |
| Hai xe cắt nhau / chồng lên nhau    | Duy trì track_id liên tục dựa trên quỹ đạo chuyển động (motion vector) và tốc độ trước khi chồng lấp          | Ngăn chặn lỗi ID switch do chồng lấp hình ảnh |

## 3. Luật bbox

| Tình huống                                   | Luật của nhóm                                                                                              |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                        | Bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                              |
| Xe bị xe khác che một phần                   | Bbox ôm phần **nhìn thấy được**                                                                            |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ       | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `từ frame xe hiện diện rõ nét cấu trúc tối thiểu` |
| Xe đang đỗ, không di chuyển                  | Vẫn gán nhãn lớp `vehicle` và duy trì track liên tục trong toàn bộ thời gian nó xuất hiện trong clip      |
| Keyframe đặt dày ở đâu                       | Đặt dày ở các đoạn xe phanh gấp, đổi hướng rẽ, hoặc có vật cản che khuất một phần                           |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01 / frame 45-50 / ID 3 chuyển sang ID 5`
- Tình huống: Xe bị khuất một phần sau xe buýt lớn và bị nhảy ID khi đi khuất.
- Quyết định: Dùng công cụ `Merge` để gộp chung hai track thành một.
- Lý do: Xe không rời khỏi khung hình và thời gian khuất ngắn hơn ngưỡng 25 frame.

### Ca 2

- Clip / frame / ID: `clip_01 / frame 185-190 / ID 2`
- Tình huống: Xe đã chạy ra khỏi biên giới khung hình nhưng bbox vẫn bám lơ lửng.
- Quyết định: Bấm phím `O` (outside) ngay tại frame xe hoàn toàn rời khỏi khung.
- Lý do: Tránh tạo ra lỗi bbox treo (FP) ở cuối video.

### Ca 3

- Clip / frame / ID: `clip_01 / frame 80-95 / ID 4`
- Tình huống: Bbox bị trôi lệch khỏi thân xe do xe di chuyển nhanh qua các frame.
- Quyết định: Thêm các keyframe trung gian và tinh chỉnh lại độ khít của bbox.
- Lý do: Đảm bảo chỉ số LocA (độ khít bounding box) đạt chuẩn qua cổng đánh giá.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Cần quy định chi tiết hơn về ranh giới thời gian khi xe bị che khuất ở các góc khuất giao lộ phức tạp.
- Thêm nguyên tắc xử lý khi các xe có màu sắc/kiểu dáng tương tự di chuyển quá sát nhau để hạn chế tối đa nhầm lẫn ID.
