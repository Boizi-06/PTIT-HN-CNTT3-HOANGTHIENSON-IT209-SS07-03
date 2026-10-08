# Bài 3: Thiết lập Cơ sở dữ liệu và Tự cấu hình dịch vụ Systemd cho Spring Boot

## 1. Thông tin môi trường

- **OS:** Ubuntu 22.04.5 LTS
- **Java:** OpenJDK 21
- **Maven:** 3.6.3
- **MySQL:** 8.0
- **Spring Boot:** 3.4.5
- **Application Port:** 8082
- **Application JAR:** `/opt/spring-app/app.jar`

---

## 2. Thiết lập MySQL

### Tạo Database

```sql
CREATE DATABASE springboot_db;
```

### Tạo User

```sql
CREATE USER 'spring-admin'@'localhost'
IDENTIFIED BY 'SpringSecure@123';
```

### Cấp quyền

```sql
GRANT ALL PRIVILEGES ON springboot_db.*
TO 'spring-admin'@'localhost';

FLUSH PRIVILEGES;
```

Kiểm tra quyền:

```sql
SHOW GRANTS FOR 'spring-admin'@'localhost';
```

Kết quả xác nhận user `spring-admin` có toàn quyền trên database `springboot_db`.

---

## 3. Tạo System User

Tạo user `spring-runner` không cho phép đăng nhập shell:

```bash
useradd --system --no-create-home \
--shell /usr/sbin/nologin spring-runner
```

Kiểm tra:

```bash
getent passwd spring-runner
```

Kết quả:

```text
spring-runner:x:997:997::/home/spring-runner:/usr/sbin/nologin
```

User được sử dụng để chạy ứng dụng Spring Boot và không có quyền đăng nhập shell thông thường.

---

## 4. Build và triển khai Spring Boot

Application được build thành:

```text
app.jar
```

Sau đó triển khai vào:

```text
/opt/spring-app/app.jar
```

Phân quyền cho user chạy ứng dụng:

```bash
chown -R spring-runner:spring-runner /opt/spring-app
```

Kiểm tra:

```bash
ls -lh /opt/spring-app/
```

Kết quả xác nhận file `app.jar` thuộc quyền sở hữu của `spring-runner`.

---

## 5. Cấu hình Spring Boot

Ứng dụng được cấu hình chạy trên port `8082`.

Database sử dụng:

```text
Database: springboot_db
Username: spring-admin
Host: localhost
Port: 3306
```

Application JAR:

```text
/opt/spring-app/app.jar
```

---

## 6. Cấu hình Systemd

File service:

```text
/etc/systemd/system/spring-app.service
```

Nội dung:

```ini
[Unit]
Description=Spring Boot Application
After=network.target mysql.service
Requires=mysql.service

[Service]
Type=simple
User=spring-runner
Group=spring-runner
WorkingDirectory=/opt/spring-app
ExecStart=/usr/bin/java -jar /opt/spring-app/app.jar
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Sau khi tạo service:

```bash
systemctl daemon-reload
systemctl enable spring-app.service
systemctl start spring-app.service
```

---

## 7. Kiểm tra trạng thái Systemd

Lệnh kiểm tra:

```bash
systemctl status spring-app.service
```

Kết quả:

```text
Loaded: loaded (/etc/systemd/system/spring-app.service; enabled)
Active: active (running)
Main PID: 15172 (java)
```

Điều này xác nhận Spring Boot đang được Systemd quản lý và service đang hoạt động.

---

## 8. Kiểm tra Port 8082

Lệnh:

```bash
ss -tlnp | grep 8082
```

Kết quả:

```text
LISTEN 0 100 *:8082 *:* users:(("java",pid=15172,fd=9))
```

Port `8082` đang được Java lắng nghe.

---

## 9. Kiểm tra Process và User

Lệnh:

```bash
ps -ef | grep '[j]ava'
```

Kết quả:

```text
spring-runner ... /usr/bin/java -jar /opt/spring-app/app.jar
```

Điều này xác nhận ứng dụng được chạy bởi user `spring-runner`.

---

## 10. Kiểm tra quyền đăng nhập của spring-runner

Lệnh:

```bash
getent passwd spring-runner
```

Kết quả:

```text
spring-runner:x:997:997::/home/spring-runner:/usr/sbin/nologin
```

User `spring-runner` sử dụng `/usr/sbin/nologin`, do đó không được phép đăng nhập shell thông thường.

---

## 11. Kết quả

Các yêu cầu chính của bài đã được hoàn thành:

- [x] Tạo database `springboot_db`
- [x] Tạo user MySQL `spring-admin`
- [x] Cấp quyền cho `spring-admin`
- [x] Tạo system user `spring-runner`
- [x] Cấu hình `spring-runner` với `nologin`
- [x] Build Spring Boot thành `app.jar`
- [x] Triển khai `app.jar` tại `/opt/spring-app/`
- [x] Cấu hình ứng dụng chạy trên port `8082`
- [x] Tạo Systemd service thủ công
- [x] Chạy service bằng user `spring-runner`
- [x] Cấu hình `Restart=on-failure`
- [x] Cấu hình `RestartSec=10`
- [x] Enable service tự động khởi động cùng hệ thống
- [x] Kiểm tra service `active (running)`
- [x] Kiểm tra port `8082`

## 12. Minh chứng

Các ảnh chụp màn hình minh chứng gồm:

1. `systemctl status spring-app.service`
2. `ss -tlnp | grep 8082`
3. `ps -ef | grep '[j]ava'`
4. `getent passwd spring-runner`
5. `SHOW GRANTS FOR 'spring-admin'@'localhost';`