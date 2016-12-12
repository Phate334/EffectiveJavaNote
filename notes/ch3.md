# Chapter3: Methods Common to All Objects #

[Object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html) 類別是被設計來擴展用，它的一些 nonfinal 方法 (equals、hashCode、toString、clone 和 finalize) 都有明確的目的 (詳見 API 文件)。該章節說明如何適當的 override 這些方法。 除了 Object 的方法外還會說明 [Comparable.compareTo](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html#compareTo-T-) 。

--------
## Item 8: Obey the general contract when overriding equals ##
當類別具有自己特有的 logical equality 時，且 superclass 並沒有提供才有必要自己實作 equals 。
通常 value class 實作 equals 是為了比較是否邏輯相等，而不是想知道是否指向同一個物件。

但有一種 value class 不需要複寫，那就是 (Item 1) 中提到受物件控制的類別，例如 Enum 。因為這種類別的 logical equality 跟 object identity 指的是同一件事，因此跟 Object 的 equals 是同樣意思。

根據 Object 的[規範](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-)， equals 應遵守以下的 equivalence relation:

1. Reflexive: 對於任何 non-null 的值，x.equals(x) => true。
2. Symmetric: 對於任何 non-null 的值，當 y.equals(x) => true，則 x.equals(y) => true。
3. Transitive: 對於任何 non-null 的值，當 x.equals(y) => true，且 y.equals(z) => true，則 x.equals(z) => true。
4. Consistent: 對於任何 non-null 的值，只要值沒被修改過，每次回傳的結果應一致。
5. 對於任何 non-null 的值，x.equals(null) => false。

由這些規範可以表列出一些 equals 的訣竅。

1. 使用 == 檢查傳進來的參數是否是自己本身，可以節省一些較昂貴的比較動作。
2. 使用 instanceof 比較參數的型態是否正確，也可以同時用來檢查參數是否為 null。
3. 將參數轉換成正確的型別。
4. 開始比較每個 關鍵 (significant) 欄位，全部檢查成功才返回 true。

除了 float 和 double 外的 primitive fieldsd 可以使用 == 比較，物件 reference fields 則可以用 equals，至於 float 和 double 可以使用 Float.compare 和 Double.compare 做比較。對於 array fields 可以使用 Arrays.equals 。

- [Why can't we use '==' to compare two float or double numbers](http://stackoverflow.com/questions/17898266/why-cant-we-use-to-compare-two-float-or-double-numbers)

因為某些欄位是允許為 null ，所以避免引發 NullPointerException 經常使用下列寫法做比較。

    (field == null ? o.field == null : field.equals(o.field))

假如兩個比較欄位經常是同一個物件，則下列作法會更有效率。

    (field == o.field || (field != null && field.equals(o.field)))

其他要注意的項目:

- 複寫 equals 時一定要複寫 hashCode (Item 9)。
- 讓 equals 的邏輯盡量保持簡單。
- 常見的錯誤是參數型別不是 Object，這樣不會複寫 Object 的 equals ，而是重載而已。

--------
## Item 9: Always override hashCode when you override equals ##

根據[規範](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--):

1. 只要在執行期間，只要被 equals 比較用到的欄位沒有更動，那麼每次呼叫應傳回同樣的整數。但同一支應用程式每次的執行過程可以回傳不同值。
2. 如果 equals 回傳 true ，則兩個物件的 hashCode 應回傳同樣的整數。
3. 但是如果 equals 回傳 false ，則兩者回傳的 hashCode 不一定要不同，只不過給予不同的回傳值可能可以提高 hash table 的效能。

因為兩個不同的物件在邏輯上是可能相等，但是如果使用 Object 類別的 hashCode 將會回傳不同的值，所以這時沒有複寫 hashCode 會違反第二條規定，

不正確複寫 hashCode ，多數 Hash 的集合可能無法正確運作，例如 HashMap 、 HashSet 和 Hashable 。
例如 HashMap.get() 時無法取得內容或取得不符期望的物件。

--------
## Item 10: Always override toString #

“a concise but informative representation that is easy for a person to read” [JavaSE6].

雖然 toString 方法並不像 equals 和 hashCode 那麼重要，但提供好的 toString 能幫助類別更好用，例如 print 、字串串聯符號、assert 或者被印出到終端時， toString 都會被自動選用。

--------
## Item 11: Override clone judiciously ##

[Object.clone()](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#clone--)

Cloneable 介面是為了供 mixin interface (Item 18) 所用，表示該物件允許複製。

直接呼叫 super.clone() 只會複製原物件中成員的參考，所以修改原物件會影響到複製出的內容。

(Item 17)

--------
## Item 12: Consider implementing Comparable ##

[Comparable](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html)

規則與 equals 類似，比較對象較小傳回*負值*，較大則傳回*正值*。 如果行為和 equals 不一致應該要明確說明。

    (x.compareTo(y) == 0) == (x.equals(y))
