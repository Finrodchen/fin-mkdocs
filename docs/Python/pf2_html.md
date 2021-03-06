# 使用 Flask 框架製作網站─公告欄網頁

## 前情提要

我們已經運用 Flask 框架完成了基礎網站結構的架設，接下來我們要幫這個網站加入公告欄的功能。

## 網站架構

![flow](https://i.imgur.com/8jq0ii3.png)

這個公告欄網站分成三個部分，分別是網頁、網站程式以及資料庫，如同圖中的架構，網頁部分呈現公告欄的外觀以及公告內容；網站程式則負責將我們新增/更新/刪除的內容傳遞給資料庫，資料庫保存資料之後再顯示於網頁上。

這篇文章將先從網頁建立開始，筆記在 Flask 框架中如何將網頁與網站程式整合，下一篇文章則會完成程式與資料庫的建立及測試。

## 基底網頁(base.html)

在 Flask 框架中提供了區塊的設計`{% 我是區塊 %}`，我們可以先建立一個基底網頁base.html，預先設定網站中每一個都相同的部分，例如`<meta>`、`<sheetstyle>`等的屬性，接著將在base.html文件中圈出特定的區塊，這些區塊在其他的網頁中會變成可變動的內容區域。簡單來說，就是整個網站的網頁都是依照基底網頁的設計產生，而在基底網頁中圈出的區塊位置，便是各個網站頁面能自訂內容的。不囉嗦現在就開始製作基底網頁吧！

首先為了符合 Flask 框架的架構，我們要在網站程式app.py的目錄內新增一個名稱為"templates"的資料夾，並在其中建立base.html網頁。

將base.html以編輯器打開，先給予它網頁的基本架構：

```html
<!doctype html>

<html lang="zh-Hant-TW"> <!-- 網頁語言 -->

    <head>  <!-- 網頁標頭 -->
    
      <meta charset="utf-8"> <!-- 網頁字元編碼 -->
        
      <meta name="description" content="以Flask框架製作的簡易公告欄">
      <meta name="author" content="finrodchen.net">
      <!-- 網頁基本資料 -->
  
    </head>

    <body>  <!-- 網頁內容 -->

    </body>

</html>
```

接著我們在head和body標籤中額外加入 Flask 框架的區塊：

```html
<head>
         
      {% block head %}  <!-- head區塊開始 -->
      
      {% endblock %}  <!-- head區塊結束 -->
      
</head>

<body>

    {% block body %}  <!-- body區塊開始 -->
      
    {% endblock %}  <!-- body區塊結束 -->

</body>
```

這樣就完成base網頁的設定了。

## 公布欄首頁(index.html)

由於基底網頁已經設定好了，因此這個網站中的其他網頁的結構就會變得非常簡單，就像是這樣：(對，就是這麼簡單)

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->
      
{% endblock %}  <!-- body區塊結束 -->
```

接下來我們先在head區塊中放入首頁的標題，這個標題會出現在瀏覽器的分頁標籤上。

```html
{% block head %}  <!-- head區塊開始 -->

<title>簡易公告欄 by Finrodchen</title>

{% endblock %}  <!-- head區塊結束 -->
```

接著我們要在body區塊中建立公告欄的內容，整個網頁中分為兩個區塊：

- 公告欄本體(包含公告事項、說明、公告人、建立時間、刪除/更新功能)
- 發布公告的表單(讓使用者能填寫上述內容的表單)

首先我們先寫上公告欄網頁的標題，接著再建立有5個欄位的表格，容納公告欄中的資訊，使用HTML語法中的`<table>`標籤。

```html
<h1 style="text-align: center;">
    重要事項公告欄  <!-- h1標題`,設定為置中-->
</h1>

<table border='1' align='center'>
    <!-- 設定表格框線及置中 -->
    <tr>  <!-- 表格表頭 -->
        <th>公告事項</th>
        <th>說明</th>
        <th>公告人</th>
        <th>建立時間</th>
        <th>操作</th>
    </tr>
    <tr>  
        <!-- 
        這裡我先填入內容，下一篇文章導入資料庫
        的時候會改為讀取資料庫中的內容。
        -->
        <td>公告001</td>
        <td>這是公告001的說明</td>
        <td>小明</td>
        <td>2020/06/08</td>
        <td>
            刪除
            <br>
            更新
        </td>
    </tr>
</table>
```

接下來我們再加入發布公告用的表單，使用的是`<form>`標籤：

```html
<br>
<hr>  <!-- 為了美觀加入分行和水平線區隔公告欄與表單 -->
<br>

<form align='center'>
    <!--
    分別建立3個要輸入的項目
    在input標籤中定義輸入屬性為文字
    給予欄位名稱及ID標記(用於對應資料庫內容)
    最後在placeholder屬性中提示使用者應填寫的資訊
    -->
    <label for="title">公告事項</label><br>
    <input type="text" name="title" id="title" placeholder="請輸入公告事項"><br><br>
    
    <label for="content">公告說明</label><br>
    <input type="text" name="content" id="content" placeholder="請輸入公告說明"><br><br>
    
    <label for="person">公告人</label><br>
    <input type="text" name="person" id="person" placeholder="請輸入公告發佈人"><br><br>
    
    <input type="submit" value="新增公告事項">
    <!-- 因為目前尚未連結資料庫，所以這個表單還無法收集資料 -->
    
</form>
```

到這邊公告欄的首頁就完成了，現階段可以看到網頁的外觀已經很好的呈現了，但公告欄和表單在連上資料庫前還無法工作。最後我們還要增加一個更新公告用的網頁，其中也包含了一個和首頁類似的表單，用於更新現有的公告事項。

## 更新網頁(update.html)

更新公告用的網頁架構相對簡單，僅需要一個表單更新現有的資訊即可，但由於目前的網頁尚未與資料庫連結，因此這個更新網頁也是先以完成架構為目標。

首先先代入基底網頁(base.html)的基本架構，並加上網站標題及網頁中的標題：

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

    <title>公告更新</title>  <!-- 網頁標題 -->

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->

    <h2 style="text-align: center;">
    公告更新  <!-- h2標題`,設定為置中-->
    </h2>
      
{% endblock %}  <!-- body區塊結束 -->
```

接著在標題後加入更新內容所使用的表單：

```html
<form align='center'>
    <!--
    分別建立3個要輸入的項目
    在input標籤中定義輸入屬性為文字
    給予欄位名稱及ID標記(用於對應資料庫內容)
    新增了value屬性，用於顯示預設的內容在表單輸入欄位中
    與資料庫連結後value屬性會直接引用特定公告的資料內容
    -->
    <label for="title">公告事項</label><br>
    <input type="text" name="title" id="title" value="公告001"><br><br>
    
    <label for="content">公告說明</label><br>
    <input type="text" name="content" id="content" value="這是公告001的說明"><br><br>
    
    <label for="person">公告人</label><br>
    <input type="text" name="person" id="person" value="小明"><br><br>
    
    <input type="submit" value="更新公告">
    <!-- 因為目前尚未連結資料庫，所以這個表單還無法收集資料 -->
    
</form>
```

到這邊我們就完成這個公告欄網頁所有需要的頁面了，下一篇文章就會修改`app.py`網站後端程式，並建立公告欄對應的資料庫，屆時會再回頭修正`index.html`和`update.html`的部分欄位，整個網站就完成了！

## 完整程式碼

- base.html

```html
<!doctype html>

<html lang="zh-Hant-TW"> <!-- 網頁語言 -->

    <head>  <!-- 網頁標頭 -->
    
        <meta charset="utf-8"> <!-- 網頁字元編碼 -->
        
        <meta name="description" content="以Flask框架製作的簡易公告欄">
        <meta name="author" content="finrodchen.net">
        <!-- 網頁基本資料 -->

        {% block head %}  <!-- head區塊開始 -->
      
        {% endblock %}  <!-- head區塊結束 -->
  
    </head>

    <body>  <!-- 網頁內容 -->

        {% block body %}  <!-- body區塊開始 -->
      
        {% endblock %}  <!-- body區塊結束 -->

    </body>

</html
```

- index.html

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

    <title>簡易公佈欄 by Finrodchen</title>

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->

    <h1 style="text-align: center;">
        重要事項公告欄  <!-- h1標題 -->
    </h1>

    <table border='1' align='center'>
        <tr>  <!-- 表格表頭 -->
            <th>公告事項</th>
            <th>說明</th>
            <th>公告人</th>
            <th>建立時間</th>
            <th>操作</th>
        </tr>
        <tr>  
            <!-- 
            這裡我先填入內容，下一篇文章導入資料庫
            的時候會改為讀取資料庫中的內容。
            -->
            <td>公告001</td>
            <td>這是公告001的說明</td>
            <td>小明</td>
            <td>2020/06/08</td>
            <td>
                刪除
                <br>
                更新
            </td>
        </tr>
    </table>

    <br>
    <hr>
    <br>

    <form align='center'>
        <!--
        分別建立3個要輸入的項目
        在input標籤中定義輸入屬性為文字
        給予欄位名稱及ID標記(用於對應資料庫內容)
        最後在placeholder屬性中提示使用者應填寫的資訊
        -->
        <label for="title">公告事項</label><br>
        <input type="text" name="title" id="title" placeholder="請輸入公告事項"><br><br>
        
        <label for="content">公告說明</label><br>
        <input type="text" name="content" id="content" placeholder="請輸入公告說明"><br><br>
        
        <label for="person">公告人</label><br>
        <input type="text" name="person" id="person" placeholder="請輸入公告發佈人"><br><br>
        
        <input type="submit" value="新增公告事項">

        <!-- 因為目前尚未連結資料庫，所以這個表單還無法收集資料 -->

    </form>
      
{% endblock %}  <!-- body區塊結束 -->
```

- update.html

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

        <title>公告更新</title>  <!-- 網頁標題 -->

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->

    <h2 style="text-align: center;">
        公告更新  <!-- h2標題`,設定為置中-->
    </h2>

    <form align='center'>
        <!--
        分別建立3個要輸入的項目
        在input標籤中定義輸入屬性為文字
        給予欄位名稱及ID標記(用於對應資料庫內容)
        新增了value屬性，用於顯示預設的內容在表單輸入欄位中
        與資料庫連結後value屬性會直接引用特定公告的資料內容
        -->
        <label for="title">公告事項</label><br>
        <input type="text" name="title" id="title" value="公告001"><br><br>
        
        <label for="content">公告說明</label><br>
        <input type="text" name="content" id="content" value="這是公告001的說明"><br><br>
        
        <label for="person">公告人</label><br>
        <input type="text" name="person" id="person" value="小明"><br><br>
        
        <input type="submit" value="更新公告">
        <!-- 因為目前尚未連結資料庫，所以這個表單還無法收集資料 -->
        
    </form>
      
{% endblock %}  <!-- body區塊結束 -->
```
