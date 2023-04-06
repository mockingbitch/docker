Dockerize ứng dụng Laravel ─ Biện pháp triển khai, quản lý version Laravel đơn giản

* Đặt vấn đề:
─ Một bạn fresher tên Tòn vừa mới join vào dự án, cậu ấy được add vào trong dự án và được leader yêu cầu clone project về và build trên máy cá nhân. Tuy nhiên, khi build thì Tòn lại fail liên tục. Mỗi lần build thì lại return lỗi version. Rồi cu cậu lại phải loay hoay đi tìm version của từng package để cài đặt trên máy. Sau 7749 lần download những thứ mà Tòn còn chẳng biết là thứ gì thì cu cậu cũng đã setup được project cho nó chạy mặc dù cài xong vẫn cấn cấn tay. Không biết khi nào nó fail tiếp.
─ Bẵng đi một thời gian, Tòn được add thêm vào 1 dự án khác. Tuy nhiên, dự án này lại phải cài đặt version php, mysql, laravel khác so với project cũ. Và lần này thì cậu ta hoang mang tột độ vì sợ sau khi cài project mới thì project cũ sẽ chết do sai version.
─ Lại tiếp tục vào 1 khoảng thời gian sau đó, Tòn lại gặp phải 1 ông khách khó tính. Ổng yêu cầu Tòn phải cài project trên máy tính cá nhân của ổng. Và lúc này cu cậu lại phải đi mò lại 7749 thứ kia để install.

Từ sau những lần gặp vấn đề đó, Tòn bắt đầu suy nghĩ. Nếu có 1 công cụ có thể "đóng gói" cái project của mình lại thì sẽ tiện lợi hơn rất nhiều. Mỗi lần di chuyển thì chỉ cần chuyển cả cái "gói" đó đi. Sau nhiều lần research thì Tòn đã biết đến sự tồn tại của 1 thứ tên là Docker.

* Docker là gì?
─ Về cơ bản thì Docker là người đóng tàu. Chúng ta sẽ phải thuê mấy thằng đóng tàu này vận chuyển các "thùng hàng" (container) lên 1 con tàu. Khi tàu đủ hàng thì sẽ được đi vận chuyển. Và khi cập bến đỗ mới thì về cơ bản nó đã có đầy đủ những thùng hàng cần có để sử dụng rồi.
Cơ sở lý thuyết của Dockerize Laravel:
Laravel là 1 framework php có thể tương tác với cơ sở dữ liệu SQL và phải cần có server để chạy. Vậy nên đối với một project Laravel thông thường sẽ phải bao gồm: SQL, PHP, Server.
Ở đây, mình sẽ sử dụng Mysql, PHP version 8.1, Server thì mình sẽ sử dụng Nginx

* Setup
─ Trước hết, chúng ta cần phải install Docker. Các bạn có thể xem link install <a href="https://docs.docker.com/engine/install/">tại đây</a>.
Về lý thuyết của docker thì các bạn có thể xem <a href="https://docs.docker.com/get─started/">tại đây </a>.
Ở bài này mình chỉ hướng dẫn cách setup docker cho project Laravel thôi nhé.
─ Các bạn sẽ không cần phải cài php, mysql hay nginx. Chỉ cần với docker, nó sẽ cân tất giúp bạn.
─ Đầu tiên, chúng ta cần clone project laravel về. Hoặc các bạn có thể setup 1 project Laravel mới.
─ Tiếp theo, tạo 1 folder docker trong thư mục Laravel và các thư mục con để chứa config của docker:
```
docker
    │── config
    │   │── nginx
    │   │   │── project1
    │   │       │── app.conf
    │   │── php─fpm
    │       │── custom.ini
    │── data
    │   │── .gitignore
    │   │── mysql
    │── images
    │   │── MYSQL.Dockerfile
    │   │── NGINX.Dockerfile
    │   │── PHP.Dockerfile
    │── logs
    │   │── nginx_log
    │   │   │── .gitignore
    │   │   │── access.log
    │   │   │── error.log
    │   │── php_fpm_log
    │── mysql
```

