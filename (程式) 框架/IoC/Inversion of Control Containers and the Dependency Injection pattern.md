---
aliases:
date: 2004-01-14
update: 2004-01-23
author: Martin Fowler
language:
sourceurl: https://martinfowler.com/articles/injection.html
tags:
  - IoC
---

# Inversion of Control Containers and the Dependency Injection pattern 控制反轉容器與相依性注入模式

_In the Java community there's been a rush of lightweight containers that help to assemble components from different projects into a cohesive application. Underlying these containers is a common pattern to how they perform the wiring, a concept they refer under the very generic name of “Inversion of Control”. In this article I dig into how this pattern works, under the more specific name of “Dependency Injection”, and contrast it with the Service Locator alternative. The choice between them is less important than the principle of separating configuration from use.
在 Java 社群中，出現了大量輕量級容器，它們幫助將來自不同專案的分件組裝成一個協調的應用程式。這些容器背後有一個共同的組裝模式，它們用一個非常普遍的名稱來指稱這個概念，即「控制反轉」。在本文中，我將深入探討這個模式如何運作，用一個更特定的名稱「相依性注入」，並將其與服務定位器替代方案進行對比。它們之間的選擇不如將配置與使用分離的原則重要。_

One of the entertaining things about the enterprise Java world is the huge amount of activity in building alternatives to the mainstream J2EE technologies, much of it happening in open source. A lot of this is a reaction to the heavyweight complexity in the mainstream J2EE world, but much of it is also exploring alternatives and coming up with creative ideas. A common issue to deal with is how to wire together different elements: how do you fit together this web controller architecture with that database interface backing when they were built by different teams with little knowledge of each other. A number of frameworks have taken a stab at this problem, and several are branching out to provide a general capability to assemble components from different layers. These are often referred to as lightweight containers, examples include [PicoContainer](http://picocontainer.com/), and [Spring](http://www.springsource.org/).
企業 Java 世界中一件有趣的事是，在建立主流 J2EE 技術的替代方案方面有大量的活動，其中許多活動發生在開放原始碼領域。這其中許多是對主流 J2EE 世界中嚴重重量級複雜性的反應，但其中也有許多是在探索替代方案並提出創意想法。一個常見的問題是如何將不同的元素連接起來：當它們由不同的團隊開發，彼此知識有限時，你如何將這個網頁控制器架構與那個資料庫介面背後整合在一起。許多框架都嘗試解決這個問題，其中幾個已擴展提供從不同層級組裝組件的通用功能。這些通常被稱為輕量級容器，例如 PicoContainer 和 Spring。

Underlying these containers are a number of interesting design principles, things that go beyond both these specific containers and indeed the Java platform. Here I want to start exploring some of these principles. The examples I use are in Java, but like most of my writing the principles are equally applicable to other OO environments, particularly .NET.
這些容器背後有一系列有趣的设计原则，這些原则超越了這些特定容器，也超越了 Java 平台本身。這裡我想開始探討其中的一些原則。我使用的例子是 Java，但像大多數我的寫作一樣，這些原則同樣適用於其他面向對象環境，特別是.NET。

## Components and Services  元件與服務

The topic of wiring elements together drags me almost immediately into the knotty terminology problems that surround the terms service and component. You find long and contradictory articles on the definition of these things with ease. For my purposes here are my current uses of these overloaded terms.
關於如何將元素綁定在一起的話題，幾乎立刻引導我進入圍繞服務和元件這些術語的麻煩術語問題。您可以輕易地找到關於這些事物的定義的長篇且相互矛盾的文章。對於我這裡的用途，這是我目前對這些重載術語的用法。

I use component to mean a glob of software that's intended to be used, without change, by an application that is out of the control of the writers of the component. By 'without change' I mean that the using application doesn't change the source code of the components, although they may alter the component's behavior by extending it in ways allowed by the component writers.
我使用元件來指代一塊旨在被一個不受元件作者控制的應用程式無改變使用的軟體。所謂「無改變」是指使用應用程式不會改變元件的原始碼，儘管它們可能通過擴展元件（以符合元件作者允許的方式）來改變元件的行為。

A service is similar to a component in that it's used by foreign applications. The main difference is that I expect a component to be used locally (think jar file, assembly, dll, or a source import). A service will be used remotely through some remote interface, either synchronous or asynchronous (eg web service, messaging system, RPC, or socket.)
一個服務與元件相似，都是被外部應用程式使用。主要差別在於我期望元件是本地使用（想想 jar 檔案、組合、dll 或來源匯入）。服務會透過遠端介面遠程使用，無論是同步或非同步（例如網服務、訊息系統、RPC 或插座。）

I mostly use service in this article, but much of the same logic can be applied to local components too. Indeed often you need some kind of local component framework to easily access a remote service. But writing “component or service” is tiring to read and write, and services are much more fashionable at the moment.
本文主要使用服務，但大多數相同邏輯也適用於本地元件。事實上，你通常需要某種本地元件框架來輕鬆存取遠端服務。但寫「元件或服務」既麻煩閱讀也麻煩撰寫，而服務在目前則流行得多。

## A Naive Example  一個天真範例

To help make all of this more concrete I'll use a running example to talk about all of this. Like all of my examples it's one of those super-simple examples; small enough to be unreal, but hopefully enough for you to visualize what's going on without falling into the bog of a real example.
為了讓這一切更具體，我將使用一個正在執行的範例來討論這些內容。像我所有的範例一樣，這是一個超級簡單的範例；小到不真實，但希望足夠讓你無法陷入真實範例的泥潭而能夠想像正在發生什麼。

In this example I'm writing a component that provides a list of movies directed by a particular director. This stunningly useful function is implemented by a single method.
在本範例中，我正在撰寫一個元件，提供由特定導演導演的電影列表。這個令人驚嘆地實用的功能是由單一方法實現的。

```java
class MovieLister...
  public Movie[] moviesDirectedBy(String arg) {
      List allMovies = finder.findAll();
      for (Iterator it = allMovies.iterator(); it.hasNext();) {
          Movie movie = (Movie) it.next();
          if (!movie.getDirector().equals(arg)) it.remove();
      }
      return (Movie[]) allMovies.toArray(new Movie[allMovies.size()]);
  }
```

The implementation of this function is naive in the extreme, it asks a finder object (which we'll get to in a moment) to return every film it knows about. Then it just hunts through this list to return those directed by a particular director. This particular piece of naivety I'm not going to fix, since it's just the scaffolding for the real point of this article.
這個功能的實現極度天真，它要求一個尋找物件（我們稍後會提到）回傳它所知道的所有電影。然後它只是透過這個列表搜尋，回傳由特定導演導演的那些電影。我並不打算修補這個特別的天真之處，因為它只是本文真正要點的基礎架構。

The real point of this article is this finder object, or particularly how we connect the lister object with a particular finder object. The reason why this is interesting is that I want my wonderful `moviesDirectedBy` method to be completely independent of how all the movies are being stored. So all the method does is refer to a finder, and all that finder does is know how to respond to the `findAll` method. I can bring this out by defining an interface for the finder.
本文真正要點是這個尋找物件，或特別是我們如何將列表器物件與特定的尋找物件連接起來。這個有趣的原因是，我希望我的偉大的 `moviesDirectedBy` 方法能夠完全獨立於所有電影的儲存方式。因此，這個方法所做的只是參考一個尋找物件，而這個尋找物件所做的只是知道如何回應 `findAll` 方法。我可以透過定義一個尋找物件的介面來將這件事表現出來。

```java
public interface MovieFinder {
    List findAll();
}
```

Now all of this is very well decoupled, but at some point I have to come up with a concrete class to actually come up with the movies. In this case I put the code for this in the constructor of my lister class.
現在這些都非常好地解耦，但在某個時候我必須想出一個具體的類別來實際產生電影。在這種情況下，我把這段代碼放在我的 lister 類別的建構子裡。

```java
class MovieLister...
  private MovieFinder finder;
  public MovieLister() {
    finder = new ColonDelimitedMovieFinder("movies1.txt");
  }
```

The name of the implementation class comes from the fact that I'm getting my list from a colon delimited text file. I'll spare you the details, after all the point is just that there's some implementation.
實現類別的名稱來自於我從一個以冒號分隔的文本檔案中獲取我的清單。我不會細說，畢竟重點只是有一個實現。

Now if I'm using this class for just myself, this is all fine and dandy. But what happens when my friends are overwhelmed by a desire for this wonderful functionality and would like a copy of my program? If they also store their movie listings in a colon delimited text file called “movies1.txt” then everything is wonderful. If they have a different name for their movies file, then it's easy to put the name of the file in a properties file. But what if they have a completely different form of storing their movie listing: a SQL database, an XML file, a web service, or just another format of text file? In this case we need a different class to grab that data. Now because I've defined a `MovieFinder` interface, this won't alter my `moviesDirectedBy` method. But I still need to have some way to get an instance of the right finder implementation into place.
現在如果我只是用這個類別來幫自己使用，這一切都很好。但當我的朋友們被這個絕妙的函數性所吸引，想要我的程式副本時會怎麼樣？如果他們也將他們的電影清單存放在名為“movies1.txt”的冒號分隔的文本檔中，那麼一切都很好。如果他們的電影檔名不同，那麼很容易將檔名放在屬性檔中。但如果他們有完全不同的電影清單儲存形式：一個 SQL 資料庫、一個 XML 檔案、一個網絡服務，或者只是一個其他格式的文本檔？在這種情況下，我們需要一個不同的類別來抓取那個數據。現在因為我定義了一個 `MovieFinder` 介面，這不會改變我的 `moviesDirectedBy` 方法。但我仍然需要某種方法來將正確的查找實現實例安置到位。

![[naive.gif]]

Figure 1: The dependencies using a simple creation in the lister class
圖 1：在清單類別中使用簡單創建的依賴關係

Figure 1 shows the dependencies for this situation. The `MovieLister` class is dependent on both the `MovieFinder` interface and upon the implementation. We would prefer it if it were only dependent on the interface, but then how do we make an instance to work with?
圖 1 顯示了這種情況下的依賴關係。 `MovieLister` 類別依賴於 `MovieFinder` 介面和實現。我們希望它只依賴於介面，但那麼我們該如何創建一個實例來使用呢？

In my book [P of EAA](https://martinfowler.com/books/eaa.html), we described this situation as a [Plugin](https://martinfowler.com/eaaCatalog/plugin.html). The implementation class for the finder isn't linked into the program at compile time, since I don't know what my friends are going to use. Instead we want my lister to work with any implementation, and for that implementation to be plugged in at some later point, out of my hands. The problem is how can I make that link so that my lister class is ignorant of the implementation class, but can still talk to an instance to do its work.
在我的書籍《Enterprise Application Architecture》中，我們將這種情況描述為一個外掛。尋找器的實現類別在編譯時並未鏈接到程序中，因為我不知道我的朋友們將使用什麼。相反，我們希望我的列表器能夠與任何實現一起工作，並且該實現在稍後的某個時刻被插入，這超出我的控制範圍。問題在於如何建立這種鏈接，以便我的列表器類別對實現類別無知，但仍然可以與實例交談以完成其工作。

Expanding this into a real system, we might have dozens of such services and components. In each case we can abstract our use of these components by talking to them through an interface (and using an adapter if the component isn't designed with an interface in mind). But if we wish to deploy this system in different ways, we need to use plugins to handle the interaction with these services so we can use different implementations in different deployments.
將這一點擴展到一個實際系統中，我們可能會有數十個這樣的服務和元件。在每個情況下，我們都可以通過通過介面與這些元件交談來抽象我們的使用方式（如果元件不是為了介面而設計的，則使用適配器）。但如果我們希望以不同的方式部署此系統，我們需要使用外掛來處理與這些服務的交互，以便在不同的部署中使用不同的實現。

So the core problem is how do we assemble these plugins into an application? This is one of the main problems that this new breed of lightweight containers face, and universally they all do it using Inversion of Control.
因此，核心問題是怎麼樣將這些外掛組裝到應用程序中？這是這種新型輕量級容器面臨的主要問題之一，它們普遍都使用控制反轉來解決這個問題。

## Inversion of Control  控制反轉

When these containers talk about how they are so useful because they implement “Inversion of Control” I end up very puzzled. [Inversion of control](https://martinfowler.com/bliki/InversionOfControl.html) is a common characteristic of frameworks, so saying that these lightweight containers are special because they use inversion of control is like saying my car is special because it has wheels.
當這些容器提到它們因實現「控制反轉」(Inversion of Control) 而非常實用時，我常常感到非常困惑。控制反轉是框架的常見特性，因此說這些輕量級容器因使用控制反轉而特殊，就像說我的車因有輪子而特殊一樣。

The question is: “what aspect of control are they inverting?” When I first ran into inversion of control, it was in the main control of a user interface. Early user interfaces were controlled by the application program. You would have a sequence of commands like “Enter name”, “enter address”; your program would drive the prompts and pick up a response to each one. With graphical (or even screen based) UIs the UI framework would contain this main loop and your program instead provided event handlers for the various fields on the screen. The main control of the program was inverted, moved away from you to the framework.
問題在於：「它們在反轉哪方面的控制？」當我第一次遇到控制反轉時，它是在使用者介面 (User Interface) 的主要控制中。早期的使用者介面是由應用程式控制的。你會有一系列的指令，例如「輸入姓名」、「輸入地址」；你的程式會主導提示並接收每個指令的回應。在圖形化 (或甚至螢幕基礎) 的 UI 中，UI 框架會包含這個主要迴圈，而你的程式則提供各種螢幕上欄位的事件處理函式。程式的 主要控制被反轉了，從你移動到框架。

For this new breed of containers the inversion is about how they lookup a plugin implementation. In my naive example the lister looked up the finder implementation by directly instantiating it. This stops the finder from being a plugin. The approach that these containers use is to ensure that any user of a plugin follows some convention that allows a separate assembler module to inject the implementation into the lister.
對於這種新型態的容器來說，控制倒置是關於它們如何查詢外掛實現的方式。在我的天真範例中，list 器是透過直接實例化來查詢 finder 實現。這阻止了 finder 成為一個外掛。這些容器使用的做法是確保任何使用外掛的人遵循某種規範，讓一個獨立的金字塔模組能夠將實現注入到 list 器中。

As a result I think we need a more specific name for this pattern. Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various IoC advocates we settled on the name _Dependency Injection_.
因此，我認為我們需要為這個模式取一個更特定的名稱。控制倒置是一個太過泛化的術語，因此人們會感到困惑。經過與許多 IoC 倡導者的討論後，我們最終確定了名稱為相依性注入。

I'm going to start by talking about the various forms of dependency injection, but I'll point out now that that's not the only way of removing the dependency from the application class to the plugin implementation. The other pattern you can use to do this is Service Locator, and I'll discuss that after I'm done with explaining Dependency Injection.
我將從討論相依性注入的多種形式開始，但現在我就要指出，這並非將相依性從應用程式類別移除至外掛實作的唯一方式。您可以使用的另一個模式是服務定位器，我將在解釋完相依性注入後再討論它。

## Forms of Dependency Injection 相依性注入的形式

The basic idea of the Dependency Injection is to have a separate object, an assembler, that populates a field in the lister class with an appropriate implementation for the finder interface, resulting in a dependency diagram along the lines of [Figure 2](https://martinfowler.com/articles/injection.html#injector.gif)
相依性注入的基本概念是有一個獨立的物件，一個組裝器，用適當的 finder 接口實現來填補清單類別中的欄位，結果產生一個相依性圖表，類似於圖 2

![[injector.gif]]

Figure 2: The dependencies for a Dependency Injector
圖 2：相依性注入器的相依性

There are three main styles of dependency injection. The names I'm using for them are Constructor Injection, Setter Injection, and Interface Injection. If you read about this stuff in the current discussions about Inversion of Control you'll hear these referred to as type 1 IoC (interface injection), type 2 IoC (setter injection) and type 3 IoC (constructor injection). I find numeric names rather hard to remember, which is why I've used the names I have here.
相依性注入有三種主要形式。我為它們使用的名稱是建構子注入、設定器注入和接口注入。如果你在關於控制反轉的當前討論中閱讀這些資訊，你會聽到它們被稱為類型 1 IoC（接口注入）、類型 2 IoC（設定器注入）和類型 3 IoC（建構子注入）。我覺得數字名稱很難記住，這就是為什麼我這裡使用了這些名稱。

### Constructor Injection with PicoContainer 建構子注入

I'll start with showing how this injection is done using a lightweight container called [PicoContainer](http://picocontainer.com/). I'm starting here primarily because several of my colleagues at Thoughtworks are very active in the development of PicoContainer (yes, it's a sort of corporate nepotism.)
我會先展示如何使用一個輕量級容器 PicoContainer 來進行這種注入。我這裡開始主要是因為我在 Thoughtworks 的許多同事都非常活躍於 PicoContainer 的開發（是的，這算是種企業內部人脈關係吧。）

PicoContainer uses a constructor to decide how to inject a finder implementation into the lister class. For this to work, the movie lister class needs to declare a constructor that includes everything it needs injected.
PicoContainer 使用建構子來決定如何將搜尋器實作注入到清單類別中。為了讓這個功能正常運作，電影清單類別需要宣告一個包含所有需要被注入的建構子。

```java
class MovieLister...
  public MovieLister(MovieFinder finder) {
      this.finder = finder;
  }
```

The finder itself will also be managed by the pico container, and as such will have the filename of the text file injected into it by the container.
搜尋器本身也會由 PicoContainer 來管理，因此容器會將文字檔的檔名注入到搜尋器中。

```java
class ColonMovieFinder...
  public ColonMovieFinder(String filename) {
      this.filename = filename;
  }
```

The pico container then needs to be told which implementation class to associate with each interface, and which string to inject into the finder.
接下來，PicoContainer 需要知道要將哪個實作類別與每個介面關聯，以及要將哪個字串注入到搜尋器中。

```java
private MutablePicoContainer configureContainer() {
    MutablePicoContainer pico = new DefaultPicoContainer();
    Parameter[] finderParams = {new ConstantParameter("movies1.txt")};
    pico.registerComponentImplementation(MovieFinder.class, ColonMovieFinder.class, finderParams);
    pico.registerComponentImplementation(MovieLister.class);
    return pico;
}
```

This configuration code is typically set up in a different class. For our example, each friend who uses my lister might write the appropriate configuration code in some setup class of their own. Of course it's common to hold this kind of configuration information in separate config files. You can write a class to read a config file and set up the container appropriately. Although PicoContainer doesn't contain this functionality itself, there is a closely related project called NanoContainer that provides the appropriate wrappers to allow you to have XML configuration files. Such a nano container will parse the XML and then configure an underlying pico container. The philosophy of the project is to separate the config file format from the underlying mechanism.
這個設定程式碼通常會設定在不同的類別中。以我們的範例來說，每個使用我的 lister 的朋友可能會在他們自己的某個設定類別中寫下適當的設定程式碼。當然，將這種設定資訊放在分開的設定檔中是很常見的。你可以寫一個類別來讀取設定檔，並適當地設定容器。儘管 PicoContainer 本身不包含這項功能，但有一個名為 NanoContainer 的密切相關專案提供適當的包裝，讓你可以使用 XML 設定檔。這樣的 nano 容器會解析 XML，然後設定底層的 pico 容器。這個專案的哲學是將設定檔格式與底層機制分開。

To use the container you write code something like this.
要使用這個容器，你會寫像這樣的程式碼。

```java
public void testWithPico() {
    MutablePicoContainer pico = configureContainer();
    MovieLister lister = (MovieLister) pico.getComponentInstance(MovieLister.class);
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
}
```

Although in this example I've used constructor injection, PicoContainer also supports setter injection, although its developers do prefer constructor injection.
儘管在這個範例中我使用了建構函式注入，但 PicoContainer 也支援設定函式注入，儘管它的開發者更偏好建構函式注入。

### Setter Injection with Spring 設定器注入

The [Spring framework](http://www.springsource.org/) is a wide ranging framework for enterprise Java development. It includes abstraction layers for transactions, persistence frameworks, web application development and JDBC. Like PicoContainer it supports both constructor and setter injection, but its developers tend to prefer setter injection - which makes it an appropriate choice for this example.
Spring 框架是一個廣泛的企業級 Java 開發框架。它包含交易、持續性框架、網站應用程式開發和 JDBC 的抽象層。像 PicoContainer 一樣，它支援建構子與設定器注入，但它的開發者傾向於偏好設定器注入——這使得它成為這個範例的適當選擇。

To get my movie lister to accept the injection I define a setting method for that service
為了讓我的電影清單工具能夠接受注入，我為該服務定義了一個設定方法

```java
class MovieLister...
  private MovieFinder finder;
  public void setFinder(MovieFinder finder) {
    this.finder = finder;
}
```

Similarly I define a setter for the filename.
類似地，我也定義了一個用於檔案名稱的設定器。

```java
class ColonMovieFinder...
  public void setFilename(String filename) {
      this.filename = filename;
  }
```

The third step is to set up the configuration for the files. Spring supports configuration through XML files and also through code, but XML is the expected way to do it.
第三步是設定檔案的相關配置。Spring 支援透過 XML 檔案和程式碼來進行設定，但 XML 是預期的設定方式。

```xml
<beans>
    <bean id="MovieLister" class="spring.MovieLister">
        <property name="finder">
            <ref local="MovieFinder"/>
        </property>
    </bean>
    <bean id="MovieFinder" class="spring.ColonMovieFinder">
        <property name="filename">
            <value>movies1.txt</value>
        </property>
    </bean>
</beans>
```

The test then looks like this.
測試看起來就像這樣。

```java
public void testWithSpring() throws Exception {
    ApplicationContext ctx = new FileSystemXmlApplicationContext("spring.xml");
    MovieLister lister = (MovieLister) ctx.getBean("MovieLister");
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
}
```

### Interface Injection  介面注入

The third injection technique is to define and use interfaces for the injection. [Avalon](http://avalon.apache.org/) is an example of a framework that uses this technique in places. I'll talk a bit more about that later, but in this case I'm going to use it with some simple sample code.
第三種注入技術是定義和使用介面來進行注入。Avalon 是一個在這方面使用此技術的框架範例。我稍後會多談一點，但在這種情況下，我打算用一些簡單的範例程式碼來使用它。

With this technique I begin by defining an interface that I'll use to perform the injection through. Here's the interface for injecting a movie finder into an object.
使用這種技術，我首先定義一個介面，我將透過它來進行注入。這裡是將一部電影尋找器注入物件的介面。

```java
public interface InjectFinder {
    void injectFinder(MovieFinder finder);
}
```

This interface would be defined by whoever provides the MovieFinder interface. It needs to be implemented by any class that wants to use a finder, such as the lister.
這個介面會由提供 MovieFinder 介面的那一方來定義。任何想要使用搜尋器的類別，例如 lister，都需要實現這個介面。

```java
class MovieLister implements InjectFinder
  public void injectFinder(MovieFinder finder) {
      this.finder = finder;
  }
```

I use a similar approach to inject the filename into the finder implementation.
我使用類似的方法將檔案名稱注入到搜尋器實現中。

```java
public interface InjectFinderFilename {
    void injectFilename (String filename);
}

class ColonMovieFinder implements MovieFinder, InjectFinderFilename...
  public void injectFilename(String filename) {
      this.filename = filename;
  }
```

Then, as usual, I need some configuration code to wire up the implementations. For simplicity's sake I'll do it in code.
接下來，照例，我需要一些設定碼來連接實作。為了簡潔起見，我會用程式碼來做。

```java
class Tester...
  private Container container;

   private void configureContainer() {
     container = new Container();
     registerComponents();
     registerInjectors();
     container.start();
  }
```

This configuration has two stages, registering components through lookup keys is pretty similar to the other examples.
這個設定有兩個階段，透過查詢鍵註冊組件與其他範例類似。

```java
class Tester...
  private void registerComponents() {
    container.registerComponent("MovieLister", MovieLister.class);
    container.registerComponent("MovieFinder", ColonMovieFinder.class);
  }
```

A new step is to register the injectors that will inject the dependent components. Each injection interface needs some code to inject the dependent object. Here I do this by registering injector objects with the container. Each injector object implements the injector interface.
一個新的步驟是註冊將注入相依組件的注入器。每個注入介面需要一些程式碼來注入相依物件。我在這裡透過註冊注入器物件到容器來完成這件事。每個注入器物件都實作了注入器介面。

```java
class Tester...
  private void registerInjectors() {
    container.registerInjector(InjectFinder.class, container.lookup("MovieFinder"));
    container.registerInjector(InjectFinderFilename.class, new FinderFilenameInjector());
  }

public interface Injector {
  public void inject(Object target);
}
```

When the dependent is a class written for this container, it makes sense for the component to implement the injector interface itself, as I do here with the movie finder. For generic classes, such as the string, I use an inner class within the configuration code.
當相依物件是為這個容器所寫的類別時，對組件來說實作注入器介面本身是有意義的，就像我在這裡用電影搜尋器所做的一樣。對於泛型類別，例如字串，我在設定程式碼中使用一個內部類別。

```java
class ColonMovieFinder implements Injector...
  public void inject(Object target) {
    ((InjectFinder) target).injectFinder(this);
  }

class Tester...

  public static class FinderFilenameInjector implements Injector {
    public void inject(Object target) {
      ((InjectFinderFilename)target).injectFilename("movies1.txt");
    }
    }
```

The tests then use the container.
測試會使用這個容器。

```java
class Tester…
  public void testIface() {
    configureContainer();
    MovieLister lister = (MovieLister)container.lookup("MovieLister");
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
  }
```

The container uses the declared injection interfaces to figure out the dependencies and the injectors to inject the correct dependents. (The specific container implementation I did here isn't important to the technique, and I won't show it because you'd only laugh.)
容器會使用宣告的注入介面來判斷相依性，並使用注入器來注入正確的相依項。(我這裡實現的具體容器並不影響這個技術，而且我不會展示它，因為你們只會笑。)

## Using a Service Locator 使用服務定位器

The key benefit of a Dependency Injector is that it removes the dependency that the `MovieLister` class has on the concrete `MovieFinder` implementation. This allows me to give listers to friends and for them to plug in a suitable implementation for their own environment. Injection isn't the only way to break this dependency, another is to use a [service locator](http://java.sun.com/blueprints/corej2eepatterns/Patterns/ServiceLocator.html).
相依性注入器的關鍵優勢在於，它移除了 `MovieLister` 類別對具體 `MovieFinder` 實現的相依性。這讓我能夠將 listers 分給朋友，並讓他們為自己的環境插入合適的實現。注入並非打破這種相依性的唯一方式，另一種方法是使用服務定位器。

The basic idea behind a service locator is to have an object that knows how to get hold of all of the services that an application might need. So a service locator for this application would have a method that returns a movie finder when one is needed. Of course this just shifts the burden a tad, we still have to get the locator into the lister, resulting in the dependencies of [Figure 3](https://martinfowler.com/articles/injection.html#locator.gif)
服務定位器背後的基本想法是擁有一個物件，它知道如何獲取應用程式可能需要的所有服務。因此，這個應用程式的服務定位器會有一個方法，當需要時就回傳一個電影尋找器。當然，這只是稍微將負擔移轉，我們仍然必須將定位器放入 lister 中，導致了圖 3 的相依性

![[locator.gif]]

Figure 3: The dependencies for a Service Locator
圖 3：服務定位器的相依性

In this case I'll use the ServiceLocator as a singleton [Registry](https://martinfowler.com/eaaCatalog/registry.html). The lister can then use that to get the finder when it's instantiated.
在此情況下，我會使用 ServiceLocator 作為單例的 Registry。然後，清單可以使用它來在實例化時取得 finder。

```java
class MovieLister...

  MovieFinder finder = ServiceLocator.movieFinder();

class ServiceLocator...

  public static MovieFinder movieFinder() {
      return soleInstance.movieFinder;
  }
  private static ServiceLocator soleInstance;
  private MovieFinder movieFinder;
```

As with the injection approach, we have to configure the service locator. Here I'm doing it in code, but it's not hard to use a mechanism that would read the appropriate data from a configuration file.
與注入方法一樣，我們必須配置服務定位器。這裡我在程式中進行配置，但使用機制從配置檔案讀取相關數據並不困難。

```java
class Tester...

  private void configure() {
      ServiceLocator.load(new ServiceLocator(new ColonMovieFinder("movies1.txt")));
  }

class ServiceLocator...

  public static void load(ServiceLocator arg) {
      soleInstance = arg;
  }

  public ServiceLocator(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
  }
```

Here's the test code.
這是測試程式碼。

```java
class Tester...

  public void testSimple() {
      configure();
      MovieLister lister = new MovieLister();
      Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
      assertEquals("Once Upon a Time in the West", movies[0].getTitle());
  }
```

I've often heard the complaint that these kinds of service locators are a bad thing because they aren't testable because you can't substitute implementations for them. Certainly you can design them badly to get into this kind of trouble, but you don't have to. In this case the service locator instance is just a simple data holder. I can easily create the locator with test implementations of my services.
我常聽到抱怨說這種服務定位器是不好的，因為它們不可測試，因為你無法替換它們的實作。當然你可以設計得不好而陷入這種麻煩，但你不必如此。在這種情況下，服務定位器的實例只是一個簡單的資料持有者。我可以輕易地用測試實作的服務來建立定位器。

For a more sophisticated locator I can subclass service locator and pass that subclass into the registry's class variable. I can change the static methods to call a method on the instance rather than accessing instance variables directly. I can provide thread–specific locators by using thread–specific storage. All of this can be done without changing clients of service locator.
為了更複雜的定位器，我可以繼承服務定位器，並將這個子類別傳入登記表的類別變數。我可以將靜態方法改為呼叫實例上的方法，而不是直接存取實例變數。我可以透過使用特定於執行的儲存來提供特定於執行的定位器。所有這些都可以在不改變服務定位器客戶端的情況下完成。

A way to think of this is that service locator is a registry not a singleton. A singleton provides a simple way of implementing a registry, but that implementation decision is easily changed.
這種思考方式是服務定位器是一個登記表而不是單例。單例提供了一種實現登記表的簡單方法，但這個實現決策很容易改變。

### Using a Segregated Interface for the Locator 使用分離介面來定位定位器

One of the issues with the simple approach above, is that the MovieLister is dependent on the full service locator class, even though it only uses one service. We can reduce this by using a [role interface](https://martinfowler.com/bliki/RoleInterface.html). That way, instead of using the full service locator interface, the lister can declare just the bit of interface it needs.
上述簡單方法的一個問題是，MovieLister 依賴於完整的服務定位器類別，即使它只使用一個服務。我們可以透過使用角色介面來減少這個依賴。這樣，列表器就不需要使用完整的服務定位器介面，而只需要宣告它需要的介面部分。

In this situation the provider of the lister would also provide a locator interface which it needs to get hold of the finder.
在此情況下，清單提供者也會提供一個定位介面，它需要透過這個介面來獲取查找器。

public interface MovieFinderLocator {
    public MovieFinder movieFinder();

The locator then needs to implement this interface to provide access to a finder.
定位器接著需要實現這個介面，以提供對查找器的訪問。

MovieFinderLocator locator = ServiceLocator.locator();
MovieFinder finder = locator.movieFinder();

public static ServiceLocator locator() {
     return soleInstance;
 }
 public MovieFinder movieFinder() {
     return movieFinder;
 }
 private static ServiceLocator soleInstance;
 private MovieFinder movieFinder;

You'll notice that since we want to use an interface, we can't just access the services through static methods any more. We have to use the class to get a locator instance and then use that to get what we need.
您會發現，由於我們想要使用介面，我們無法再透過靜態方法來訪問服務。我們必須使用類別來獲取定位器實例，然後使用該實例來獲取我們需要的內容。

### A Dynamic Service Locator

動態服務定位器

The above example was static, in that the service locator class has methods for each of the services that you need. This isn't the only way of doing it, you can also make a dynamic service locator that allows you to stash any service you need into it and make your choices at runtime.
上述範例是靜態的，因為服務定位器類別為您需要的每個服務都提供了方法。這不是唯一的方法，您可以建立一個動態服務定位器，讓您能在執行時將任何需要的服務存入其中並做出選擇。

In this case, the service locator uses a map instead of fields for each of the services, and provides generic methods to get and load services.
在此情況下，服務定位器使用映射（map）而不是每個服務的字段，並提供通用方法來獲取和加載服務。

class ServiceLocator...

  private static ServiceLocator soleInstance;
  public static void load(ServiceLocator arg) {
      soleInstance = arg;
  }
  private Map services = new HashMap();
  public static Object getService(String key){
      return soleInstance.services.get(key);
  }
  public void loadService (String key, Object service) {
      services.put(key, service);
  }

Configuring involves loading a service with an appropriate key.
設定涉及使用適當的鍵來加載服務。

class Tester...

  private void configure() {
      ServiceLocator locator = new ServiceLocator();
      locator.loadService("MovieFinder", new ColonMovieFinder("movies1.txt"));
      ServiceLocator.load(locator);
  }

I use the service by using the same key string.
我使用服務時使用相同的鍵字串。

class MovieLister...

  MovieFinder finder = (MovieFinder) ServiceLocator.getService("MovieFinder");

On the whole I dislike this approach. Although it's certainly flexible, it's not very explicit. The only way I can find out how to reach a service is through textual keys. I prefer explicit methods because it's easier to find where they are by looking at the interface definitions.
整體上我不喜歡這種方法。雖然它確實很靈活，但它不是很明確。我能找到的達到服務的唯一方式是透過文字鍵。我偏好明確的方法，因為透過查看介面定義更容易找到它們的位置。

### Using both a locator and injection with Avalon

在 Avalon 中使用定位器和注入

Dependency injection and a service locator aren't necessarily mutually exclusive concepts. A good example of using both together is the Avalon framework. Avalon uses a service locator, but uses injection to tell components where to find the locator.
相依性注入與服務定位器不一定互斥。使用這兩者的一個良好範例是 Avalon 框架。Avalon 使用服務定位器，但使用注入來告訴組件去哪裡尋找定位器。

Berin Loritsch sent me this simple version of my running example using Avalon.
Berin Loritsch 給我寄來了使用 Avalon 的我的範例的簡單版本。

public class MyMovieLister implements MovieLister, Serviceable {
    private MovieFinder finder;

    public void service( ServiceManager manager ) throws ServiceException {
        finder = (MovieFinder)manager.lookup(“finder”);
    } 
      

The service method is an example of interface injection, allowing the container to inject a service manager into MyMovieLister. The service manager is an example of a service locator. In this example the lister doesn't store the manager in a field, instead it immediately uses it to lookup the finder, which it does store.
服務方法是一個介面注入的例子，允許容器將服務管理員注入到 MyMovieLister 中。服務管理員是一個服務定位器的例子。在這個例子中，列表器不將管理員存儲在欄位中，相反它立即使用它來查找查找器，它將存儲。

## Deciding which option to use

決定使用哪個選項

So far I've concentrated on explaining how I see these patterns and their variations. Now I can start talking about their pros and cons to help figure out which ones to use and when.
到目前為止，我已經專注於解釋我如何看待這些模式及其變體。現在我可以開始談論它們的優缺點，以幫助確定使用哪種模式以及何時使用。

### Service Locator vs Dependency Injection

服務定位器與相依性注入

The fundamental choice is between Service Locator and Dependency Injection. The first point is that both implementations provide the fundamental decoupling that's missing in the naive example - in both cases application code is independent of the concrete implementation of the service interface. The important difference between the two patterns is about how that implementation is provided to the application class. With service locator the application class asks for it explicitly by a message to the locator. With injection there is no explicit request, the service appears in the application class - hence the inversion of control.
基本的選擇在於服務定位器與相依性注入之間。第一點是，兩種實現都提供了在簡單示例中缺失的基本解耦——在兩種情況下，應用程式代碼都不依賴服務介面的具體實現。兩種模式之間的重要區別在於如何將該實現提供給應用程式類別。使用服務定位器時，應用程式類別透過向定位器發送訊息明確地請求它。使用注入時，沒有明確的請求，服務會出現在應用程式類別中——因此這是控制倒置。

Inversion of control is a common feature of frameworks, but it's something that comes at a price. It tends to be hard to understand and leads to problems when you are trying to debug. So on the whole I prefer to avoid it unless I need it. This isn't to say it's a bad thing, just that I think it needs to justify itself over the more straightforward alternative.
控制反轉是框架的常見功能，但它是有代價的。它往往難以理解，且在嘗試除錯時會導致問題。因此，我整體上傾向避免使用它，除非我需要它。這並不是說它不好，只是我認為它需要比更直接的替代方案更有說服力。

The key difference is that with a Service Locator every user of a service has a dependency to the locator. The locator can hide dependencies to other implementations, but you do need to see the locator. So the decision between locator and injector depends on whether that dependency is a problem.
關鍵區別在於，使用服務定位器時，服務的每個使用者都有對定位器的相依性。定位器可以隱藏對其他實現的相依性，但你確實需要看到定位器。因此，定位器與注入器的決策取決於這個相依性是否是問題。

Using dependency injection can help make it easier to see what the component dependencies are. With dependency injector you can just look at the injection mechanism, such as the constructor, and see the dependencies. With the service locator you have to search the source code for calls to the locator. Modern IDEs with a find references feature make this easier, but it's still not as easy as looking at the constructor or setting methods.
使用相依性注入有助於讓元件相依性更清晰。使用相依性注入器，你只需要看著注入機制，例如建構子，就能看到相依性。使用服務定位器，你必須搜尋源代碼中對定位器的呼叫。現代具有查找參考功能的 IDE 讓這件事更容易，但它仍然沒有看建構子或設定方法那麼容易。

A lot of this depends on the nature of the user of the service. If you are building an application with various classes that use a service, then a dependency from the application classes to the locator isn't a big deal. In my example of giving a Movie Lister to my friends, then using a service locator works quite well. All they need to do is to configure the locator to hook in the right service implementations, either through some configuration code or through a configuration file. In this kind of scenario I don't see the injector's inversion as providing anything compelling.
這很多取決於服務的使用者的性質。如果你正在建立一個使用多個類別的應用程式，那麼應用程式類別到定位器的相依性並不是一個大問題。在我給朋友們提供電影清單的範例中，使用服務定位器運作得相當好。他們只需要配置定位器以連接正確的服務實作，無論是透過一些配置程式碼或透過配置檔案。在這種情況下，我看不出注入器的反向控制提供任何令人信服的東西。

The difference comes if the lister is a component that I'm providing to an application that other people are writing. In this case I don't know much about the APIs of the service locators that my customers are going to use. Each customer might have their own incompatible service locators. I can get around some of this by using the segregated interface. Each customer can write an adapter that matches my interface to their locator, but in any case I still need to see the first locator to lookup my specific interface. And once the adapter appears then the simplicity of the direct connection to a locator is beginning to slip.
差異在於，如果這個清單器（list 器）是一個我提供給其他人在寫的應用程式的元件。在這種情況下，我對我的客戶即將使用的服務定位器（service locators）的 API 了解不多。每個客戶可能有自己的不相容服務定位器。我可以透過使用分離介面來解決部分問題。每個客戶可以寫一個適配器（adapter），將我的介面匹配到他們的定位器，但在任何情況下我仍然需要看到第一個定位器來查詢我的特定介面。而且一旦適配器出現，直接連接到定位器的簡單性就開始滑落。

Since with an injector you don't have a dependency from a component to the injector, the component cannot obtain further services from the injector once it's been configured.
由於使用注入器（injector）時，元件與注入器之間沒有相依性，一旦元件被配置完成後，它就無法再從注入器獲取進一步的服務。

A common reason people give for preferring dependency injection is that it makes testing easier. The point here is that to do testing, you need to easily replace real service implementations with stubs or mocks. However there is really no difference here between dependency injection and service locator: both are very amenable to stubbing. I suspect this observation comes from projects where people don't make the effort to ensure that their service locator can be easily substituted. This is where continual testing helps, if you can't easily stub services for testing, then this implies a serious problem with your design.
一個常見的偏好相依性注入的理由是它讓測試更輕鬆。這裡的關鍵在於進行測試時，你需要能輕易地用 stub 或 mock 替換實際服務實現。然而，相依性注入和服務定位器之間並沒有真正的區別：兩者都非常適合進行 stubbing。我猜測這個觀察來自於那些人沒有努力確保他們的服務定位器能夠輕易被替換的專案。這就是持續測試的作用所在，如果你無法輕易地為測試 stub 服務，那麼這意味著你的設計存在嚴重的問題。

Of course the testing problem is exacerbated by component environments that are very intrusive, such as Java's EJB framework. My view is that these kinds of frameworks should minimize their impact upon application code, and particularly should not do things that slow down the edit-execute cycle. Using plugins to substitute heavyweight components does a lot to help this process, which is vital for practices such as Test Driven Development.
當然，在非常侵入性的組件環境中，測試問題會更加嚴重，例如 Java 的 EJB 框架。我的看法是，這類框架應該盡量減少對應用程式碼的影響，特別是不應該做那些延緩編輯 - 執行週期的操作。使用插件替換重型組件能顯著幫助這個過程，而這對測試驅動開發等實踐至關重要。

So the primary issue is for people who are writing code that expects to be used in applications outside of the control of the writer. In these cases even a minimal assumption about a Service Locator is a problem.
所以主要的問題是對於那些寫的代碼預期會在作者控制的範圍之外的應用程式中被使用的開發者。在這些情況下，即使對服務定位器有一個最小的假設也是一個問題。

### Constructor versus Setter Injection

建構子與設定器注入

For service combination, you always have to have some convention in order to wire things together. The advantage of injection is primarily that it requires very simple conventions - at least for the constructor and setter injections. You don't have to do anything odd in your component and it's fairly straightforward for an injector to get everything configured.
對於服務組合，你總是必須有一些約定來將事物連接在一起。注入的優勢主要在於它需要非常簡單的約定——至少對於建構子與設定器注入而言。你不需要在你的元件中做任何奇怪的事情，而且對於注入器來說，配置所有事情相當直觀。

Interface injection is more invasive since you have to write a lot of interfaces to get things all sorted out. For a small set of interfaces required by the container, such as in Avalon's approach, this isn't too bad. But it's a lot of work for assembling components and dependencies, which is why the current crop of lightweight containers go with setter and constructor injection.
介面注入更具侵入性，因為你必須寫很多介面才能把事情都弄清楚。對於容器所需的介面集合較小，例如在 Avalon 的方法中，這並不至於太壞。但它對於組裝元件和依賴性來說工作量很大，這就是為什麼目前的輕量級容器選擇使用設定器和建構器注入。

The choice between setter and constructor injection is interesting as it mirrors a more general issue with object-oriented programming - should you fill fields in a constructor or with setters.
在設定函數與建構函數注入之間的選擇很有趣，因為它反映了物件導向程式設計中一個更普遍的問題——你應該在建構函數中還是透過設定函數來填入欄位。

My long running default with objects is as much as possible, to create valid objects at construction time. This advice goes back to Kent Beck's [Smalltalk Best Practice Patterns](https://www.amazon.com/gp/product/013476904X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=013476904X&linkCode=as2&tag=martinfowlerc-20): Constructor Method and Constructor Parameter Method. Constructors with parameters give you a clear statement of what it means to create a valid object in an obvious place. If there's more than one way to do it, create multiple constructors that show the different combinations.
我對物件長久以來的默認做法是盡可能在建構時間創建有效的物件。這個建議源於 Kent Beck 的 Smalltalk 最佳實踐模式：建構函數方法與建構函數參數方法。帶參數的建構函數在明顯的位置清楚地表明了創建有效物件的含義。如果有多種方法可以做到，就創建多個建構函數來展示不同的組合。

Another advantage with constructor initialization is that it allows you to clearly hide any fields that are immutable by simply not providing a setter. I think this is important - if something shouldn't change then the lack of a setter communicates this very well. If you use setters for initialization, then this can become a pain. (Indeed in these situations I prefer to avoid the usual setting convention, I'd prefer a method like `initFoo`, to stress that it's something you should only do at birth.)
建構函數初始化的另一個優勢是它允許你透過簡單地不提供設定函數來明確隱藏任何不可變的欄位。我認為這很重要——如果某個東西不應該改變，那麼沒有設定函數就能很好地傳達這一點。如果你使用設定函數進行初始化，那麼這可能會變得麻煩。(確實在這些情況下，我更傾向於避免通常的設定規範，我更喜歡一種像 `initFoo` 的方法，以強調這是你只應該在出生時才做的東西。)

But with any situation there are exceptions. If you have a lot of constructor parameters things can look messy, particularly in languages without keyword parameters. It's true that a long constructor is often a sign of an over-busy object that should be split, but there are cases when that's what you need.
但任何情況下都有例外。如果你有很多建構子參數，情況可能會亂七八糟，特別是在沒有關鍵字參數的語言中。確實，一個很長的建構子通常是表示一個過於忙碌的物件，應該被拆分，但在某些情況下，這正是你所需要的。

If you have multiple ways to construct a valid object, it can be hard to show this through constructors, since constructors can only vary on the number and type of parameters. This is when Factory Methods come into play, these can use a combination of private constructors and setters to implement their work. The problem with classic Factory Methods for components assembly is that they are usually seen as static methods, and you can't have those on interfaces. You can make a factory class, but then that just becomes another service instance. A factory service is often a good tactic, but you still have to instantiate the factory using one of the techniques here.
如果你有多種方式來建構一個有效的物件，透過建構子來表示這一點可能很困難，因為建構子只能根據參數的數量和類型來變化。這時候工廠方法就派上用場了，這些方法可以結合私有的建構子和設定器來實現它們的工作。經典工廠方法在組件組裝上的問題在於，它們通常被視為靜態方法，而你無法在介面上有這些方法。你可以建立一個工廠類別，但那樣它就只是另一個服務實例。工廠服務通常是一個不錯的策略，但你還是必須使用這裡提到的一種技術來實例化工廠。

Constructors also suffer if you have simple parameters such as strings. With setter injection you can give each setter a name to indicate what the string is supposed to do. With constructors you are just relying on the position, which is harder to follow.
建構函式也會受到困擾，如果你有簡單的參數，例如字串。使用設定器注入，你可以為每個設定器指定名稱，以表示這個字串的用途。使用建構函式，你只是依賴位置，這更難追蹤。

If you have multiple constructors and inheritance, then things can get particularly awkward. In order to initialize everything you have to provide constructors to forward to each superclass constructor, while also adding you own arguments. This can lead to an even bigger explosion of constructors.
如果你有多個建構函式且涉及繼承，那麼情況可能會特別尷尬。為了初始化所有內容，你必須提供建構函式來傳遞給每個超類別建構函式，同時也必須添加你自己的參數。這可能會導致建構函式的爆炸性增長。

Despite the disadvantages my preference is to start with constructor injection, but be ready to switch to setter injection as soon as the problems I've outlined above start to become a problem.
儘管有這些缺點，我的偏好是從建構函式注入開始，但當上述問題開始變成問題時，要準備切換到設定器注入。

This issue has led to a lot of debate between the various teams who provide dependency injectors as part of their frameworks. However it seems that most people who build these frameworks have realized that it's important to support both mechanisms, even if there's a preference for one of them.
這個問題已經引發了提供相依性注入器作為其框架一部分的各個團隊之間的許多爭論。然而，似乎建立這些框架的大多數人已經意識到支援這兩種機制很重要，即使他們偏好其中一種。

### Code or configuration files

程式碼或設定檔

A separate but often conflated issue is whether to use configuration files or code on an API to wire up services. For most applications that are likely to be deployed in many places, a separate configuration file usually makes most sense. Almost all the time this will be an XML file, and this makes sense. However there are cases where it's easier to use program code to do the assembly. One case is where you have a simple application that's not got a lot of deployment variation. In this case a bit of code can be clearer than a separate XML file.
另一個獨立但常被混洽的問題是，是否要在 API 上使用設定檔或程式碼來綁定服務。對於大多數可能會部署在許多地方的應用程式，一個獨立的設定檔通常更合理。幾乎在所有情況下，這會是一個 XML 檔案，這是合理的。然而，在某些情況下，使用程式碼來進行組裝更容易。一種情況是您有一個簡單的應用程式，部署變化不大。在這種情況下，一點程式碼可能比一個獨立的 XML 檔案更清楚。

A contrasting case is where the assembly is quite complex, involving conditional steps. Once you start getting close to programming language then XML starts breaking down and it's better to use a real language that has all the syntax to write a clear program. You then write a builder class that does the assembly. If you have distinct builder scenarios you can provide several builder classes and use a simple configuration file to select between them.
一個對比的情況是，當組裝相當複雜，涉及條件步驟時。一旦您開始接近程式設計語言，XML 就開始失敗，最好使用一種具有所有語法來撰寫清晰程式的真正語言。然後您撰寫一個建構類別來進行組裝。如果您有獨特的建構情況，您可以提供幾個建構類別，並使用一個簡單的設定檔來在它們之間選擇。

I often think that people are over-eager to define configuration files. Often a programming language makes a straightforward and powerful configuration mechanism. Modern languages can easily compile small assemblers that can be used to assemble plugins for larger systems. If compilation is a pain, then there are scripting languages that can work well also.
我常常認為人們過於急於定義設定檔。通常一種程式語言會提供直接且強大的設定機制。現代語言可以輕易編譯小型組裝器，可用於組裝大型系統的插件。如果編譯很麻煩，那麼也有腳本語言可以很好地工作。

It's often said that configuration files shouldn't use a programing language because they need to be edited by non-programmers. But how often is this the case? Do people really expect non-programmers to alter the transaction isolation levels of a complex server-side application? Non-language configuration files work well only to the extent they are simple. If they become complex then it's time to think about using a proper programming language.
常說設定檔不應使用程式語言，因為它們需要非程式設計師來編輯。但這種情況有多常發生？人們真的預期非程式設計師會修改複雜伺服器端應用程式的交易隔離層級嗎？非語言設定檔只有在它們簡單的情況下才運作良好。如果它們變得複雜，那麼就是思考使用正確的程式語言的時候了。

One thing we're seeing in the Java world at the moment is a cacophony of configuration files, where every component has its own configuration files which are different to everyone else's. If you use a dozen of these components, you can easily end up with a dozen configuration files to keep in sync.
目前在 Java 世界中，我們看到的是一堆設定檔的混亂，其中每個元件都有自己的設定檔，而且彼此不同。如果你使用十幾個這樣的元件，你很輕易就會有十幾個需要同步的設定檔。

My advice here is to always provide a way to do all configuration easily with a programmatic interface, and then treat a separate configuration file as an optional feature. You can easily build configuration file handling to use the programmatic interface. If you are writing a component you then leave it up to your user whether to use the programmatic interface, your configuration file format, or to write their own custom configuration file format and tie it into the programmatic interface
我的建議是，要一直提供一種能夠輕鬆進行所有設定工作的程式化介面，然後將獨立的設定檔視為一個可選功能。您可以輕鬆建置設定檔處理來使用程式化介面。如果您正在撰寫一個元件，那麼您可以讓使用者自行決定是否使用程式化介面、您的設定檔格式，或是撰寫他們自己的自訂設定檔格式並將其與程式化介面連結起來。

### Separating Configuration from Use

將設定與使用分離

The important issue in all of this is to ensure that the configuration of services is separated from their use. Indeed this is a fundamental design principle that sits with the separation of interfaces from implementation. It's something we see within an object-oriented program when conditional logic decides which class to instantiate, and then future evaluations of that conditional are done through polymorphism rather than through duplicated conditional code.
在所有這些事情中，重要的是要確保服務的設定與其使用是分離的。事實上，這是一個與介面與實作分離的基本設計原則相關的原則。在面向物件的程式中，我們可以看到當條件邏輯決定要實例化哪個類別時，未來對該條件的評估是透過多型而非重複的條件程式碼來完成的。

If this separation is useful within a single code base, it's especially vital when you're using foreign elements such as components and services. The first question is whether you wish to defer the choice of implementation class to particular deployments. If so you need to use some implementation of plugin. Once you are using plugins then it's essential that the assembly of the plugins is done separately from the rest of the application so that you can substitute different configurations easily for different deployments. How you achieve this is secondary. This configuration mechanism can either configure a service locator, or use injection to configure objects directly.
若這種分離在單一程式碼基礎中很有用，那麼當您使用外來元素（例如組件和服務）時，它尤其重要。第一個問題是您是否希望將實作類別的選擇延後到特定的部署。如果是這樣，您需要使用某種外掛的實作。一旦您開始使用外掛，那麼將外掛的組裝與應用程式的其他部分分開就至關重要，以便您可以為不同的部署輕鬆替換不同的配置。您如何達成這點是次要的。這種配置機制可以配置服務定位器，或者使用注入直接配置物件。

## Some further issues  一些進一步的問題

In this article, I've concentrated on the basic issues of service configuration using Dependency Injection and Service Locator. There are some more topics that play into this which also deserve attention, but I haven't had time yet to dig into. In particular there is the issue of life-cycle behavior. Some components have distinct life-cycle events: stop and starts for instance. Another issue is the growing interest in using aspect oriented ideas with these containers. Although I haven't considered this material in the article at the moment, I do hope to write more about this either by extending this article or by writing another.
在本篇文章中，我專注於使用相依性注入（Dependency Injection）和服務定位器（Service Locator）進行服務配置的基本問題。有一些更多與此相關的議題也值得關注，但我還沒有時間深入探討。特別是生命週期的行為問題。有些組件具有獨特的生命週期事件：例如停止和啟動。另一個問題是對使用面向方面思想（aspect oriented ideas）與這些容器的興趣日益增長。儘管我目前沒有在文章中考慮這些材料，但我希望未來能夠寫更多有關這方面的內容，無論是擴展這篇文章還是寫另一篇文章。

You can find out a lot more about these ideas by looking at the web sites devoted to the lightweight containers. Surfing from the [picocontainer](http://picocontainer.com/) and [spring](http://www.springsource.org/) web sites will lead to you into much more discussion of these issues and a start on some of the further issues.
您可以透過查看專注於輕量級容器的網站來了解這些想法的更多內容。從 picocontainer 和 Spring web 網站開始瀏覽，將會引導您進入更多有關這些問題的討論以及一些進一步問題的開始。

## Concluding Thoughts  總結性思考

The current rush of lightweight containers all have a common underlying pattern to how they do service assembly - the dependency injector pattern. Dependency Injection is a useful alternative to Service Locator. When building application classes the two are roughly equivalent, but I think Service Locator has a slight edge due to its more straightforward behavior. However if you are building classes to be used in multiple applications then Dependency Injection is a better choice.
目前輕量級容器都有一個共同的基礎模式來處理服務組裝 - 相依性注入模式。相依性注入是服務定位器的有用替代方案。在建立應用程式類別時，這兩者大致上相等，但我認為服務定位器由於其更直觀的行為而略有優勢。然而，如果你正在建立要在多個應用程式中使用的類別，那麼相依性注入是更好的選擇。

If you use Dependency Injection there are a number of styles to choose between. I would suggest you follow constructor injection unless you run into one of the specific problems with that approach, in which case switch to setter injection. If you are choosing to build or obtain a container, look for one that supports both constructor and setter injection.
如果你使用相依性注入，有許多風格可供選擇。我建議你遵循建構函式注入，除非你遇到該方法的特定問題，那麼切換到設定函式注入。如果你正在選擇建立或獲取一個容器，尋找支援建構函式和設定函式注入的容器。

The choice between Service Locator and Dependency Injection is less important than the principle of separating service configuration from the use of services within an application.
服務定位器與相依性注入之間的選擇不如將服務組態與應用程式中的服務使用分離的原則重要。

---

## Acknowledgments  致謝

My sincere thanks to the many people who've helped me with this article. Rod Johnson, Paul Hammant, Joe Walnes, Aslak Hellesøy, Jon Tirsén and Bill Caputo helped me get to grips with these concepts and commented on the early drafts of this article. Berin Loritsch and Hamilton Verissimo de Oliveira provided some very helpful advice on how Avalon fits in. Dave W Smith persisted in asking questions about my initial interface injection configuration code and thus made me confront the fact that it was stupid. Gerry Lowry sent me lots of typo fixes - enough to cross the thanks threshold.
衷心感謝許多幫助我完成這篇文章的人。Rod Johnson、Paul Hammant、Joe Walnes、Aslak Hellesøy、Jon Tirsén 和 Bill Caputo 帮助我理解這些概念，並對文章的早期草稿提出評論。Berin Loritsch 和 Hamilton Verissimo de Oliveira 就 Avalon 如何融入其中提供了一些非常有用的建議。Dave W Smith 堅持詢問我初始介面注入配置程式碼的問題，因此讓我面對到它很愚蠢的事實。Gerry Lowry 寄給我許多拼字修正——足夠跨越感謝的門檻。

Significant Revisions  重大修訂

_23 January 2004:_ Redid the configuration code of the interface injection example.
2004 年 1 月 23 日：重新修改了介面注入範例的設定程式碼。

_16 January 2004:_ Added a short example of both locator and injection with Avalon.
2004 年 1 月 16 日：添加了使用 Avalon 的定位器和注入的簡短示例。

_14 January 2004:_ First Publication
2004 年 1 月 14 日：首次發表

[![](https://martinfowler.com/thoughtworks_white.png)](https://www.thoughtworks.com/engineering)

© Martin Fowler | [Disclosures](https://martinfowler.com/aboutMe.html#disclosures)
