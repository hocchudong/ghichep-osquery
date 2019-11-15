# Hướng dẫn cài đặt osquery, kolide fleet

## Mô hình
## IP Planning

# Bước cài đặt

## Thiết lập IP, hostname và các môi trường khác

- Đặt IP tĩnh cho máy cài Osquery

```

```

## Cài đặt các gói bổ trợ

- Cập nhật OS và cài đặt các gói bổ trợ.

```
yum update -y && yum install yum-utils wget byobu -y
```


## Cài đặt và cấu hình osquery

- Thực hiện cài đặt osquery

```
curl -L https://pkg.osquery.io/rpm/GPG | tee /etc/pki/rpm-gpg/RPM-GPG-KEY-osquery
yum-config-manager --add-repo https://pkg.osquery.io/rpm/osquery-s3-rpm.repo
yum-config-manager --enable osquery-s3-rpm
yum install osquery -y
cp /usr/share/osquery/osquery.example.conf /etc/osquery/osquery.conf
```

- Cấu hình osquery để lập lịch thu thập dữ liệu



## Cài đặt kolide fleet

- Kolid fleet yêu cầu các phần mềm bổ trợ, trong bước này cần cài đặt chúng trước khi cài đặt kolide fleet.
- Thực hiện cài đặt mysql

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




