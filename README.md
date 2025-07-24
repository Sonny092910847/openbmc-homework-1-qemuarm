# OpenBMC 作業一：QEMU (qemuarm) IPMI 成功連線

本文檔記錄了成功完成作業一的過程：在 QEMU 上執行自行編譯的 OpenBMC 映像檔（qemuarm 目標），並成功透過 IPMI over LAN 進行通訊。

## 環境設定

* **主機作業系統：** Windows
* **虛擬化軟體：** VirtualBox
* **虛擬機作業系統：** Ubuntu 22.04 Server
* **OpenBMC 版本：** v2.13.0（或最新 master）
* **目標平台：** qemuarm


## 1. 關鍵 local.conf 自訂設定

```bash
# 在 build/qemuarm/conf/local.conf 中加入以下設定，以啟用開發功能和必要的服務。
# --- 作業最終自訂設定 ---
# 強制啟用開發功能，允許空密碼和自動序列登入
EXTRA_IMAGE_FEATURES += " empty-root-password allow-empty-password serial-autologin-root"

# 新增 SSH 和 IPMI 必要套件
IMAGE_INSTALL:append = " dropbear phosphor-ipmi-net"

# 強制移除預設靜態密碼套件
IMAGE_INSTALL:remove = " obmc-phosphor-static-password"
```

## 2. QEMU 啟動指令

```bash
# 在成功執行 bitbake obmc-phosphor-image 編譯後，使用以下指令啟動 QEMU，並增加記憶體和正確的網路埠轉發設定。
/home/sonny/openbmc/build/qemuarm/tmp/work/x86_64-linux/qemu-helper-native/1.0/recipe-sysroot-native/usr/bin/qemu-system-arm \
-M virt \
-m 1024 \
-kernel /home/sonny/openbmc/build/qemuarm/tmp/deploy/images/qemuarm/zImage \
-drive file=/home/sonny/openbmc/build/qemuarm/tmp/deploy/images/qemuarm/obmc-phosphor-image-qemuarm.ext4,if=none,id=disk0,format=raw \
-device virtio-blk-device,drive=disk0 \
-append "root=/dev/vda rw" \
-nic user,id=net0,hostfwd=udp::6230-:623,mac=52:54:00:12:34:02 \
-nographic
```

## 3. IPMI 驗證指令

```bash
# 在 QEMU 執行時，使用以下 ipmitool 指令成功查詢 sensor 列表，證明 IPMI over LAN 連線已建立。
bashipmitool -I lanplus -H localhost -p 6230 -U root -P 0penBmc -C 17 sdr elist
# 這確認了作業一的成功完成。
```
