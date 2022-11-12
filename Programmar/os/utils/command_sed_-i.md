# sed -i 命令举例

## 举例

```shell
# 对每行匹配到的第一个字符串进行替换
sed -i 's/原字符串/新字符串/' ab.txt 
 
# 对全局匹配上的所有字符串进行替换
sed -i 's/原字符串/新字符串/g' ab.txt 
 
# 删除所有匹配到字符串的行
sed -i '/匹配字符串/d'  ab.txt  
 
# 特定字符串的行后插入新行
sed -i '/特定字符串/a 新行字符串' ab.txt 
 
# 特定字符串的行前插入新行
sed -i '/特定字符串/i 新行字符串' ab.txt
 
# 把匹配行中的某个字符串替换为目标字符串
sed -i '/匹配字符串/s/源字符串/目标字符串/g' ab.txt
 
# 在文件ab.txt中的末行之后，添加bye
sed -i '$a bye' ab.txt   
 
# 对于文件第3行，把匹配上的所有字符串进行替换
sed -i '3s/原字符串/新字符串/g' ab.txt 

# file的[N,M]行都被删除
sed -i 'N,Md' filename 

# 第n行前添加x内容（换行）
sed -i 'ni\x' file.txt 

# 第n行后添加x内容（换行）
sed -i 'na\x' file.txt 

# 匹配m字符的行前面添加x内容
sed -i '/m/i\x' file.txt 

# 匹配m字符的行后面添加x内容
sed -i '/m/a\x' file.txt 
```

## 参考文献

1. [sed -i 命令详解](https://blog.csdn.net/qq_42767455/article/details/104180726)
1. [linux中 删除指定行多行sed命令](https://blog.csdn.net/han_cui/article/details/126604143)