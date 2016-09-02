# Chapter2: Creating and Destroying Objects #

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

--------
## Item 2: Consider a builder when faced with many constructor parameters ##

Static factories 和 constructors 遇到大量可選參數時並沒辦法容易擴充。以往會使用 telescoping constructor pattern 替每種組合撰寫建構子，這樣不但難以維護且遇到相鄰兩種相同型態的參數也容易讓使用者誤會。

第二種方法是 JavaBeans pattern ，建構子只傳入必要參數，其後使用 setter 設定選用參數，雖然囉嗦但簡單易讀。這種模式最大缺點是建構物件的過程並沒有經過強制檢查，產生出來的物件可能不一致，使得程式碼難以除錯。雖然可以透過凍結的方式等物件結構完整後才允許使用，但方式較笨拙不被多數採用。

Builder pattern 結合 telescoping constructor pattern 的安全性和 JavaBeans pattern 的可讀性。

1. 使用者利用 constructor 或 static factory 輸入必要參數得到一個 builder 物件，
2. 呼叫 builder 的 setter 去設定其他選用的參數。
3. 最後呼叫 builder 中不帶參數的方法去產生物件，產生的物件可以是 immutable 。

- Example:

    public class SampleObj {
        private final int requirementA;
        private final int optionA;
        private final int optionB;

        public static class Builder{
            // Required parameters
            private final int requirementA;
            
            // Optional parameters - initialized to default values
            private final int optionA = 0;
            private final int optionB = 0;
            public Builder(int requirementA) {
                this.requirementA = requirementA;
            }

            public Builder setOptionA(int a){
                optionA = a;
                return this;
            }
            public Builder setOptionB(int b){
                optionB = b;
                return this;
            }
            
            public SampleObj build(){
                return new SampleObj(this);
            }
        }
        private SampleObj(Builder builder){
            requirementA = builder.requirementA;
            optionA = builder.optionA;
            optionB = builder.optionB;
        }
    }


    SampleObj sample = new SampleObj.Builder(240).setOptionA(100).setOptionB(200).build();

--------
## Item 3: Enforce the singleton property with a private constructor or an enum type ##

- [Singleton Pattern](http://openhome.cc/Gossip/DesignPattern/SingletonPattern.htm)

在 JAVA5 前有兩種方法實作 Singleton ，引用自書中範例。

    // Singleton with public final field
    public class Elvis {
        public static final Elvis INSTANCE = new Elvis();
            private Elvis() { ... }
        public void leaveTheBuilding() { ... }
    }

或是使用 static factory method 的 public 成員建立物件。

    // Singleton with static factory
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
        private Elvis() { ... }
        public static Elvis getInstance() { return INSTANCE; }
        public void leaveTheBuilding() { ... }
    }

要避免反序列化後重新建立物件。

    private Object readResolve() {
        // Return the one true Elvis and let the garbage collector
        // take care of the Elvis impersonator.
        return INSTANCE;
    }

JAVA5 後可以使用 enum 來建立 singleton 物件，下面例子等同 public final field 。

    // Enum singleton - the preferred approach
    public enum Elvis {
        INSTANCE;
        public void leaveTheBuilding() { ... }
    }

## Item 4: Enforce noninstantiability with a private constructor ##

有一些靜態方法或變數組成的類別，例如 [java.lang.Math](https://docs.oracle.com/javase/8/docs/api/java/lang/Math.html) 或 [java.util.Arrays](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html)，或是用 factory methods 產生介面的[java.util.Collections](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)。如果類別中沒有建構子，編譯器會自動產生，所以這幾個類別都用一個空的建構子並標上 private ，而 Math 甚至加上 final 防止被繼承。

不能使用抽象的方式來禁止使用者實例化，因為這樣還是能被繼承的子類別產生實例，而且這樣會誤導使用者。
保險的方式是在建構子中丟出 AssertionError 例外，這能確保該類別不會被實例化，也可以保證繼承的類別不會呼叫父類別的建構子。

    // Noninstantiable utility class
    public class UtilityClass {
        // Suppress default constructor for noninstantiability
        private UtilityClass() {
            throw new AssertionError();
        }
        ... // Remainder omitted
    }