---
aliases:
date: 2022-04-01
update:
author: Jay Hsieh
language:
sourceurl: https://hackmd.io/@XEBT9JooT1SIims9CsXXGA/designpatternseries1
tags:
  - OOP
---

# OOP 六種不同的關係

根據 OOP 的三大特性: **封裝**、**繼承**、**多形**，歸納和擴展出類與類之間的六種不同的關係

## 前言

廣泛地說，類與類之間的關係其實只有三種: **繼承**、**實現**、**關聯** (**依賴**是一種弱關聯，**聚合**和**組合**是關聯中的特例)。
繼承與實現是縱向關係，而關聯是橫向關係

pycharm 裡畫 .puml 的寫法

- 繼承 Inheritance　　　　　  --|>
- 實現 Implementation　　　  ..|>
- 關聯 Association　　　　　 -->
- 依賴 (弱關聯) Dependency　..>
- 聚合 Aggregation　　　　　o--
- 組合 Composition　　　　　\*--

複習一下 OOP 的 UML 圖示

![[OOP的UML.png]]

1. 繼承關係 Inheritance: 類與類，或接口與接口之間的父子關係
2. 實現關係 Implementation: 接口與類定義好的操作集合，由實現類完成接口或類的具體操作
3. 依賴關係 Dependency: 在局部變量、方法的參數 (不會出現在 init)，或者對靜態方法的調用中實現
4. 關聯關係 Association: 在類的成員變量中實現 (會在 init 時綁定關係)，可以雙向也可以是單向
5. 聚合關係 Aggregation: 強關聯，整體與部分的關係，但是可以分離
6. 組合關係 Composition: 更強關聯，整體與部分的關係，不可分離

---

## 1. 繼承關係 (Inheritance)

1. 子類繼承自抽象類或普通類
2. 子接口繼承自父接口: 適用於 Java

**Dog is an Animal**

![[Inheritance.png]]

```python
from abc import ABC, abstractmethod

# 子類繼承自抽象類 ABC
# 必須實現父類中@abstractmethod修飾的抽象方法 
# 否則會出現錯誤訊息
class Animal(ABC):
    def __init__(self):
        self.name = "Animal"

    @abstractmethod
    def run(self):
        pass

    def play(self):
        print(self.name + " is playing.")

class Dog(Animal):
    def __init__(self):
        super().__init__()
        self.name = "Dog"

    def run(self):
        print(self.name + " is running.")

    def __bark(self):
        print(self.name + " is barking.")

class Cat(Animal):
    def __init__(self):
        super().__init__()
        self.name = "Cat"

    def run(self):
        print(self.name + " is running.")

    def play(self):
        print(self.name + " is playing.")

    def __jump(self):
        print(self.name + " is jumping.")

dog = Dog()
cat = Cat()
dog.run()
dog.play()
cat.run()
cat.play()
```

```python
# 子類繼承自普通類，子類方法重寫父類同名非私有方法
class Animal:
    def __init__(self):
        self.name = "Animal"

    def run(self):
        print(self.name + " is running.")

    def play(self):
        print(self.name + " is playing.")

class Dog(Animal):
    def __init__(self):
        super().__init__()
        self.name = "Dog"

    def run(self):
        print(self.name + " is running.")

    def __bark(self):
        print(self.name + " is barking.")

class Cat(Animal):
    def __init__(self):
        super().__init__()
        self.name = "Cat"

    def play(self):
        print(self.name + " is playing.")

    def __jump(self):
        print(self.name + " is jumping.")

dog = Dog()
cat = Cat()
dog.run()
dog.play()
cat.run()
cat.play()
```

---

## 2. 實現關係 (Implemention)

1. 類具體實現接口中所聲明的操作，如 Java 中支持原生 interface，可以直接 implement
2. 類具體實現接口類中所聲明的操作，如 python 中無原生 interface，這裡的接口類更多的是指邏輯上的契約或規範

**Benz, BMW implement Car**

![[Implemention.png]]

