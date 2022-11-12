# Shell脚本确保目标文件中只有N组信息，确保信息不重复。

## 背景

1. 升级脚本中需要为`nginx.conf`中插入新的配置，此时我们需要保证**幂等性**。即无论脚本执行多少次，最终目标文件中也只有一组我们要插入的信息。
2. 降级过程中我们需要移除`nginx.conf`中插入的新配置，此时最终目标是文件中不会有本次升级相关的配置，但也不会影响到其他配置。
3. 不需要关注性能问题。

## 思路

1. 升级
    1. 先移除已存在的新配置，移除到没有对应配置为止。
    2. 插入新配置到文件中。
2. 降级
    1. 移除已存在的新配置，移除到没有对应配置为止。

分类后我们发现就2步操作，于是决定封装2个函数，对应两步操作。

1. 移除新配置
    1. 循环判断一个文件内对应配置是否存在。
    2. 存在则移除对应配置。
    3. 移除范围确定，不要影响其他配置。
2. 插入新配置
    1. 插入配置到文件的指定位置中。

## CODE

### 移除已存在的新配置

```shell
delete_nginx_conf_content(){
  while [[ `grep -E '^[ \t]*location[ \t]+\/fileclient/api/filetransfer/filePush[ \t]+{$' ${1} | wc -l` -ge 1 ]]; do
    startLine=`grep -nE '^[ \t]*location[ \t]+\/fileclient/api/filetransfer/filePush[ \t]+{$' ${1} | head -n 1 | cut -d ':' -f 1`
    lineNumberIncrement=`tail -n +${startLine} ${1} | grep -nE '^[ \t]*}$' | head -n 1 | cut -d ':' -f 1`
    endLine=`expr ${startLine} + ${lineNumberIncrement} - 1`
    sed -i "${startLine},${endLine}d" ${1}
  done
}
delete_nginx_conf_content /etc/nginx/nginx.conf
```

1. `grep -E`中的-E表示使用**扩展的正则表达式**。
2. 正则表达式`[ \t]`表示匹配空格或制表符。
3. `wc -l`可以衡量数据行数，大于等于1(`-ge 1`)时执行删除操作。
4. `grep -nE`会输出匹配字符串的行号
5. `cut -d ':' -f 1`表示使用冒号分割字符串，取数组中的第1个值，注意这里数组从1开始数。
6. `tail -n +${startLine}`从第startLine行开始输出数据。
7. `head -n 1`表示仅取数据中的第1行。
8. `expr`用以将字符串转换为数值进行计算。
9. `sed -i`对文件进行操作。
10. `sed -i "${startLine},${endLine}d"`表示删除文件的startLine行到endLine行，包括endLine行。
11. `${1}`值调用该函数时输入的第1个实参

### 插入新配置

```shell
insert_nginx_conf(){
  grep -qE '^[ \t]*location[ \t]+\/fileclient[ \t]+{$' ${1}
  if [[ $? -eq 0 ]]; then
        sed -i '/location \/fileclient {$/i \
        location /fileclient/api/filetransfer/filePush {\
            proxy_request_buffering off;\
            proxy_pass http://127.0.0.1:8082/api/filetransfer/filePush;\
            include /etc/nginx/conf.d/proxy.conf;\
        }\
      
        ' ${1}
  fi
}
insert_nginx_conf /etc/nginx/nginx.conf
```

1. 检查是否插入锚点是否存在，插入锚点为`location /fileclient {`
2. `grep -qE`中`-q`为不输出程序结果
3. 如果找到了插入锚点，则`$? -eq 0`的值为`true`
4. 在插入锚点前插入要增加的新配置