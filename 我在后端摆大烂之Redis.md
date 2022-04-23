# 我在后端摆大烂之Redis

## 引入

相信大家都用过思政学时小程序，也经历过活动开始报名不到一秒就结束的场景，那么大家有没有想过，我们的程序是怎样保证报名的学生不超过报名人数上限的呢？也就是说，假设有一千个人在一秒内同时点击报名活动，这个活动的报名人数上限是200人，我们的程序要如何精准的控制只让200人成功报名，而不会出现报名成功的人数超出上限的情况。接下来我们就来看一下如何通过Redis来处理这个问题。

## 介绍

Redis是现在最受欢迎的NoSQL数据库之一，它是使用C语言编写的开源、支持网络、基于内存、分布式、可选持久性的、灵活的高性能 key-value 数据结构存储数据库，可以用来作为数据库、缓存和消息队列。

Redis具备的特点是：

- C/S通讯模型
- 单进程单线程模型
- 丰富的数据类型
- 操作具有原子性
- 持久化
- 高并发读写
- 支持lua脚本

Redis 的应用场景包括：缓存系统（“热点”数据：高频读、低频写）、计数器、消息队列系统、排行榜、社交网络和实时系统。

## 基本操作

Redis提供的数据类型主要分为5种自有类型和一种自定义类型，这5种自有类型包括：String类型、哈希类型、列表类型、集合类型和顺序集合类型。

