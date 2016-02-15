---
date: 2016-02-15T13:06:32+08:00
description: ""
disqus_identifier: "20160001"
slug: ""
tags:
- liunx
- shell
- NAS
title: Liunx查找删除历史备份文件批命令
topics:
- 运维
---

NAS作为数据备份中心，但需要经常清理历史备份文件，仅需要保留一部分备份文件即可。由于NAS是改造后的Liunx 操作系统，故有些命令无法直接执行，因此独立写了一个脚本来搜索历史备份文件，并记录待删除文件信息，自动生成脚本删除文件。

###  Liunx查找删除历史备份文件批命令脚本

具体执行脚本如下：

```bash
#!/bin/bash

NOW=$(date +"%Y%m%d%M%S")
FILE="NeedDeleteFile_$NOW.sh"


# 遍历文件，写入文件中
function findFile(){ 
	# 记录查找目录
	echo "# find from $1 -mtime $2" >> $FILE
	
	#遍历目录，将指定日期的
    find $1 -mtime $2 -type f \( -iname "*.bak" -o -iname "*.log" -o -iname "*.zip" -o -iname "*.rar" \)  -print0 | while IFS= read -r -d $'\0' line; 
    do
        echo  "rm \"$line\";" >> $FILE
    done
}

# 标记sh文件
echo "#!/bin/bash" >> $FILE

# 记录创建日期
echo "# $NOW create file." >>$FILE

# 数据中心，清理30天前备份数据
findFile "/share/MD0_DATA/DC/" "+30"
# 营销平台，清理10天前备份数据
findFile "/share/MD0_DATA/YX/" "+10" 

# 执行后输出时间
echo "echo \"\$(date +%Y%m%d%M%S) DONE.\"" >> $FILE

#执行脚本
bash $FILE
```


### 脚本说明

脚本执行时，会在当前目录生成一个 `NeedDeleteFile_YYmmDDMMSS.sh`的脚本文件，再将搜索到的指定类型，指定日期前的文件依次生成 `rm`删除命令并`cat` 到自动脚本中。最后执行该脚本，执行删除。

这里当然也可以在变量文件时便删除文档，但我还是希望保证安全性，先统一写入脚本文件中，再执行脚本进行删除。好处是为了安全性，可以注释掉掉末尾的`bash $FILE`，而手工执行脚本。
