# Hướng dẫn cài đặt osquery trên linux

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

### 2.2. Cài đặt osquery

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

- Thử chạy câu lệnh truy vấn về hệ điều hành.
    ```
    select * from os_version; 
    ```
- Kết quả trả về là:

    ```
    osquery> select * from os_version;
    +--------------+--------------------------------------+-------+-------+-------+-------+----------+---------------+----------+
    | name         | version                              | major | minor | patch | build | platform | platform_like | codename |
    +--------------+--------------------------------------+-------+-------+-------+-------+----------+---------------+----------+
    | CentOS Linux | CentOS Linux release 7.7.1908 (Core) | 7     | 7     | 1908  |       | rhel     | rhel          |          |
    +--------------+--------------------------------------+-------+-------+-------+-------+----------+---------------+----------+
    ```

Có thể chạy một số câu lệnh khách như:

- Lấy ra các giá trị về tham số của cpu (tương tự một phần lệnh top)
    ```
    select * from cpu_time;
    ```

- Lệnh lấy các các thông số về hostname, loại cpu, loại phần cứng:
    ```
    select hostname, cpu_brand, hardware_vendor, hardware_model from system_info; 
    ```

Để thoát màn hình sql của osquery, ta thực hiện phím `CTL +D`
 
 - Sử dụng lệnh `tailf` hoặc `tail -f` để theo dõi nội dung file lưu log của các câu truy vấn được khai báo ở tùy chọn `shedule` 

 ```
 tailf /var/log/osquery/osqueryd.results.log
 ```

 - Ta có kết quả như sau

    ```
    {"name":"cpu_time","hostIdentifier":"osqueryclient","calendarTime":"Sat Nov 16 00:52:34 2019 UTC","unixTime":1573865554,"epoch":0,"counter":9,"logNumericsAsNumbers":false,"decorations":{"host_uuid":"95B2A9C0-785F-2B8A-EF22-ECFA7C04E745","username":"reboot"},"columns":{"core":"0","guest":"0","guest_nice":"0","idle":"265659","iowait":"374","irq":"0","nice":"19","softirq":"15","steal":"10","system":"2929","user":"4463"},"action":"added"}
    {"name":"dns_resolvers","hostIdentifier":"osqueryclient","calendarTime":"Sat Nov 16 00:45:04 2019 UTC","unixTime":1573865104,"epoch":0,"counter":0,"logNumericsAsNumbers":false,"decorations":{"host_uuid":"95B2A9C0-785F-2B8A-EF22-ECFA7C04E745","username":"reboot"},"columns":{"address":"8.8.8.8","id":"0","netmask":"32","options":"524993","type":"nameserver"},"action":"added"}   
    ```   

Chuyển sang bước dưới để thực hiện các câu truy vấn cơ bản. 

## 3. Hướng dẫn sử dụng các câu truy vấn cơ bản của osquery.

- Thực hiện đăng nhập vào osquery với lệnh `osqueryi`, a sẽ ở màn hình nhắc lệnh.
- Lưu ý: Để thoát màn hình sql của osquery, ta thực hiện phím `CTL +D`.
- Nếu bạn từng làm việc với các ngôn ngữ SQL của SQLite MySQL, MariaDB hoặc MS SQL ... thì việc thao thác này khá đơn giản, các kết quả sẽ là các giá trị của cột (colum) hoặc hàng (row) của các bảng (tables).
- Schema (Cấu trúc các bảng) của osquery được mô tả tại đây: `https://osquery.io/schema/4.0.2`
- Kết thúc các lệnh của sql là dấu `;`.

Trong các ghi chép dưới đây, tôi sẽ hướng dẫn thao tác với một số bản cơ bản.

### 3.1. Kiểm tra uptime của hệ thống

- Sử dụng `select * from uptime;` để kiểm tra uptime của hệ thống, ta có kết quả
    ```
    osquery> select * from uptime;
    +------+-------+---------+---------+---------------+
    | days | hours | minutes | seconds | total_seconds |
    +------+-------+---------+---------+---------------+
    | 0    | 2     | 13      | 5       | 7985          |
    +------+-------+---------+---------+---------------+
    ```


### 3.2. Bảng user

- Kiểm tra danh sách các user trong hệ thống 


```
select * from users;
```

- Kết quả sẽ trả về danh sách các user