- Tiếp theo đó, chúng ta sẽ phải tạo file config của nginx trong thư mục docker/config/nginx , tại đây các bạn tạo 1 folder tên của project và 1 file app.conf nằm trong folder đó. Sau đó copy đoạn config sau vào file app.conf :

```
server {
    listen   80; ## listen for ipv4; this line is default and implied

    root /var/www/html/project1/public;
    index index.php index.html index.htm;

    server_name project1.docker www.project1.docker; ##Server name dùng để rewrite host thay vì sử dụng localhost

    gzip on;
    gzip_min_length 10240;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css text/xml application/json text/javascript application/x-javascript application/xml;
    gzip_disable "MSIE [1-6]\.";

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri $uri/ /index.php?$query_string;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

        location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$ {
                expires           5d;
        }

    location ~ /\. {
            log_not_found off;
            deny all;
    }

}
```

- Chúng ta cần phải viết Dockerfile để pull các images từ dockerhub về. Ở đây mình sẽ tách làm 3 files là MYSQL.Dockerfile, NGINX.Dockerfile, PHP.Dockerfile.
Tất cả đều được đặt ở thư mục docker/images.
+ MYSQL.Dockerfile:

```
FROM mysql:5.7 #ở đây mình sử dụng version mysql là 5.7, các bạn có thể thay đổi tùy dự án
```

+ NGINX.Dockerfile:

```
FROM nginx:latest #Đối với server nginx thì mình sẽ pull version mới nhất
```

+ PHP.Dockerfile:

```
FROM php:8.1-fpm

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

RUN apt-get update \
    && apt-get install -y \
    libzip-dev \
    zip \
    vim \
    unzip \
    git \
    curl \
    && docker-php-ext-install zip pdo pdo_mysql

# RUN chown -R www-data:www-data /var/www

# RUN chmod -R 777 /var/www/html/docker-socialnetwork/storage
# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions

RUN docker-php-ext-install pdo_mysql zip exif pcntl
# RUN docker-php-ext-configure gd --with-freetype --with-jpeg
# RUN docker-php-ext-install gd

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Change current user to www
USER www
```

- Và cuối cùng là tạo 1 file docker-compose.yml ở cùng cấp với folder docker vừa tạo (file docker-compose và .env nên cùng cấp với nhau):

```
version: "3"

services:
  nginx:
    dns:
      - 8.8.8.8
      - 4.4.4.4
    build:
      context: ./docker/images
      dockerfile: NGINX.Dockerfile # gọi tới file NGINX.Dockerfile trong thư mục docker/images để pull image Nginx:latest từ Dockerhub về
    working_dir: /var/www/html/project1 # tạo working_dir trên máy ảo
    container_name: nginx_project1 # đặt tên cho container
    ports:
      - "8080:80" # ánh xạ cổng 8080 trên máy thật vào cổng 80 trên máy ảo
    volumes:
      - .:/var/www/html/project1  # ánh xạ current folder vào thư mục project1 trên máy ảo
      - ./docker/logs/nginx_log:/var/log/nginx
      - ./docker/config/nginx/project1/app.conf:/etc/nginx/conf.d/project1.conf # ánh xạ file config của server nginx từ thư mục config vào máy ảo
      # - ./config/nginx/project2.conf:/etc/nginx/conf.d/project2.conf
    links:
      - php
      - mysql
    networks:
      - app-network
  php:
    build:
      context: ./docker/images
      dockerfile: PHP.Dockerfile  # gọi tới file PHP.Dockerfile để pull image php-8.1-fpm về. Và run các câu lệnh, cài đặt các package tương tự như ở trên máy thật
    container_name: php_project1
    volumes:
      - .:/var/www/html/project1  # ánh xạ toàn bộ các thư mục vào máy ảo
      - ./docker/logs/php_fpm_log:/var/log/php-fpm # ánh xạ log
      - ./docker/config/php-fpm/custom.ini:/usr/local/etc/php/conf.d/custom.ini # ánh xạ custom.ini, việc sử dụng method post có thể bị limit size. File này sẽ custom lại size cho nó
    networks:
      - app-network

  mysql:
    platform: linux/x86_64
    build:
      context: ./docker/images
      dockerfile: MYSQL.Dockerfile  #pull image mysql 5.6
    container_name: mysql_project1
    ports:
      - "3306:3306" #ánh xạ cổng 3306 từ máy thật vào máy ảo
    volumes:
      - ./docker/mysql:/docker-entrypoint-initdb.d  #volume data
    environment:  #các config của file .env được gọi ở đây. Chúng ta nên đặt file docker-compose và file .env cùng cấp để dễ gọi biến.
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      # MYSQL_USER: ${DB_USERNAME}
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    expose:
      - '3306'
    restart: unless-stopped
    networks:
      - app-network
networks:
  app-network:
    driver: bridge
```

