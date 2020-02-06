```sh
find /opt/kafka -maxdepth 10 -mindepth 1 -name "*.sh" -type f -exec grep -iH "Xmx1G" {} \;
```