![reids](https://yansp.oss-cn-beijing.aliyuncs.com/reids.png)

String类型：

它是一个二进制安全的字符串，意味着它不仅能够存储字符串、还能存储图片、视频等多种类型, 最大长度支持512M。

对每种数据类型，Redis都提供了丰富的操作命令，如：

- GET/MGET
- SET/SETNX/SETEX/MSET
- INCR/DECR
- GETSET
- DEL

我们暂时只需要知道第一行中的GET命令、第二行中的SET命令和SETNX命令以及最后一行的DEL命令。

- GET命令用于获取指定key的值

  ```sh
  GET key_name
  ```

- SET命令用于设置给定key的值，如果key已经存储其他值， SET命令会覆盖旧值

  ```sh
  SET key_name value
  ```

- SETNX命令用于在指定的key不存在时，为key设置指定的值，否则不做任何操作

  ```sh
  SETNX key_name value
  ```

- DEL命令用于删除已存在的键，不存在的键会忽略

  ```sh
  DEL key_name
  ```

  

其他数据类型我们就先不展开了，大家有兴趣可以参考别的资料学习~

**大家看到这里可能已经懵了，不过没关系(~~看不懂就放弃~~)，我们直接看案例。**

## 案例引入

我们先来看一下报名的流程，用户发送报名请求到服务器，服务器接收到用户请求后，先读取Redis中存储的剩余报名容量数据，判断容量是否足够(容量大于0)。如果容量足够，则将容量减1写回Redis，并返回给用户报名成功信息；如果容量不足，则直接返回给用户报名人数已满的信息。

流程图如下：

![image-20220420032056918](https://yansp.oss-cn-beijing.aliyuncs.com/image-20220420032056918.png)

代码实现：

```java
// 代码01
@RestController
public class ApplyController {

    // 自动注入一个主要操作字符串的Redis模板类
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/reduce_apply_number")
    private String reduceApplyNumber() {

        // 从redis中取出剩余报名容量数据
        int applyNumber = Integer.parseInt(stringRedisTemplate.opsForValue().get("applyNumber"));

        /*
        	判断容量
        	如果容量足够，容量减1并写入数据库
        	如果容量不足，返回错误信息
        */
        if (applyNumber > 0) {
            // 新报名容量
            int newApplyNumber = applyNumber - 1;
            // 更新Redis中的报名容量信息
            stringRedisTemplate.opsForValue().set("applyNumber", newApplyNumber + "");
            System.out.println("报名成功，剩余报名容量: " + newApplyNumber);
        } else {
            System.out.println("报名失败，剩余报名容量不足");
            // 返回报名失败信息
            return "error";
        }

        // 返回报名成功信息
        return "success";
    }
}
```

上面这段代码很朴素，它的主要逻辑就三步，读数据 -> 判断 -> 返回。这段代码乍一看好像没什么问题，完美实现了我们的需求。

那我们来思考一下这样的情景，*在某一个时刻有两个用户**恰巧**同时发送了报名请求，又**恰巧**同时执行了第12行代码（读取剩余报名容量数据，假设取到的applyNumber是101），那么当这两个请求进入if语句块时，得到的newApplyNumber值都是100。大家发现问题了么？当程序执行第23行语句向Redis写数据时，两个请求写入的数据都是100。*诶，出大问题！我这原本剩101个报名位置，来了2个人报名，结果我还剩100个报名位置，这不搞笑呢么？

这时候有的小伙伴可能就会说了：**`这不简单，你加个synchronized锁一下不就完事了。`**

说的很对(~~下次不要再说了~~)，我们可以加个锁去解决这个问题。上面这段代码的问题在于，没有保证操作**原子性**。也就是说，我们程序的读数据操作和写入新数据操作要一起执行，**不能分割开来**。我们可以通过synchronized关键字来实现操作原子性。

这里解释一下**synchronized**，它是Java语言的关键字，可用来给对象和方法或者代码块加锁，当它锁定一个方法或者一个代码块的时候，同一时刻最多只有一个线程执行这段代码。当两个并发线程访问同一个对象object中的这个加锁同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块。

我们可以用synchronized关键字，将代码01中的第11行到第31行代码块包裹起来，保证我们的读操作、判断操作和写操作一起执行，即一次只能有一个用户执行这段代码。

代码修改如下：

```java
// 代码02
@RestController
public class ApplyController {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/reduce_apply_number")
    private String reduceApplyNumber() {

        // 给代码块"上锁"，一直锁到第22行
        synchronized (this) {
            int applyNumber = Integer.parseInt(stringRedisTemplate.opsForValue().get("applyNumber"));
            if (applyNumber > 0) {
                int newApplyNumber = applyNumber - 1;
                stringRedisTemplate.opsForValue().set("applyNumber", newApplyNumber + "");
                System.out.println("报名成功，剩余报名容量: " + newApplyNumber);
            } else {
                System.out.println("报名失败，剩余报名容量不足");
                return "error";
            }
        }

        return "success";
    }
}
```

有的小伙伴可能就会问了：**`这样就解决问题了吗？感觉也不是很难嘛`**

对，这样就能解决问题，但是只能解决**单机部署**的问题。单机部署指的是我们的服务只部署在一台服务器上，那为什么不能解决**不单机部署**的问题呢？

让我们再来思考一下这样一个场景，学校第一次部署思政学时服务的时候经验不足，租用了一台性能一般的服务器来部署，以致于小伙伴们在使用思政学时小程序的时候，经常出现卡顿、不流畅的情况，于是纷纷建议学校改善这种情况。学校经过深思熟虑之后，决定(~~氪金~~)多租用一台服务器来部署思政学时，减轻另一台服务器的压力。这时候我们就有两台服务器部署思政学时服务了，也就是**不单机部署**。我们可以通过Nginx（一个高性能的HTTP和反向代理web服务器）来实现两台服务器之间有效地分配客户端请求。即Nginx可以帮助我们弹性分配用户请求，将请求发送到更空闲的那台服务器去处理。好了，进入正题，让我们看看下面这张流程图。

![image-20220420044058447](https://yansp.oss-cn-beijing.aliyuncs.com/image-20220420044058447.png)



小伙伴们发现问题出在哪里了吗？对，坏就坏在请求分发上！真是成也萧何，败也萧何。

我们来设想这样一个场景，当A、B和C三个用户同时发送报名请求时，Nginx服务器将A、B用户的请求分发到服务器1处理，将C用户的请求分发到服务器2处理。我们先来看服务器1，假设A请求先进入代码02中的synchronized代码块中执行，那么B请求就会被拦截在synchronized代码块外，直到A请求处理完毕才能进入其中执行，这里没有任何问题。我们再来看服务器2，即便服务器1中的A请求先进入了synchronized代码块中，并且还未执行结束，C请求也能顺利进入synchronized代码块中执行。

**`哈？为什么C能进去，不是被A锁住了吗？`** 

对，A请求是锁住了synchronized代码块，但是只锁住了服务器1中的synchronized代码块。注意，我们在服务器1和服务器2分别部署了思政学时服务，虽然代码内容相同，但它们是两个完全不同的程序，也就是说会有两把不同的锁，因此A请求的锁能锁住B请求，但是锁不住C请求。就像我和A都有《Java核心技术卷Ⅰ》这本书，虽然书里的内容相同，但是我阅读我的书并不影响A阅读他的书。如果A的这本书是和B合买的，他们俩只有这一本书，A阅读A想看的部分时，B就不能阅读B想看的部分（假设A和B想看的部分不一样，~~严谨~~）。

**`那这样不就又出现了2个人报名，剩余名额只减1的情况了吗，怎么办呢？`**

是的，synchronized关键字已经不管用了，接下来由我们的主角Redis来解决这个问题。只需要通过上面提到的SETNX操作，就可以实现一把朴素的分布式锁，同步服务器之间的请求。

代码如下：

```java
// 代码03
@RestController
public class ApplyController {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/reduce_apply_number")
    private String reduceApplyNumber() {

        /*
        	setIfAbsent()方法相当于SETNX方法，第一个参数是key，第二个参数是value
        	SETNX命令用于在指定的key不存在时，为key设置指定的值，否则不做任何操作
        	setIfAbsent()在指定的key不存在时，为key设置指定的值，并返回ture，否则返回false
        */
        Boolean lock = stringRedisTemplate.opsForValue().setIfAbsent("lock_key", "1");
        
        // 当lock不为true时，说明锁已被他人获取，返回错误信息
        if (!lock) {
            return "error";
        }

        int applyNumber = Integer.parseInt(stringRedisTemplate.opsForValue().get("applyNumber"));
        if (applyNumber > 0) {
            int newApplyNumber = applyNumber - 1;
            stringRedisTemplate.opsForValue().set("applyNumber", newApplyNumber + "");
            System.out.println("报名成功，剩余报名容量: " + newApplyNumber);
        } else {
            System.out.println("报名失败，剩余报名容量不足");
            return "error";
        }
        
        // 报名逻辑执行完毕，删除key（释放锁）
        stringRedisTemplate.delete("lock_key");

        return "success";
    }
}
```

我们来观察一下代码03，同代码02相比，它加入了第16、19、20、21、34行共五行代码，去除了synchronized关键字。

第16行代码中setIfAbsent()方法相当于执行了一个SETNX操作，这有什么用呢？

让我们回到上面那个场景，A和B请求被分发至服务器1，C请求被分发至服务器2。

还是先看服务器1，假设A请求先进入方法执行，A请求会先执行第16行代码，也就是在Redis中执行了一个SETNX的操作。由于A请求是第一个到达的请求，Redis里不存在"lock_key"这个key，因此这个SETNX操作会执行成功，setIfAbsent()方法将返回true，此时lock的值就为true，A请求很顺利的通过第19-21行的if语句，执行下面的报名逻辑代码。服务器1中紧随其后的B请求执行第16行代码时，由于Redis里已经存在"lock_key"这个key了，setIfAbsent()方法将返回false，B请求不能通过第19-21行的if语句，也就不会影响A请求的原子性操作了。

我们再来看服务器2，还是假设A请求比C请求早了那么一点点执行第16行代码，当C请求执行第16行代码时，SETNX命令发现Redis中已经存在"lock_key"了，因此setIfAbsent()方法也会返回false，C请求也被if语句拦截，无法执行报名逻辑代码，成功解决多台服务器之间的同步问题！

当服务器1中的A请求执行完报名逻辑代码后，会执行第34行代码，删除Redis中的"lock_key"，即释放锁的过程。当A请求释放了这把锁之后，接下来的新请求（B和C重新报名或者D、E等其他用户报名）又能重新申请这把锁，并执行报名逻辑代码。

**`这就是分布式锁吗，学到了学到了！`**

还没完，代码03只是实现了一把朴素分布式锁，里面还存在很大的问题。比如说，当A请求执行完第16行代码拿到锁之后，在A请求执行第34行释放锁的代码之前，我们的报名逻辑代码抛出了一个异常。这时候A请求已经退出程序，且没有执行释放锁的代码，这样就造成其他正常请求申请锁一直不成功，导致活动报名崩溃。

**`这个我会，咱可以try catch一下，把释放锁的代码放到finally语句块中，这样就能保证单个请求出现异常也能释放锁了！`**

说得好！这在一定程度上确实优化了我们的代码，可万一这个异常...是这台服务器宕机了呢？比如说机房突然断电，管理员误操作关机，服务器突然被黑客攻击打崩了等等。显然，这种情况finally语句块也不能保证锁一定能释放掉，服务器都挂了就更不用说程序了。这样就会导致服务器1处于宕机状态，服务器2处于正常工作状态，但是无法提供正常的报名服务。

那这种情况我们怎么应对呢？很简单，我们只需要给这把锁设置一个过期时间。如果请求正常执行，执行完毕后会自己释放这把锁；如果请求在程序执行过程中出现了异常，程序不能正常释放这把锁，那么当这把锁的过期时间一到，Redis中的"lock_key"就不存在了，锁自然就被释放掉了。

代码如下：

```java
// 代码04
@RestController
public class ApplyController {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/reduce_apply_number")
    private String reduceApplyNumber() {

        // 在setIfAbsent()方法中多传入两个参数
        // 10代表时间数值，TimeUnit.SECONDS代表这个数值的单位是秒
        Boolean lock = stringRedisTemplate.opsForValue().setIfAbsent(
            			"lock_key", "1", 10, TimeUnit.SECONDS);
        if (!lock) {
            return "error";
        }

        // 加入try catch finally语句块
        try {
            int applyNumber = Integer.parseInt(stringRedisTemplate.opsForValue().get("applyNumber"));
            if (applyNumber > 0) {
                int newApplyNumber = applyNumber - 1;
                stringRedisTemplate.opsForValue().set("applyNumber", newApplyNumber + "");
                System.out.println("报名成功，剩余报名容量: " + newApplyNumber);
            } else {
                System.out.println("报名失败，剩余报名容量不足");
                return "error";
            }
        } finally {
            stringRedisTemplate.delete("lock_key");
        }

        return "success";
    }
}
```

在代码04中，我们向setIfAbsent()多传入两个参数10和TimeUnit.SECONDS，代表设置过期时间为10秒。这个方法保证了**SETNX操作**和**设置过期时间操作**的原子性，因此我们不需要担心这里的线程安全问题。这里的线程安全问题是指，如果setIfAbsent()方法不能保证原子性，那么可能会出现这样一种情况：当我SETNX操作执行完成后，即锁已经被加上了，这时服务器崩溃，锁无法被正常释放，而设置过期时间操作还没有执行，导致锁一直无法被释放，形成我们讨论的上一种情况。

你以为这样就万无一失了吗？错了，还没有，让我们来看一下下面这张图。

![4bb06a7efdb34fd79cec7c2f16ce9e08](https://yansp.oss-cn-beijing.aliyuncs.com/4bb06a7efdb34fd79cec7c2f16ce9e08.png)

> 图源：[CSDN@承与](https://blog.csdn.net/cheng__yu/article/details/122620213)

这张图很好理解，如果当A请求获取到锁且正在执行报名逻辑代码时，锁的过期时间到了，这时候Redis会删除"lock_key"，也就是释放了锁。这时候B请求成功获取了锁，正在执行报名逻辑代码，而这时候A请求才刚好执行完报名逻辑代码，紧接着执行释放锁的代码。问题来了，A请求的锁已经超时被释放，而A请求又执行了释放锁的代码，也就是说A请求将B请求的锁释放掉了。这时候C请求进来，B请求将C请求的锁释放，D请求进来，C请求将D请求的锁释放。

显然，这样会有导致超报名情况发生的可能。那我们如何避免这种情况的发生呢？将锁的过期时间延长？这不是一个好办法。理论上无论过期时间设置的多长，都有可能出现上述情况。而且如果我们将过期时间设置的过于长，反而失去了设置它的意义。比如说我们将过期时间设置为1小时，那么当A请求发生意外而无法释放锁时，其他用户要等待1小时才能继续报名活动，这并不是我们想要的结果。

我们换个角度思考一下，产生这个问题的原因，是一个进程释放了另一个进程的锁，即A请求释放了B请求的锁，那我们只需要保证每个进程都只能释放自己的锁，而无法释放其他进程的锁，这个问题就解决了。

代码如下：

```java
// 代码05
@RestController
public class ApplyController {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @RequestMapping("/reduce_apply_number")
    private String reduceApplyNumber() {

        // 为每个进程生成一个随机且唯一的字符串，用于标识进程（用户）
        String lockId = UUID.randomUUID().toString();
        // 将lock_key的值设置为用户唯一标识
        Boolean lock = stringRedisTemplate.opsForValue().setIfAbsent(
            			"lock_key", lockId, 10, TimeUnit.SECONDS);
        if (!lock) {
            return "error";
        }

        try {
            int applyNumber = Integer.parseInt(stringRedisTemplate.opsForValue().get("applyNumber"));
            if (applyNumber > 0) {
                int newApplyNumber = applyNumber - 1;
                stringRedisTemplate.opsForValue().set("applyNumber", newApplyNumber + "");
                System.out.println("报名成功，剩余报名容量: " + newApplyNumber);
            } else {
                System.out.println("报名失败，剩余报名容量不足");
                return "error";
            }
        } finally {
            // 判断lock_key的值是否属于该进程，是则释放，否则什么也不做
            if (lockId.equals(stringRedisTemplate.opsForValue().get("lock_key"))) {
                stringRedisTemplate.delete("lock_key");
            }
        }

        return "success";
    }
}
```

代码05在第12行为每个进程生成了一个唯一的字符串，用来唯一标识该进程。第14行执行获取锁操作时，将这个唯一标识作为lock_key的值存入Redis，在第32行释放锁之前，判断Redis中lock_key的值与该进程的唯一标识是否相同，如果是则释放锁，如果不是则什么也不做，这样就避免了进程释放不属于自己的锁的问题。

细心的小伙伴可能发现了，即便是代码05也没有从根源上解决问题，因为第32行的判断代码和第33行的释放锁代码并不是原子操作。假设A请求在锁过期前执行完第32行代码，进入if语句块准备执行第33行代码时锁过期了，此时B请求已经获得锁，A请求执行释放锁，那么还是会出现一个进程释放另一个进程锁的情况。

相信大家看到这里已经累了，感兴趣的小伙伴可以自行了解一下锁续命机制来解决这个问题，这里就不再展开啦~

虽然代码05还有很多小瑕疵，还有很多值得优化的地方，但是做到这个程度，已经足够应对一般的高并发场景了，我们的优化也告一段落(~~除非学校加钱~~)。我敢说就以代码05的程度，就算全校学生同时报名只有10个名额的活动，也不会出现超报名问题！(~~出问题了当我没说~~) 

债见~

## 参考资料

> [Redis官网-简介](https://www.redis.com.cn/redis-intro.html)
>
> [维基百科-Redis](https://zh.wikipedia.org/wiki/Redis)
>
> [菜鸟教程-Redis](https://www.runoob.com/redis/redis-tutorial.html)
>
> [Redis是什么？看这一篇就够了](https://www.cnblogs.com/powertoolsteam/p/redis.html)
>
> [使用Redis解决秒杀过程中超卖问题](https://blog.csdn.net/cheng__yu/article/details/122620213)

## 推荐阅读

>[synchronized篇](https://ccqstark.github.io/p/concurrent_synchronized/)
>
>[Redis：Redisson分布式锁的使用](https://blog.csdn.net/chuanchengdabing/article/details/121210426)



made by Yansp