- Trong file .env cũng cần phải sửa lại DB_HOST thành tên của service mysql trong docker-compose.yml . Ở trên mình đang để là mysql nên DB_HOST = mysql . Cần thêm cả DB_PASSWORD nữa nhé. :v
Build project:
Chúng ta sẽ sử dụng Docker Compose để build project này. Các bạn có thể sử dụng terminal/command prompt để chạy câu lệnh: ```docker compose up --build``` để build project. Sau khi build thành công, sẽ có 3 container được tạo ra theo container_name các bạn đặt ở trong docker-compose.yml . Các bạn có thể sử dụng terminal/command prompt hoặc Docker Desktop để quản lý chúng. Ở đây mình sẽ sử dụng terminal nhé. :v
Để check các container status chúng ta có câu lệnh ```docker ps -a```
Để thực thi/xử lý 1 container: ```docker exec -it + 3 ký tự đầu của container id + bash``` VD: ```docker exec -it 1ab bash```
Để check ip của container: ```docker inspect abc | grep "IP"``` hoặc nếu dùng command prompt thì ```docker inspect abc``` và bạn ctrl + F IP.
Nếu build thành công rồi thì về cơ bản là đã hoàn thành việc đóng gói. Tuy nhiên, để dễ dàng truy cập vào project thì chúng ta có thể rewrite host name.
- Ở Ubuntu/Linux/MacOs: Các bạn sử dụng sudo và edit file hosts ```sudo nano /etc/hosts``` Ở hộp thoại mở ra, các bạn thêm tên host các bạn mới config trong file app.conf lúc nãy vào:

```
127.0.0.1 project1.docker www.project1.docker
```
- Ở Window: chạy notepad với quyền admin, mở file host tại C:\Windows\System32\Drivers\etc và sửa tương tự như trên.

Sử dụng:
- Tất cả đã được setup. Chúng ta có thể truy cập vào project thông qua http://project1.docker:8080 . Lưu ý, port 8080 là cổng đã được ánh xạ ở trong file docker-compose.yml . Nếu cổng đã bị trùng thì có thể sửa lại ở docker-compose.yml và build lại.
- Để setup database bằng migration hoặc chạy composer update, ta cần phải exec container php_project1 bằng command: ```docker exec -it container_id bash``` . Và tiếp theo là có thể chạy lệnh ```php artisan migrate``` như thường.
Gợi ý: Để truy cập vào database dễ dàng hơn chúng ta có thể download adminer và cho thẳng vào folder project. Thêm config nginx tương tự và ánh xạ trong docker-compose.yml. Khi truy cập vào DB thì chỉ cần sử dụng IP của container mysql + username + password đã setup.

Kết luận:
Đây là 1 trong những cách Dockerize Laravel đơn giản đã được mình tóm gọn lại. Bạn có thể setup theo nhiều cách tùy theo dự án của bạn. Chúc các bạn may mắn ra khơi cùng Docker!