# Monad

## pointed functor

在繼續後面的內容之前，我得向你坦白一件事：關於我們先前創建的容器類型上的 `of` 方法，我並沒有說出它的全部實情。真實情況是，`of` 方法不是用來避免使用 `new` 關鍵字的，而是用來把值放到*默認最小化上下文*（default minimal context）中的。是的，`of` 沒有真正地取代構造器——它是一個我們稱之為 *pointed* 的重要接口的一部分。

> *pointed functor* 是實現了 `of` 方法的 functor。

這裡的關鍵是把任意值丟到容器里然後開始到處使用 `map` 的能力。

```js
IO.of("tetris").map(concat(" master"));
// IO("tetris master")

Maybe.of(1336).map(add(1));
// Maybe(1337)

Task.of([{id: 2}, {id: 3}]).map(_.prop('id'));
// Task([2,3])

Either.of("The past, present and future walk into a bar...").map(
concat("it was tense.")
);
// Right("The past, present and future walk into a bar...it was tense.")
```

如果你還記得，`IO` 和 `Task` 的構造器接受一個函數作為參數，而 `Maybe` 和 `Either` 的構造器可以接受任意值。實現這種接口的動機是，我們希望能有一種通用、一致的方式往 functor 里填值，而且中間不會涉及到複雜性，也不會涉及到對構造器的特定要求。「默認最小化上下文」這個術語可能不夠精確，但是卻很好地傳達了這種理念：我們希望容器類型里的任意值都能發生 `lift`，然後像所有的 functor 那樣再 `map` 出去。

有件很重要的事我必須得在這裡糾正，那就是，`Left.of` 沒有任何道理可言，包括它的雙關語也是。每個 functor 都要有一種把值放進去的方式，對 `Either` 來說，它的方式就是 `new Right(x)`。我們為 `Right` 定義 `of` 的原因是，如果一個類型容器*可以* `map`，那它就*應該* `map`。看上面的例子，你應該會對 `of` 通常的工作模式有一個直觀的印象，而 `Left` 破壞了這種模式。

你可能已經聽說過 `pure`、`point`、`unit` 和 `return` 之類的函數了，它們都是 `of` 這個史上最神秘函數的不同名稱（譯者注：此處原文是「international function of mystery」，源自惡搞《007》的電影 *Austin Powers: International Man of Mystery*，中譯名《王牌大賤諜》）。`of` 將在我們開始使用 monad 的時候顯示其重要性，因為後面你會看到，手動把值放回容器是我們自己的責任。

要避免 `new` 關鍵字，可以借助一些標準的 JavaScript 技巧或者類庫達到目的。所以從這裡開始，我們就利用這些技巧或類庫，像一個負責任的成年人那樣使用 `of`。我推薦使用 `folktale`、`ramda` 或 `fantasy-land` 里的 functor 實例，因為它們同時提供了正確的 `of` 方法和不依賴 `new` 的構造器。

## 混合比喻

<img src="images/onion.png" alt="http://www.organicchemistry.com/wp-content/uploads/BPOCchapter6-6htm-41.png" />

你看，除了太空墨西哥卷（如果你聽說過這個傳言的話）（譯者注：此處的傳言似乎是說一個叫 Chris Hadfield 的宇航員在國際空間站做墨西哥卷的事，[視頻鏈接](https://www.youtube.com/watch?v=f8-UKqGZ_hs)），monad 還被喻為洋蔥。讓我以一個常見的場景來說明這點：

