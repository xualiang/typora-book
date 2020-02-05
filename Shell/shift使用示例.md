- 例1： 

- ```bash
  [root@361way bash]$ cat shift.sh
  #!/bin/bash
  until [ $# -eq 0 ]
  do
    echo "第一个参数为: $1 参数个数为: $#"
    shift
  done
  
  [root@361way bash]$ sh shift.sh 1 2 3 4
  第一个参数为: 1 参数个数为: 4
  第一个参数为: 2 参数个数为: 3
  第一个参数为: 3 参数个数为: 2
  第一个参数为: 4 参数个数为: 1
  ```

- 例2：

- ```sh
  [root@361way bash]$ cat shift1.sh
  until [ -z "$1" ]   # Until all parameters used up
  do
    echo "$@"
    shift
  done
  
  [root@361way bash]$ sh shift1.sh 1 2 3 4 5 6
  1 2 3 4 5 6
  2 3 4 5 6
  3 4 5 6
  4 5 6
  5 6
  6 
  ```