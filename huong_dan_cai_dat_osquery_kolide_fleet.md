# Hướng dẫn cài đặt osquery, kolide fleet

## Mô hình


## IP Planning

```
GraylogSrv		: CentOS 7				: Graylog
KolideFleetSrv 	: CentOS 7 				: KolideFleet  
OsqueryClient1 	: CentOS 7  			: osquery 4.0.2
OsqueryClient2 	: ubuntu 18.04 			: osquery 4.0.2
OsqueryClient3	: CentOS 6 				: osquery 4.0.2
OsqueryClient4	: Windows server 2012	: osquery 4.0.2
```


# Bước cài đặt

## Thiết lập IP, hostname và các môi trường khác

- Đặt IP tĩnh cho máy cài osquery theo IP Planning.

```

```

## Cài đặt các gói bổ trợ

- Cập nhật OS và cài đặt các gói bổ trợ.

```
yum update -y && yum install yum-utils wget byobu -y
```

## Cài đặt và cấu hình osquery trên `OsqueryClient1`

- Thực hiện cài đặt osquery

```
curl -L https://pkg.osquery.io/rpm/GPG | tee /etc/pki/rpm-gpg/RPM-GPG-KEY-osquery
yum-config-manager --add-repo https://pkg.osquery.io/rpm/osquery-s3-rpm.repo
yum-config-manager --enable osquery-s3-rpm
yum install osquery -y
cp /usr/share/osquery/osquery.example.conf /etc/osquery/osquery.conf
```

- Cấu hình osquery để lập lịch thu thập dữ liệu



## Cài đặt kolide fleet trên server `KolideFleetSrv`

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



