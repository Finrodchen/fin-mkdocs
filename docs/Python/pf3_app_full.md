# 使用Flask框架製作網站─公告欄網頁程式

## 前情提要

之前我們的 Flask 公告欄網站已經完成網頁的製作了，在這一篇文章我們要完成網站程式的建置，也就是系列文章第一篇中完成的`app.py`，在這個章節中我們會建立資料庫，完成資料庫運作的程式，下篇文章會與公告欄網頁串聯，並測試產品運作狀態。

上次我們的`app.py`做到這邊：

```python
from flask import Flask #引入Flask模組

app = Flask(__name__) #告訴Flask我們現在要執行一個叫做app的網站程式，__name__是Flask內的模組名稱

@app.route("/") #描述app網站程式的路徑，"/"表示是首頁
def home(): #定義首頁要運作的功能
    return "Hello World. I am Finrodchen!" #回傳Hello訊息

if __name__ == "__main__":
    app.run() #設定若__name__這個模組被指定為主要網站模組時，則啟動這app這個網站，執行app中的功能

```

## 工作流程

1) 資料庫 kanban.db
2) 建立新增公告功能
3) 建立刪除公告功能
4) 建立更新公告功能
5) 本機建立資料庫

## 需要的模組

- flask (網站框架)
- flask_sqlalchemy (適用於flask框架的資料庫擴充模組)
- datetime

引入模組：

```python
from flask import Flask, render_template, request, redirect
#render_template 用於渲染網頁，將base.html的設定分享其他網頁
#request和redirect則是用來將接收或送出網頁的資料，擔任資料庫與前端網頁的溝通橋樑

from flask_sqlalchemy import SQLAlchemy
from datetime import datetime
```

## 資料庫 kanban.db

在`app.py`中我們要先建立儲存公告內容的資料庫，首先我們要設定資料庫的名稱和儲存位置，接著將資料庫在`app.py`中定義為`db`

```python
app = Flask(__name__) #告訴Flask我們現在要執行一個叫做app的網站程式，__name__是Flask內的模組名稱

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///kanban.db'

# 設定資料庫格式為SQLite，儲存在根目錄下，名稱為kanban.db

db = SQLAlchemy(app)

# 將app裡由SQLAlchemy建立的資料庫定義為db

```

接著我們定義資料庫中的第一個資料表為Todo，接著定義資料表中各欄位要儲存的資料及屬性。

```python
class Todo(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    #第一欄定義為ID，整數，用於排序
    
    priority = db.Column(db.String(200), nullable = False)
    #第二欄定義為優先順序，字串，長度限制為200位元
    
    title = db.Column(db.String(200), nullable = False)
    #第三欄定義為任務標題，字串，長度限制為200位元
    
    content = db.Column(db.String(400), nullable = False)
    #第四欄定義為任務內容，字串，長度限制為200位元
    
    people = db.Column(db.String(200), nullable = False)
    #第四欄定義為任務負責人，字串，長度限制為200位元
    
    completed = db.Column(db.Integer, default = 0)
    #第五欄定義為任務完成狀態，整數，0或1
    
    date_created = db.Column(db.DateTime, default = datetime.utcnow)
    #第六欄定義為任務建立時間，日期，預設為建立任務裝置的本地時間

    def __repr__(self):
        return '<Task %r>' % self.id
    #將網頁收到的資料寫入資料庫中
```

## 新增公告功能

接下來要把主要功能都建立起來，首先是新增公告的功能：

新增公告的功能將直接出現在首頁，因此我們要在`@app.route("/")`下面定義首頁的程式。

```python
@app.route("/", methods=['POST', 'GET'])

# 在首頁運作程式，網頁中會使用到POST和GET兩種機制

def index():
    if request.method == 'POST':
    #收到來自網頁的POST時，執行以下程式

        task_priority = request.form['priority']
        task_title = request.form['title']
        task_content = request.form['content']
        task_people = request.form['people']
        #從表格中對應的欄位收集資料存到變數中
        
        new_task = Todo(priority=task_priority, title=task_title, content=task_content, people=task_people)
        #將以上變數存到db對應的欄位中

        try:
            db.session.add(new_task)
            db.session.commit()
            return redirect('/')
            #將任務資料存入db，並顯示於首頁中
            
        except:
            return "Fail to add new issue to your task."
            #如果db寫入失敗了顯示錯誤訊息
    else:
        tasks = Todo.query.order_by(Todo.date_created).all()
        return render_template('index.html', tasks=tasks)
        #沒有POST新任務的話，就顯示目前資料中的任務
        #首頁使用的模板是index.html

```

## 刪除公告功能

接著是刪除公告功能，刪除功能建立在首頁裡面，不需要使用網頁模板，點選後就會刪除對應列的公告。

```python
@app.route("/delete/<int:id>")
#在/delete/<int:id>按鈕執行以下程式

def delete(id):
    task_to_delete = Todo.query.get_or_404(id)
    #將資料庫收到刪除要求的任務存到變數中

    try:
        db.session.delete(task_to_delete)
        db.session.commit()
        return redirect("/")
        #從資料庫中刪除任務，將結果回傳到首頁
    
    except:
        return "Deleteing problem."
        #如果db刪除失敗了顯示錯誤訊息
```

## 更新公告功能

更新公告功能需要用到另一個`update.html`網頁，網頁中的表格會先讀取目前公告的內容，使用者修正送出後改寫資料庫中的資料。