```python
from abc import ABC, abstractmethod

class Car(ABC):
    @abstractmethod
    def engine(self):
        raise NotImplementedError

class Benz(Car):
    def engine(self):
        print("Benz is running.")

        
class BMW(Car):
    def engine(self):
        print("BMW is running.")

benz = Benz()
bmw = BMW()
benz.engine()
bmw.engine()
```

---

## 3. 依賴關係 (Dependency)

一個類 A 使用到另一個類 B，僅僅是 " 使用到 "，類 B 本身不屬於類 A

依賴關係是**偶然性**、**臨時性**的，但是因為類 B 本身被類 A 引用到，所以 **B 類的變化也會影響到類 A**。

**Person uses a Plane**

![[Dependency.png]]

1. 類 B 作為類 A 的參數

```python
class Plane:
    def __init__(self):
        self.name = "plane"

    def fly(self):
        print(self.name + " is flying.")

        
class Person:
    def __init__(self):
        self.name = "person"

    def flying(self, local_plane: Plane):
        local_plane.fly()
        print(self.name + " uses a plane.")

person = Person()
plane = Plane()
person.flying(plane)
```

2. 類 B 作為類 A 方法的局部變量

```python
class Plane:
    def __init__(self):
        self.name = "plane"

    def fly(self):
        print(self.name + " is flying.")

        
class Person:
    def __init__(self):
        self.plane = None

    def flying(self):
        self.plane = Plane()
        self.plane.fly()

        
person = Person()
person.flying()
```

3. 類 A 調用類 B 的靜態方法

```python
class Person():
    def flying(self):
        Plane.fly()
        
class Plane():
    @staticmethod
    def fly():
        print("The plane is flying.")

person = Person()
person.flying()
```

---

## 4. 關聯關係 (Association)

依賴關係與關連關係的區別有動靜之分
依賴關係的偶然性和臨時性說明了動態性
關聯關係的長期性、擁有性靜態地展示了對被關聯類的引用

關聯關係一般是**長期性**的、**擁有性**的關係，被關聯類 B 以類的**屬性形式**出現在關聯 A 中 (init or construtor 中綁定關係)，關聯可以是單向，也可以是雙向的

1. 單向關聯: 單向擁有關係，只有一個類知道另一個類的屬性與方法

**School has a student**

![[Association_單向.png]]

```python
class Student:
    def __init__(self):
        self.title = "Seno"

    def study(self):
        print("Studying...")

class School:
    def __init__(self):
        self.address = "ABC Street"
        self.student = Student()

    def act(self):
        print(self.student.title)  
        self.student.study()

school = School()
school.act()
```

2. 雙向關聯: 雙向擁有關係，雙方都知道對方的屬性和方法

    ![[Association_雙向.png]]

```python
class Student:
    def __init__(self):
        self.school = School()

class School:
    def __init__(self):
        self.student = Student()
```

3. 自身關聯: 自己關聯自己，這種情況比較少用到，如鏈表

    ![[Association_自身.png]]

```python
class Node:
    def __init__(self):
        self.next = Node()
```

4. 多重性關聯: 表示兩個類的對象在書量上的對應關係，多重性可以在關聯線上用數字範圍表示

    ![[Association_多重性.png]]

多重性 (multiplicity):
(1) 1 -> 僅為 1
(2) * -> 從 0 到無窮大
(3) 0..1 -> 0 或者 1
(4) n..m -> [n,m] 之間的任何數

---

## 5. 聚合關係 (Aggregation)

聚合關係是關聯關係的特例

普通**關聯關係**的兩個類一般處於同一平等的層次上
而**聚合關係**的兩個類處於不同的層次，是整體與部分的關係
聚合關係中的整體和部分是可以分離的，生命週期也是可以互相獨立的

1. 類 A 由類 B 聚合而成，類 A 包含有類 B 的全局對象，但類 B 的對象可以不在類 A 創建的時刻創建
2. UML 中使用空心菱形指向類 A(整體)，實線指向類 B(部分)

