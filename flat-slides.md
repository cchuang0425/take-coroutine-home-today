# 投影片基稿

## 今天就能帶回家的 Java coroutine！

## 警語

1. 架構設計有賺有賠，導入前請詳閱公開說明文件
1. 所有的設計都是 trade off
1. 沒有最好，只有...
1. 大家考慮清楚、討論充足後，願意一同承擔風險的方案

## 今天的主角

### 不是我

1. 黃俊鈞
1. 一名程式員，現任職於資策會

### 也不是它

1. Quasar
1. 

### 是各位



## 今天的脈絡

1. 問題 -> 需求
1. 需求 -> 機制
1. 機制 -> 實踐
1. 實踐 -> 驗證

## 問題 -> 需求

### 不知道，各位有沒有碰過這樣的問題？

### 假設我們有個這樣的計算問題

* 在 1 <= n <= 45 的費氏數列裡，有多少個數字是質數呢？

1. 為了表示耗時操作，本例皆用最慢演算法
1. 各位都是專業的工程師，請勿模仿本例寫法

### 費氏數列

```java
long fib(long n){
  if(n == 0 || n == 1){
    return n;
  }else{
    return fib(n - 1) + fib(n - 2);
  }
}
```

### 質數判定

```java
boolean isPrime(long value) {
  if (value <= 1) {
    return false;
  } else {
    for (long div = value - 1; div > 0; div--) {
      if ((value % div) == 0) { return div == 1; }
    }
    return false;
  }
}
```

### [同步執行](同步執行)

### 同步執行的程式

```java
List<Long> checkPrimalityFib(long n){
  List<Long> numbers = new ArrayList<>();
  for(long i = 1; i <= n; i++){
    long fibNum = fib(i);
    if (isPrime(fibNum)){ numbers.add(i); }
  }
  return numbers;
}
```

### [非同步執行](非同步執行)

### 非同步環境下的同步執行(示意)

```java
List<Long> checkPrimalityFib(long n){
  List<Long> numbers = new ArrayList<>();
  for(long i = 1; i <= n; i++){
    long fibNum = CALL_MQ_AND_WAIT(fib(i));
    if (CALL_MQ_AND_WAIT(isPrime(fibNum))){ numbers.add(i); }
  }
  return numbers;
}
```

### 真正的非同步執行程式

_<async by reactor>_

### 程式脈絡已經與先前完全不同！

### 所以，我們有以下需求

* 呼叫 Message Queue 的時候，程式可以暫停，而非等待
* Message Queue 傳回執行結果後，程式可以恢復原本執行時的狀態

## 需求 -> 機制

### 解法一：Producer Consumer Pattern

* [Producer Consumer Pattern](Producer Consumer Pattern)

### 評價

1. 優點：
    * 使用 Java 原生的機制，具有最好的相容性與穩定性
    * 使用同步式開發，程式碼易於理解、維護
    * Thread 與底層高度相依，能有效使用 CPU 計算資源
1. 缺點：
    * 所有 `wait` 在 `LinkedBlockingQueue` 上的 Producer 與 Consumer 都是活著的(activated) Thread
    * 若 ThreadPool 裡的 Thread 都在 `wait`，則會造成 ThreadPool 堵塞住
    * 若使用 `sleep` 則會造成頻繁 context switch 負擔
    * 若只是存取外部服務(Web、MQ...)，為何我們要耗用 CPU 的資源來等待呢？

### 解法二：Asynchronous

* [Asynchronous](Asynchronous)

### 評價

1. 優點：
    * 本質上完全相容於 Message Queue 訊息來往的特性
    * 已有許多成熟的專案可支援
        * Akka
        * Vert.x
        * Reactor Project
1. 缺點：
    * 既有架構必須完全翻新
    * 需要使用記憶體快取或資料持久化機制，來儲存每一個階段做到一半的資料
    * Akka 引入了 Scala，造成額外的編程範式的負擔
    * 學習曲線較高，不一定適用於團隊每個人

### 解法三：Coroutine & Continuation

* [Coroutine & Continuation](Coroutine & Continuation)

### 評價

1. 優點：
    * 可以在同步式開發的程式裡，具備非同步的執行特性
    * 輕量化的 Thread - Fiber，可以開大量的執行單元
    * 適用於非佔用 CPU 型的任務，如 IO、網路通訊等
1. 缺點：
    * 引入了外部(Java Agent)工具，增加部署上的負擔
    * 為了保持邏輯正確性，仍然需要把 Fiber 暫停前的資料暫存起來
    * 框架間相容性需要更深入研究
    * 框架都很冷門，甚至有的已經沒有在維護
      * Commons JavaFlow
      * Kilim
      * Java RIFE
      * 還有今天的 Quasar

### 最後選擇

[Quasar](Quasar)

## 機制 -> 實踐

## Quasar 101

### [QuasarWeb](QuasarWeb)

### [Quasar@MvnRepo](Quasar@MvnRepo)

### Fiber & Channel

[Fiber & Channel](Fiber & Channel)

### PrimalityFiber

_<直接看程式>_

### Gradle

```groovy
configurations {
  quasar
}
dependencies {
  compile  "co.paralleluniverse:quasar-core:0.7.10:jdk8"
  quasar   "co.paralleluniverse:quasar-core:0.7.10:jdk8@jar"
}
test {
  jvmArgs "-javaagent:${configurations.quasar.singleFile}"
}
```

### 小眉角

```groovy
task copyQuasar(type: Copy) {
    into "$projectDir/lib"
    from configurations.quasar
}
build.finalizedBy(copyQuasar)
```

## 實踐 -> 驗證

_<直接看程式>_

## 接下來，我們要講古...

## 深入探討 － Coroutine

### 定義



### 其他語言如何使用

### 在 Java 的現況

## 深入探討 － Continuation

### 定義

### 其他語言如何使用

### 在 Java 的現況

## 深入探討 － Quasar

### JavaAgent?

