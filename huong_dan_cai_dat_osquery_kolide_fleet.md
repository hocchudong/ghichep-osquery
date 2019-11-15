# Hướng dẫn cài đặt osquery, kolide fleet

## 1. Mô hình

Mô hình triển khai hệ thống osquery và công cụ quản lý kolide fleet

![OSquery topo](https://image.prntscr.com/image/1KLf7ZviQKqRamkP_WJXBw.png)

## 2. IP Planning

IP Planning cho các máy 

![OSquery IP Planning](https://image.prntscr.com/image/QxiNdFXUSyepHmrWirY09A.png)


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
echo "127.0.0.1 localhost osqueryclient01" > /etc/hosts
echo "192.168.80.142 osqueryclient01" >> /etc/hosts
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

## 5. Cài đặt kolide fleet trên server `KolideFleetSrv`

- Kolid fleet yêu cầu các phần mềm bổ trợ, trong bước này cần cài đặt chúng trước khi cài đặt kolide fleet.


### Thực hiện cài đặt mysql

```
wget https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
rpm -ivh mysql80-community-release-el7-3.noarch.rpm
yum update

yum install -y mysql-community-server.x86_64 mysql-community-client.x86_64 -y
```

- Kiểm tra file log của mysql 8.0 xem mật khẩu được sinh ngẫu nhiên à gì.
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

- Trong hướng dẫn trên, tôi đã thiết lập mật khẩu cho tài khoản `root` là `Welcome123+`.


### Cài đặt redis

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



