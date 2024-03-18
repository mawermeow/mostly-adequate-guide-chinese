# Hindley-Milner 類型簽名

## 初識類型

剛接觸函數式編程的人很容易深陷類型簽名（type signatures）的泥淖。類型（type）是讓所有不同背景的人都能高效溝通的元語言。很大程度上，類型簽名是以 「Hindley-Milner」 系統寫就的，本章我們將一起探究下這個系統。

類型簽名在寫純函數時所起的作用非常大，大到英語都不能望其項背。這些簽名輕輕訴說著函數最不可告人的秘密。短短一行，就能暴露函數的行為和目的。類型簽名還衍生出了 「自由定理（free theorems）」 的概念。因為類型是可以推斷的，所以明確的類型簽名並不是必要的；不過你完全可以寫精確度很高的類型簽名，也可以讓它們保持通用、抽象。類型簽名不但可以用於編譯時檢測（compile time checks），還是最好的文檔。所以類型簽名在函數式編程中扮演著非常重要的角色——重要程度遠遠超出你的想象。

JavaScript 是一種動態類型語言，但這並不意味著要一味否定類型。我們還是要和字符串、數值、布爾值等等類型打交道的；只不過，語言層面上沒有相關的集成讓我們時刻謹記各種數據的類型罷了。別擔心，既然我們可以用類型簽名生成文檔，也可以用注釋來達到區分類型的目的。

JavaScript 也有一些類型檢查工具，比如 [Flow](http://flowtype.org/)，或者它的靜態類型方言 [TypeScript](http://www.typescriptlang.org/) 。由於本書的目標是讓讀者能夠熟練使用各種工具去書寫函數式代碼，所以我們將選擇所有函數式語言都遵循的標準類型系統。

## 神秘的傳奇故事

從積塵已久的數學書，到浩如煙海的學術論文；從每周必讀的博客文章，到源代碼本身，我們都能發現 Hindley-Milner 類型簽名的身影。Hindley-Milner 並不是一個複雜的系統，但還是需要一些解釋和練習才能完全掌握這個小型語言的要義。

```js
// capitalize :: String -> String
var capitalize = function(s){
return toUpperCase(head(s)) + toLowerCase(tail(s));
}

capitalize("smurf");
//=> "Smurf"
```

這裡，`capitalize` 接受一個 `String` 並返回了一個 `String`。先別管實現，我們感興趣的是它的類型簽名。

在 Hindley-Milner 系統中，函數都寫成類似 `a -> b` 這個樣子，其中 `a` 和`b` 是任意類型的變量。因此，`capitalize` 函數的類型簽名可以理解為「一個接受 `String` 返回 `String` 的函數」。換句話說，它接受一個 `String` 類型作為輸入，並返回一個 `String` 類型的輸出。

再來看一些函數簽名：

```js
// strLength :: String -> Number
var strLength = function(s){
return s.length;
}

// join :: String -> [String] -> String
var join = curry(function(what, xs){
return xs.join(what);
});

// match :: Regex -> String -> [String]
var match = curry(function(reg, s){
return s.match(reg);
});

// replace :: Regex -> String -> String -> String
var replace = curry(function(reg, sub, s){
return s.replace(reg, sub);
});
```

`strLength` 和 `capitalize` 類似：接受一個 `String` 然後返回一個 `Number`。

至於其他的，第一眼看起來可能會比較疑惑。不過在還不完全瞭解細節的情況下，你盡可以把最後一個類型視作返回值。那麼 `match` 函數就可以這麼理解：它接受一個 `Regex` 和一個 `String`，返回一個 `[String]`。但是，這裡有一個非常有趣的地方，請允許我稍作解釋。

對於 `match` 函數，我們完全可以把它的類型簽名這樣分組：

```js
// match :: Regex -> (String -> [String])
var match = curry(function(reg, s){
return s.match(reg);
});
```

是的，把最後兩個類型包在括號里就能反映更多的信息了。現在我們可以看出 `match` 這個函數接受一個 `Regex` 作為參數，返回一個從 `String` 到 `[String]` 的函數。因為 curry，造成的結果就是這樣：給 `match` 函數一個 `Regex`，得到一個新函數，能夠處理其 `String` 參數。當然了，我們並非一定要這麼看待這個過程，但這樣思考有助於理解為何最後一個類型是返回值。

