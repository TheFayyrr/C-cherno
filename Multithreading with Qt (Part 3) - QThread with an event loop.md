在简短介绍了不带事件循环的 QThread 之后，本视频将向您介绍带事件循环的线程。这样就可以在二级线程中处理对象的事件，这对于在这些对象上调用插槽是必不可少的。
本视频将介绍两种不同的方法：在同一级别创建线程和 Worker，或将线程封装到 Worker 中。
有关多线程培训的更多信息，请点击此处：


![image](https://github.com/user-attachments/assets/1393f863-af5b-4d4c-9bc6-6dd5d2cac52a)
在处理计时器、网络、排队连接等时，事件循环是必要的。Qt支持单线程事件循环：
每个线程本地事件循环传递事件或该线程中的qobject
![image](https://github.com/user-attachments/assets/39503859-5d9b-4b7c-86c2-f75e4bb1804a)


QThread::run() 的默认实现实际上调用了 QThread::exec()。这使我们能够在其他线程中运行代码而无需对 QThread 进行子类化。

![image](https://github.com/user-attachments/assets/773dcb64-bb6d-44a7-a548-90ce0fba5ede)


QThread是线程，而worker是一个工作对象，要完成工作的对象；从工作线程中取一个槽，QThread的启动信号，工作线程就是&worker::dowork槽函数中完成；
一旦启动，worker就开始工作。
当其工作完成之后，提供结果时就会发送信号称为workdone，可以连接此处得到结果。
也可将信号连接到线程退出方法（&QThread::quit）,这就是到某个实际点完成的方式
将finish信号连接到worker的deletelater槽，
将工作线程移动到辅助线程（movetothread）
一切都设置好之后就是thread->start();线程上调用start
经典模式，运行很好。问题是：但是需要处理两个不同的对象，工作程序和线程，并且要留意删除。
有一个单一对象可以同时处理这两件事，使用此模式，成为活动；
-把线程移动到工作对象中，看不到辅助线程，只能看到worker


![image](https://github.com/user-attachments/assets/3cd627a6-4cdf-457d-a60b-a895c9a8d19a)

工作方式：在工作器的实现中，我们将线程存储在成员变量m_thread中，因而，我们做的第一件事就是，使用新的QThread创建线程实例（永远不要把父对象传递给该对象！！！），；工作结束后删除线程时在辅助线程中完成，而不是调用结构在主线程中完成，用的是invoke方法在辅助线程中调用这个清理槽函数（因为该对象的所有事件都在辅助线程中处理，意味着该插槽在那里被调用）
这意味着我们可以正确的线程(处理事件的线程)中执行我们需要的任何类型的清理，(例如删除计时器、套接字或其他内容)
->quit 之后，再析构函数中，删除唯一一个指针
在不同的线程中，由处理套接字之类的事情，但我们实际看不到该线程。 我们所看到的只是套接字处理程序。

