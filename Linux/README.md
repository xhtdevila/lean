| 基本语法                        | 功能描述                                             |
| :-----------------------    | :-------------------------------------------------- |
| ls -l >filename            | 列表内容写入到file中                                 |
| ls -l >>filename                   | 列表的内容追加到file末尾                                  |
|cat 文件1 > 文件2                    | 将文件1的内容覆盖到文件2中                              |
| echo “内容” > filename   | rclone绑定网盘时的名字                               |
| mv file1 file2 |移动1到2 |
|mv /root/a/a.txt /root/a/b.txt|将a.txt重命名b.txt|
| cp file1 file2 |复制1到2 |
|rm file |删除文件|
|rm /root/a/*|删除root下a目录所有文件|
|rm /root/a/xx* |删除路径下xx头的文件|
| $(date +%Y%m%d%H%M%S)       | 以时间创建文件夹作为备份目录 |
|>/dev/null 2>&1 | 忽略日志输出 |
