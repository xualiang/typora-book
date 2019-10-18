```shell
#!/bin/bash

imgDir=/tmp/downImgs
mkdir -p $imgDir
cd $imgDir

while read line
do
  vsdId=`echo $line | awk -F'vsdId":"' '{print $2}' | awk -F'","eventId' '{print $1}'`
  carPosition=`echo $line | awk -F'carPosition":"' '{print $2}' | awk -F'","driverPosition' '{print $1}'`
  imageUrl=`echo $line | awk -F'imageUrl":"' '{print $2}'|awk -F'","deviceType' '{print $1}'`
  imageName=`echo $imageUrl | awk -F'/' '{print $NF}'`

  wget $imageUrl

  mv $imageName ${vsdId}_渣土车_${carPosition}.jpg
done < /tmp/down-trunck.log
```

