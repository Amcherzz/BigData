## windows WSL2操作

- 本地和windows文件切换

  ```
  # 在WSL2中访问windows D盘
  cd /mnt/d
  ```

- 将宿主机文件拷贝到容器内

  ```
  docker cp 要拷贝的文件路径 容器名：要拷贝到容器里面对应的路径
  ```




### hadoop中hadoop01节点操作

```
# 运行中的container先commit
docker commit -m'添加了hive' 3d03ab4fc90f  hadoop01:v1.1

# 删除原来运行的容器
docker rm -f 3d03ab4fc90f

# 重新将images启动为容器
sudo docker run -d --name hadoop01 --hostname hadoop01 --net mynetwork --ip 172.22.0.2 -P -p 9870:9870 -p 8088:8088 -p 19888:19888 -p 10000:10000  -p 3306:3306 --privileged  c1344ee82401 /usr/sbin/init

# 进入容器
sudo docker exec -it --privileged=true 2accebd56642 /bin/bash
```



