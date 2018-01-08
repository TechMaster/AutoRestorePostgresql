Viết một shell script tự động hoá việc:
1- Tải về database back up dạng tar file dùng curl
2- Kiểm tra đã có container nào có tên theo yêu cầu và dùng image postgresql:latest
3- Nếu chưa có thì tạo mới dùng docker run
4- Nếu có rồi thì docker start
5- Sau khi container đã chạy thì chờ 5 giây cho nó ổn định (warm up)
6- Kết nối vào postgresql trong container tạo ra cơ sở dữ liệu có tên theo yêu cầu
7- Khôi phục dữ liệu sao lưu từ file tar vào cơ sở dữ liệu
8- Thử truy vấn