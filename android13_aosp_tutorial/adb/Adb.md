# ADB ENABLE FOR RPI4

### Enable

1.Sau khi boot được android lên các bạn chọn vào "Setting" 

2.Chọn "About Table"

3.Click vào "Build Number" 7 8 lần đến khi nào nó hiện là "you are now developer"

4.Trong System sẽ có mục "Developer options"

5.Click "Developer options" 

6.Tìm và bật "USB debugging".

7.Reboot lại rpi4


### Adb cmd
```
adb devices 
adb shell 
```


adb shell : để kết nối với rpi4 xem các thư mục của nó

adb devices : xem các thiết bị đang kết nối 