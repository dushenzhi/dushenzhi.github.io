---
layout:     post
title:      Python读取Hadoop Sequence File
subtitle:   一个用python读取SequenceFile示例
date:       2018-12-07
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - python
    - hadoop
    - Sequence File
    - 读文件
---

示例代码如下：

```python
import sys

from hadoop.io import SequenceFile

if __name__ == '__main__':
    if len(sys.argv) < 3:
        print 'usage: SequenceFileReader <filename> <output>'
    else:
        reader = SequenceFile.Reader(sys.argv[1])

    key_class = reader.getKeyClass()
    value_class = reader.getValueClass()

    key = key_class()
    value = value_class()

    #reader.sync(4042)
    position = reader.getPosition()
    f = open(sys.argv[2],'w')
    while reader.next(key, value):
        f.write(value.toString()+'\n')
    reader.close()
    f.close()
```
参考链接:[https://stackoverflow.com/questions/33684625/how-to-load-data-from-hdfs-sequencefile-in-python](https://stackoverflow.com/questions/33684625/how-to-load-data-from-hdfs-sequencefile-in-python)

python读取更多hadoop文件格式，如:ArrayFile,MapFile,SetFile等，可以参见:[https://github.com/matteobertozzi/Hadoop](https://github.com/matteobertozzi/Hadoop)

