---
aliases:
created: 
update: 
author: Jason Chen
language: Python
sourceurl: https://jason-chen-1992.weebly.com/home/-kalman-filter
tags:
date: 2022-03-12
---

# 【演算法】卡爾曼濾波器 Kalman Filter

![[kalmanfilter-cover_orig.jpg]]

​卡爾曼濾波器（Kalman filter）又稱作最佳線性濾波器，它是一個純時域的濾波器（不需要做頻域變換），所以實現簡單，也因此在工程上有很多應用，像是用於估計動態系統的狀態。像當初 NASA 在登月的時候就是利用卡爾曼濾波來估計登月飛船的狀態！

可以想像在那個年代，感測器技術不如現在那麼進步，加上登月飛船在該死的太空環境中會受到各式各樣的干擾（太陽風、宇宙射線...），以至於它勢必沒有辦法準確的量測到真實的數值。但這可不行啊~ 如果速度、仰角、方向等數據的量測誤差過大，是會導致整台登月飛船墜毀在月球之上的。還有最後有卡爾曼博士出手相救，我們才得以見證這個人類工程史上的奇蹟。

![[kalmanfilter-chart_orig.jpg]]

然而卡爾曼濾波器不僅可以用來估計動態系統的狀態，它也可以被應用在當下正紅的自駕車上！
其實一台自駕車上，是會安裝多種不同的感測器（e.g. Radar、LIDAR、Camera），而這些感測器也各有各的用途！至於最後智能車要怎麼彙整這些來自不同感測器的資訊，我們就需要感測器融合（Sensor Fusion）技術。
卡爾曼濾波器也是一種可以用來​​實現感測器融合的方法。

![[kalmanfilter-sensorfusion_orig.jpg]]

接下來 Jaosn 也不想太數學的方式來寫，畢竟如果你真的想了解它底層數學邏輯/計算，那你隨便去 Youtube 上搜一下，就有一堆 PhD 拍的授課影片，講的可精彩了~ 我怎麼可能有他們專業 >"<
這邊呢主要還是想寫一下我對卡爾曼濾波器的理解。
​
​卡爾曼濾波器說穿了就兩個步驟：「預估」以及「測量更新」。
舉例來說，今天張三開著車，想要從甲地前往乙地。
​如果說甲地跟乙地之間的距離是 60 公里，並且張三以每小時 60 公里的速度均速前進的話，那我們自然就會預估張三會在一個小時後抵達目的地，但在真實世界中哪有那麼美好~ 姑且不論等紅綠燈、塞車之類的狀況，即便你能夠平穩的在道路上行駛，那車子的時速還是會受到上下坡、風阻等外在因素所影響，讓我們想讓車輛維持均速前進這件事變得相當困難。

既然沒辦法保證車輛能夠平穩地均速前行，那我們就很難 " 準確 " 估計下一個時間點車輛出現的位置。
這時候我們就用一個高斯機率密度函數來表示，車輛於該時間點 " 可能 " 出現的位置。
接著我們再通過 " 量測 "，來輔助確認車輛的位置。
那你們一定會想說~ 啊都可以用量的了，不是比較準嗎? 那還預估的屁?
那是因為，很多時候我們量測到的數據並不會是 " 真確 " 的。
以今天張三的這個例子來說，如果我們使用 GPS 訊號做 " 量測 "，一般民用 GPS 會有 10~25 公尺左右的誤差。然後我們假設用行駛速度預估出來的位置是有 100~200 公尺的誤差，而卡爾曼濾波器厲害的地方就是，它可以用這兩個不是那麼準的位置資訊，幫你預估出更加真確的位置。

![[kalmanfilter-move_orig.jpg]]

Ok, 那​關於卡爾曼濾波器的部分就簡單介紹到這邊，如果有想要自己動手玩玩看卡爾曼濾波器的話，其實這個演算法 OpenCV 就有 built-in 在他們 Library 裡面了！
Python 的話你直接 import cv2 然後 cv2.KalmanFilter 這個就是了！
最後再附上一段使用卡爾曼濾波器追蹤滑鼠移動事件的 Python Demo Code 給你們參考~
不過這 Code 不是我寫的，是以前學習相關知識時在網路上找到並保存下來的，應該是對岸的某論壇或某博主吧~ 至於詳細出處就不記得了~

Python Demo Code : Track Mouse Movement Using Kalman Filter

