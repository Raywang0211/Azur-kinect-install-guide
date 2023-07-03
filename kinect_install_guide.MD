# Azure Kinect SDK 安裝

可以從官方網站下載SDK，按照官方網站的步驟也可以正確的安裝。

[https://docs.microsoft.com/zh-tw/azure/kinect-dk/sensor-sdk-download](https://docs.microsoft.com/zh-tw/azure/kinect-dk/sensor-sdk-download)

基本介紹

[https://docs.microsoft.com/zh-tw/azure/kinect-dk/about-sensor-sdk](https://docs.microsoft.com/zh-tw/azure/kinect-dk/about-sensor-sdk)

---

### 確認Microsoft 相依套件包是否有在路徑中

[https://docs.microsoft.com/zh-tw/windows-server/administration/linux-package-repository-for-microsoft-software](https://docs.microsoft.com/zh-tw/windows-server/administration/linux-package-repository-for-microsoft-software)

在ubuntu 18.04的環境之下執行以下

```bash
curl -sSL https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo apt-add-repository https://packages.microsoft.com/ubuntu/18.04/prod
sudo apt-get update
```

若有遇到以下問題i386 與 amd64 問題的解決方法

![Azure%20Kinect%20SDK%20%E5%AE%89%E8%A3%9D%202bcc01cea1d147799ceb5cb660062a13/Untitled.png](Azure%20Kinect%20SDK%20%E5%AE%89%E8%A3%9D%202bcc01cea1d147799ceb5cb660062a13/Untitled.png)

安裝套件時他沒辦法用i386的架構去下載因此抱錯，此時需要強制修改sources.list 將i386架構改成amd64就可以解決問題。

```bash
curl -sSL https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
sudo apt-add-repository https://packages.microsoft.com/ubuntu/18.04/prod
cd /etc/apt
sudo vim sources.list
```

執行完新增路徑 <curl> 之後再做以下的更改

將 'deb [https://packages.microsoft.com/ubuntu/18.04/prod](https://packages.microsoft.com/ubuntu/18.04/prod) bionic main'

更改成 'deb [arch=amd64] [https://packages.microsoft.com/ubuntu/18.04/prod](https://packages.microsoft.com/ubuntu/18.04/prod) bionic main'

強迫指定下載的框架 amd64

### 安裝 k4a-tools 套件並且執行

開啟的方式與realsense viewer 很像 realsense-viewer

[https://docs.microsoft.com/zh-tw/azure/kinect-dk/sensor-sdk-download](https://docs.microsoft.com/zh-tw/azure/kinect-dk/sensor-sdk-download)

```bash
sudo apt install k4a-tools
sudo apt install libk4a1.4-dev
sudo k4aviewer  #start k4aviewer
```

---

### Azure Kinect DK 簡介

(FoV) 模式的窄欄位，很適合在 X 和 Y 維度中具有較小範圍的場景，但在 Z 維度中較大的範圍。 如果場景的 X 和 Y 範圍很大，但 Z 範圍較小，則 寬 FoV 模式 較適合。

[https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___enumerations_ga3507ee60c1ffe1909096e2080dd2a05d.html](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___enumerations_ga3507ee60c1ffe1909096e2080dd2a05d.html)

解釋相機binned and unbinned，將2*2範圍內的pixel合併起來得到相同的像似值

![Azure%20Kinect%20SDK%20%E5%AE%89%E8%A3%9D%202bcc01cea1d147799ceb5cb660062a13/Untitled%201.png](Azure%20Kinect%20SDK%20%E5%AE%89%E8%A3%9D%202bcc01cea1d147799ceb5cb660062a13/Untitled%201.png)

---

### Using python to read depth image

[https://www.codenong.com/cs107042721/](https://www.codenong.com/cs107042721/)

以上網址中提到的大神github

[https://github.com/ibaiGorordo/pyKinectAzure](https://github.com/ibaiGorordo/pyKinectAzure)

使用該範例程事前需要安裝兩個另外的套件

1. pyk4a
    
    安裝pyk4a之前一定要安裝sdk(要安裝成功)，不然pyk4a安裝上會有問題且無法正確開啟套件
    
    kinect 連接到電腦的時候要注意 usb 插到主機板後面，不然會偵測不到相機，可能要多插拔幾次才會感應到。
    
2. Azure-Kinect-Python
    
    比較難用(大神說的)，但是我沒有安裝這個一樣可以使用
    

pyk4a範例(同時顯示rgb以及depth)：

```python
from pyk4a import Config, PyK4A
from matplotlib import pyplot as plt

k4a = PyK4A()
k4a.start()
capture = k4a.get_capture()
img_color = capture.color
img_depth = capture.depth
print('img_depth.shape = ',img_depth.shape)

# Display with pyplot
plt.imshow(img_color[:, :, 2::-1]) # BGRA to RGB
plt.figure()
plt.imshow(img_depth)
plt.show()
```

若啟動時報錯：

pyk4a.errors.K4AException

可以嘗試以下解法：

先檢查是否有其他程式佔用了相機，接著嘗試以下的權限設定（[https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/usage.md)）

![Azure%20Kinect%20SDK%20%E5%AE%89%E8%A3%9D%202bcc01cea1d147799ceb5cb660062a13/Untitled%202.png](Azure%20Kinect%20SDK%20%E5%AE%89%E8%A3%9D%202bcc01cea1d147799ceb5cb660062a13/Untitled%202.png)

如果都已經嘗試過了最後可以嘗試將usb多插拔幾次。

### Config 檔設定以及以及SDK設定

config設定：

```python
k4a = PyK4A(Config(
            color_resolution=pyk4a.ColorResolution.RES_2160P,
            depth_mode=pyk4a.DepthMode.WFOV_UNBINNED,camera_fps = pyk4a.FPS.FPS_15
        ))
```

有4種深度相機模式可以使用，可以調整FPS，也可以選用不同解析度的BGRA影像

### pyk4a import error

```bash
>>> import pyk4a
RuntimeError: module compiled against API version 0xf but this version of numpy is 0xd
Traceback (most recent call last):
  File "/home/nmed/Documents/Code/Azure-Kinect-python/pyk4a/pyk4a/module.py", line 5, in <module>
    import k4a_module  # noqa: F401
ImportError: numpy.core.multiarray failed to import

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/nmed/Documents/Code/Azure-Kinect-python/pyk4a/pyk4a/__init__.py", line 1, in <module>
    from .calibration import Calibration, CalibrationType
  File "/home/nmed/Documents/Code/Azure-Kinect-python/pyk4a/pyk4a/calibration.py", line 8, in <module>
    from .module import k4a_module
  File "/home/nmed/Documents/Code/Azure-Kinect-python/pyk4a/pyk4a/module.py", line 24, in <module>
    raise ImportError(
ImportError: Cannot import k4a_module. Make sure `libk4a.so` can be found. Add the directory to your `LD_LIBRARY_PATH` if required. Also make sure pyk4a is properly built.
```

solution -

update numpy version 

```bash
pip uninstall pyk4a
pip install --upgrade numpy
pip install pyk4a
```