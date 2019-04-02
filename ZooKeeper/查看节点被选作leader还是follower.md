**查看结点被选作leader或者follower** 

```
zkServer.sh status
```

![image-20190402100539754](../images/image-20190402100539754.png)

或者使用

```
echo stat|nc 127.0.0.1 2181 
```

![image-20190402100602645](../images/image-20190402100602645.png)

