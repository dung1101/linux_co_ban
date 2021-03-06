# 16.Security
## 16.1 Sự khác biệt giữa su và sudo
|su|sudo|
|--|----|
|Để sử dụng su phải biết root passwd|	Để sử dụng sudo user phải là sudoers và phải nhập passwd của chính user để có thể chạy|
|Đăng nhập vào một lần có thể là mộ thứ user root có thể theo ý muốn|	Hệ thống sẽ quy định một khoảng thời gian sau khi gõ đúng pass user sử dụng sudo sẽ không phải nhập lại pass|
## 16.2 Password encryption & pasword aging
| |Mục tiêu, công dụng|Cơ chế sử dụng|Ngữ cảnh áp dụng|
|-|-------------------|--------------|----------------|
|Password encryption|	Mã hóa passwd|	Sử dụng hàm băm SHA-512|Áp dụng để mã hóa dữ liệu trong các giao thức SSH,SSL,TLS…|
|Password aging|	Nhắc nhở người dùng thay đổi pass sau một khoảng thời gian|	Sử dụng câu lênh change |Thay đổi pass theo chu kì nhằm tăng tính bảo mật	|

## 16.3 Pirvate key & Public key 
|Mục tiêu, công dụng|Cơ chế sử dụng|Ngữ cảnh áp dụng|
|-------------------|--------------|----------------|
|	Thay thế cho user passwd phục vụ cho việc đăng nhập server từ xa nâng cao tính bảo mật|Private key được lưu trữ tại máy client và public key được lưu trữ tại máy server|ứng dụng trong SSH|
### Tạo key
|  |Command | Descrption |
|--|--------|------------|
|Server|apt-get install openssh-server| Install ssh server|
||ufw allow 22| tường lửa cho phép truyền nhậ dữ liệu qua port 22|
||ssh-keygen –t sra|Tạo key sử dụng thuật toán RSA |
|||Private key:  /root/.ssh/id_rsa|
|||Public key : /root/user/.ssh/id_rsa.pub|
||chmod 700 ~/.ssh|Thay đổi quyền|
||chmod 600 ~/.ssh/id_rsa|	Thay đổi quyền| 
||cat id_rsa.pub>>~/.ssh/authorized_keys|Install pub key vào list|
||rm –rf ~/.ssh/id_rsa.pub|	xóa key|  
||scp ~/.ssh/id_rsa root@clientmachine:/root/.ssh|Gửi private key cho client|
||rm –rf ~/.ssh/id_rsa|	xóa key|
|client|apt-get install openssh-client	|Install ssh|
||ssh –i  ~/.ssh/id_rsa root@servermachine|	Đăng nhập |
## 16.4 SSL & TSL
| |Mục tiêu, công dụng|Cơ chế sử dụng|Ngữ cảnh áp dụng|
|-|-------------------|--------------|----------------|
|TLS(Transport Layer Security)&SSL(Secure Sockets Layer)|Bảo mật thông tin trên kênh truyền giữa server và client mà không phải bận tâm một bên thứ 3 có thể chặn và đọc được thông tin|Bao lại đường truyền và mã hóa thông tin truyền|sử dụng trong giao thức https|
### SSL
|Command | Descrption |
|--------|------------|
|apt-get install openssl|install ssl|
|a2enmod ssl|enable ssl module|
|service apache2 restart|khởi động lại apache2|
|mkdir /etc/apache2/ssl|tạo thư mục ssl|
|cd /etc/apache2/ssl|di chuyển vào thư mục ssl|
|openssl genrsa -aes256 -out ca_key.pem 4096|tạo Certification Authority key(RSA private key),sử dụng thuật toán aes256,độ dài 4096bit|
|openssl rsa -in ca-key.pem -text|xem key dưới dạng text|
|openssl req -new -x509 -days 3650 -key ca-key.pem -sha256 -out ca.pem|Tạo Certification Authority ,x509:chuẩn về chứng thực,thời hạn 3650 ngày,mã hóa sử dụng sha256|
|openssl x509 -in ca.pem -noout -text|xem CA dưới dạng text |
|openssl genrsa -out server-key.pem 4096|tạo private key cho server|
|HOST=abc|đặt tên host|
|openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr|tạo Certificate Signing Request sử dụng private key của server|
|openssl x509 -req -days 3650 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server.pem|tạo certificate file kết hợp với server_key.pem tạo thành một cặp|
|nano /etc/apache2/sites-available/default-ssl.conf|sửa SSLCertificateFile /etc/apache2/ssl/server.pem SSLCertificateKeyFile /etc/apache2/ssl/server_key.pem|
|sudo a2ensite default-ssl.conf|enable default-ssl.conf in virtual host|
|service apache2 restrart|khởi động lại apache2|
||sử dụng web browser https://ipserver:443 để test|
### The cloudflare TLS toolkit
|Command | Descrption |
|--------|------------|
|wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64|download|
|wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64|download|
|chmod +x cfssl_linux-amd64|thêm quyền thực thi|
|chmod +x cfssljson_linux-amd64|thêm quyền thực thi|
|mv cfssl_linux-amd64 /usr/local/bin/cfssl|di chuyển|
|mv cfssljson_linux-amd64 /usr/local/bin/cfssljson|di chuyển|
|nano ca-config.json|tạo file với nội dung: {"signing": {"default": {"expiry": "8760h"},"profiles": {"custom": {"usages": ["signing", "key encipherment", "server auth", "client auth"],"expiry": "8760h"}}}}|
|nano ca-crs.json|tạo file với nội dung {"CN": "NoverIT","key": {"algo": "rsa","size": 4096},"names": [{"C": "IT","ST": "Italy","L": "Milan","O": "My Own Certification Authority"}]}|
|cfssl gencert -initca ca-csr.json/cfssljson -bare ca|tạo ca.pem ca-key.pem|
|nano server-csr.json|tạo file với nội dung {"CN": "docker-engine","hosts": ["centos","10.10.10.1","127.0.0.1","localhost"],"key": {"algo": "rsa","size": 4096}}|
|cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=custom server-csr.json/cfssljson -bare server|tạo server.gem|
||sau đó thực hiện các bước như khi sử dụng ssl|