**School owns Students**

![[Aggregation.png]]

```python
class School:
    def __init__(self):
        self.__students = []

    def add_student(self, student):
        self.__students.append(student)

class Student:
    pass

student = Student()
school = School()
school.add_student(student)
```

---

## 6. 組合關係 (Composition)

組合關係也是關聯關係的特例，屬於強聚合，也表示整體與部分的關係

組合關係中的整體和部分是**不可分離的**
**整體生命週期的結束時也是部分的生命週期結束時**

1. 組合關係在代碼上體現為整體類的構造方法中直接實例化成員類，因為類組合關係中整體與部分是共生的
2. 注意組合關係與關聯關係的不同

**Brain is a part of Person**

![[Composition.png]]

```python
class Person:
    def __init__(self):
        print("Person Iinitialization Start")
        self.__brain = Brain()
        print("Person Iinitialization End")

    def run(self):
        print("Running...")

class Brain:
    def __init__(self):
        print("Brain Initialization Start")
        print("Brain Initialization End")

person = Person()
person.run()
```

---

# GRASP、SOLID 和 Design Pattern 關係

GRASP 九大原則、SOLID 5 大原則、GoF 23 種設計模式。
GRASP 處於最上層，SOLID 基於它再進一步細化闡述，GOF 再根據這些原則進一步的歸納出更具體的模式。

## GRASP 9 大原則

GRASP 是 design pattern 的基礎

GRASP 的 9 個原則，以**低耦合**、**高內聚**原則爲核心

1. **information 信息專家原則** （若一個類擁有完成這個職責所需要的數據，則把這個職責分配給這個類。）
2. **creator 創造者原則**（類對象職責委託給哪個誰，誰應該負責產生某個類的實例）
3. **low coupling 低耦合原則**（不具有操作性，是一種評價原則。極端情況，沒有耦合，就會形成一個很差的設計，一個類完成全部工作。若穩定或流傳度廣的，才可以有高耦合，例如 系統軟件和 JDK。）
4. **high cohesion 高內聚原則**（功能內聚）
5. **controller 控制器原則**（事件的接受和處理委託給一個對象）
6. **polymorphism 多態原則**（根據類型變化分配職責）
7. **pure fabrication 純虛構**（解決高內聚和低耦合的矛盾，基於功能進行劃分）
8. **indirect 中介原則**（建立中間對象協調 2 個對象間的交互，避免高耦合度）
9. **protected variations 受保護變量原則**（類似 OCP 開閉原則，找到變化點，統一的用接口封裝，通過接口擴展來擴展新功能。）

---

## SOLID 5 大原則

### Single Responsibility Principle (SRP) 單一職責原則

體現**高內聚**，**低耦合**
每個物件，不管是類別、函數，負責的功能，都應該**只做一件事 or 業務功能**。
對函數而言，一個函數內，同時做了兩件以上的事情。當發生錯誤時，很難快速定位錯誤的原因。另外，也容易間接導至程式碼的可閱讀性降低。

### Open-Close Principle (OCP) 開放封閉原則

用意為代碼擴充

**對擴展開放，對修改關閉。**
變化通過擴展實現，變化不應影響已有代碼

- 優點：降低修改風險；只有新增程式碼，舊有程式因為沒修改，所以理論上問題當然會比較少
- 潛在問題：擴展的情境並不一定在設計階段就會發現，常常要到了需求調整才會知道

### Liscov Substitution Principle (LSP) 里氏替換原則

父類子類設計原則

任何使用父類的地方，都可以無差別的使用子類替換。
當程式中使用一個類別，那麼**將它替換成它的子類別不應該破壞程式原有的行為**。
這樣==我們在寫程式的時候，可以先宣告父類別的物件，而在 runtime 時指到子類別的實例==，跟物件導向的多型 polymorphism 特性有密切相關。

