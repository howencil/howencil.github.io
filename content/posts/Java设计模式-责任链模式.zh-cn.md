---
title: "Java设计模式 责任链模式"
date: 2022-06-13T11:44:49+08:00
draft: false
author: 插画师
tags: ["Java","设计模式"]
categories: ["Tech"]
---
# 1.责任链模式的定义和特点
责任链模式的定义： **为了避免发送者与多个请求处理者耦合在一起，于是将所有请求的处理者通过前一对象记住其下一个对象的引用而连成一条链；当有请求发生时，可将请求沿着这条链传递，直到有对象处理它为止。**

在责任链模式中，客户只需要将请求发送到责任链上即可，无须关心请求的处理细节和请求的传递过程，请求会自动进行传递。所以责任链将请求的发送者和处理者解耦了。
#### 优点
1. **降低对象之间的耦合度。** 该模式使得一个对象无须知道到底是哪一个对象处理其请求以及链的结构，发送者和接收者也无须拥有对方的明确信息。
2. **增强了系统的可扩展性。** 可以根据需要增加新的请求处理类，满足开闭原则。
3. **增强了给对象指派职责的灵活性。** 当工作流程发生变化，可以动态地改变链内的成员或者调动它们的次序，也可动态地新增或者删除责任。
4. **责任链简化了对象之间的连接。** 每个对象只需保持一个指向其后继者的引用，不需保持其他所有处理者的引用，这避免了使用众多的 if 或者 if···else 语句。

5. **责任分担。** 每个类只需要处理自己该处理的工作，不该处理的传递给下一个对象完成，明确各类的责任范围，符合类的单一职责原则。
#### 缺点
1. **不能保证每个请求一定被处理。** 由于一个请求没有明确的接收者，所以不能保证它一定会被处理，该请求可能一直传到链的末端都得不到处理。
2. **对比较长的职责链，请求的处理可能涉及多个对象，系统性能将受到一定影响。**
3. **职责链建立的合理性需要客户端来保证，增加了客户端的复杂性，可能会由于职责链的错误设置而导致系统出错，如可能会造成循环调用。**

# 2.责任链模式的结构与实现
### 1.责任链模式的结构
1. **抽象处理者（Handler）角色：** 定义一个处理请求的接口，包含抽象处理方法和一个后继连接。
2. **具体处理者（Controller Handler）角色：** 实现抽象处理者的处理方法，判断能否处理本次请求，如果可以处理请求则处理，否则将该请求转给它的后继者。
3. **客户端（Client）角色：** 创建处理链，并向链头的具体处理者对象提交请求，它不关心处理细节和请求的传递过程。

### 2.代码实现
开发一个请假流程控制系统。请假一天以下的假只需要小组长同意即可；请假1天到3天的假还需要部门经理同意；请求3天到7天还需要总经理同意才行。