```python
@app.route("/update/<int:id>", methods=['POST', 'GET'])

# 在/update/<int:id>按鈕執行以下程式，網頁中會使用到POST和GET兩種機制

def update(id):
    task = Todo.query.get_or_404(id)
    #將資料庫收到更新要求的任務存到變數中

    if request.method == "POST":
        task.priority = request.form['priority']
        task.title = request.form['title']
        task.content = request.form['content']
        task.people = request.form['people']
        #從表格中對應的欄位收集資料存到變數中

        try:
            db.session.commit()
            return redirect("/")
            #更新資料庫，將結果回傳到首頁
            
        except:
            return "Updating issue."
            #如果db更新失敗了顯示錯誤訊息
    else:
        return render_template('update.html', task=task)
        #更新頁面使用的模板是update.html

```

最後再加上Flask框架的執行程式碼就完成啦！

```python
if __name__ == "__main__":
    app.run() #設定若__name__這個模組被指定為主要網站模組時，則啟動這app這個網站，執行app中的功能
```

## 本機建立資料庫

程式完成之後，我們必須先在本機建立資料庫`kanban.db`，才能開始進行測試：

首先我們要進到Python cosole中，並切換目錄到`app.py`所在的資料夾：

```bash
Python 3.7.7 (default, May  6 2020, 11:45:54) [MSC v.1916 64 bit (AMD64)] :: Anaconda, Inc. on win32

Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

接著輸入`from app import db`，從app.py中引入資料庫架構。

最後輸入`db.create_all()`，就完成資料庫建立了。

```bash
>>> from app import db
>>> db.create_all()
>>> 
```

可以看到在資料夾中新增了一個`kanban.db`檔案。

到這邊就完成網站程式的部分了，下一篇文章我們會回頭修改`index.html`和`update.html`兩個網頁，加入與資料庫互動的功能，完成後便可以在本機進行測試了！

## 完整程式碼

```python
from flask import Flask, render_template, request, redirect

# render_template 用於渲染網頁，將base.html的設定分享其他網頁

# request和redirect則是用來將接收或送出網頁的資料，擔任資料庫與前端網頁的溝通橋樑

from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__) #告訴Flask我們現在要執行一個叫做app的網站程式，__name__是Flask內的模組名稱

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///kanban.db'

# 設定資料庫格式為SQLite，儲存在根目錄下，名稱為kanban.db

db = SQLAlchemy(app)

# 將app裡由SQLAlchemy建立的資料庫定義為db

class Todo(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    #第一欄定義為ID，整數，用於排序

    priority = db.Column(db.String(200), nullable = False)
    #第二欄定義為優先順序，字串，長度限制為200位元
    
    title = db.Column(db.String(200), nullable = False)
    #第三欄定義為任務標題，字串，長度限制為200位元
    
    content = db.Column(db.String(400), nullable = False)
    #第四欄定義為任務內容，字串，長度限制為200位元
    
    people = db.Column(db.String(200), nullable = False)
    #第四欄定義為任務負責人，字串，長度限制為200位元
    
    completed = db.Column(db.Integer, default = 0)
    #第五欄定義為任務完成狀態，整數，0或1
    
    date_created = db.Column(db.DateTime, default = datetime.utcnow)
    #第六欄定義為任務建立時間，日期，預設為建立任務裝置的本地時間

    def __repr__(self):
        return '<Task %r>' % self.id
    #將網頁收到的資料寫入資料庫中

@app.route("/", methods=['POST', 'GET'])

# 在首頁運作程式，網頁中會使用到POST和GET兩種機制

def index():
    if request.method == 'POST':
    #收到來自網頁的POST時，執行以下程式

        task_priority = request.form['priority']
        task_title = request.form['title']
        task_content = request.form['content']
        task_people = request.form['people']
        #從表格中對應的欄位收集資料存到變數中
        
        new_task = Todo(priority=task_priority, title=task_title, content=task_content, people=task_people)
        #將以上變數存到db對應的欄位中

        try:
            db.session.add(new_task)
            db.session.commit()
            return redirect('/')
            #將任務資料存入db，並顯示於首頁中
            
        except:
            return "Fail to add new issue to your task."
            #如果db寫入失敗了顯示錯誤訊息
    else:
        tasks = Todo.query.order_by(Todo.date_created).all()
        return render_template('index.html', tasks=tasks)
        #沒有POST新任務的話，就顯示目前資料中的任務
        #首頁使用的模板是index.html

@app.route("/delete/<int:id>")

# 在/delete/<int:id>按鈕執行以下程式

def delete(id):
    task_to_delete = Todo.query.get_or_404(id)
    #將資料庫收到刪除要求的任務存到變數中

    try:
        db.session.delete(task_to_delete)
        db.session.commit()
        return redirect("/")
        #從資料庫中刪除任務，將結果回傳到首頁
    
    except:
        return "Deleteing problem."
        #如果db刪除失敗了顯示錯誤訊息

@app.route("/update/<int:id>", methods=['POST', 'GET'])

# 在/update/<int:id>按鈕執行以下程式，網頁中會使用到POST和GET兩種機制

def update(id):
    task = Todo.query.get_or_404(id)
    #將資料庫收到更新要求的任務存到變數中

    if request.method == "POST":
        task.priority = request.form['priority']
        task.title = request.form['title']
        task.content = request.form['content']
        task.people = request.form['people']
        #從表格中對應的欄位收集資料存到變數中

        try:
            db.session.commit()
            return redirect("/")
            #更新資料庫，將結果回傳到首頁
            
        except:
            return "Updating issue."
            #如果db更新失敗了顯示錯誤訊息
    else:
        return render_template('update.html', task=task)
        #更新頁面使用的模板是update.html

if __name__ == "__main__":
    app.run() #設定若__name__這個模組被指定為主要網站模組時，則啟動這app這個網站，執行app中的功能
```