該原則目的: **提高強健性**
子類擴展新功能，但儘量不要重寫/重載父類接口。克服了重寫父類造成的複用性變差的缺點；同時，避免了擴展子類不會對已有系統引入新的錯誤。

- child class 必須完全實作 parent class 的方法
- child class 可以有屬於自己的屬性和方法
- 覆寫或實作 parent class 的 method 時，*參數* 要與 parent class 定義的一樣，或是更**寬鬆**。
  比方說：parent class 的定義是 List，child class 則可以是 List 或 Collection。**Param type: child > parent**
- 覆寫或實作 parent class 的 method 時，*回傳結果* 要跟 parent class 定義的一樣，或是**縮小**。
  比方說：parent class 是回傳 List ，child class 則可以回傳 List 或 ArrayList。**Return type: parent > child**

### Interface Segregation Principle (ISP) 介面隔離原則

接口的設計原則

不應強迫客戶依賴他們不需要的接口。大接口拆分成更小更具體的接口

- 優點：在需要多型時，會比較容易為 class 實作對應的 method

### Dependency Inversion Principle (DIP) 依賴反轉原則

接口和類的設計原則

inversion 是相對傳統的結構化方法中的高層模塊依賴低層模塊，進而導致依賴於具體實現細節。
DIP 原則，**要依賴於抽象，不依賴具體**。**要針對接口編程，不針對實現編程**。**具體實現依賴於抽象，高層模塊和低層模塊都依賴於抽象 (abstractions)**。

1. 高階模組／類別（high-level classes/modules）不應該依賴於低階模組/類別（low-level modules/classes）
2. 兩者皆應該依賴於一抽象介面（abstraction）
3. 而抽象介面不應該依賴於實作細節（implementation details），實作細節應該依賴於抽象介面。

這樣當我們今天必須要改變、更換低階模組時，也不會影響到高階模組的運作。這其實就是所謂的「Program to an interface, not an implementation」。

- 依賴：模塊 Package A 調用模塊 Package B，稱爲 A 依賴 B。
- 耦合：依賴也就是耦合。
- 低層模塊：實現了基本的或初級的操作。
- 高層模塊：封裝了複雜邏輯，依賴於低層模塊。

![[Dependency Inversion Principle.png]]

DIP 中提出的評價軟件的定性描述:

- 靈活性 Flexible，魯棒性 Robust，可重用性 Reusable
- 僵化 Rigidity，脆弱 Fragility，複用性差 Immobility

### Least Knowledge Principle (LKP) SOLID 之外的最小知識原則

Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.
每個單元應該只對其他單元有有限的知識：只有與當前單元「密切」相關的單元。

---

## GoF 設計模式

### Creational Design Patterns 創建型模式

是跟物件創立（object creation）的過程有關的模式，能夠提升已有代碼的靈活性和可覆用性

#### 1. Factory Method

![[Factory Method.png]]

不用吧，明明就有兩個比較簡單的設計方法呀

1. **讓陸路運輸公司獨立生產貨車，設計貨車的送貨方法；而海路運輸公司獨立生產船，設計船的送貨方法呢？**
    Ans: 陸路運輸公司、海路運輸公司都自己寫了一份送貨方法，不只多餘，每次要改什麼還要改兩次。如果之後要加入飛機空運，送貨方法再寫一次，崩潰。
2. **運輸總公司一起生產貨車、船，設計一套送貨邏輯給船跟貨車一起用呢？**
    Ans: 送貨邏輯裡面就會夾雜很多 if/else：看到貨車就跑陸路，看到船就走海路。之後又要加入飛機空運，再度崩潰。

在一般專案初期，我們可以先使用較簡單的 Factory Method 來創建 Object，之後如果想要更靈活地創建 Object，我們也可以改用較複雜的 Abstract Factory，Prototype，或是 Builder 等創建型設計模式。

- 符合單一責任原則 (SRP)
  創建 product 的程式碼都在同一個地方 (factory_method)，而使其更容易維護
- 符合開關原則 (OCP)
  因為送貨方法和 product 類型無關，所以就算加入新的 product 類型也無需更改送貨方法