**LeaveRequest**
```Java
package com.zhuang.responsibility;

/**
 * @Classname LeaveRequest
 * @Description 请假条
 * @Date 2021/3/31 16:21
 * @Created by dell
 */

public class LeaveRequest {
    //姓名
    private String name;
    // 请假天数
    private int num;
    // 请假内容
    private String content;

    public LeaveRequest(String name, int num, String content) {
        this.name = name;
        this.num = num;
        this.content = content;
    }

    public String getName() {
        return name;
    }

    public int getNum() {
        return num;
    }

    public String getContent() {
        return content;
    }

}
```
**Handler**
```Java
package com.zhuang.responsibility;

/**
 * @Classname Handler
 * @Description 用一句话描述类的作用
 * @Date 2021/3/31 16:23
 * @Created by dell
 */

public abstract class Handler {
    protected final static int NUM_ONE = 1;
    protected final static int NUM_THREE = 3;
    protected final static int NUM_SEVEN = 7;

    //该领导处理的请假天数区间
    private int numStart;
    private int numEnd;


    //领导上还有领导
    private Handler nextHandler;

    //设置请假天数范围
    public Handler(int numStart) {
        this.numStart = numStart;
    }

    //设置请假天数范围
    public Handler(int numStart, int numEnd) {
        this.numStart = numStart;
        this.numEnd = numEnd;
    }

    //设置上级领导
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }

    //提交请假条
    public final void submit(LeaveRequest leaveRequest) {
        if (this.numStart == 0) {
            return;
        }
        //请假天数达到领导处理要求
        if (leaveRequest.getNum() >= this.numStart) {
            this.handleLeave(leaveRequest);

            //如果还有上级 并且请假天数超过当前领导的处理范围
            if (this.nextHandler != null && leaveRequest.getNum() > numEnd) {
                //继续提交
                this.nextHandler.submit(leaveRequest);
            } else {
                System.out.println("流程结束！！！");
            }
        }
    }

    //各级领导处理请假条方法
    protected abstract void handleLeave(LeaveRequest leave);

}
```
**GroupLeader**
```Java
package com.zhuang.responsibility;

/**
 * @Classname GroupLeader
 * @Description 小组长类
 * @Date 2021/3/31 16:33
 * @Created by dell
 */

public class GroupLeader extends Handler {
    //1-3天的假
    public GroupLeader() {
        super(Handler.NUM_ONE, Handler.NUM_THREE);
    }

    @Override
    protected void handleLeave(LeaveRequest leave) {
        System.out.println(leave.getName() + "请假" + leave.getNum() + "天，" + leave.getContent() + "!");
        System.out.println("小组长审批通过：同意！");
    }
}
```
**Manager**
```Java
package com.zhuang.responsibility;

/**
 * @Classname Manager
 * @Description 部门经理类
 * @Date 2021/3/31 16:36
 * @Created by dell
 */

public class Manager extends Handler {
    //3-7天的假
    public Manager() {
        super(Handler.NUM_THREE, Handler.NUM_SEVEN);
    }

    @Override
    protected void handleLeave(LeaveRequest leave) {
        System.out.println(leave.getName() + "请假" + leave.getNum() + "天，" + leave.getContent() + "!");
        System.out.println("部门经理审批通过：同意！");
    }
}
```
**GeneralManager**
```Java
package com.zhuang.responsibility;

/**
 * @Classname GeneralManager
 * @Description 总经理类
 * @Date 2021/3/31 16:38
 * @Created by dell
 */

public class GeneralManager extends Handler{
    //7天以上的假
    public GeneralManager() {
        super(Handler.NUM_THREE, Handler.NUM_SEVEN);
    }

    @Override
    protected void handleLeave(LeaveRequest leave) {
        System.out.println(leave.getName() + "请假" + leave.getNum() + "天，" + leave.getContent() + "!");
        System.out.println("总经理审批通过：同意！");
    }
}
```
**Client**
```Java
package com.zhuang.responsibility;

/**
 * @Classname Client
 * @Description 责任链模式 测试类
 * @Date 2021/3/31 16:39
 * @Created by dell
 */

public class Client {
    public static void main(String[] args) {
        //请假条
        LeaveRequest leave = new LeaveRequest("小庄", 3, "出去旅游");

        //各位领导
        Manager manager = new Manager();
        GroupLeader groupLeader = new GroupLeader();
        GeneralManager generalManager = new GeneralManager();

        /*
        小组长上司是经理 经理上司是总经理
         */
        groupLeader.setNextHandler(manager);
        manager.setNextHandler(generalManager);

        //提交
        groupLeader.submit(leave);

    }
}
```

### 3.责任链模式的应用场景
1. 多个对象可以处理一个请求，但具体由哪个对象处理该请求在运行时自动确定。
2. 可动态指定一组对象处理请求，或添加新的处理者。
3. 需要在不明确指定请求处理者的情况下，向多个处理者中的一个提交请求。