# SnowFlake
`雪花算法`英文叫`SnowFlake`算法，是`Twitter`开源的分布式id生成算法。其核心思想就是：使用一个64bit的long型的数字作为全局唯一id。在分布式系统中的应用十分广泛，且ID引入了时间戳，基本上保持自增的，后面的代码中有详细的注解。

![avatar](https://pic1.zhimg.com/80/v2-89659f2e11fdbdacd672a26b7be42068_1440w.jpg)

其中分四部分：
1. 第一部分：1位，正负位，在java中由于long的最高位是符号位，一般生成ID都是正数所以一般写死 二进制0；
2. 第二部：41位，时间戳，是当前系统时间减去手动设置的起始时间，一般如果起始时间设置为今天，41位的时间戳可以使用69年，(1L << 41) / (1000L 60 60 24 365) = 69年；
3. 第三部分：10位，是机房机器编号，这里为了区分不同的机器生成的ID防止相同时间生成ID冲突，很好的散列策略，可以部署1024个节点；
4. 第四百分：12位，填充的是一个序列，可以理解为递增数字。支持同一毫秒内同一个节点可以生成4096个ID。

## 1 Java实现算法
```java

public class SnowFlake {

    /**
     * 起始的时间戳
     * 2021-01-30 12:43:05
     */
    private final static long START_STMP = 1611981793853L;

    /**
     * 每一部分占用的位数
     */
    private final static long SEQUENCE_BIT = 12; //序列号占用的位数
    private final static long MACHINE_BIT = 5;   //机器标识占用的位数
    private final static long DATACENTER_BIT = 5;//数据中心占用的位数

    /**
     * 每一部分的最大值
     */
    private final static long MAX_DATACENTER_NUM = -1L ^ (-1L << DATACENTER_BIT);
    private final static long MAX_MACHINE_NUM = -1L ^ (-1L << MACHINE_BIT);
    private final static long MAX_SEQUENCE = -1L ^ (-1L << SEQUENCE_BIT);

    /**
     * 每一部分向左的位移
     */
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    private final static long DATACENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    private final static long TIMESTMP_LEFT = DATACENTER_LEFT + DATACENTER_BIT;

    private long datacenterId;  //数据中心
    private long machineId;     //机器标识
    private long sequence = 0L; //序列号
    private long lastStmp = -1L;//上一次时间戳

    public SnowFlake(long datacenterId, long machineId) {
        if (datacenterId > MAX_DATACENTER_NUM || datacenterId < 0) {
            throw new IllegalArgumentException("datacenterId can't be greater than MAX_DATACENTER_NUM or less than 0");
        }
        if (machineId > MAX_MACHINE_NUM || machineId < 0) {
            throw new IllegalArgumentException("machineId can't be greater than MAX_MACHINE_NUM or less than 0");
        }
        this.datacenterId = datacenterId;
        this.machineId = machineId;
    }

    /**
     * 产生下一个ID
     *
     * @return
     */
    public synchronized long nextId() {
        long currStmp = getNewstmp();
        if (currStmp < lastStmp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }

        if (currStmp == lastStmp) {
            //相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            //同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currStmp = getNextMill();
            }
        } else {
            //不同毫秒内，序列号置为0
            sequence = 0L;
        }

        lastStmp = currStmp;

        return (currStmp - START_STMP) << TIMESTMP_LEFT //时间戳部分
                | datacenterId << DATACENTER_LEFT       //数据中心部分
                | machineId << MACHINE_LEFT             //机器标识部分
                | sequence;                             //序列号部分
    }

    private long getNextMill() {
        long mill = getNewstmp();
        while (mill <= lastStmp) {
            mill = getNewstmp();
        }
        return mill;
    }

    private long getNewstmp() {
        return System.currentTimeMillis();
    }

}

```



## 2 调用生成全局ID

```java
// 其中第一个参数为数据中心ID[机房号]，第二个为机器编号
SnowFlake sf = new SnowFlake(1,2);
long id = sf.nextId();
```

## 3 优缺点
1. 实现简单，性能比Java的UUID生成效率高；
2. 需要对不同机房不同机器做不同的ID编号，在大量机器部署的时候需要想办法解决这个编号问题，如果生产环境主机少，则比较实用。
