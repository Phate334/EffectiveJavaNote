# Chapter4: Classes and Interfaces #

## Item 13: Minimize the accessibility of classes and members ##

良好的模組設計應該要對外隱藏所有內部細節， API 和 implementation 應該要完全分開，讓模組之間只透過 API 溝通，稱為 Information hiding 或 Encapsulation (封裝)。

設計模組首先讓每個類別或成員不被外部存取，使可訪問的層級最小化。當把成員宣告為 public 就要考慮未來改版的相容性。

Access level 從小到大:

1. private - 只有在該 class 內部才能存取
2. package-private - 只有在 package 內才能存取，沒有聲明時的預設層級。 
3. protected - 子類別也可以存取，但有限制 [JLS, 6.6.2](https://docs.oracle.com/javase/specs/jls/se7/html/jls-6.html#jls-6.6.2)。
4. public

不良的設計可能會使內容在實作 Serializable 介面時洩漏 (Item 74、75)。

[JLS, 8.4.8.3](https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.4.8.3) 當複寫方法時，子類方法的層級不應低於父類，這樣可以確保
任何使用父類方法的地方都可以使用子類的物件。

--------
## Item 14:  In public classes, use accessor methods, not public fields ##

撰寫 Degenerate classes 時要注意的情況，讓這種 public class 遍布在程式碼中的後果， (Item 55) 是個例子。

永遠不要讓 public class 中的成員直接暴露在外，尤其是可變動的欄位。

--------
## Item 15:  Minimize mutability ##

Immutable class 在建立物件時就必須提供所有資訊，在物件的生命週期中都不能被更動。 在 Java 中像是 String 或 primitive type 的包裝類都是。不可變的特性讓類別更容易設計也更安全。

Immutable class 有以下五條規則:

1. 不提供任何會修改物件狀態的方法 (mutator)。
2. 使類別不可被繼承。
3. 所有成員都設為 final 。
4. 所有成員都設為 private 。
5. 取得 (accessor) 可變成員的方法。不要用使用者提供的物件參考來初始化這些成員，也不要在 accessor 中回傳這些參考。在 constructors、accessors、readObject (Item 76) 中應該使用 defensive copy (Item 39)。

當需要改變不可變物件的狀態時通常都是回傳一個新的物件，例如 String 的串聯。這種 functional 的作法使得這類物件可以簡單維護。

不可變物件是 thread-safe ，不需要要求同步且可以被自由共享。