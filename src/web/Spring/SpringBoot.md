# CompletableFuture：异步编程利器原理深究

## Completable Future 的结构

既然 CompletableFuture 这么好用，那么它的原理是什么？下面我们一起来看看。CompletableFuture 实现了两个接口，如下图所示：Future、CompletionStage。

![](../../assets/future1.png)

- **Future** 接口代表了一个异步计算的结果，也就是说，它代表了某个未来可能完成的计算结果。Future 接口提供了获取结果、检查是否完成、取消任务等方法。
- **CompletionStage** 用于表示异步执行过程中的一个步骤（Stage），这个步骤可能是由另外一个 CompletionStage 触发的，随着当前步骤的完成，也可能会触发其他一系列 CompletionStage 的执行。从而我们可以根据实际业务对这些步骤进行多样化的编排组合，CompletionStage 接口正是定义了这样的能力，我们可以通过其提供的 thenAppy、thenCompose 等函数式编程方法来组合编排这些步骤.

CompletableFuture 中包含两个字段：result 和 stack。

![future2.png](../../assets/future2.png)

- **result** 用于存储当前 CF 的结果。
- **stack（Completion）** 表示当前 CF 完成后需要触发的依赖动作，去触发依赖它的 CF 的计算，依赖动作可以有多个（表示有多个依赖它的 CF），以栈的形式存储，stack 表示栈顶元素。

这种方式类似“观察者模式”，依赖动作（Dependency Action）都封装在一个单独 Completion 子类中。CompletableFuture 中的每个方法都对应了图中的一个 Completion 的子类，Completion 本身是观察者的基类。

![future3.png](../../assets/future3.png)

## CompletableFuture 的设计思想

按照类似“观察者模式”的设计思想，原理分析从“观察者”和“被观察者”两个方面着手。由于回调种类多，但结构差异不大，所以这里以一元依赖中的 thenApply 为例。

![future4.png](../../assets/future4.png)

### 被观察者

1. 每个 CompletableFuture 都可以被看作一个被观察者，其内部有一个 Completion 类型的链表成员变量 stack，用来存储注册到其中的所有观察者。当被观察者执行完成后会弹栈 stack 属性，依次通知注册到其中的观察者。上面例子中步骤 fn2 就是作为观察者被封装在 UniApply 中。
2. 被观察者 CF 中的 result 属性，用来存储返回结果数据。这里可能是一次 RPC 调用的返回值，也可能是任意对象，在上面的例子中对应步骤 fn1 的执行结果。

### 观察者

CompletableFuture 支持很多回调方法，例如 thenAccept、thenApply、exceptionally 等，这些方法接收一个函数类型的参数 f，生成一个 Completion 类型的对象（即观察者），并将入参函数 f 赋值给 Completion 的成员变量 fn，然后检查当前 CF 是否已处于完成状态（即 result != null），如果已完成直接触发 fn，否则将观察者 Completion 加入到 CF 的观察者链 stack 中，再次尝试触发，如果被观察者未执行完则其执行完毕之后通知触发。

1. 观察者中的 dep 属性：指向其对应的 CompletableFuture，在上面的例子中 dep 指向 CF2。
2. 观察者中的 src 属性：指向其依赖的 CompletableFuture，在上面的例子中 src 指向 CF1。
3. 观察者 Completion 中的 fn 属性：用来存储具体的等待被回调的函数。这里需要注意的是不同的回调方法（thenAccept、thenApply、exceptionally 等）接收的函数类型也不同，即 fn 的类型有很多种，在上面的例子中 fn 指向 fn2。

## 整体流程

### 一元依赖

以 thenApply 为例来说明一元依赖的流程：

1. 将观察者 Completion 注册到 CF1，此时 CF1 将 Completion 压栈。
2. 当 CF1 的操作运行完成时，会将结果赋值给 CF1 中的 result 属性。
3. 依次弹栈，通知观察者尝试运行。

### 二元依赖

thenCombine 操作表示依赖两个 CompletableFuture。其观察者实现类为 BiApply，BiApply 通过 src 和 snd 两个属性关联被依赖的两个 CF，fn 属性的类型为 BiFunction。与单个依赖不同的是，在依赖的 CF 未完成的情况下，thenCombine 会尝试将 BiApply 压入这两个被依赖的 CF 的栈中，每个被依赖的 CF 完成时都会尝试触发观察者 BiApply，BiApply 会检查两个依赖是否都完成，如果完成则开始执行。

![future5.png](../../assets/future5.png)

### 多元依赖

依赖多个 CompletableFuture 的回调方法包括 allOf、anyOf，区别在于 allOf 观察者实现类为 BiRelay，需要所有被依赖的 CF 完成后才会执行回调；而 anyOf 观察者实现类为 OrRelay，任意一个被依赖的 CF 完成后就会触发。二者的实现方式都是将多个被依赖的 CF 构建成一棵平衡二叉树，执行结果层层通知，直到根节点，触发回调监听。

![future6.png](../../assets/future6.png)

### 执行流程

```Java | pure
CompletableFuture<String> base = new CompletableFuture<>();
CompletableFuture<String> future =
        base.thenApply(
                s -> {
                    System.out.println("2");
                    return s + " 2";
                });
base.thenAccept(s -> System.out.println("3-1")).thenAccept(aVoid -> System.out.println("3-2"));
base.thenAccept(s -> System.out.println("4-1")).thenAccept(aVoid -> System.out.println("4-2"));
base.complete("1");
System.out.println("base result: "+ base.join());
System.out.println("future result: "+ future.join());
```

我们分解执行步骤：

1. base.complete("1")后 base 里的 result 属性会变成 1
2. 取 base 中 stack（对象 4-1）执行，出栈
3. 取对象 1 中 dep 属性的 stack（对象 4-2）执行，出栈
4. 取 base 中 stack（对象 3-1）执行，出栈
5. 取对象 3 中 dep 属性的 stack（对象 3-2）执行，出栈
6. 取 base 中 stack（对象 2）执行，出栈
   最终 stack 之间的引用关系如何所示：

![future8.png](../../assets/future8.png)
