# Chapter3: Methods Common to All Objects #

[Object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html) 類別是被設計來擴展用，它的一些 nonfinal 方法 (equals、hashCode、toString、clone 和 finalize) 都有明確的目的 (詳見 API 文件)。該章節說明如何適當的 override 這些方法。 除了 Object 的方法外還會說明 [Comparable.compareTo](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html#compareTo-T-) 。

--------
## Item 8: Obey the general contract when overriding equals ##
當類別具有自己特有的 logical equality 時，且 superclass 並沒有提供才有必要自己實作 equals 。
通常 value class 實作 equals 是為了比較是否邏輯相等，而不是想知道是否指向同一個物件。

但有一種 value class 不需要複寫，那就是 (Item 1) 中提到受物件控制的類別，例如 Enum 。因為這種類別的 logical equality 跟 object identity 指的是同一件事，因此跟 Object 的 equals 是同樣意思。

根據 Object 在 JavaSE6 的規範， equals 應遵守以下的 equivalence relation:

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

因為某些引用對象是允許為 null ，所以避免引發 NullPointerException 經常使用下列寫法做比較。

    (field == null ? o.field == null : field.equals(o.field))

假如兩個比較欄位通常是同一個對象，則下列作法會更有效率。

    (field == o.field || (field != null && field.equals(o.field)))

其他要注意的項目:

- 複寫 equals 時一定要複寫 hashCode (Item 9)。
- 讓 equals 的邏輯盡量保持簡單。
- 常見的錯誤是參數型別不是 Object，這樣不會複寫 Object 的 equals ，而是重載而已。