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

SampleObj sampleObj = new SampleObj.Builder(240)
        .setOptionA(100).setOptionB(200).build();
