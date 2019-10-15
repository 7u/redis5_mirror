# Reset Password 思路

## 数据库

### 记录表

* status：0-成功，非0-失败
* 增加lastpass字段，记录上一次随机密码

## 流程

1. 启动
2. 初始化
   1. 查数据库配置，并缓存，若不是“自动更新密码”则跳到d
   2. 查数据库失败的任务，重新执行（调用执行类，多线程并行），并设置“正在执行”为true
   3. 设置定时任务
   4. 初始化完成
3. 服务
   1. 查询配置：查数据库返回，并更新缓存（Atomic）
   2. 修改配置：若服务正在执行，返回错误；否则根据是否“自动更新密码”选择是否“执行计划”
   3. 密码校验：拿着用户名和密码执行一遍登录看返回
   4. 查看执行记录：查数据库对应状态的记录，直接返回
4. 执行任务的线程：本周期任务结束后调用接口，更新“正在执行”配置项为false，并更新缓存（Atomic）





## 工具类

### UUID

```java
    /**
     * 返回指定长度随机数字+字母(大小写敏感)组成的字符串
     *
     * @param length          指定长度
     * @param caseSensitivity 是否区分大小写
     * @return 随机字符串
     */
    public static String captchaChar(int length, boolean caseSensitivity) {
        StringBuilder sb       = new StringBuilder();
        Random        rand     = new Random();// 随机用以下三个随机生成器
        Random        randdata = new Random();
        int           data     = 0;
        for (int i = 0; i < length; i++) {
            int index = rand.nextInt(caseSensitivity ? 3 : 2);
            // 目的是随机选择生成数字，大小写字母
            switch (index) {
                case 0:
                    data = randdata.nextInt(10);// 仅仅会生成0~9, 0~9的ASCII为48~57
                    sb.append(data);
                    break;
                case 1:
                    data = randdata.nextInt(26) + 97;// 保证只会产生ASCII为97~122(a-z)之间的整数,
                    sb.append((char) data);
                    break;
                case 2: // caseSensitivity为true的时候, 才会有大写字母
                    data = randdata.nextInt(26) + 65;// 保证只会产生ASCII为65~90(A~Z)之间的整数
                    sb.append((char) data);
                    break;
                default:
                    break;
            }
        }
        return sb.toString();
    }
```



## 资料

[Spring+quartz实现动态化定时任务](https://my.oschina.net/u/1760791/blog/887956)



[分布式事务的实现原理](https://draveness.me/distributed-transaction-principle)

[分布式系统的事务处理](https://coolshell.cn/articles/10910.html)

[分布式系统互斥性与幂等性问题的分析与解决](https://tech.meituan.com/2016/09/29/distributed-system-mutually-exclusive-idempotence-cerberus-gtis.html)

[再有人问你分布式事务，把这篇扔给他](https://juejin.im/post/5b5a0bf9f265da0f6523913b)

[聊聊分布式事务，再说说解决方案](https://www.cnblogs.com/savorboard/p/distributed-system-transaction-consistency.html)

[分布式事务？No, 最终一致性](https://zhuanlan.zhihu.com/p/25933039)

[分布式事务的 N 种实现]([http://myfjdthink.com/2019/04/26/%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%E7%9A%84-n-%E7%A7%8D%E5%AE%9E%E7%8E%B0/](http://myfjdthink.com/2019/04/26/分布式事务的-n-种实现/))

[面互联网公司必备的分布式事务方案](http://www.justdojava.com/2019/04/17/java-distributed-transaction/)

[如何选择分布式事务形态（TCC、SAGA、补偿、基于消息的最终一致等等](http://springcloud.cn/view/413)

[浅谈分布式事务](https://ehlxr.me/2019/01/25/distributed-system-transaction/)

[分布式事务-本地消息表：最终一致性](https://quguang.wang/post/transaction-local-msg-tb/)

[基于 KV 的分布式事务方案](http://kaiyuan.me/2019/01/19/2pc/)

[漫画-分布式事务](https://www.itcodemonkey.com/article/2657.html)

[分布式事务-01-概览](https://houbb.github.io/2019/04/05/distributed-tx-01-overview-01)



[蚂蚁金服分布式事务开源以及实践 | SOFA 开源一周年献礼](https://www.sofastack.tech/blog/seata-distributed-transaction-open-source/)

[分布式事务中间件Seata的设计原理](http://objcoding.com/2019/07/11/seata/)

[蚂蚁金服大规模分布式事务实践及四种分布式事务模式](https://linux.cn/article-11164-1.html)

[蚂蚁金服大规模分布式事务实践和 开源介绍 ](https://gw.alipayobjects.com/os/basement_prod/514cacb8-a7b9-4b39-b277-5e12ecb723d3.pdf)

