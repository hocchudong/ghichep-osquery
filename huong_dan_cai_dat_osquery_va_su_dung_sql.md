# Hướng dẫn cài đặt osquery và sử dụng các câu truy vấn.

## 1. Giới thiệu
- osquery là giải pháp nguồn mở do facebook phát triển, osquery có chức nnăg thu thập các thông tin phần cứng theo cơ chế của ngôn ngữ truy vấn. Tức là để hiển thị các dữ liệu mong muốn, người sử dụng (system admin hoặc dev hoặc người dùng thông thường) sẽ sử dụng các câu truy vấn theo ngôn ngữ SQL để làm việc.
- osquery có các phiên bản dành cho Linux, Windows, MacOS. Có thể cài cả trên máy trạm hoặc máy chủ.
- osquery đóng vai trò như một agent trong các giải pháp về bảo mật theo cơ chế HIDS (Host-based intrusion detection system).
- osquery được cài lên từng máy hay còn gọi là host và có thể được quản lý tập trung bởi giải pháp kolide fleet. Bộ stack kết hợp với osquery có thể là: `osquery, kolide fleet, graylog` hoặc `osquery, kolide fleet, ELK` hoặc `osquery, kolide fleet, Splunk`.

## .2 Hướng dẫn cài đặt osquery
- Trong phạm vi bài viết này tôi sẽ hướng dẫn cài đặt osquery dạng độc lập. Sau đó sẽ hướng dẫn sử dụng các câu SQL cơ bản để làm việc với osquery.

### 2.1. Chuẩn bị

- Sử dụng một máy chủ CentOS7, có kết nối internet.

#### 2.1.1 Thiết lập cơ bản cho hệ điều hành

-  Cập nhật hệ điều hành và các gói bổ trợ.
    ```
    yum update -y && yum install -y wget yum-utils byobu 
    ```

- Thiết lập hostname.
    ```
    hostnamectl set-hostname osqueryclient

    bash
    ```
- Cấu hình IP.
    ```
    Cấu hình IP tĩnh theo quy hoạch IP của bạn
    ```

- Khởi động lại hệ điều hành
    ```
    init 6
    ````

#### 2.1.2. Cài đặt osquery

- Thực hiện cài đặt osquery
    ```
    curl -L https://pkg.osquery.io/rpm/GPG | tee /etc/pki/rpm-gpg/RPM-GPG-KEY-osquery
    yum-config-manager --add-repo https://pkg.osquery.io/rpm/osquery-s3-rpm.repo
    yum-config-manager --enable osquery-s3-rpm
    yum install osquery -y
    cp /usr/share/osquery/osquery.example.conf /etc/osquery/osquery.conf
    ```

- Kiểm tra lại phiên bản của osquery khi cài đặt xong bằng lệnh `osqueryd -version`
    ```
    [root@osqueryclient ~]# osqueryd -version
    osqueryd version 4.0.2
    ```

Tiếp theo, trong hướng dẫn này sẽ tiến hành sửa một số cấu hình cơ bản của osquery trước khi start.

- File cấu hình của osquery sẽ nằm tại file `/var/osquery/osquery.conf`. File này có định dạng json theo cấu trúc

```
{
    "options": {
        // Các khai báo 
    },

    "schedule": {
        // Các khai báo
    },

     "decorators": {
         //Các khai báo

     },

     "packs": {
         //Các khai báo
     },

     // Và các khai báo khác.

}
```

- Cú pháp của file khai báo trên như sau:
  - Dòng bắt đầu là ký tự `//` là dòng commend, không có tác dụng khi osquery được khởi động.
  - `"options"`: Khai các tùy chọn về lưu trữ dữ liệu, tùy chọn về file log.
  - `"schedule"`:  Khai báo để lập lịch định kỳ chạy các câu truy vấn mà ta chỉ định.
  - `"packs"`: Là đường dẫn chứa các khai báo các cấu hình chuyên biệt theo nhóm được cấu hình sẵn.

Thực hiện khai báo một số cấu hình căn bẳn trong file `/etc/osquery/osquery.conf`. Cụ thể như sau

Mở file `/etc/osquery/osquery.conf` và sửa các dòng dưới:

- Bỏ dòng comment (`//`) tại dòng 
    ```
    //"logger_path": "/var/log/osquery",
    ```

- Thành dòng 
    ```
    "logger_path": "/var/log/osquery",
    ```

- Dưới dòng `"schedule": {` thêm đoạn cấu hình dưới

```
    // for example, get CPU Time per 60 seconds
    "cpu_time": {
      "query": "SELECT * FROM cpu_time;",
      "interval": 60
    },

    // for example, get settings of resolv.conf per an hour
    "dns_resolvers": {
      "query": "SELECT * FROM dns_resolvers;",
      "interval": 120
    },

```

- Ta sẽ có file dạng sau:

![Ảnh cấu hình](https://image.prntscr.com/image/trqfRM0jQsCUbWv6RZPMOw.png)


- Khởi động `osqueryd`
    ```
    systemctl start osqueryd
    systemctl enable osqueryd
    ```

- Kiểm tra trạng thái của osquery
    ```
    systemctl status osqueryd
    ```

- Ta có kết quả của osquery như sau là ok
    ```
    ● osqueryd.service - The osquery Daemon
    Loaded: loaded (/usr/lib/systemd/system/osqueryd.service; disabled; vendor preset: disabled)
    Active: active (running) since Sat 2019-11-16 07:43:35 +07; 4s ago
    Process: 1223 ExecStartPre=/bin/sh -c if [ -f $LOCAL_PIDFILE ]; then mv $LOCAL_PIDFILE $PIDFILE; fi (code=exited, status=0/SUCCESS)
    Process: 1220 ExecStartPre=/bin/sh -c if [ ! -f $FLAG_FILE ]; then touch $FLAG_FILE; fi (code=exited, status=0/SUCCESS)
    Main PID: 1226 (osqueryd)
    CGroup: /system.slice/osqueryd.service
            ├─1226 /usr/bin/osqueryd --flagfile /etc/osquery/osquery.flags --config_path /etc/osquery/osquery.conf
            └─1229 /usr/bin/osqueryd

    Nov 16 07:43:34 osqueryclient systemd[1]: Starting The osquery Daemon...
    Nov 16 07:43:35 osqueryclient systemd[1]: Started The osquery Daemon.
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: osqueryd started [version=4.0.2]
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: I1116 07:43:35.080093  1229 database.cpp:570] Checking database version for migration
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: I1116 07:43:35.080296  1229 database.cpp:594] Performing migration: 0 -> 1
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: I1116 07:43:35.082423  1229 database.cpp:626] Migration 0 -> 1 successfully completed!
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: I1116 07:43:35.082475  1229 database.cpp:594] Performing migration: 1 -> 2
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: I1116 07:43:35.084009  1229 database.cpp:626] Migration 1 -> 2 successfully completed!
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: I1116 07:43:35.140584  1229 events.cpp:863] Event publisher not enabled: auditeventpublisher: Publ...guration
    Nov 16 07:43:35 osqueryclient osqueryd[1226]: I1116 07:43:35.140792  1229 events.cpp:863] Event publisher not enabled: syslog: Publisher disable...guration
    Hint: Some lines were ellipsized, use -l to show in full.
    ```

- Thực hiện đang nhập vào osquery bằng lệnh `osqueryi`, ta có màn hình tương tác ở chế độ của SQL
    ```
    [root@osqueryclient ~]# osqueryi
    Using a virtual database. Need help, type '.help'
    osquery>
    ```