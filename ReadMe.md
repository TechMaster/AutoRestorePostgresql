#Viết một shell script tự động hoá việc:
1. Tải về database back up dạng tar file dùng curl
2. Kiểm tra đã có container nào có tên theo yêu cầu và dùng image postgresql:latest
3. Nếu chưa có thì tạo mới dùng docker run
4. Nếu có rồi thì docker start
5. Sau khi container đã chạy thì chờ 5 giây cho nó ổn định (warm up)
6. Kết nối vào postgresql trong container tạo ra cơ sở dữ liệu có tên theo yêu cầu
7. Khôi phục dữ liệu sao lưu từ file tar vào cơ sở dữ liệu
8. Thử truy vấn

#Đây là source code
```shell
#!/bin/sh
postgres_password="123"
backupfile="dvdrental.tar"
container="db"
dbName="dvdrental"
postgres_image="postgres:latest"

if [ ! -f $backupfile ]; then
    echo "Download from robconery github"
    curl -O https://raw.githubusercontent.com/robconery/dvdrental/master/dvdrental.tar
    if [ ! -f dvdrental.tar ]; then
    	echo "Failed to dowload $backupfile"
    	exit
    else
    	echo "Downloading $backupfile completed"
    fi
fi

result=$(docker ps -a --format '{{.Names}}' | grep -w $container)
if [ $container != "$result" ]
then
	echo "Container '$container' is not found. docker run now!"
	docker run --name $container -e POSTGRES_PASSWORD=$postgres_password -d -p 5432:5432 $postgres_image
else
	echo "Container '$container' is found"
	found_image=$(docker ps -a --format '{{.Image}}' -f name=$container | grep -w $postgres_image)
	if [ $postgres_image != "$found_image" ]
	then
		echo "Base image of container $container is not $postgres_image. docker run to create new container"
		docker stop $container && docker rm $container
		docker run --name $container -e POSTGRES_PASSWORD=$postgres_password -d -p 5432:5432 $postgres_image
	else
		echo  "docker start $container"
		docker start $container
	fi
fi
echo "sleep 5 seconds for container warms up"
sleep 5
found_db=$(docker exec -it -u postgres $container psql -c "SELECT datname FROM pg_database;" | grep $dbName)
echo $found_db
if [ $found_db != "" ]
then
	echo "Found $dbName then drop it first"
	docker exec -it -u postgres $container dropdb $dbName	
fi
docker exec -it -u postgres $container createdb $dbName
docker exec -i -u postgres $container pg_restore -d $dbName < $backupfile
echo "show first 5 records in table actor"
docker exec -it -u postgres $container psql -d $dbName -c "table actor limit 5"
```