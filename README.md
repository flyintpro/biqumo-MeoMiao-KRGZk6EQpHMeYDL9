[合集 \- 花100块做个摸鱼小网站!(10\)](https://github.com)[1\.《花100块做个摸鱼小网站! · 序》灵感来源07\-30](https://github.com/wlovet/p/18333053)[2\.《花100块做个摸鱼小网站! 》第一篇—买云服务器和初始化环境08\-05](https://github.com/wlovet/p/18336522)[3\.《花100块做个摸鱼小网站! 》第二篇—后端应用搭建和完成第一个爬虫08\-12](https://github.com/wlovet/p/18350862)[4\.《花100块做个摸鱼小网站! 》第三篇—热搜表结构设计和热搜数据存储08\-19](https://github.com/wlovet/p/18367044)[5\.《花100块做个摸鱼小网站! 》第四篇—前端应用搭建和完成第一个热搜组件08\-26](https://github.com/wlovet/p/18376100)[6\.《花100块做个摸鱼小网站! 》第五篇—通过xxl\-job定时获取热搜数据09\-02](https://github.com/wlovet/p/18392182)[7\.《花100块做个摸鱼小网站! 》第六篇—将小网站部署到云服务器上09\-09](https://github.com/wlovet/p/18403018)[8\.《花100块做个摸鱼小网站! 》第七篇—谁访问了我们的网站？10\-09](https://github.com/wlovet/p/18454074)[9\.《花100块做个摸鱼小网站! 》第八篇—增加词云组件和搜索组件10\-22](https://github.com/wlovet/p/18492618)10\.《花100块做个摸鱼小网站! 》第九篇—我的小网站被攻击了！11\-22收起
> **目录**
> 
> [一、前言](#%E4%B8%80%E3%80%81%E5%89%8D%E8%A8%80)[二、被攻击的过程](#%E4%BA%8C%E3%80%81%E8%A2%AB%E6%94%BB%E5%87%BB%E7%9A%84%E8%BF%87%E7%A8%8B):[蓝猫机场](https://fenfang.org)[三、小网站的统计规则](#%E4%B8%89%E3%80%81%E5%B0%8F%E7%BD%91%E7%AB%99%E7%9A%84%E7%BB%9F%E8%AE%A1%E8%A7%84%E5%88%99)[四、限流大法好](#%E5%9B%9B%E3%80%81%E9%99%90%E6%B5%81%E5%A4%A7%E6%B3%95%E5%A5%BD)[1、滑动窗口限流](#1%E3%80%81%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E9%99%90%E6%B5%81)[2、原理说明](#2%E3%80%81%E5%8E%9F%E7%90%86%E8%AF%B4%E6%98%8E)[3、代码实现](#3%E3%80%81%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0)[五、小结一下](#%E4%BA%94%E3%80%81%E5%B0%8F%E7%BB%93%E4%B8%80%E4%B8%8B)



> ⭐️基础链接导航⭐️
> 
> 
> 服务器 → [☁️ 阿里云活动地址](https://github.com)
> 
> 
> 看样例 → [🐟 摸鱼小网站地址](https://github.com)
> 
> 
> 学代码 → [💻 源码库地址](https://github.com)


# 一、前言


大家好呀，我是summo，最近不是被裁员了嘛，找工作又难找，老是心烦意乱的，也导致了一个多月断更，不好意思哈。虽然工作没了，但是小网站的故事却一直在发生，比如阿里云RDS到期导致我的小网站崩了一个月（已经修复）；比如我重构了一下设计和代码（代码已上传Gitee）；比如有些“好心人”刷我的接口，给我提高访问量（本篇会讲的故事）... ...


还有一些更有意思的，比如济南有位老板看中了我的小网站希望我可以根据他的一些需求进行二次开发（一期已经做好上线了）；比如通过这个小网站让我赚到1千多的外快（意外之喜）等等。


工作是一时的，兴趣是一辈子的，无论如何，从今天开始，我会继续更新小网站的故事，这一篇主题：我的小网站被攻击了！


# 二、被攻击的过程


在我小网站的右侧有一块访问统计的展示，显示着今日PV，今天UV、总PV和总UV（PV：页面浏览量，访问一次我加一次；UV：独立访客数，一个IP我算一个），如下图：
![](https://img2024.cnblogs.com/blog/1127399/202411/1127399-20241122145915760-1436334970.png)



> 这是2024\-11\-22截的图，总UV才2W多，但是PV已经8W多了，相当于每个用户平均访问了小网站4次，虽然看起来也不算很离谱，但实际上有几个“好心人”单独就贡献了1\.5W次的访问，如下图：
> ![](https://img2024.cnblogs.com/blog/1127399/202411/1127399-20241122150450976-983285836.png)
> 至于他们为什么攻击我，跟我也是有很大关系，我曾经在博客园的闪存“大放厥词”：
> ![](https://img2024.cnblogs.com/blog/1127399/202411/1127399-20241122142933628-939264247.png)


# 三、小网站的统计规则


之前我们在[第七篇—谁访问了我们的网站？](https://github.com)中介绍了小网站的统计逻辑，核心逻辑就是加了一个`@VisitLog`注解，使用切面的方式记录访问的IP地址。然后将这个注解放在了`IndexController.java`的index方法上，如下：



```
@Controller
public class IndexController {

    @GetMapping("/")
    @VisitLog
    public String index(HttpServletRequest request) {
        if (isFromMobile(request)) {
            return "mp/index";
        } else {
            return "web/index";
        }
    }
}

```


> 也就是说，只要你们疯狂调用https://sbmy.fun，就会不断地增加访问记录。正如我前面所说的我并没有使用CDN、OSS这种按按流量计费的组件，无论你们怎么刷都是不会给我造成资损的，但是有一个问题我不得不处理一下：
> 前端的资源来自于我的应用，资源包括js、css、图片、字体等文件，这些资源还都不小，尤其是chunk\-vendors.js，体积达到了1\.2M，如下图：
> ![](https://img2024.cnblogs.com/blog/1127399/202411/1127399-20241122153506046-207371313.png)
> 阿里云的ECS公网IP带宽默认只有3M，也就是说小网站同时有三个人访问就会出现访问效率问题，问题其实也不大，浏览器只要访问一次这些资源就会将它们缓存下来，后续再刷新也不会很慢。但是如果被攻击了呢，那就麻烦了，正常用户想访问都不行了... ... 所以我必须得想个法子了。


# 四、限流大法好


像我们这种小网站，流量也不大，被攻击了好像也没啥意义，有些攻击纯粹就是兄弟们开的玩笑，搞着玩的。但既然兄弟们出招了，我这边也得想个法子解决不是？其实很简单，搞个滑动窗口限流就可以了。



> 我先讲一下什么是限流，王者荣耀大家都玩过吧，里面的英雄都有一个攻击间隔，当我们连续的点击普通攻击的时候，英雄的攻速并不会随着我们点击的越快而更快的攻击。这个就是限流，英雄会按照自身攻速的系数执行攻击，我们点的再快也没用。


## 1、滑动窗口限流


先上一张流程图，帮助大家理解原理
![](https://img2024.cnblogs.com/blog/1127399/202411/1127399-20241122154815469-146239414.png)


## 2、原理说明


![](https://img2024.cnblogs.com/blog/1127399/202401/1127399-20240110190301568-1554061615.png)



> 从图上可以看到时间创建是一种滑动的方式前进， 滑动窗口限流策略能够显著减少临界问题的影响，但并不能完全消除它。滑动窗口通过跟踪和限制在一个连续的时间窗口内的请求来工作。与简单的计数器方法不同，它不是在窗口结束时突然重置计数器，而是根据时间的推移逐渐地移除窗口中的旧请求，添加新的请求。
> 举个例子：假设时间窗口为10s，请求限制为3，第一次请求在10:00:00发起，第二次在10:00:05发起，第三次10:00:11发起，那么计数器策略的下一个窗口开始时间是10:00:11，而滑动窗口是10:00:05。所以这也是滑动窗口为什么可以减少临界问题的影响，但并不能完全消除它的原因。


如果大家想详细了解一些常用的限流算法，可以看我这篇文章：[《优化接口设计的思路》系列：第七篇—接口限流策略](https://github.com)。


## 3、代码实现


#### SlidingWindowRateLimit.java



```
package com.summo.demo.config.limitstrategy.slidingwindow;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface SlidingWindowRateLimit {
    /**
     * 请求的数量
     *
     * @return
     */
    int requests();

    /**
     * 时间窗口，单位为秒
     *
     * @return
     */
    int timeWindow();
}

```

#### SlidingWindowRateLimitAspect.java



```
package com.summo.sbmy.common.limit.slidingwindow;

import com.summo.sbmy.common.util.HttpContextUtil;
import com.summo.sbmy.common.util.IpUtil;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.TimeUnit;

@Slf4j
@Aspect
@Component
@Order(5)
public class SlidingWindowRateLimitAspect {
    /**
     * 使用 ConcurrentHashMap 保存每个方法的请求时间戳队列
     */
    private final ConcurrentHashMap>> METHOD_IP_REQUEST_TIMES_MAP = new ConcurrentHashMap<>();

    @Around("@annotation(com.summo.sbmy.common.limit.slidingwindow.SlidingWindowRateLimit)")
    public Object rateLimit(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        // 获取客户端IP地址
        String clientIp = IpUtil.getIpAddr(HttpContextUtil.getHttpServletRequest());


        SlidingWindowRateLimit rateLimit = method.getAnnotation(SlidingWindowRateLimit.class);
        // 允许的最大请求数
        int requests = rateLimit.requests();
        // 滑动窗口的大小(秒)
        int timeWindow = rateLimit.timeWindow();

        // 获取方法名称字符串
        String methodName = method.toString();
        // 如果不存在当前方法和IP的请求时间戳队列，则初始化一个新的队列
        ConcurrentHashMap> ipRequestTimesMap = METHOD_IP_REQUEST_TIMES_MAP.computeIfAbsent(methodName, k -> new ConcurrentHashMap<>());
        ConcurrentLinkedQueue requestTimes = ipRequestTimesMap.computeIfAbsent(clientIp, k -> new ConcurrentLinkedQueue<>());

        // 当前时间
        long currentTime = System.currentTimeMillis();
        // 计算时间窗口的开始时间戳
        long thresholdTime = currentTime - TimeUnit.SECONDS.toMillis(timeWindow);

        // 这一段代码是滑动窗口限流算法中的关键部分，其功能是移除当前滑动窗口之前的请求时间戳。这样做是为了确保窗口内只保留最近时间段内的请求记录。
        // requestTimes.isEmpty() 是检查队列是否为空的条件。如果队列为空，则意味着没有任何请求记录，不需要进行移除操作。
        // requestTimes.peek() < thresholdTime 是检查队列头部的时间戳是否早于滑动窗口的开始时间。如果是，说明这个时间戳已经不在当前的时间窗口内，应当被移除。
        while (!requestTimes.isEmpty() && requestTimes.peek() < thresholdTime) {
            // 移除队列头部的过期时间戳
            requestTimes.poll();
        }

        // 检查当前时间窗口内的请求次数是否超过限制
        if (requestTimes.size() < requests) {
            // 未超过限制，记录当前请求时间
            requestTimes.add(currentTime);
            return joinPoint.proceed();
        } else {
            // 超过限制，抛出限流异常
            throw new RuntimeException("Too many requests, please try again later.");
        }
    }
}

```

#### 使用方式



```

@Controller
public class IndexController {
    @GetMapping("/")
    @VisitLog
    @SlidingWindowRateLimit(requests = 2, timeWindow = 2)
    public String index(HttpServletRequest request) {
        if (isFromMobile(request)) {
            return "mp/index";
        } else {
            return "web/index";
        }
    }
}

```

# 五、小结一下


上面的限流使用的是ConcurrentHashMap来存储每个方法的请求时间戳队列，适用于单机，如果是分布式的环境则可以换成Redis。通过在资源接口上加一个限流的方式我们可以防止单个IP刷爆我们的index接口，防止带宽打满，我试了下应该是用的，就是不知道实战如何。


作者：[不若为止](https://github.com)欢迎任何形式的转载，但请务必注明出处。限于本人水平，如果文章和代码有表述不当之处，还请不吝赐教。

