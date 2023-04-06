Dockerize ứng dụng Laravel - Biện pháp triển khai, quản lý version Laravel đơn giản

* Đặt vấn đề:
- Một bạn fresher tên Tòn vừa mới join vào dự án, cậu ấy được add vào trong dự án và được leader yêu cầu clone project về và build trên máy cá nhân. Tuy nhiên, khi build thì Tòn lại fail liên tục. Mỗi lần build thì lại return lỗi version. Rồi cu cậu lại phải loay hoay đi tìm version của từng package để cài đặt trên máy. Sau 7749 lần download những thứ mà Tòn còn chẳng biết là thứ gì thì cu cậu cũng đã setup được project cho nó chạy mặc dù cài xong vẫn cấn cấn tay. Không biết khi nào nó fail tiếp.
- Bẵng đi một thời gian, Tòn được add thêm vào 1 dự án khác. Tuy nhiên, dự án này lại phải cài đặt version php, mysql, laravel khác so với project cũ. Và lần này thì cậu ta hoang mang tột độ vì sợ sau khi cài project mới thì project cũ sẽ chết do sai version.
- Lại tiếp tục vào 1 khoảng thời gian sau đó, Tòn lại gặp phải 1 ông khách khó tính. Ổng yêu cầu Tòn phải cài project trên máy tính cá nhân của ổng. Và lúc này cu cậu lại phải đi mò lại 7749 thứ kia để install.

Từ sau những lần gặp vấn đề đó, Tòn bắt đầu suy nghĩ. Nếu có 1 công cụ có thể "đóng gói" cái project của mình lại thì sẽ tiện lợi hơn rất nhiều. Mỗi lần di chuyển thì chỉ cần chuyển cả cái "gói" đó đi. Sau nhiều lần research thì Tòn đã biết đến sự tồn tại của 1 thứ tên là Docker.

* Docker là gì?
- Về cơ bản thì Docker là người đóng tàu. Chúng ta sẽ phải thuê mấy thằng đóng tàu này vận chuyển các "thùng hàng" (container) lên 1 con tàu. Khi tàu đủ hàng thì sẽ được đi vận chuyển. Và khi cập bến đỗ mới thì về cơ bản nó đã có đầy đủ những thùng hàng cần có để sử dụng rồi.
Cơ sở lý thuyết của Dockerize Laravel:
Laravel là 1 framework php có thể tương tác với cơ sở dữ liệu SQL và phải cần có server để chạy. Vậy nên đối với một project Laravel thông thường sẽ phải bao gồm: SQL, PHP, Server.
Ở đây, mình sẽ sử dụng Mysql, PHP version 8.1, Server thì mình sẽ sử dụng Nginx

* Setup
- Trước hết, chúng ta cần phải install Docker. Các bạn có thể xem link install <a href="https://docs.docker.com/engine/install/">tại đây</a>.
Về lý thuyết của docker thì các bạn có thể xem <a href="https://docs.docker.com/get-started/">tại đây </a>.
Ở bài này mình chỉ hướng dẫn cách setup docker cho project Laravel thôi nhé.
- Các bạn sẽ không cần phải cài php, mysql hay nginx. Chỉ cần với docker, nó sẽ cân tất giúp bạn.
- Đầu tiên, chúng ta cần clone project laravel về. Hoặc các bạn có thể setup 1 project Laravel mới.
- Tiếp theo, tạo 1 folder docker và các thư mục con để chứa config của docker:
```
docker
        |-- config
        |   |-- nginx
        |   |   |-- socialnetwork
        |   |       |-- app.conf
        |   |-- php-fpm
        |       |-- custom.ini
        |-- data
        |   |-- .gitignore
        |   |-- mysql
        |-- images
        |   |-- MYSQL.Dockerfile
        |   |-- NGINX.Dockerfile
        |   |-- PHP.Dockerfile
        |-- logs
        |   |-- nginx_log
        |   |   |-- .gitignore
        |   |   |-- access.log
        |   |   |-- error.log
        |   |-- php_fpm_log
        |-- mysql
```