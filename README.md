# Raspbian BLE
> 使用開源套件：https://github.com/nettlep/gobbledegook </br>
> 相關套件安裝教學：https://b8807053.pixnet.net/blog/post/347553287-updating-bluez-on-raspberry-pi-%285.48%29 </br>
> dbus權限設定教學：https://stackoverflow.com/questions/43085699/update-dbus-on-raspberry-pi

提供TCP/IP socket服務，可直接由其他程序存取本地藍芽讀寫，詳請參閱ble-gatt-server_socket.cpp

## 安裝與設定
1.  安裝相關套件：
```
sudo apt-get update
sudo apt-get install libglib2.0-dev
sudo apt-get install libudev-dev
sudo apt-get install libical-dev
sudo apt-get install libreadline-dev
sudo apt-get install libbluetooth-dev
```
2. 啟用D-Bus權限</br>
先在pi目錄中建立以下檔案(作為dbus權限設定檔)</br>
檔案名稱：gobbledegook.conf</br>
檔案內容：</br>
```
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN" "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
  <policy user="root">
    <allow own="com.gobbledegook"/>
    <allow send_destination="com.gobbledegook"/>
    <allow send_destination="org.bluez"/>
  </policy>
  <policy at_console="true">
    <allow own="com.gobbledegook"/>
    <allow send_destination="com.gobbledegook"/>
    <allow send_destination="org.bluez"/>
  </policy>
  <policy context="default">
    <deny send_destination="com.gobbledegook"/>
  </policy>
</busconfig>
```
> com.gobbledegook為套件預設(standalone.cpp)的權限名稱</br>
> 如果編譯使用ble-gatt-server.cpp須將com.gobbledegook改為com.raspberry_ggk_ble</br>
> 只要有更新權限檔內容的動作，都須執行以下的Step.2相關步驟，以刷新權限

Step.2-1 將pi目錄中的gobbledegook.conf放至/etc/dbus-1/system.d中</br>
```
sudo cp gobbledegook.conf /etc/dbus-1/system.d/
```

Step.2-2 更新dbus設定
(過程中可能會重開機，出現錯誤是正常的，需記得上次執行到哪步，接著做就可以)
```
sudo systemctl stop dbus
sudo systemctl daemon-reload
sudo systemctl start dbus
```

3. 啟用藍芽
```
sudo btmgmt -i 0 power off
sudo btmgmt -i 0 le on
sudo btmgmt -i 0 connectable on
sudo btmgmt -i 0 advertising on
sudo btmgmt -i 0 power on
```

4. 建立及啟動範例程式
cd 至該套件包存放位置(需自行解壓縮) </br>
順利即啟動一個BLE server(如果無法啟動，可能是dbus權限沒設定好，請檢查權限檔)</br>
例如：cd gatt-server-master(放置於pi目錄下)</br>
```
./configure && make
sudo src/standalone -d
```

## 快速使用
> 請參照"安裝與設定"，先安裝相關套件，並測試套件是否能運行(standalone)

1. 壓縮檔解壓縮 -> 出現ble-gatt-server資料夾

2. 終端機進入ble-gatt-server資料夾內

3. 將ble-gatt-server中的gobbledegook.conf放至/etc/dbus-1/system.d中
sudo cp gobbledegook.conf /etc/dbus-1/system.d/

4. 參考安裝與設定，刷新D-Bus權限

5. 重啟完畢後，到ble-gatt-server資料夾內編譯
```
g++ -o bleServer ble-gatt-server.cpp libggk.a -lgobject-2.0 -lgio-2.0 -lglib-2.0 -lpthread
```

6. 執行
```
sudo ./bleServer
```

## 原始碼撰寫與編譯
1. 先到gatt-server套件的目錄中，輸入指令：
```
./configure
```

2. 更新Server的服務參數</br>
直接修改"套件目錄/src/Server.cpp"中的內容即可</br>
修改完畢後，回到套件目錄</br>
生成libggk.a (只在更新BLE Server設定時才需使用)</br>
例如：cd gatt-server-master(放置於pi目錄下)</br>
語法：make</br>
順利即生成新的libggk.a檔案

3. 編寫及編譯(自己的程式碼)</br>
原始碼編寫可參考套件中的standalone.cpp</br>
原始碼標頭需引入"套件目錄/include/Gobbledegook.h"這份檔案</br>
Setter是ble的接收器</br>
Getter是ble的發送器</br>
Notify透過UART送資料請用此特性</br>
ggkNofifyUpdatedCharacteristic("/com/gobbledegook/服務名稱/特性名稱");

編譯語法：
```
g++ -o [輸出檔名] [原始碼檔名.cpp] libggk.a -lgobject-2.0 -lgio-2.0 -lglib-2.0 -lpthread
```
"套件目錄/src/libggk.a" 需放置到自行撰寫的原始碼同個目錄中，或是編譯時指定libggk.a的檔案路徑

4. 執行語法(需要加上sudo，因為有用到D-Bus權限)：
```
sudo ./輸出檔名
```



