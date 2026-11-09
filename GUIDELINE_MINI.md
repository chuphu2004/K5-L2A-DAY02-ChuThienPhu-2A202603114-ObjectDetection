# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | xe con, sedan, SUV, taxi, xe bán tải | xe thấp, không quá lớn, chở từ 4-9 người |
| `truck` | xe tải chở hàng, xe đầu kéo | lớn, có phần cabin lái tách rời với phần hàng phía sau |
| `bus` | xe buýt, minibus chở khách| kích thước lớn, có nhiều cửa sổ chạy dọc thân xe, cỏ cửa cho khách |
| `van` | xe van, minivan chở hàng hoặc chở người| thường có cửa lùa bên hông, có hình dáng khác với car ở đuôi, mui xe  |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: box ôm sát phần thân xe nhìn thấy được
- Vật thể bị che khuất: Xe bị che vẫn gán nếu còn nhận ra lớp
- Vật thể bị cắt bởi rìa ảnh: Xe bị cắt ở rìa thì box chạm đúng rìa ảnh.
- Vật thể rất nhỏ/mơ hồ: bỏ qua không dán nhãn

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_044` | xe bị cắt bởi rìa ảnh, khó phân biệt | gán lớp 'bus' | có cửa sổ dọc thân xe |
| `drive_030` | các xe ô tô phía xa không phân biệt được viền | không gán nhãn | quá mơ hồ, không phân biệt được các xe với nhau |
| `drive_027` | các xe ô tô phía xa không phân biệt được viền | không gán nhãn | quá mơ hồ, không phân biệt được các xe với nhau |