```python
import cv2
import numpy as np
#建立一個大小800*800的空幀
frame = np.zeros((800,800,3),np.uint8)
#初始化測量座標和滑鼠運動預測的陣列
last_measurement = current_measurement = np.array((2,1),np.float32)

last_predicition = current_prediction = np.zeros((2,1),np.float32)
'''
    mousemove()函式在這裡的作用就是傳遞X,Y的座標值，便於對軌跡進行卡爾曼濾波
'''
def mousemove(event,x,y,s,p):
    #定義全域性變數
    global frame,current_measurement,measurements,last_measurement,current_prediction,last_prediction
    #初始化
    last_measurement = current_measurement
    last_prediction = current_prediction
    #傳遞當前測量座標值
    current_measurement = np.array([[np.float32(x)],[np.float32(y)]])
    #用來修正卡爾曼濾波的預測結果
    kalman.correct(current_measurement)
    # 呼叫kalman這個類的predict方法得到狀態的預測值矩陣，用來估算目標位置
    current_prediction = kalman.predict()
    #上一次測量值
    lmx,lmy = last_measurement[0],last_measurement[1]
    #當前測量值
    cmx,cmy = current_measurement[0],current_measurement[1]
    #上一次預測值
    lpx,lpy = last_prediction[0],last_prediction[1]
    #當前預測值
    cpx,cpy = current_prediction[0],current_prediction[1]
    #繪製測量值軌跡（綠色）
    cv2.line(frame,(lmx,lmy),(cmx,cmy),(0,100,0))
    #繪製預測值軌跡（紅色）
    cv2.line(frame,(lpx,lpy),(cpx,cpy),(0,0,200))

cv2.namedWindow("kalman_tracker")
#呼叫函式處理滑鼠事件，具體事件必須由回撥函式的第一個引數來處理，該引數確定觸發事件的型別（點選和移動）
'''
void setMousecallback(const string& winname, MouseCallback onMouse, void* userdata=0)
       winname:視窗的名字
       onMouse:滑鼠響應函式，回撥函式。指定窗口裡每次滑鼠時間發生的時候，被呼叫的函式指標。
                這個函式的原型應該為void on_Mouse(int event, int x, int y, int flags, void* param);
       userdate：傳給回撥函式的引數

 void on_Mouse(int event, int x, int y, int flags, void* param);
        event是 CV_EVENT_*變數之一
        x和y是滑鼠指標在影象座標系的座標（不是視窗座標系）
        flags是CV_EVENT_FLAG的組合， param是使用者定義的傳遞到setMouseCallback函式呼叫的引數。
    常用的event：
        CV_EVENT_MOUSEMOVE
        CV_EVENT_LBUTTONDOWN
        CV_EVENT_RBUTTONDOWN
        CV_EVENT_LBUTTONUP
        CV_EVENT_RBUTTONUP
        和標誌位flags有關的：
        CV_EVENT_FLAG_LBUTTON
'''
cv2.setMouseCallback("kalman_tracker",mousemove)
'''
Kalman這個類需要初始化下面變數：
轉移矩陣，測量矩陣，控制向量(沒有的話，就是0)，
過程噪聲協方差矩陣，測量噪聲協方差矩陣，
後驗錯誤協方差矩陣，前一狀態校正後的值，當前觀察值。
    在此cv2.KalmanFilter(4,2)表示轉移矩陣維度為4，測量矩陣維度為2

卡爾曼濾波模型假設k時刻的真實狀態是從(k − 1)時刻的狀態演化而來，符合下式：
            X(k) = F(k) * X(k-1) + B(k)*U(k) + W（k）
其中
F(k)  是作用在xk−1上的狀態變換模型（/矩陣/向量）。 
B(k)  是作用在控制器向量uk上的輸入－控制模型。 
W(k)  是過程噪聲，並假定其符合均值為零，協方差矩陣為Qk的多元正態分佈。
'''
kalman = cv2.KalmanFilter(4,2)
#設定測量矩陣
kalman.measurementMatrix = np.array([[1,0,0,0],[0,1,0,0]],np.float32)
#設定轉移矩陣
kalman.transitionMatrix = np.array([[1,0,1,0],[0,1,0,1],[0,0,1,0],[0,0,0,1]],np.float32)
#設定過程噪聲協方差矩陣
kalman.processNoiseCov = np.array([[1,0,0,0],[0,1,0,0],[0,0,1,0],[0,0,0,1]],np.float32)*0.03

while True:
    cv2.imshow("kalman_tracker",frame)
    if (cv2.waitKey(30) & 0xff) == 27:
        break

cv2.destroyAllWindows()
```
