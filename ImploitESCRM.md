1. Cần phải có apache2 + mariaDB
> sudo apt install apache2

> sudo apt-get install mariadb-server mariadb-client

2. Sau khi tải xong apache2 và mariaDB thì khởi động 2 dịch vụ lên
> sudo systemctl start apache2.service

> sudo systemctl start mariadb.service

3. Tiếp theo sẽ bật máy chủ của mariaDB và tạo mật khẩu root.
> sudo mysql_secure_installation

![image](https://user-images.githubusercontent.com/63194321/156552290-1afb3f8a-d729-41e5-a8f5-36eefe3f00c3.png)


Sau khi tạo mật khẩu root của DB và không cho phép quyền truy cập root từ xa thì ta sẽ khởi động lại dịch vụ của mariaDB.
> sudo systemctl restart mysql.service

4. Tải PHP và các thư viện cần thiết.
> sudo apt install php libapache2-mod-php php-cli php-ldap php-zip php-curl php-mysql php-mbstring php-gd php-xml

5. Sau khi cài xong PHP thì ta sẽ chỉnh lại tệp mặc định của Apache2.
> sudo nano /etc/php/7.4/apache2/php.ini

Tìm và chỉnh sửa các thông số thành: 
```file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 30
zend.assertions = 0
display_errors = Off
max_input_vars = 1500
max_execution_time = 180
max_input_time = 180
post_max_size  = 20M
```
5. Tiến hành tạo database cho EspoCRM
- Khởi chạy mySql
> sudo mysql -u root -p
- Tạo một cơ sở dữ liệu là `espocrm`
> CREATE DATABASE espocrm;
- Tạo người dùng cơ sở dữ liệu có tên `espocrmuser` và mật khẩu mới
> CREATE USER 'espocrmuser'@'localhost' IDENTIFIED BY 'new_password_here';
- Cấp cho người dùng toàn quyền truy cập vào cơ sở dữ liệu
> GRANT ALL ON espocrm.* TO 'espocrmuser'@'localhost' IDENTIFIED BY 'user_password_here' WITH GRANT OPTION;
- Lưu các thay đổi và thoát.
> FLUSH PRIVILEGES;
>
>EXIT;
6. Tải và setup EspoCRM
- Truy cập vào `/var/www/html` để tải và giải nén tệp.  
> cd /tmp
- Tải EspoCRM bản mới nhất.
> wget https://www.espocrm.com/downloads/EspoCRM-7.0.9.zip

> unzip EspoCRM-7.0.9.zip
- Đặt quyền cho tệp vừa giải nén.
> sudo chown -R www-data:www-data /var/www/html/EspoCRM-7.0.9
> 
> sudo chmod -R 755 /var/www/html/EspoCRM-7.0.9
7. Cấu hình Apache2.
- Tạo 1 tệp cấu hình apache2 mới.
> sudo nano /etc/apache2/sites-available/espocrm.conf
```<VirtualHost *:80>
     ServerAdmin admin@example.com
     DocumentRoot /var/www/html/EspoCRM-7.0.9

     <Directory /var/www/html/EspoCRM-7.0.9/>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost> 
```
- Sau khi viết tệp cấu hình xong thì tắt modul cũa và bật modul vừa tạo lên.
> sudo a2ensite espocrm.conf && sudo a2dissite 000-default.conf 
- Bật chế độ rewrite
> sudo systemctl restart apache2.service 
- Khởi động lại Apache2.
> sudo systemctl restart apache2.service

8. Truy cập vào `http://localhost` để vào giao diện chính.
- Giao diện chính của trang web.
![image](https://user-images.githubusercontent.com/63194321/156557964-4c2b8a5d-ca61-487b-a870-bcc6f978f9b6.png)
- Điền vào thông tin tạo từ bước 5
![image](https://user-images.githubusercontent.com/63194321/156558327-4e7ab28a-9847-4268-9d31-63b01461daf7.png)
-Tiếp theo sẽ đến phần kiểm tra lại các cấu hình của PHP đã cài đặt.
![image](https://user-images.githubusercontent.com/63194321/156560110-97a48cac-7320-42a8-92ee-58998cf58375.png)
- họn install và đến phần cài đặt tài khoản admin.
![image](https://user-images.githubusercontent.com/63194321/156560283-031acec7-1aa2-4b3e-8e4b-be093087c4ef.png)
- Tếp sau là cài đặt thông tin hệ thống.
![image](https://user-images.githubusercontent.com/63194321/156560468-3426dd32-5f14-4cee-9076-74992e716da3.png)
- Và cuối cùng cài đặt thành công.
![image](https://user-images.githubusercontent.com/63194321/156560617-41b65216-f4e3-4686-93f0-1179b666150b.png)

END.
