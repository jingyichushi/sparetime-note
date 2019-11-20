## 案例

以上次的文档的[《消息队列之分布式事务》](https://www.bcoder.top/2019.11.12/2019/11/16/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E4%B9%8B%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1/)购物车为例，来进行验证分布式事务，具体如图所示：

![](http://img.bcoder.top/2019.11.12/1.png)



## RocketMQ如何实现分布式事务

在 RocketMQ 中的事务实现中，**增加了事务反查的机制来解决事务消息提交失败的问题**。如果 Producer 也就是订单系统，在提交或者回滚事务消息时发生网络异常，RocketMQ 的 Broker 没有收到提交或者回滚的请求，Broker 会定期去 Producer 上反查这个事务对应的本地事务的状态，然后根据反查结果决定提交或者回滚这个事务。

为了支撑这个事务反查机制，我们的业务代码需要实现一个反查本地事务状态的接口，告知 RocketMQ 本地事务是成功还是失败。

在我们这个例子中，反查本地事务的逻辑也很简单，我们只要根据消息中的订单 ID，在订单库中查询这个订单是否存在即可，如果订单存在则返回成功，否则返回失败。RocketMQ 会自动根据事务反查的结果提交或者回滚事务消息。

这个反查本地事务的实现，并不依赖消息的发送方，也就是订单服务的某个实例节点上的任何数据。这种情况下，即使是发送事务消息的那个订单服务节点宕机了，RocketMQ 依然可以通过其他订单服务的节点来执行反查，确保事务的完整性。



![](http://img.bcoder.top/2019.11.12/2.png)



## 场景构造

### 正常场景

#### RocketMQ生产者代码（订单系统）

**1.yml配置**

```yaml
rocketmq:
  name-server: test-host:9876
  producer:
    group: zln
```



**2.核心代码：**

```java
@Service
@Slf4j
public class OrdersServiceImpl implements OrdersService {

    private final OrdersMapper ordersMapper;

    private final RocketMQTemplate rocketMQTemplate;

    @Autowired
    public OrdersServiceImpl(OrdersMapper ordersMapper, RocketMQTemplate rocketMQTemplate) {
        this.ordersMapper = ordersMapper;
        this.rocketMQTemplate = rocketMQTemplate;
    }
    
    @Override
    @Transactional
    public String addOrder() {
        log.info("创建订单");
        Orders orders = Orders.builder().name(UUID.randomUUID().toString()).piece(12.0).build();
        ordersMapper.insert(orders);

        log.info("订单消息发送给rocketmq");
        rocketMQTemplate.convertAndSend("order-topic", orders);

        return "创建成功";
    }

}
```

#### RocketMQ消费者者代码（购物车系统）

**1.yml配置**

```yml
rocketmq:
  name-server: test-host:9876
```



**2.核心代码：**

```java
@Slf4j
@Service
@RocketMQMessageListener(topic = "order-topic", consumerGroup = "zln")
public class OrdersListener implements RocketMQListener<String>{

    @Override
    public void onMessage(String s) {
        log.info("收到订单信息，清除当前购物车信息。订单信息为：{}", s);
    }
}
```



#### 验证

发送消息（生产端）：

![](http://img.bcoder.top/2019.11.12/4.png)



![](http://img.bcoder.top/2019.11.12/3.png)



![](http://img.bcoder.top/2019.11.12/5.png)



消费端：

![](http://img.bcoder.top/2019.11.12/6.png)





### 异常场景

只需修改生产端代码即可：

```java
@Service
@Slf4j
public class OrdersServiceImpl implements OrdersService {

    private final OrdersMapper ordersMapper;

    private final RocketMQTemplate rocketMQTemplate;

    @Autowired
    public OrdersServiceImpl(OrdersMapper ordersMapper, RocketMQTemplate rocketMQTemplate) {
        this.ordersMapper = ordersMapper;
        this.rocketMQTemplate = rocketMQTemplate;
    }

    @Override
    @Transactional
    public String addOrder() {
        log.info("创建订单");
        Orders orders = Orders.builder().name(UUID.randomUUID().toString()).piece(12.0).build();
        ordersMapper.insert(orders);

        log.info("订单消息发送给rocketmq");
        rocketMQTemplate.convertAndSend("order-topic", orders);
        log.info("异常了");
        int i = 1 / 0;
        return "创建成功";
    }

}
```



#### 验证

发送消息（生产端）：

![](http://img.bcoder.top/2019.11.12/7.png)



![](http://img.bcoder.top/2019.11.12/8.png)



![](http://img.bcoder.top/2019.11.12/9.png)



消费端：

![](http://img.bcoder.top/2019.11.12/10.png)



可以看出，生产端抛出异常后，本地数据库回滚，但是消息已经发出，消费端针对该消息进行了处理，因此需要造成了消息的不一致。



### 分布式事务场景

#### 核心代码

```java
@Service
@Slf4j
public class OrdersServiceImpl implements OrdersService {


    private final RocketMQTemplate rocketMQTemplate;

    @Autowired
    public OrdersServiceImpl(RocketMQTemplate rocketMQTemplate) {
        this.rocketMQTemplate = rocketMQTemplate;
    }

    @Override
    @Transactional
    public String addOrder() {
        log.info("创建订单");
        Orders orders = Orders.builder().name(UUID.randomUUID().toString()).piece(12.0).build();
        Message<String> message = MessageBuilder.withPayload(JacksonUtils.bean2Json(orders)).build();
        rocketMQTemplate.sendMessageInTransaction("send-order", "order-topic", message, null);
        return "创建成功";
    }

}
```



```java
@Slf4j
@Service
@RocketMQTransactionListener(txProducerGroup = "send-order")
public class TransactionListenerImpl implements RocketMQLocalTransactionListener {

    private final OrdersService ordersService;

    @Autowired
    public TransactionListenerImpl(OrdersService ordersService) {
        this.ordersService = ordersService;
    }

    @Transactional
    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
        try {
            log.info("消息发送成功执行本地事务");
            String jsonString = new String((byte[]) msg.getPayload());
            Orders orders = JacksonUtils.json2Bean(jsonString, Orders.class);
            log.info("订单创建");
            ordersService.add(orders);
            return RocketMQLocalTransactionState.COMMIT;
        } catch (Exception e) {
            log.error("本地事务执行失败");
            e.printStackTrace();
            return RocketMQLocalTransactionState.ROLLBACK;
        }
    }
    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
        log.info("检查事务执行状态");
        return RocketMQLocalTransactionState.COMMIT;
    }
}
```



```java
@Service
@Slf4j
public class OrdersServiceImpl implements OrdersService {

    private final OrdersMapper ordersMapper;

    @Autowired
    public OrdersServiceImpl(OrdersMapper ordersMapper) {
        this.ordersMapper = ordersMapper;
    }


    @Transactional
    @Override
    public void add(Orders orders) {
        ordersMapper.insert(orders);
    }

}
```







其他不变。

#### 验证正常情况

生产端：

![](http://img.bcoder.top/2019.11.12/11.png)



![](http://img.bcoder.top/2019.11.12/12.png)



![](http://img.bcoder.top/2019.11.12/13.png)



消费端：



![](http://img.bcoder.top/2019.11.12/14.png)





#### 验证异常情况

修改TransactionListenerImpl.java的executeLocalTransaction方法：

```java
@Service
@Slf4j
public class OrdersServiceImpl implements OrdersService {

    private final OrdersMapper ordersMapper;

    @Autowired
    public OrdersServiceImpl(OrdersMapper ordersMapper) {
        this.ordersMapper = ordersMapper;
    }


    @Transactional
    @Override
    public void add(Orders orders) {
        int i = 1 / 0;
        ordersMapper.insert(orders);
    }

}
```



生产端：

![](http://img.bcoder.top/2019.11.12/16.png)



![](http://img.bcoder.top/2019.11.12/17.png)





消费端：



![](http://img.bcoder.top/2019.11.12/18.png)



可以发现，生产端发送的消息因为本地事务异常，并没有收到，消息一致性得到保证。



在验证最后一个场景时，遇到了一个异常：Transaction rolled back because it has been marked as rollback-only,这里做一个记录。









