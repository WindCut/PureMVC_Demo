# 我的笔记

## Demo启动流程

1. MyFacade中使用静态方法实例化了private static m_instance,实例化过程中，会根据父类构造方法中的模板依次初始化Model，Controller,View
2. Model中注册Proxy(代理)
3. Controller中对定义的Command进行注册
4. App.cs作为与Unity3D链接的启动入口，在`MonoBehaviur.OnAwake()`中调用Demo作者的 `MyFacede.GetInstance().Launch()` 启动游戏。 在`MyFacede.GetInstance()`实例化MyFacade的过程中，会调用框架中的构造方法
5. MyDacede.Launch会发送Command 启动游戏逻辑


## Demo运行逻辑
1. 发送Command是指通过SendNotification，然后执行Facade中的NotifyObservers。这里就是一个Observer模式，具体实现方式代码暂时不看（这里的具体实现设计知识盲区）
2. Command 中执行具体的逻辑时，通过Facade和key来获取相应的Mediator(对应View)或Proxy(对应Model)，并执行相关的逻辑。应当注意的是Command与Controller中不包含状态
3. 通过Proxy更改数值时，会根据需求发送相应的Command


## Demo运行逻辑分析
这一段只描述Demo中的逻辑流，直接从启动之后开始。这一段流程写了一堆，主要就是流程通过事件的发送与接收进行连接，command从Facade中获取相应的mediatr与proxy进行处理，
1. 游戏启动startUpCcommand接受` startUP`之后实例化MainPanel，注册相应的Mediator,发送更新道具命令
2. RefreshRewardPoolCommand 接受刷新命令之后，通过Facade获取奖励池的proxy并执行Proxy中的方法设置Model的数据。之后获取Mediator并根据数据初始化或更新Mediator中。以上步骤执行完之后会发送更新奖励UI(REFRESH_BONUS_UI)的事件
3. MainPanelMediator 接受 EFRESH_BONUS_UI之后执行相应的方法更新UI。之后等待用户的操作
4. 用户点击Play按钮之后，Mediator发送PLAY事件
5. PlayCommand接受Play事件之后，通过Facade获取奖励池的proxy并随机到相应物品，通过Facade获取用户数据的proxy并调用proxy中的方法将奖励数据设置到给玩家数据中，**注意command只调用proxy的方法，不直接操作数据，也无需知晓data的结构**
6. playerDataProxy更改玩家数据时，由于数据变化，proxy会发送UPDATE_PLAYER_DATA事件，
7. MainPanelMediator接受命令，更新页面并发送`REWARD_TIP_VIEW`奖励提示事件
8. RewardTipCommand 接收`REWARD_TIP_VIEW`命令，打开RewardTipMediator(如果没有则新建)，之后发送`UPDATE_REWARD_TIP_VIEW`
9. RewardTipViewMediator 接受`UPDATE_REWARD_TIP_VIEW`命令并执行更新页面。之后玩游主动发送的事件，等待玩家操作

## 框架的目录分析
1. FrameWork的Core中为核心的Model、View、Model。这三部分基本不会由用户直接调用，完成了但部分的操作
2. Interface 框架的比较底层的抽象接口。定义了框架的操作
3. Pattern 我理解为设计模式的模板，也是接口的实现。根据设计模式分了子目录，很有启发

## 模块的目录分析
~~模块中仍然分成ViewModelController，但是其中只有相应的。~~
不说了，就是一般的目录分析就可以了


## 记录问题
1. 通知的流向？
   >Mediator、Proxy、Controller都可以发送事件。但是Proxy(model)不接受通知
2. View，Model，Controller可不可以再继承
   >继承时可以继承的。但是重写方法也没什么必要，而且重写之后还需要修改Facade，使其代码使用即成的view model controller
3. facede中存在神奇的GameManager和一些其他的Manager，这些manager是做什么用的？
   >注意到框架代码中manager的类型时Object，且获取方法使用了泛型(限定为UnityEngine.Component)，猜测这里应该是一些自定义的Manager
   >gameManager 是一个UnityEnginr.GameObject似乎不是框架原有的而是一位大佬加进去的，就是作为Unity通常的GameManager。
   >框架中提供了创建manager时将Manager添加到GameManager中
4. RegisterMediator 中会根据传入的Mediator的name做关键词，那么也就是说同一类型的的多个Mediator Register时需要线设置好各自的名字。那么命名方法框架有没有提供以及获取Mediator时怎么知道需要获得的Mediator名字
   >确实需要再创件时设置好自己的名字，在View的MediatorDictionary中会使用名字作为Key。获取Mediator时也需要传入名字
5. Mediator,Proxy,Command的销毁处理？
   >框架中存在RemoveXXX以及OnRemove可以用作销毁处理
6. 若有多个数据同时改变，单个不同的Mediator监听所有改变数据中的多个时，怎么处理？（如：Mediator1关注v1，v3变化，而v1，v3的变化被分到两个事件中）
   >不需要共同表现，分开就分开
   >如果需要共同对多个属性进行变化，具体情况具体分析吧。。
7. Facade初始化时就吧所有的View(这里指UI GameObject)都实例化了？
   >可以根据获取Mediator时是否获取了null来判断是否需要初始化
   >不公具体创建哪一个mediator需要通过硬编码的方式来解决
   >仿佛有其他的框架时解决了这个问题的
8. 业务逻辑与域逻辑：
   >业务逻辑:指的是那些需要协调Model与View的逻辑。域逻辑:指的是仅仅是针对数据模型的操作，不论是对于客户端还是对于服务端，不论是同步的操作还是异步的操作。
   >业务逻辑理所当然应该放在Command里来完成，而域逻辑应当放在Proxy里完成
   >
   
9. Demo启动流程中Mediator的注册在哪里？？为什么没有都在Facade中注册
