# 使用Python的 Flask 框架製作網站程式

![logo](https://i.imgur.com/3Wyrabq.png)

## Flask

[Flask](https://flask.palletsprojects.com/en/1.1.x/) 是Python的一個套件， Flask 提供的是"網頁程式框架功能"，藉由這個框架，我們可以很簡單將程式與網站緊密結合，導入資料庫、模組化的建立頁面等功能都非常的方便。

Flask框架的實作會分成3個部分：

1) 基本Flask框架介紹，做出Hello World網頁
2) 導入資料庫，做出簡易的佈告欄網頁
3) 將做好的網頁發布到heroku上，網站上線

## 背景知識及準備工作

- 基本的Python及HTML語言知識
- Python開發環境

那我們就開始Hello World吧！

## 使用 Flask 模組

基本的Flask網頁僅需要安裝一個模組就是Flask！

```bash
pip install Flask #安裝Flask模組
```

裝好之後就可以開始寫網頁/程式了。

## 建立基本 Flask 框架網站

首先建立一個新的Python程式命名為`app.py`，在編輯器中打開。

程式中第一步要引入Flask模組：

```python
from flask import Flask #引入Flask模組
```

接著就是建立基本的Flask程式框架：

```python
app = Flask(__name__) #告訴Flask我們現在要執行一個叫做app的網站程式，__name__是Flask內的模組名稱
```

再來建立網站的內容：

```python
@app.route("/") #描述app網站程式的路徑，"/"表示是首頁
def home(): #定義首頁要運作的功能
    return "Hello World. I am Finrodchen!" #回傳Hello訊息
```

最後設定網站程式的運作方式：

```python
if __name__ == "__main__":
    app.run() #設定若__name__這個模組被指定為主要網站模組時，則啟動這app這個網站，執行app中的功能
```

就這樣，網站的基本架構已經完成了！接下來只要執行這個程式`app.py`，在終端機中會看到網站伺服器啟動的敘述， Flask 會在本機中建立網站，你看到的網址會是(<http://127.0.0.1:5000)，只要在瀏覽器中打開這個網址就可以看到Hello> World訊息啦！

![hello](https://i.imgur.com/R9dB2N8.png)

到這裡我們已經成功用 Flask 做出網站了，下一篇文章我們要來改造這個網站，導入資料庫，做一個簡單的線上的佈告欄。

## 完整程式碼

```python
from flask import Flask #引入Flask模組

app = Flask(__name__) #告訴Flask我們現在要執行一個叫做app的網站程式，__name__是Flask內的模組名稱

@app.route("/") #描述app網站程式的路徑，"/"表示是首頁
def home(): #定義首頁要運作的功能
    return "Hello World. I am Finrodchen!" #回傳Hello訊息

if __name__ == "__main__":
    app.run() #設定若__name__這個模組被指定為主要網站模組時，則啟動這app這個網站，執行app中的功能
```