- 符合里氏替換原則 (LSP)
  工廠方法 factory_method 返回 Transport 的子類

[程式碼](https://github.com/jayhsieh/GoF-Design-Patterns-/tree/main/Creational%20design%20patterns/Factory_Method)

#### 2. Abstarct Factory

將多個工廠方法透過抽象工廠整合

![[Abstarct Factory.png]]

抽象工廠模式適合應用場景

1. 如果代碼需要與多個不同系列的相關產品交互，但是由於無法提前獲取相關資訊，或者出於對未來擴展性的考慮，你不希望代碼基於產品的具體類進行構建，在這種情況下，你可以使用抽象工廠。

> 抽象工廠為你提供了一個介面，可用於創建每個系列產品的物件。
> 只要代碼通過該介面創建物件，那麼你就不會生成與應用程式已生成的產品類型不一致的產品。

2. 如果你有一個基於一組抽象方法的類，且其主要功能因此變得不明確，那麼在這種情況下可以考慮使用抽象工廠模式。

> 在設計良好的程式中，每個類僅負責一件事。
> 如果一個類與多種類型產品交互，就可以考慮將工廠方法抽取到獨立的工廠類或具備完整功能的抽象工廠類中。

設計

1. 讓工廠類別（Factory Class）去做建立物件的細節，至於使用者只要使用 Factory 去建立需要的物件，不需要知道物件怎麼建立的，或是細節
2. 常用的工廠模式：
    a) Static factory（兩種使用方式）: 藉由參數來選擇要建立哪種連線。直接呼叫對應的方法。
    b) Abstract factory: 建立一個抽象工廠，再各自建立擴充的工廠來覆寫它
    c) Generic factory

優缺點

1. Pros:
    A) 確保同一工廠生成的產品相互匹配
    B) 避免用戶端和具體產品代碼的耦合
    C) SRP，將產品生成代碼抽取到同一位置，使得代碼易於維護
    D) OCP，向應用程式中引入新產品變體時，你無需修改用戶端代碼
2. Cons: 由於採用該模式需要向應用中引入眾多介面和類，代碼可能會比之前更加複雜

