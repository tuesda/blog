ubuntu 15.10不能启动mongodb的解决办法：

1. `sudo mongod --dbpath /var/lib/mongodb/ --journal`使用这条命令启动mongodb，然后新打开一个终端，输入`mongo`就进入mongodb shell界面