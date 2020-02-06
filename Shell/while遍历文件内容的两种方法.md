## 使用这种方式：

```sh
while read ip1 ip2 dp
do
  {

  }&
done < /tmp/node.list
```



## 而不要使用：

```sh
cat /tmp/node.list | while read ip1 ip2 dp
do
  {

  }&
done 
```

 

