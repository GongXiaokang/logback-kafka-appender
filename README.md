# logback-kafka-appender

该开源版本的KafkaAppender实现了AppenderAttachable接口，
目的是在Kafka发送日志失败的时候，有一个fallback方案，可以用Console或者file等方式。

在KafkaAppender中有一个BlockingQueue，为什么会有一个BlockingQueue，原因是Kafka在发送消息的时候也会有消息产生，
如果业务触发打印日志是动作A，记录消息a，Kafka发送消息a的过程，自身会产生消息b、c，也需要经过kafka发送，
如果不做特殊处理，消息b的发送过程中又回产生e、f，消息c的发送过程会产生g、h。
那么只要触发日志记录，就会形成环路，Kafka不断处理自身打印的日志。

他的实现方式是业务日志A，kafka发送A，kafka自身产生B、C，先不发送，先放入队列中。
等到业务日志D到来，Kafka先发送B、C，再处理日志D，这过程中产生的消息再次入队。

这么看来，避免Kafka消息形成环路本身并没有解决这个问题，这个队列迟早会被打爆。

唯一的办法就是Kafka运行过程中的消息单独处理，例如记录到单独的文件中。

