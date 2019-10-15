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

