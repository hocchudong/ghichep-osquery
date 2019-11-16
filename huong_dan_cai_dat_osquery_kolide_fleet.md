# Hướng dẫn cài đặt osquery, kolide fleet

## 1. Mô hình

Mô hình triển khai hệ thống osquery và công cụ quản lý kolide fleet

![OSquery topo](https://image.prntscr.com/image/1KLf7ZviQKqRamkP_WJXBw.png)

## 2. IP Planning

IP Planning cho các máy 

![OSquery IP Planning](https://image.prntscr.com/image/ly9beWf2SceieCq85xW9YA.png)


## 3. Thực hiện cài đặt trên các node client

### 3.1. Cài đặt trên `OsqueryClient1`

#### 3.1.1 Chuẩn bị cài đặt trên `OsqueryClient1`

- Thực hiện cập nhật os và cài các gói bổ trợ.
```
yum update -y && yum install yum-utils wget byobu -y
```

- Thiết lập hostname 

```
hostnamectl set-hostname osqueryclient01
bash 
```

```
echo "192.168.80.142 osqueryclient01" >> /etc/hosts
echo "127.0.0.1 localhost osqueryclient01" > /etc/hosts
```

- Thiết lập IP


#### 3.1.2. Cài đặt và cấu hình osquery trên `OsqueryClient1`

- Thực hiện cài đặt osquery

```
curl -L https://pkg.osquery.io/rpm/GPG | tee /etc/pki/rpm-gpg/RPM-GPG-KEY-osquery
yum-config-manager --add-repo https://pkg.osquery.io/rpm/osquery-s3-rpm.repo
yum-config-manager --enable osquery-s3-rpm
yum install osquery -y
cp /usr/share/osquery/osquery.example.conf /etc/osquery/osquery.conf
```

Sửa cấu hình của file `/etc/osquery/osquery.conf` 

- Thay dòng 

```
//"logger_path": "/var/log/osquery",
``` 

- thành dòng 

```
"logger_path": "/var/log/osquery",
```

- Tìm dòng `"schedule": {` thêm đoạn cấu hình sau ngay bên dưới.

```
// for example, get CPU Time per 60 seconds
    "cpu_time": {
      "query": "SELECT * FROM cpu_time;",
      "interval": 60
    },
```

Mục tiêu của đoạn cấu hình trên là để lập lịch cho osquery thực hiện các câu truy vấn về dữ liệu ta muốn thu thập và ghi vào file log trong thư mục `/var/log/osquery`. Đoạn sửa có hình tương tự ảnh dưới

![cấu hình schedule](https://image.prntscr.com/image/JJ04vQvlS12yiec4_HcQLw.png)


## 4. Cài đặt trên `KolideFleetSrv`

### 4.1. Thiết lập ban đầu cho KolideFleetSrv

#### 4.1.1. Cấu hình cơ bản cho KolideFleetSrv

- Thao tác trên node `KolideFleetSrv`
- Thực hiện cập nhật os và cài các gói bổ trợ.
```
yum update -y && yum install yum-utils wget byobu unzip -y
```

- Thiết lập IP

- Thiết lập hostname 

    ```
    hostnamectl set-hostname KolideFleetSrv
    bash 
    ```

    ```
    echo "192.168.80.141 KolideFleetSrv" >> /etc/hosts
        echo "127.0.0.1 localhost KolideFleetSrv" > /etc/hosts

    ```

- Khởi động lại os 

    ```
    init 6
    ```


#### 4.1.2. Thực hiện cài đặt mysql
- Kolid fleet yêu cầu các phần mềm bổ trợ, trong bước này cần cài đặt chúng trước khi cài đặt kolide fleet.
    ```
    wget https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
    rpm -ivh mysql80-community-release-el7-3.noarch.rpm
    yum update -y 

    yum install -y mysql-community-server.x86_64 mysql-community-client.x86_64 -y
    ```

- Khởi động mysql 8.0
    ```
    systemctl start mysqld
    systemctl enable mysqld
    ```

- Đối với bản mysql 8.0, sau khi khởi động ta kiểm tra file log của mysql 8.0 xem mật khẩu được sinh ngẫu nhiên à gì.
    ```
    cat /var/log/mysqld.log    
    ```

- Ta có mật khẩu sinh ngẫu nhiên.
![Mật khẩu mysql](https://image.prntscr.com/image/ercAYtvHRNywDptycAq3DQ.png)

- Dùng mật khẩu này để đăng nhập vào mysql khi được hỏi ở lệnh dưới
    ```
    mysql -u root -p
    ```

- Sau khi đăng nhập được mysql, ta tiến hành tạo mật khẩu và database để chuẩn bị cài kolide. Lưu ý mật khẩu cần đủ độ phức tạp, bao gồm ký tự hoa, ký tự thường, ký tự số và ký tự đặc biệt.
    ```
    alter user "root"@"localhost" identified by "Welcome123+";   
    flush privileges;
    create database kolide;
    exit
    ```

- Trong hướng dẫn trên, tôi đã thiết lập mật khẩu cho tài khoản `root` là `Welcome123+`. Lưu ý dùng mật khẩu này ở bước dưới.


#### 4.1.3. Cài đặt redis

- Thực hiện cài đặt redis

```
wget http://download.redis.io/redis-stable.tar.gz
yum install -y gcc g++
tar -zxvf redis-stable.tar.gz && cd redis-stable
make && make install
cp redis.conf /etc/redis.conf
sed -i 's/daemonize no/daemonize yes/' /etc/redis.conf
redis-server /etc/redis.conf
```

#### 4.1.4. Thực hiện cài đặt kolide fleet

- Tải bộ cài của kolide fleet về 

```
mkdir /root/fleet
cd /root/fleet

wget https://github.com/kolide/fleet/releases/download/2.4.0/fleet.zip
```

- Giải nén file fleet.zip

```
yum install -y unzip

unzip fleet.zip
```

- Ta sẽ có các file dưới sau khi giải nén

```
[root@kolidefleetsrv fleet]# ls -alh
total 101M
drwxr-xr-x  5 root root   65 Nov 15 16:11 .
dr-xr-x---. 4 root root  255 Nov 15 16:08 ..
drwxr-xr-x  2 root root   35 Nov 13 06:11 darwin
-rw-r--r--  1 root root 101M Nov 13 06:17 fleet.zip
drwxr-xr-x  2 root root   35 Nov 13 06:11 linux
drwxr-xr-x  2 root root   43 Nov 13 06:11 windows
```

- Thực hiện copy các file cần thiết

```
cp linux/fleet /usr/bin/fleet
 
cp linux/fleetctl /usr/bin/fleetctl
```

- Thực hiện khai báo cấu hình cho fleet

```
/usr/bin/fleet prepare db --mysql_address=127.0.0.1:3306 --mysql_database=kolide --mysql_username=root --mysql_password=Welcome123+
```

- Kết quả xuất diện dòng `Migrations completed.` là đã hoàn thành bước trên, nếu sai xin kiểm tra lại các bước từ đầu :).

- Thực hiện khai báo cert cho kolide

```
openssl genrsa -out /etc/pki/tls/private/server.key 4096

openssl req -new -key /etc/pki/tls/private/server.key -out /etc/pki/tls/certs/server.csr
```

Khi được hỏi các thông tin, nhập tương tự như ảnh dưới.

![khai bao cert](https://image.prntscr.com/image/uR_lL3gLRcOJku0uegfNCA.png)

- Tạo file cert 

```
openssl x509 -req -days 3650 -in /etc/pki/tls/certs/server.csr -signkey /etc/pki/tls/private/server.key -out /etc/pki/tls/certs/server.cert
```

- Kết quả báo như bên dưới là thành công: 

```
Signature ok
subject=/C=VN/ST=HN/L=HN/O=HCD/OU=HCD/CN=KolideFleetSrv/emailAddress=tcvn1985@gmail.com
```

- Tạo thư mục log cho kolide

```
mkdir /var/log/kolide
```

- Thực hiện cài đặt kolide, hãy lưu ý chuỗi mật khẩu `Welcome123+`, nếu bạn thay chuỗi khác thì hãy sửa cho phù hợp.

```
/usr/bin/fleet serve \
    --mysql_address=127.0.0.1:3306 \
    --mysql_database=kolide \
    --mysql_username=root \
    --mysql_password=Welcome123+ \
    --redis_address=127.0.0.1:6379 \
    --server_cert=/etc/pki/tls/certs/server.cert \
    --server_key=/etc/pki/tls/private/server.key \
    --logging_json \
    --osquery_result_log_file=/var/log/kolide/osquery_result \
    --osquery_status_log_file=/var/log/kolide/osquery_status
```

- Kết quả của lệnh trên trả về như sau:

```
{"DEPRECATED":"use filesystem.status_log_file.","level":"info","msg":"using osquery.status_log_file value for filesystem.status_log_file","ts":"2019-11-15T09:18:41.933982248Z"}
{"DEPRECATED":"use filesystem.result_log_file.","level":"info","msg":"using osquery.result_log_file value for filesystem.result_log_file","ts":"2019-11-15T09:18:41.934195463Z"}
################################################################################
# ERROR:
#   A value must be supplied for --auth_jwt_key. This value is used to create
#   session tokens for users.
#
#   Consider using the following randomly generated key:
#   mbvPQMslkWFP7rT+5Hb8MUuMs7SJXAnl
################################################################################
```

Sửa dụng chuỗi `mbvPQMslkWFP7rT+5Hb8MUuMs7SJXAnl` để khai báo cho tùy chọn `--auth_jwt_key` khi thực hiện cài đặt kolide. Bước ở trên chỉ là thao tác để lấy chuỗi `--auth_jwt_key`, ta sẽ thực hiện cài đặt kolide với lệnh đầy đủ như sau: 

```
/usr/bin/fleet serve \
    --mysql_address=127.0.0.1:3306 \
    --mysql_database=kolide \
    --mysql_username=root \
    --mysql_password=Welcome123+ \
    --redis_address=127.0.0.1:6379 \
    --server_cert=/etc/pki/tls/certs/server.cert \
    --server_key=/etc/pki/tls/private/server.key \
    --logging_json \
    --osquery_result_log_file=/var/log/kolide/osquery_result \
    --osquery_status_log_file=/var/log/kolide/osquery_status \
    --auth_jwt_key=mbvPQMslkWFP7rT+5Hb8MUuMs7SJXAnl
```

Lưu ý: Nhớ thay đúng chuỗi bạn nhận được và chuỗi mật khẩu đúng với mật khẩu bạn nhập.

- Sau khi thực hiện lệnh trên, màn hình sẽ có kết quả như bên dưới. Lúc này mở trình duyệt để truy cập vào trang quản trị của kolide

![Man hinh quan tri kolide fleet](https://image.prntscr.com/image/sIyUJv3bQm_mYkZDLJZ8ow.png)

Truy cập vào địa chỉ https://192.168.80.141:8080 và nhập đầy đủ các thông tin setup ban đầu như hình ảnh dưới.

![Khai bao kolide](https://image.prntscr.com/image/5J5yN83gSPGU8S3v4ZgrbA.png)

- Sau khi chọn submit ta sẽ chuyển sang màn hình khai báo tiếp theo.

![Khai bao kolide](https://image.prntscr.com/image/e3H-9w1mS9K1Uyv8YAYusA.png)

- Tiếp tục chọn submit ta sẽ có mà hình dưới

![Khai bao kolide](https://image.prntscr.com/image/OPTN6xJ1RxW_LSFXWfsJwg.png)

- Cuối cùng, ta sẽ có màn hình chào mừng 

![Khai bao kolide](https://image.prntscr.com/image/Mfca1CUORJyd4EGckX4Ttg.png)


- Chọn finish ta vào màn hình quản lý như bên dưới

![Khai bao kolide](https://image.prntscr.com/image/-iZHPHScSZe1ij8R6lqj7Q.png)

Ta sẽ chuyển sang bước add các client cài đặt osquery


### 4.2. Thực hiện khai báo các osquery client.

#### 4.2.1. Lấy file `Osquery Enroll Secret` và `Fleet Certificate`

- Lựa chọn tab `ADD HOST` trên giao diện

![add host](https://image.prntscr.com/image/6S-5XsTuQXWMfbQCwJeFEQ.png)


- Lựa chọn biểu tượng copy chuỗi `Enroll Secret` và lưu lại để dùng cho bước tiếp.

![add host](https://image.prntscr.com/image/kUoNg7gcS9SPhxxFvhl-3w.png)


- Lựa chọn biểu tượng certificate để lấy file key. File này sẽ được dùng để copy vào thư mục quy định của các máy client.

![add host](https://image.prntscr.com/image/zcYxtUhLTemXMa_Ti3FlTQ.png)

Tới đây đã hoàn thành các bước để có file key và chuỗi `Enroll Secret`. Chuyển sang node client để thực hiện các bước tiếp theo.

#### 4.2.2 Thực hiện khai báo key, chuỗi Enroll Secret trên client

- SSH với quyền `root` vào trong máy client và thực hiện các khai báo trước khi được add lên Kolide fleet server.

- Lưu ý: Đối với hướng dẫn này, trước mắt cần thực hiện stop `osqueryd`, thực hiện bằng lệnh.
    ```
    systemctl stop osqueryd
    ```

- Ta kiểm tra lại trạng thái bằng lệnh dưới.
     ```
    systemctl status osqueryd
    ```

- Thực hiện khai báo chuỗi `Enroll Secret`
    ```
    echo '4WPAs52IOFJc8dPzyRqeLbHDx9rz38dC' > /var/osquery/enroll_secret 
    ```

- Copy file key lên thư mục /root/ của osqueryclient. Tên file trong hướng dẫn này là `192.168.80.141_8080.pem`

![add host](https://image.prntscr.com/image/Z7eypW_xQkiuaqJyxZ-CHA.png)


- Di chuyển file pem vào trong thư mục quy định

```
mv 192.168.80.141_8080.pem /var/osquery/server.pem
```

- Thực kiện kết nối client tới kolide fleet. Lưu ý khai báo IP của kolide fleet. Tại bước này ta có thể sử dụng `byobu` để truy trì phiên làm việc của osquery bởi vì chúng ta đang trong chế độ tương tác lệnh, nếu không dùng byobu khi tắt màn hình sẽ bị mất phiên.

```
byobu
```

- Kết nối osquery client với server fleet

```
/usr/bin/osqueryd \
    --enroll_secret_path=/var/osquery/enroll_secret \
    --tls_server_certs=/var/osquery/server.pem \
    --tls_hostname=192.168.80.141:8080 \
    --host_identifier=hostname \
    --enroll_tls_endpoint=/api/v1/osquery/enroll \
    --config_plugin=tls \
    --config_tls_endpoint=/api/v1/osquery/config \
    --config_tls_refresh=10 \
    --disable_distributed=false \
    --distributed_plugin=tls \
    --distributed_interval=3 \
    --distributed_tls_max_attempts=3 \
    --distributed_tls_read_endpoint=/api/v1/osquery/distributed/read \
    --distributed_tls_write_endpoint=/api/v1/osquery/distributed/write \
    --logger_plugin=tls \
    --logger_tls_endpoint=/api/v1/osquery/log \
    --logger_tls_period=10
```

- Mở giao diện web của máy KolideFleetSrv ta sẽ thấy xuất hiện osquery client như hình ảnh dưới.

![osqueryclient](https://image.prntscr.com/image/I4SK2nZGQzW9R7-8lKxKxw.png)

Việc add thêm các client khác sẽ lặp lại tương tự như bước trên. 

- Từ giao diện này ta có thể thực hiện các câu query. Sau hướng dẫn này các bạn sẽ chuyển sang các bước sử dụng giao diện của fleet.

