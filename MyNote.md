# 我的笔记

## Demo启动流程

1. MyFacade中使用静态方法实例化了private static m_instance,实例化过程中，会根据父类构造方法中的模板依次初始化Model，Controller,View
1. Model中注册Proxy(代理)
1. Controller中对定义的Command进行注册
1. App.cs作为与Unity3D链接的启动入口，在' MonoBehaviur.OnAwake() ' 中调用' MyFacede.GetInstance().Launch()'（该方法非框架中)。在MyFacede.GetInstance()实例化MyFacade的过程中，会调用框架中的构造方法
1. MyDacede.Launch会发送Command 启动游戏逻辑



## Demo运行逻辑

2. 发送Command是指通过SendNotification，然后执行Facade中的NotifyObservers。这里就是一个Observer模式，具体实现方式代码暂时不看（这里的具体实现设计知识盲区）
2. Command 中执行具体的逻辑时，通过Facade和key来获取相应的Mediator(对应View)或Proxy(对应Model)，并执行相关的逻辑。应当注意的是Command与Controller中不包含状态
3. 通过Proxy更改数值时，会根据需求发送相应的Command。

## Demo运行逻辑
