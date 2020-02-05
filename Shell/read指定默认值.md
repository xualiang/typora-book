- 方法1：

- 5s 以后则 num =10 作为默认值。直接回车，也会传入一个默认值，num=10

- ```sh
  #!/bin/bash
  read -t 5 -p "请输入一个数：" num
  num=${num:-10}
  echo "num=$num"
  ```

- 

-  方法2  按回车提供一个默认值

- ```sh
  #!/bin/bash
  read -p "请输入一个数：" num
  if [ -z "${num}" ];then
    num=10
  fi
  echo "num      is $num"
  ```