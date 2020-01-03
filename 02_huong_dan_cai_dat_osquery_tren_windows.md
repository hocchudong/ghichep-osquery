# Hướng dẫn cài đặt osquery trên windows

Trong phần 01, tôi đã hướng dẫn bạn cài đặt osquery trên Linux, trong hướng dẫn này tôi sẽ hướng dẫn bạn cài đặt osquery trên windows. 

Ta có thể cài osquery trên các hệ điều hành windows bất kỳ, trong hướng dẫn này tôi sẽ lấy môi trường windows 10 để làm hướng dẫn.

## Chuẩn bị

Chuẩn bị một máy windows 10 có kết nối internet.

## Tải file cài osquery cho windows

Truy cập vào trang chủ của osquery và tìm mục download. Sau đó tải file cài cho windows

![osquerywin1](https://image.prntscr.com/image/MnDl690NQbiVntK4UWGGRQ.png)

Mở file vừa được tải về và chọn `Next`

![osquerywin2](https://image.prntscr.com/image/1QEcSr0RSKiXBfnV_b-aZA.png)

Chọn mục `I accept the terms in the License Agreement`, sau đó chọn `Next`

![osquerywin3](https://image.prntscr.com/image/7dOX65puTw6gBujz5nK40A.png)

Giữ nguyên thư mục cài đặt và chọn `Next`

![osquerywin4](https://image.prntscr.com/image/UJ8aiqgBQEi6QBJb6GfZXw.png)


Sau đó chọn `Install`

![osquerywin4](https://image.prntscr.com/image/rKdBEHmUQDqtybEUz-EdnA.png)

Một cửa sổ mới hiện lên, chọn `Yes`

![osquerywin5](https://image.prntscr.com/image/82uSt1fWS1qvi92mesfhYw.png)


Sau đó chọn `Finsh`

![osquerywin6](https://image.prntscr.com/image/3WkiKVgiSY6TaPiVKRtctQ.png)

Sau đó bắt đầu bước kiểm tra và sử dụng osquery trên windows.

Ta có thể kiểm tra xem osquery đã hoạt động hay chưa trong services của windows.

![osquerywin7](https://image.prntscr.com/image/SIMj9o1YS66B_vfeN3rLuw.png)

## Sử dụng osquery trên windows.

Cũng giống như trong linux, ta sẽ thực hiện sử dụng osquery ở chế độ live (chế độ tương tác trực tiếp). Để sử dụng osquery ở chế độ live trên windows ta sẽ mở cửa sổ `powershell` của windows để thực hiện.

Mở cửa sổ `cmd` trong windows và gõ lệnh

```
cd "C:\Program Files\osquery"
```

Sau đó thực hiện lệnh 

```
osqueryi.exe
```

Các lệnh trên sẽ giống như ảnh bên dưới

![osquerywin8](https://image.prntscr.com/image/1-9VF2ghQ2Ccz5JH16D5og.png)


Lúc này, ta sẽ ở chế độ live của `osquery`. Ta đã có thể bắt đầu thực hiện các lệnh mà osquery hỗ trợ với windows.

### Kiểm tra phiên bản của hệ điều hành

Sử dụng lệnh `select * from os_version;` để kiểm tra phiên bản hệ điều hành. Ta có kết quả:

![osquerywin9](https://image.prntscr.com/image/TFB512XrS76-Qo_5XNtDYw.png)


### Kiểm tra thông tin chung về phần cứng

Sử dụng lệnh dưới để liệt kê loại CPU, hãng phần cứng

```
select cpu_type, cpu_brand, hardware_vendor, hardware_model from system_info;
```

Kết quả ta sẽ có 

![osquerywin10](https://image.prntscr.com/image/yUjEgoXWSEmfIWRf-GnSbQ.png)

### Kiểm tra 05 chương trình sử dụng nhiều RAM nhất.

Sử dụng câu lệnh dưới để hiển thị 05 chương trình sử dụng nhiều RAM nhất.

```
SELECT pid, name, ROUND((total_size * '10e-7'), 2) AS used FROM processes ORDER BY total_size DESC LIMIT 5;
```

Ta sẽ có kết quả

![osquerywin11](https://image.prntscr.com/image/8cCjQUCNTcCfuyihmmISlg.png)

Tới đây, bạn có thể tìm hiểu thêm các lệnh mà osquery đã hỗ trợ với windows tại: https://osquery.io/schema/4.1.2