[程式碼](https://github.com/jayhsieh/GoF-Design-Patterns-/tree/main/Creational%20design%20patterns/Abstract_Factory)

#### 3. Singleton

單例模式、單件模式，**保證一個類只有一個實例**，並提供一個訪問該實例的全局節點
很多時候會希望一個類別只會有唯一一個實體，像是**DB 的接口、應用程式的偏好設定**

想要確保一個類別只有一個實體的話，要怎麼做到呢？首先標準的物件導向語言都有提供 private, protected 等關鍵字來限制存取範圍（通常稱為 scope），只要使用以上關鍵字來限制建構子的存取範圍，再搭配公開的靜態方法 (public static method)、靜態成員變數（static field），就能讓外部存取指定的實例，而且確保該實例是全域唯一的實體。

```csharp

class DBHelper {

   //通常我們會命名該實體為 'instance' 
   private static DBHelper instance;
   
   //只能透過這個 method 來拿到唯一實體
   public static DBHelper getInstance() {
      if (instance == null) {
          instance = new DBHelper();
      }
      
      return instance;
   }
   
   //建構子不能公開
   private DBHelper() {}
   
   //DBHelper 提供的 public method, 
   public boolean insert(SomeBean bean) {
      ...
   }
}
```

優點

1. 簡單、方便。提供一個唯一的的入口來操作物件。不管何時何地都能透過 XXX.getInstance() 來進行操作，不用經過一系列的物件初始化動作
2. 確保資料的正確性，假設有一個物件存放著使用者的登入狀態，如果再建立另一個一樣的物件將無法確保他們都會有一樣的登入狀態。因此產生不同步的行為

缺點

1. 不要在 Singleton 裡寫複雜邏輯，Singleton 應該隨時保持單純，可以把他當成簡單的 wrapper ，通常可以搭配 Facade pattern 一起使用
2. Singleton 不好寫測試
3. 不要到處使用 getInstance()，相對的應該考慮使用 dependency injection 將需要的類別注入到將要使用的類別，使用 getInstnace() 將會使得單元測試極為困難

注意

1. Singleton 要解決的問題是唯一實例。同一個系統裡面只會有同一種狀態，但是由於他方便的特性（可以隨時呼叫靜態方法），會讓大家開始濫用。因此在使用時要隨時注意有沒有犯了文章上面的錯誤。
2. 另一方面，很多語言都有提供 DI 以及 IOC 框架，這些框架可以幫你建立實例，只要預先寫好物件的相依關係即可。同時還可以幫你解決 Singleton 要做的事情，可以不用自己實作 。但是要注意一但用了這些框架，Singleton 所提供的保護將不會存在，如果有一個新人不知道框架的運作方式的話，還是有可能會有問題的。

- [程式碼](https://github.com/jayhsieh/GoF-Design-Patterns-/tree/main/Creational%20design%20patterns/Singleton)

#### 4. Builder

又稱生成器模式、建造者模式

情境

> 她的心裡在考慮兩種車: 轎車和跑車。她委託懂車的工頭/工人來負責製造。
> 工頭告訴珍妮: 轎車的零件有車體、引擎，而跑車的零件有車體、引擎、酷炫擾流板。
> 如果珍妮委託工頭做一台轎車，工頭就會請工人安裝車體、安裝引擎；如果珍妮委託工頭做一台跑車，工頭就會請工人安裝車體、安裝引擎、安裝酷炫擾流板。

![[Builder.png]]

生成器模式適合適用場景

1. 使用生成器模式可避免 " 重疊構造函數 (telescopic constructor)" 的出現

> 假設你的構造函數中有十個可選參數，那麼調用該函數會非常不方便；因此，你需要重載這個構造函數， 新建幾個只有較少參數的簡化版。
> 但這些構造函數仍需調用主構造函數，傳遞一些預設數值來替代省略掉的參數。
> 通常是 C#或 Java 等支持方法重載的語言中才能寫出如此複雜的構造函數

2. 當你希望使用代碼創建不同形式的產品 (例如石頭或木頭房屋) 時，可使用生成器模式
3. 使用生成器構造組合樹或其他複雜物件

> 生成器模式讓你能分步驟構造產品。你可以延遲執行某些步驟而不會影響最終產品。
> 生成器在執行製造步驟時不能對外發佈未完成的產品。這可以避免用戶端代碼獲取到不完整結果物件的情況。

設計
    工人就是 Builder，工頭則是 Director，而那些被製造的轎車和跑車稱作 Product。**任何複雜 object (稱作 product) 的創建過程都只是請工人安裝一連串的零件而已。**

1. 建造者模式包含了以下元素:
    a) Builder(建造者 or 工人): 負責建造
    b) Director(總監 or 工頭): 自己不懂建造，請工人依序執行建造的動作

不用吧，明明就有兩個比較簡單的設計方法呀

1. **兩個建構子（Constructor）各自負責轎車、跑車就好啦？**
    Ans: 當有幾十種車時，就會有幾十種建構子，每個建構子裡面都有很類似的「安裝引擎」的功能，就造成「安裝引擎」的功能被重複寫了幾十次，以後每次改都要改幾十次。比較麻煩。
2. **只要一個建構子負責接收 3 個參數：車體種類、引擎種類、擾流板種類，再根據接收的參數製造不同車種就好啦？**
    Ans: 當要製造的車種類繁多時 (包含計程車 垃圾車 救護車 靈車 閃電霹靂車…)，假設總共有 10 個參數，大部分的轎車只要 2 個參數就能製造。那呼叫那個大建構子來製造大部分的車的時候，輸入參數的部分常常有 8 個欄位都是空的。比較醜。

優缺點

