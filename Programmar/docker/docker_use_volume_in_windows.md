# docker -v挂载windows路径

## 正文

以nginx为例
```
docker run -d -p 80:80 --name file-server -v /d/workspace/nginx:/etc/nginx nginx:latest
```

windows目录的挂载也可以使用和linux一样的格式，`/盘符/对应路径`