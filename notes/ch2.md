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

有一些靜態方法或變數組成的類別，例如 [java.lang.Math](https://docs.oracle.com/javase/8/docs/api/java/lang/Math.html) 、 [java.util.Arrays](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html)，或是用 factory methods 產生介面的[java.util.Collections](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)。如果類別中沒有建構子，編譯器會自動產生，所以這幾個類別都用一個空的建構子並標上 private ，而 Math 甚至加上 final 防止被繼承。

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

## Item 5: Avoid creating unnecessary objects ##

盡可能重新使用物件而不是新建一個，例如 String 。

- [那些字串二三事](http://openhome.cc/Gossip/JavaEssence/String.htm)

除了重用 immutable 物件外，如果知道 mutable 物件不會再更改也應該盡量重用，例如以下書中範例的 Calendar、TimeZone 和 Date 物件，建立後可以不斷重複使用，而不是每次呼叫 isBabyBoomer 方法時都重新計算一次。

    class Person {
        private final Date birthDate;
        // Other fields, methods, and constructor omitted
        /**
        * The starting and ending dates of the baby boom.
        */
        private static final Date BOOM_START;
        private static final Date BOOM_END;
        static {
            Calendar gmtCal =
            Calendar.getInstance(TimeZone.getTimeZone("GMT"));
            gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
            BOOM_START = gmtCal.getTime();
            gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
            BOOM_END = gmtCal.getTime();
        }
        public boolean isBabyBoomer() {
            return birthDate.compareTo(BOOM_START) >= 0 &&
            birthDate.compareTo(BOOM_END) < 0;
        }
    }

另外要注意的是基本型態 (primitive type) 在 J2SE 5.0 後提供了 autoboxing 功能，如下範例編譯器會自動將 int 包裹 (wrap) 成 Integer。

    Integer temp = 10;

其他的基本型態 boolean、byte、short、char、long、float 和 double 都有對應的 Wrapper Type ，這雖然會使程式碼變得精簡，但這只是 Compiler sugar ，上述程式碼編譯後等同於:

    Integer temp = new Integer(10);

除了可能對效能造成影響外，也可能出現執行時期錯誤，更詳細範例可參考良葛格的 [第 4 章 從 autoboxing、unboxing 認識物件](https://github.com/JustinSDK/JavaSE6Tutorial/blob/master/docs/CH04.md)。

小節:

並非一定要避免建立物件，只不過建立小一點的物件或回收物件成本較便宜，以現代 JVM 的實作，如果能使得程式碼變得更簡潔這是好事。除非不得已，否則維護自己的 Object pool 會是一個壞主意，因為現代 JVM 已對垃圾處理做很好的優化，所以除非像是建立資料庫的連結物件過程很昂貴，那存放池中重用是合理。