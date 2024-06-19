# sonarqube v10.5.1

## Cài đặt Java
```bash
sudo apt install openjdk-17-jdk
```

## Cài đặt PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib
```
## Tạo một cơ sở dữ liệu và người dùng mới trong PostgreSQL
```bash
sudo -u postgres psql
CREATE DATABASE sonarqube;
CREATE USER sonaruser WITH ENCRYPTED PASSWORD '123456';
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonaruser;
\q
```

## Cài đặt sonarqube
```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip

unzip sonarqube-10.5.1.90531.zip

sudo mv sonarqube-10.5.1.90531 /opt/sonar
```
### Chỉnh sửa file cấu hình SonarQube
```bash
sudo vi /opt/sonar/conf/sonar.properties
```
Thêm cấu hình
```bash
# DATABASE
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
sonar.jdbc.username=sonaruser
sonar.jdbc.password=123456
```

## Tạo người dùng hệ thống mới cho SonarQube
```bash
sudo adduser --system --no-create-home --group --disabled-login sonarqube
```
## Thay đổi quyền sở hữu cho thư mục SonarQube
```bash
sudo chown sonarqube:sonarqube /opt/sonar -R

#sudo chown sonarqube:sonarqube sonar.sh
```

## Tăng bộ nhớ trên hệ thống để Elaticsearch hoạt động
```bash
sudo vi /etc/sysctl.conf
```
Thêm cấu hình
```bash
vm.max_map_count=524288
fs.file-max=131072
```

Tạo file 99-sonarqube.conf
```bash
sudo vi /etc/security/limits.d/99-sonarqube.conf
```
Thêm cấu hình vào file 99-sonarqube.conf
```bash
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```

## Chạy sonarqube
```bash
sudo -u sonarqube ./sonar.sh start
```

# Cấu hình Sonar Service
## Tạo file service
```bash
sudo nano /etc/systemd/system/sonarqube.service
```
Thêm cấu hình
```bash
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonar/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonar/bin/linux-x86-64/sonar.sh stop

User=sonarqube
Group=sonarqube
PermissionsStartOnly=true
Restart=always

StandardOutput=syslog
LimitNOFILE=131072
LimitNPROC=8192
TimeoutStartSec=5
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```
## Chạy SonarQube service
```bash
sudo systemctl start sonarqube
#sudo systemctl status sonarqube
```

# Thực hiện scanner
## Tải somar-scanner
```bash
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-6.0.0.4432-linux.zip

unzip sonar-scanner-cli-6.0.0.4432.zip

sudo mv sonar-scanner-cli-6.0.0.4432 /opt/sonarscanner

cd /opt/sonarscanner
```
## Cấu hình scanner
```bash
sudo nano conf/sonar-scanner.properties
```
Thêm cấu hình
```bash
sonar.host.url=http://localhost:9000
```

# Thực hiện chạy scanner
Đăng nhập sonarqube (admin:admin) thực hiện tạo project mới để lấy token. Thực hiện chạy lệnh tùy cấu hình project. VD:
```bash
~/opt/sonarscanner/bin/sonar-scanner   -Dsonar.projectKey=monti   -Dsonar.sources=.   -Dsonar.host.url=http://localhost:9000   -Dsonar.token=sqp_a48123dda49081fce58cfd5f44d277cbf7a3df7f
```