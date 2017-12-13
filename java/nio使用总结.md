* buffer申请的大小以字节为单位,`hello`为5个字节,`你好`在`utf-8`下为6个字节.
* `buffer.flip()`从写模式切换到读模式,连着调用并不会从读模式切换到写模式
* `buffer.equals()`:
    当满足下列条件时，表示两个Buffer相等：
    * 有相同的类型（byte、char、int等）。
    * Buffer中剩余的byte、char等的个数相等。
    * Buffer中所有剩余的byte、char等都相同。
* `buffer.compareTo()`

    compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：

    第一个不相等的元素小于另一个Buffer中对应的元素 。
    所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。
* `selector`如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：
    ```
    int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
    ```
* select()阻塞到至少有一个通道在你注册的事件上就绪了。

  select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。

  selectNow()不会阻塞，不管什么通道就绪都立刻返回（译者注：此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）
* wakeUp()

  某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。

  如果有其它线程调用了wakeup()方法，但当前没有线程阻塞在select()方法上，下个调用select()方法的线程会立即“醒来（wake up）”。
* 注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。
