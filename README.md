# fft

```
import cv2
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import font_manager
from PIL import Image


# グレースケールに変換
#https://www.hello-python.com/2018/02/16/numpyとopencvを使った画像のフーリエ変換と逆変換/
img = cv2.imread("pic_demo.png",0)
wi,hi = img.shape
im = cv2.resize(img, (int(hi/4),int(wi/4)))
w,h = im.shape

#フーリエ変換
global fimg
fimage = np.fft.fft2(im)
fshift = np.fft.fftshift(fimage)
mag = 20*np.log(np.abs(fshift))
cv2.normalize(mag, mag, 0.0, 1.0, cv2.NORM_MINMAX)
mag = np.float32(mag)


class mouseParam:
    def __init__(self, input_img_name):
        #マウス入力用のパラメータ
        self.mouseEvent = {"x":None, "y":None, "event":None, "flags":None}
        #マウス入力の設定
        cv2.setMouseCallback(input_img_name, self.__CallBackFunc, None)
    
    #コールバック関数
    def __CallBackFunc(self, eventType, x, y, flags, userdata):
        
        self.mouseEvent["x"] = x
        self.mouseEvent["y"] = y
        self.mouseEvent["event"] = eventType    
        self.mouseEvent["flags"] = flags    

    #マウス入力用のパラメータを返すための関数
    def getData(self):
        return self.mouseEvent
    
    #マウスイベントを返す関数
    def getEvent(self):
        return self.mouseEvent["event"]                

    #マウスフラグを返す関数
    def getFlags(self):
        return self.mouseEvent["flags"]                

    #xとyの座標を返す関数
    def getPos(self):
        return (self.mouseEvent["x"], self.mouseEvent["y"])
    
    #逆変換重ね合わせ
    def ifft(self,p):
        ifft = np.zeros(im.shape)
        for i in range(len(p)): #10倍で出力
            px = p[i][0]
            py = p[i][1]
            ifft[py:py+5,px:px+5] = 1
        ifft_back = fshift*ifft
        ifft_back = np.fft.ifftshift(ifft_back)
        ifft_back = np.fft.ifft2(ifft_back)
        ifft_back = np.abs(ifft_back)
        ifft_back = np.uint8(ifft_back.real)
        cv2.imshow('ifft', ifft_back)
        
    #正弦波
    def sin(self,p):
        sin = np.zeros(im.shape)
        x = p[0]
        y = p[1]
        sin[y,x] = 1
        imsin = fshift*sin
        imsin = np.fft.fftshift(imsin)
        imsin = np.fft.ifft2(imsin)
        imsin = np.uint8(imsin)
        cv2.imshow('sin',imsin)
        
click_points = []
draw = False
window_name = 'mouse'

cv2.namedWindow('ifft',cv2.WINDOW_AUTOSIZE)
cv2.namedWindow('sin',cv2.WINDOW_AUTOSIZE)
cv2.namedWindow('original',cv2.WINDOW_AUTOSIZE)
cv2.namedWindow('fft',cv2.WINDOW_AUTOSIZE)

while True:
    
    # 描画エリアの指定
    mouseframe = np.zeros((w, h, 3), np.uint8)

    # 描画する
    cv2.imshow('original',im)
    cv2.imshow('fft', mag)
    [cv2.circle(mouseframe, point,1, (255,255,255), thickness=-1, lineType=cv2.LINE_8, shift=0) for point in click_points]
    cv2.imshow(window_name, mouseframe)
    

    # 描画結果の中でマウスの状態を取得する
    mouseData = mouseParam(window_name)

    # キー入力を1ms待って、k が13（Enter）だったらBreakする
    k = cv2.waitKey(1)
    if k == 13:
        break

    #左クリックがあったら表示
    if mouseData.getEvent() == cv2.EVENT_LBUTTONDOWN: # 左ボタンを押下したとき
        draw = True
        click_points.append(mouseData.getPos())
        mouseData.sin(click_points[-1])
        mouseData.ifft(click_points)
        
    if mouseData.getEvent() == cv2.EVENT_LBUTTONUP: # 左ボタンを上げたとき
        draw = False
        
    if mouseData.getEvent() == cv2.EVENT_MOUSEMOVE and draw: # マウスが動いた時
        if draw:
            click_points.append(mouseData.getPos())
            mouseData.sin(click_points[-1])
            mouseData.ifft(click_points)


cv2.destroyAllWindows()
```
OpenCVで取得した画像の２次元フーリエ変換，逆変換をリアルタイムで行うプログラム．

マウスにより周波数を指定する．

# 依存ライブラリ，バージョン
python3.7 openCV

# 実行の仕方
変換を行いたい画像を実行環境と同じディレクトリに用意しpicture.ipynbを実行すると，
５つのウィンドウが表示される．
  - original:オリジナルのグレースケール画像
  - fft:フーリエ変換後の画像
  - ifft:フーリエ逆変換の画像
  - sin:正弦波の画像
  - mouse:マウス操作を行うウィンドウ
mouse上でクリックorドラックを行うとsinおよびifftが更新される．
Enterを押すとプログラムは終了する．

# 使用例
![demo](demo.gif)

#参考　引用

フーリエ変換（参考および応用）：
https://www.hello-python.com/2018/02/16/numpyとopencvを使った画像のフーリエ変換と逆変換/

マウス入力（引用）：https://ensekitt.hatenablog.com/entry/2018/06/17/200000
