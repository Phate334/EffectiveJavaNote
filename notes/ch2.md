# Creating and Destroying Objects #

## Item 1: Consider static factory methods instead of constructors ##

- [以 Java 程式範例來探討 Design Pattern： Factory Method](http://www.dotspace.idv.tw/Jyemii/patternscolumn/articles/FactoryMethodForJava.htm)

好處:

1. overloading constructor 使得 API 難以記憶，static factory methods 讓每個方法都有獨特名稱。
2. 不用每次都重複建立新的物件，對效能影響大。
3. 可以回傳 nonpublic 的類別物件，用來隱藏實作讓API更簡潔，例如: Java Collections Framework。也能依照需求回傳不同的 subtype  讓程式碼更好維護，例如 java.util.EnumSet 沒有建構子，而 static factories 會依照輸入決定回傳的物件類型，使用者只須要知道回傳的是 EnumSet 的子類，改版時也方便維護。
4. 簡化產生 parameterized type 物件的方法。

缺點:

1. 主要缺點是沒有建構子能被繼承，例如 Collections Framework 只提供 factory methods 。
2. 在文件中不像建構子一樣無法與其他靜態方法區分，可以使用一些常用的命名規則，例如 valueOf 、 getInstance 等。

