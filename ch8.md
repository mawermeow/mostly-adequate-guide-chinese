# 特百惠

（譯者注：特百惠是美國家居用品品牌，代表產品是塑料容器。）

## 強大的容器

<img src="images/jar.jpg" alt="http://blog.dwinegar.com/2011/06/another-jar.html" />

我們已經知道如何書寫函數式的程序了，即通過管道把數據在一系列純函數間傳遞的程序。我們也知道了，這些程序就是聲明式的行為規範。但是，控制流（control flow）、異常處理（error handling）、異步操作（asynchronous actions）和狀態（state）呢？還有更棘手的作用（effects）呢？本章將對上述這些抽象概念賴以建立的基礎作一番探究。

首先我們將創建一個容器（container）。這個容器必須能夠裝載任意類型的值；否則的話，像只能裝木薯布丁的密封塑料袋是沒什麼用的。這個容器將會是一個對象，但我們不會為它添加面向對象觀念下的屬性和方法。是的，我們將把它當作一個百寶箱——一個存放寶貴的數據的特殊盒子。

```js
var Container = function(x) {
this.__value = x;
}

Container.of = function(x) { return new Container(x); };
```

這是本書的第一個容器，我們貼心地把它命名為 `Container`。我們將使用 `Container.of` 作為構造器（constructor），這樣就不用到處去寫糟糕的 `new` 關鍵字了，非常省心。實際上不能這麼簡單地看待 `of` 函數，但暫時先認為它是把值放到容器里的一種方式。

我們來檢驗下這個嶄新的盒子：

```js
Container.of(3)
//=> Container(3)


Container.of("hotdogs")
//=> Container("hotdogs")


Container.of(Container.of({name: "yoda"}))
//=> Container(Container({name: "yoda" }))
```

如果用的是 node，那麼你會看到打印出來的是 `{__value: x}`，而不是實際值 `Container(x)`；Chrome 打印出來的是正確的。不過這並不重要，只要你理解 `Container` 是什麼樣的就行了。有些環境下，你也可以重寫 `inspect` 方法，但我們不打算涉及這方面的知識。在本書中，出於教學和美學上的考慮，我們將把概念性的輸出都寫成好像 `inspect` 被重寫了的樣子，因為這樣寫的教育意義將遠遠大於 `{__value: x}`。

在繼續後面的內容之前，先澄清幾點：

* `Container` 是個只有一個屬性的對象。儘管容器可以有不止一個的屬性，但大多數容器還是只有一個。我們很隨意地把 `Container` 的這個屬性命名為 `__value`。
* `__value` 不能是某個特定的類型，不然 `Container` 就對不起它這個名字了。
* 數據一旦存放到 `Container`，就會一直待在那兒。我們*可以*用 `.__value` 獲取到數據，但這樣做有悖初衷。

如果把容器想象成玻璃罐的話，上面這三條陳述的理由就會比較清晰了。但是暫時，請先保持耐心。

## 第一個 functor

一旦容器里有了值，不管這個值是什麼，我們就需要一種方法來讓別的函數能夠操作它。

```js
// (a -> b) -> Container a -> Container b
Container.prototype.map = function(f){
return Container.of(f(this.__value))
}
```

這個 `map` 跟數組那個著名的 `map` 一樣，除了前者的參數是 `Container a` 而後者是 `[a]`。它們的使用方式也幾乎一致：

```js
Container.of(2).map(function(two){ return two + 2 })