1. Pros:
    A) 你可以分步創建物件，暫緩創建步驟或遞迴運行創建步驟
    B) 生成不同形式的產品時，你可以複用相同的製造代碼
    C) SRP，可以將複雜構造代碼從產品的業務邏輯中分離出來
2. Cons: 由於該模式需要新增多個類，因此代碼整體複雜程度會有所增加。

符合單一責任原則 (SRP)

> 複雜的安裝 XX 功能都被獨立於 builder 裡面了。

模式比較

1. Builder 關注如何分步驟生成複雜對象，在取得產品前可以執行一些額外的建造步驟； Abstract Factory 用於生產一系列相關對象，抽象工廠會馬上返回產品。

[程式碼](https://github.com/jayhsieh/GoF-Design-Patterns-/tree/main/Creational%20design%20patterns/Builder)

#### 5. Prototype

原型模式、克隆、Clone，使你能夠複製已有對象，而又無需使代碼依賴它們所屬的類

情境

> 很多編輯軟體中都有複製這個功能，例如 Google Slides 的複製投影片，或是 PicCollage 的複製文字。這些情況就很理所當然的使用了 copy 或 clone。但更進一步的是，通常複製了之後，是想要用這些已經做好的原型 (prototype) 來做一些修改，例如更換文字顏色，或是改變字體。
> 這就是 Prototype Pattern，非常適合用在有深層結構而且高度自定義的物件上面，在這種情況下你通常不會想從頭建立這些物件，另一方面也可以提高程式碼的可讀性。

克隆模式適合適用場景

1. 如果你需要複製一些物件，同時又希望代碼獨立於這些物件所屬的具體類，可以使用原型模式

> 這一點考量通常出現在代碼需要處理協力廠商代碼通過介面傳遞過來的物件時。即使不考慮代碼耦合的情況，你的代碼也不能依賴這些物件所屬的具體類，因為你不知道它們的具體資訊。
> 原型模式為用戶端代碼提供一個通用介面，用戶端代碼可通過這一介面與所有實現了克隆的物件進行交互，它也使得用戶端代碼與其所克隆的對象具體類獨立開來。

2. 如果子類的區別僅在於其物件的初始化方式，那麼你可以使用該模式來減少子類的數量。別人創建這些子類的目的可能是為了創建特定類型的物件

> 在原型模式中，你可以使用一系列預生成的、各種類型的物件作為原型。
> 用戶端不必根據需求對子類進行產生實體，只需找到合適的原型並對其進行克隆即可

設計

1. 原型模式有更深層的意義在於:
    a) 儲放原型
    b) 方便的調用原型複製 (Clone) 功能

![[Prototype.png]]

使用時機與優缺點

1. Pros:
    A) 有些時候建立物件是非常耗時的，做了很多 IO 操作（DB, file, Api and parse json）才拿到所需要的物件。在需要建立類似物件時，會希望耗時操作越少越好，使用 Prototype 就可以減少這類型的消耗
    B) 多執行緒或跨類別操作時可以藉由 Prototype 來避免共享變數，共享變數會造成不可預期的資料修改或是 Deadlock 。雖然在原書中該 pattern 沒有限制物件要是 immutable 的，但是在這種情形下搭配 immutable 物件會很適合的
    C) 克隆物件，而無需與它們所屬的具體類相耦合
    D) 克隆預生成原型，避免反覆運行初始化代碼
    E) 更方便地生成複雜物件
    F) 用繼承以外的方式來處理複雜物件的不同配置
    
2. Cons:
    A) 缺點是實作 copy 比較繁瑣而且無聊，有些 library 或是語言本身可以幫助你完成這件事，像是 Java 的 Lombok 可以藉由 annotation 自動程式碼 (嚴格的來說這並不符合 Prototype Pattern 的定義，但是能夠獲得一樣的效果)
    B) 克隆包含迴圈引用的複雜物件可能會非常麻煩

[程式碼](https://github.com/jayhsieh/GoF-Design-Patterns-/tree/main/Creational%20design%20patterns/Prototype)
