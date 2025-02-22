<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250221%E4%B8%8B%E5%8D%8853813243.png" alt="image-20250221下午53813243" style="zoom:50%;" />

# 作业帮-Golang后端开发实习-一面-50min

## 自我介绍 & 实习

1. 自我介绍
2. 如果发offer，多久能到岗，能实习多久（能转正）
3. 介绍一下美团的实习
   1. 难点是什么？
   2. 场景：如果想要扫描检测一个ip地址，该怎么实现
   3. LLM与静态代码扫描有没有落地，说一下你刚刚说的两种实现方案（思维链 & ReAct）        

##  场景 & 八股

1. 有A、B两个分支，如何将B分支的一个commit合并到A分支

   > 答案：
   >
   > ```bash
   > git cherry-pick <commit_hash>
   > ```

2. 介绍一下GMP模型？怎么手动设置P的数量？

3. 联合索引场景题，下面几个sql在执行上有什么区别

   ```mysql
   #联合索引 （A， B）
   select * from xx where A = ? and B = ?;
   select * from xx where B = ? and A = ?;
   select * from xx where A > ? and B > ?;
   select * from xx where A > ? and B = ?;
   select * from xx where B > ? and A = ?;
   ```

# 多线程 & 算法

1. 两个协程交叉打印，从1-100 （A从1开始，B从2开始， 用Go写）
   - 解释一下代码如何运行（WaitGroup和channel怎么配合使用）
   - 说一下channel的底层实现
   - 多个接收者等待channel的消息，当channel有一个消息时，是通知所有协程还是如何？
2. [143. 重排链表](https://leetcode.cn/problems/reorder-list/)

## 反问

1. 部门负责的主要业务
2. 部门对实习生的培养方案，对实习生的要求是什么
3. 面试管问我，如果有多个offer，我会如何考虑去选哪一个