![User](https://image.prntscr.com/image/koBjDnFBTtWlPPv2pu6YZQ.png)


### 3.3. Bảng listening_ports

- Sử dụng lệnh `select * from listening_ports limit 10;` để hiển thị 10 ports đang được mở 

```
+-------+------+----------+--------+-----------+----+--------+-----------------------------+---------------+
| pid   | port | protocol | family | address   | fd | socket | path                        | net_namespace |
+-------+------+----------+--------+-----------+----+--------+-----------------------------+---------------+
| 10691 | 22   | 6        | 2      | 0.0.0.0   | 3  | 28612  |                             | 4026531956    |
| 3441  | 25   | 6        | 2      | 127.0.0.1 | 13 | 24633  |                             | 4026531956    |
| 10691 | 22   | 6        | 10     | ::        | 4  | 28614  |                             | 4026531956    |
| 11321 | 323  | 17       | 2      | 127.0.0.1 | 5  | 30292  |                             | 4026531956    |
| 11321 | 323  | 17       | 10     | ::1       | 6  | 30293  |                             | 4026531956    |
| 1542  | 0    | 0        | 1      |           | 4  | 0      | /run/systemd/journal/socket | 4026531956    |
| 1542  | 0    | 0        | 1      |           | 5  | 0      | /dev/log                    | 4026531956    |
| 9499  | 0    | 0        | 1      |           | 7  | 0      | /root/.osquery/shell.em     | 4026531956    |
| 9535  | 0    | 0        | 1      |           | 6  | 0      | public/pickup               | 4026531956    |
| 3441  | 0    | 0        | 1      |           | 49 | 0      | private/proxymap            | 4026531956    |
+-------+------+----------+--------+-----------+----+--------+-----------------------------+---------------+
```



### 3.4. Bảng shell_history

Sử dụng lệnh `select * from shell_history;` để hiển thị các lệnh được thực hiện bởi người dùng.

```
+-----+------+----------------------------------+---------------------+
| uid | time | command                          | history_file        |
+-----+------+----------------------------------+---------------------+
| 0   | 0    | yum update -y                    | /root/.bash_history |
| 0   | 0    | ip a                             | /root/.bash_history |
| 0   | 0    | yum update -y                    | /root/.bash_history |
| 0   | 0    | hostnamectl set-hostname c7srv04 | /root/.bash_history |
| 0   | 0    | ip a                             | /root/.bash_history |
| 0   | 0    | hostnamectl set-hostname c7srv04 | /root/.bash_history |
| 0   | 0    | vi /etc/hosts                    | /root/.bash_history |
| 0   | 0    | ip a                             | /root/.bash_history |
| 0   | 0    | yum update -y                    | /root/.bash_history |
| 0   | 0    | ip a                             | /root/.bash_history |
+-----+------+----------------------------------+---------------------+
```

### 3.5 Kết hợp bảng user và bảng shell_history 

Ta có thể join các bảng lại để cho ra một kết quả như ý, trong hướng dẫn này sẽ sử dụng bảng `shell_history` và bảng `user` để cho biết các lệnh được thực hiện bởi user nào. Cú pháp sẽ như sau:

```
SELECT * FROM shell_history WHERE shell_history.uid IN (SELECT uid FROM users);
```

Ta sẽ có kết quả tương tự như bên dưới nếu có nhiều user thực hiện các lệnh

```
+------+------+-----------------------------------------------------------------------------------+----------------------------+
| uid  | time | command                                                                           | history_file               |
+------+------+-----------------------------------------------------------------------------------+----------------------------+
| 0    | 0    | ip a                                                                              | /root/.bash_history        |
| 11   | 0    | timedatectl                                                                       | /root/.bash_history        |
| 11   | 0    | vi /etc/chrony.conf                                                               | /root/.bash_history        |
| 11   | 0    | systemctl restart chronyd                                                         | /root/.bash_history        |
| 11   | 0    | chronyc sources                                                                   | /root/.bash_history        |
| 11   | 0    | timedatectl                                                                       | /root/.bash_history        |
| 11   | 0    | timedatectl set-local-rtc 0                                                       | /root/.bash_history        |
| 11   | 0    | ip a                                                                              | /root/.bash_history        |
| 11   | 0    | cat osqueryd.INFO                                                                 | /root/.bash_history        |
| 11   | 0    | tailf  osqueryd.INFO                                                              | /root/.bash_history        |
| 11   | 0    | ll -tr                                                                            | /root/.bash_history        |
| 11   | 0    | cat osqueryd.                                                                     | /root/.bash_history        |
| 11   | 0    | cat osqueryd.INFO                                                                 | /root/.bash_history        |
| 11   | 0    | tailf /var/log/osquery/osqueryd.results.log                                       | /root/.bash_history        |
| 11   | 0    | cat  /var/log/osquery/osqueryd.results.log                                        | /root/.bash_history        |
| 11   | 0    | adduser congto                                                                    | /root/.bash_history        |
| 11   | 0    | passwd congto                                                                     | /root/.bash_history        |
| 11   | 0    | su - congto                                                                       | /root/.bash_history        |
| 11   | 0    | vi /etc/osquery/osquery.conf                                                      | /root/.bash_history        |
| 1000 | 0    | ping dantri.com                                                                   | /home/congto/.bash_history |
| 1000 | 0    | hostname                                                                          | /home/congto/.bash_history |
| 1000 | 0    | ls -alh                                                                           | /home/congto/.bash_history |
| 1000 | 0    | cat .bashrc                                                                       | /home/congto/.bash_history |
| 1000 | 0    | exit                                                                              | /home/congto/.bash_history |
| 1000 | 0    | osqueryi                                                                          | /home/congto/.bash_history |
| 1000 | 0    | ll -tr /var/log/osquery/                                                          | /home/congto/.bash_history |
| 1000 | 0    | ping hocchudong.com                                                               | /home/congto/.bash_history |
| 1000 | 0    | exit                                                                              | /home/congto/.bash_history |
+------+------+-----------------------------------------------------------------------------------+----------------------------+

```

