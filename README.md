# <p align="center">PDS Project1 Report</p>

**<p align="center">Masen Wen</p>**
**<p align="center">2025-10-18</p>**




## Table of Contents
* 实验要求与思路整理
  * [两个实验要求](#两个实验要求)
  * [实验设计思路](#实验设计思路)
* 实验环境搭建与实验程序设计
  * [实验环境搭建](#实验环境搭建)
  * [下载PostgreSQL源代码](#下载PostgreSQL源代码)
  * [实验程序设计-测试功能模块](#实验程序设计-测试功能模块)
  * [实验程序设计-计时功能](#实验程序设计-计时功能)
  * [实验程序设计-绘图功能](#实验程序设计-绘图功能)
* 实验与分析
  * [Experiment01-缓存-重复3遍提速87倍证明Cache价值](#Experiment01-缓存-重复3遍提速87倍证明Cache价值)
  * [Experiment02-缓存-干扰Cache命中率后无法提速证明Cache工作原理](#Experiment02-缓存-干扰Cache命中率后无法提速证明Cache工作原理)
  * [Experiment03-优化器-不同SELECT语句查询计划相同证明DBMS有优化器](#Experiment03-优化器-不同SELECT语句查询计划相同证明DBMS有优化器)
  * [Experiment04-使用工具-导入800万条记录](#Experiment04-使用工具-导入800万条记录)
  * [Experiment05-大数据集-8千到800万条数据的查询、修改性能](#Experiment05-大数据集-8千到800万条数据的查询、修改性能)
  * [Experiment06-并行-Postgresql并行执行](#Experiment06-并行-Postgresql并行执行)
  * [Experiment07-PgSQL与OpenGauss大数据集的读写性能比较](#Experiment07-PgSQL与OpenGauss大数据集的读写性能比较)


---

## 两个实验要求

* **要求1：与文件中的数据操作相比，DBMS 的独特优势是什么？**
  * 使用足够大的数据集（我们直接使用平时用的movies和people数据集）将数据同时存储在PostgreSQL数据库表和一个csv文件中 进行两个子实验
  * 在 SQL 中使用 SELECT 来查找标题中带有单词“Star”的电影（我们使用DatabaseManipulation类的函数调用SQL），在java实现的FileManipulation类中调用select函数做相同的事，都记录执行时间（我们选择分别在DatabaseManipulation类和FileManipulation类中记录，都在java的Client类里中打印并对比），以体现DBMS的优越性。
  * 在 SQL 中使用 UPDATE 将人名中所有的“To”更改为“TTOO”（我们使用DatabaseManipulation类的函数调用SQL），在java实现的FileManipulation类中调用select函数做相同的事，都记录执行时间（我们选择分别在DatabaseManipulation类和FileManipulation类中记录，都在java的Client类里中打印并对比），以体现DBMS的优越性。

* **要求2: 哪个 DBMS 更好？PostgreSQL 还是 OpenGauss，以及采用什么标准？**
  * 由于movies数据集不到10000行可能偏小，我们会选择更大的数据集进行进一步实验
  * 进行更大规模数据的复杂查询（我们使用DatabaseManipulation类的函数调用各数据库），都记录执行时间（我们选择在DatabaseManipulation类），同理打印并对比，我想我们应该要体现特化了的Opengauss的一些优点。
  * 在这个部分分析“数据重组”创建索引的价值
  * 在有条件的情况下 使用尽可能大的数据集

---

## 实验设计思路

**根据我对DBMS优势的理解：**

  |  优势   | 理解  | 
  |  ----  | ----  |
  |  并发的使用  | 利用有限的cpu(gpu)可以大大减少IO的等待时间 而对于联网服务器的很多情形 调取更多的gpu可以全面提升速度  |
  |  缓存的运用  | 根据使用频率（我注意到DataGrip的提示中有上下箭头表示使用频率的标志） 可能数据库在后台类似于类似于@cache的装饰符进行了缓存提速  |
  |  适配的数据结构  | 例如Tree数据结构。<mark>我自己写着玩儿的一个'cmd'工具里 我使用Tree存储目录和文件地址 效果不错。</mark> DBMS肯定做的更好 |
  |  多用户操作和权限管理  | 我安装opengauss后 没有建任何表 就发现用户表已存在（有用户名密码权限等）  |
  |  日志功能  | 例如opengauss的日志比较详细，对我安装过程提供了很大帮助  |
  |  并发中锁的运用  | 加锁会降低并发用户数，但能保护数据一致性  |
  |  查询优化器  | 课上提到过 数据库会对命令进行某种"解释"或"编译" 自动替换为更快执行的形式  |


**我的实验要体现出：**

* **缓存优势验证** 
  * 我会在进行实验之前对数据库进行"暖机"操作 即多次调用某一数据保证其被缓存（重连数据库清楚缓存可以体现其优化）
* **查询优化器验证**
  *  我们编写一些逻辑较差的查询来体现查询优化器的作用（课上讲过的两个例子）
* **索引机制研究**
  * PSQL和OpenGauss实现的索引机制
* **并发优势验证** 
  * 这依赖于足够大的数据量 以及足够大量的读写（我想update操作会体现出端倪）
* **OpenGauss的优势是什么呢？要体现这种细微优势是有难度的**
  * 1. 超大型数据集（？存疑）
  * 2. 超大量Client的并发调取（单机实现不现实）
  * 3. 索引机制的特化
* **我画了下图，在DBMS架构图上标出我的实验目标（缓存、查询优化器、索引、DBMS内并行）**
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/1设计001.png)



---

## 实验环境搭建

* **程序骨架**
  * 使用了Lab3的code and data项目
* **新增开发库**
  * 从华为官网下载了openGauss的驱动.jar包 添加在项目目录下
* **准备DB数据**
  * 配置了PostgreSQL和OpenGauss数据库导入了课上的基础表
  * 从Kaggle上下载了800万行的 纽约出租车数据 并导入数据库
* **准备File数据**
  * 划分为四个文件: Size = 8k，80k, 800k, 8M

![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/1设计002.png)



---

## 下载PostgreSQL源代码

* 下载PostgreSQL源码安装包
* 源码是百万级别的 只应当尝试阅读感兴趣的部分
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/2搭建001.png)
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/2搭建002.png)



---
 
## 实验程序设计-测试功能模块

* **测试功能模块设计**
  * DataManipulation接口
    * 数据查询功能
      * 例如String findMovieByTitleLike(String like)
      * @return的字符串是方法名称标记和耗时(nm)
    * 数据修改功能
      * 例如String updateDistance_mt_d(int d, int group)
      * @return的字符串是方法名称标记和耗时(nm)
    * 缓存干扰功能
      * void bustCache()模拟大量访问不同Table的操作“冲掉”上个实验的缓存数据
  * 接口的两个实现类
    * DatabaseManipulation实现类：采用DB存储
    * FileManipulation实现类：采用File存储

```Java
//
// 我们在这里定义所有要实现的对照组 并且分别用DatabaseManipulation和FileManipulation实例化
// 最终我们在Client类里调用实验
//
public interface DataManipulation {
    //
    // 支持表的创建和基本功能
    //
    public void bustCache();
    public int addOneMovie(String str);
    public String findMovieById(int id);

    //======================================================================================
    // Q1. What are the unique advantages of a DBMS compared with data operations in files?
    //======================================================================================
    //       实验1.大批量SELECT实验 检索带有‘XXX’(以Star Wars系列为例)的字符串的电影名称 [体现并发/缓存的优势]
    //            组1.调用数据库查询(暖机) [并发+缓存]
    //            组2.调用数据库查询(清理缓存)  [缓存]
    //            组3.调用Java代码查询    [什么都没有]
    //       预期: 组1最快 组2略慢 组3慢很多(由数据规模决定)
    public String findMovieByTitleStrict(String title);
    public String findMovieByTitleLike(String like);

    //       实验2.大批量Update实验 将带有‘to’的人名字符串部分替换为'TTOO' [尤其体现并发IO的优势]
    //            组1.调用数据库替换(暖机) [并发+缓存]
    //            组2.调用数据库替换(清理缓存)  [缓存]
    //            组3.调用Java代码替换    [什么都没有]
    //       预期: 组1最快 组2略慢 组3慢很多(比实验1更明显)
    public String updatePeopleNamesTTOO();
    public void refreshTTOO();

    //       实验3.使用课上查询优化器例子中本应较差表现的逻辑查询语句
    //            组1.调用数据库查询(好逻辑)[运用查询优化器]
    //            组2.调用数据库查询(差逻辑)[运用查询优化器]
    //            组3.调用Java代码查询    [执行较好的逻辑]
    //            组4.调用Java代码查询    [执行较差的逻辑]
    //       预期: 组1和2不应该有很大差别 组3慢很多 组4再慢很多
    public String findMovieByConstraintNationAndReleaseYear_usingGoodLogic(String nation, int year1, int year2);
    public String findMovieByConstraintNationAndReleaseYear_usingBadLogic(String nation, int year1, int year2);

    //======================================================================================
    // Q2. Which DBMS is better? PostgreSQL or openGauss, and by which standard?
    //======================================================================================
    //       实验4.大批量SELECT实验 检索带有载有4个人的trip
    //            组1. 8K+
    //            组2. 80K+
    //            组3. 800K+
    //            组4. 8M+
    //       作图: 规模-时间图
    public String findTripOf_n_Passengers(int n, int group);

    //       实验5.大批量Update实验 将trip_distance = d的值设为 -d
    //            组1. 8K+
    //            组2. 80K+
    //            组3. 800K+
    //            组4. 8M+
    //       作图: 规模-时间图
    public String updateDistance_mt_d(int d, int group);
    public String refreshDistance_mt_d(int d, int group);
}
```


---

## 实验程序设计-计时功能

在DataGrip中每条操作记录中有如下时间信息，但不方便对比:
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/15c73837-0363-4e39-908f-00353ed79b03.png)

我选择直接在Java程序中调取SQL指令，并记录时间(同样是寻找"Star Wars"的代码):
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/3fd5c111-7c1c-4dc3-b4b4-2dbc35a15452.png)

下附计时类对应的代码，支持选择用ms或ns计时

```Java
//
// 按毫秒进行计时(后续我换成了纳秒)
//
public class MillisecondTimer {
    private long startTime;
    private long stopTime;
    private boolean isRunning = false;

    public void start() {...}
    public long stop() {...}

    public long getElapsedTimeMillis() {...}
    public long getElapsedTimeNanos() {...}
}
```

---

## 实验程序设计-绘图功能
```Java
import matplotlib.pyplot as plt
import numpy as np

sizes = ['8K', '80K', '800K', '8M']
database_runtime = [...]
file_runtime = [...]
diff_percent = [...]

x = np.arange(len(sizes))
width = 0.35

fig, ax1 = plt.subplots(figsize=(9,5))
bars1 = ax1.bar(x - width/2, database_runtime, width, label='Database', color='#4B8BBE')
bars2 = ax1.bar(x + width/2, file_runtime, width, label='File', color='#FFD43B')

for bars in [bars1, bars2]:
    for bar in bars:
        height = bar.get_height()
        ax1.text(bar.get_x() + bar.get_width()/2, height * 1.05,
                 f'{height/1e6:.2f}M', ha='center', va='bottom', fontsize=9)

ax1.set_xlabel('Data Size')
ax1.set_ylabel('Runtime (ns)')
ax1.set_title('Runtime Comparison: Database vs File')
ax1.set_xticks(x)
ax1.set_xticklabels(sizes)
ax1.set_yscale('log')
ax1.legend()
plt.tight_layout()
plt.show()
```

---

## Experiment01-缓存-重复3遍提速87倍证明Cache价值

* **SELECT * FROM movies WHERE title = 'Star Wars'的实验程序**
  * 连续查询3遍
  * 比较第一次与第三次的查询速度
* **基于File模拟SELECT title = 'Star Wars'的实验程序**
  * 连续查询3遍
  * 比较第一次与第三次的查询速度
* **基于File、DB实验的交叉对比**
  * 第1遍：File与DB查询速度比较
  * 第3遍：File与DB查询速度比较
 ![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/e6d521b9-5c5b-4446-b438-4731b9df871f.png)


* **SELECT * FROM movies WHERE title LIKE 'Star%'的实验程序**
  * 连续查询3遍
  * 比较第一次与第三次的查询速度
* **基于File模拟SELECT title LIKE 'Star%'的实验程序**
  * 连续查询3遍
  * 比较第一次与第三次的查询速度
* **基于File、DB实验的交叉对比**
  * 第1遍：File与DB查询速度比较
  * 第3遍：File与DB查询速度比较
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/0fdc8c53-0609-4f77-a0e1-a77d18eca9b8.png)

* **实验结果：title = 'Star Wars'查询**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/f217db01-2f93-4a4f-a9fc-6de23f86b7d6.png)

* **实验结果：LIKE 'Star%'查询**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/9cc884ef-1470-4b98-82a1-53abfd2188b7.png)


* **实验结果分析**
  * DB查询
    * 第1遍9961900ns
    * 第3遍 113800ns。提速87倍，证明了DB Cache的价值
* **现实意义**
  * 这样的例子提醒我们要善于利用Cache机制设计和运用数据库调用
  * 在我最初的设想里，DBMS系统最大的性能优势应该是源于并发，而缓存(Cache)只是次之。但一些在movies和people上的小实验（这两个表都只有10000行左右的数据）和重复调试代码的过程让我注意到了**随着重复次数的增加**同一查询的执行时间明显依次递减。以找到"Star Wars"这一部电影的查询为例，缓存（重复执行三次）提升了效率**87倍**，显然DBMS的缓存模块“记住”了这部电影。 
  * 相应的，在找到"Star Wars"系列电影时，“记忆”就不那么强烈。
* 后续实验，我要尝试实现了一个bustCache()函数，干扰Cache效率


---

## Experiment02-缓存-干扰Cache命中率后无法提速证明Cache工作原理

* **step1-编写bustCache()函数**
  * 设计思想是：多个查询分别查询不同table，这样才能使Cache数据快速失效，分散Cache“注意力”
  * 模拟多次重复操作，代码中for (int i = 0; i < 30; i++)

```Java
@Override
    public void bustCache() {
        getConnection();
        try {
            String[] queries = {
                    "SELECT * FROM bus_lines WHERE station_id IN (1,2,7,8,12,17,18,28,38,61)",
                    "SELECT * FROM color_names WHERE name IN ('AliceBlue','AntiqueWhite','Aqua','Aquamarine')",
                    "SELECT * FROM station WHERE district IN ('Luohu','Futian')",
                    "SELECT * FROM station WHERE latitude BETWEEN 22.53 AND 22.55"};
            for (int i = 0; i < 30; i++) {
                for (String query : queries) {
                    try (Statement stmt = con.createStatement();
                         ResultSet rs = stmt.executeQuery(query)) {
                        while (rs.next()) {
                            // 读取所有列
                            int cols = rs.getMetaData().getColumnCount();
                            for (int j = 1; j <= cols; j++) {
                                rs.getObject(j);
                            }
                        }
                    }
                }
            }
        } catch (SQLException e) {
            //
        } finally {
            closeConnection();
        }
    }
```

* **step2-调整DBMS配置，把Cache值改得很小，确保bustCache()干扰缓存效果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/e8fe23fb-5879-4bb7-8fab-d5c60bd6f558.png)


* **step3-再次运行title = 'Star Wars查询实验程序**
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/3缓存实验001.png)

* **实验结果分析**
  * 效果是有的 但并不明显
  * 这里的8%与上图的18%对应
  * 对于数据库的复杂性来说这并不足以说明太多（即使这可以复现）
  * 这是因为数据库有很大的缓存池 (Cache Pool)
  * Cache基本原理是空间换时间。就是内存占用多了、响应时间短了。
  * Cache大小的设置影响性能，太大浪费内存、太小影响Cache命中率。
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b6227a87-6a65-4a90-b18e-c55805005640.png)


* **看PostgreSQL网课 学习缓存机制原理**
  * 这部分内容来自: PostgreSQL源码分析 系列文章十余篇 CSDN
  * 内容对学生而言较难 选取了少量内容
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/3缓存实验002.png)

* **看PostgreSQL源码 cache部分**
  * 看源码[...\cache]
  * 以下出现的源码在文件目录："src/backend/util/cache"
  * SysCache用于缓存系统表元组
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/3缓存实验-代码阅读001.png)


* **看PostgreSQL源码 cache部分**
  * RelCache用于表模式信息缓存
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/3缓存实验-代码阅读002.png)


* **看PostgreSQL源码 cache部分**
  * 源码中进行查询的部分
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/3缓存实验-代码阅读003.png)


* **看PostgreSQL源码 cache部分**
  * 缓冲的争抢
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/3缓存实验-代码阅读004.png)


* **现在我们可以全面的解释:**
  * 为什么缓存大大加速了查询？
    * 因为缓存豁免了IO时间
    * 通过实现了系统表元组和表模式信息缓存的哈希表/桶快速映射 本质是一种索引
  * 为什么尤其在中小规模数据中有用？
    * 因为缓存豁免的时间是受空间限制的
    * 如果缓存空间（如缓存池）被占用殆尽 数据库会通过释放较老的缓存来腾出空间
    
* **更加稳妥的方案:以下四个操作**
  * 在现实使用中我们往往调用几个表的组合（如movies, people, credits等的组合） 而一个项目里也会有不相关的多个组合同时运作 争抢的程度取决于工程的大小
  * bustCache() 函数倒是可以不错的模拟数据库争抢缓存的一般情况
    * 1.断开数据库清除数据库缓存或者执行等价的DataGrip操作
    * 2.重启 尤其对于JavaIO 由于Windows不支持缓存清除 必须重启
    * 3.使用 bustCache() 添加使用场景下的噪声
    * 4.设定数据库的缓存池大小（可能不安全）
```Java
sudo systemctl restart postgresql
```
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/777cdac4-b6cc-4516-8d04-2236d3d710a7.png)




---

## Experiment03-优化器-不同SELECT语句查询计划相同证明DBMS有优化器

* **下列两个查询：**
  * 第二个语句的逻辑被认为”本应该“更好，命名为GoodLogic
  * 另一个命名为BadLogic
  * 逻辑上，被认为在执行时是会等价的
  ![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/0dc9d443-f2e0-4717-8cf6-3a4bcfde0b63.png)



* **查看EXPLAIN的查询计划**
  * 两个语句执行计划一样
  * 证明DBMS有查询优化器

EXPLAIN [Execution Plan][Query Plan]
  >![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/99a52ed8-0a71-4f87-9aa8-8e2f85c3f958.png)

EXPLAIN ANALYSIS [Execution Plan][Query Plan]
  >![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/c7a920df-54d2-4726-8274-2b56a4973a59.png)


* **查询优化器 原理学习**
  * 找文章来读
    * 来自文章: Postgresql中的explain https://www.cnblogs.com/flying-tiger/p/8039055.html
  * 归纳总结
    * 数据库可以做一些基元操作{opertaion1..operationn)
    * 这些操作具有相对稳定的时间{runtime....runtimen}
    * 一个实际操作可以由一系列基元操作复合而成
    * 复合方式不止一种 有的逻辑效率很高（如利用索引直接找到） 有的则有限（遍历）
  * 设计思想
    * 用算法找到(被认为)最高效的查询逻辑 执行这个逻辑
    * 找到最优的操作组合的搜索时间 << 执行时间
    * 最优的操作组合包括：用索引、用缓存、并行等
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/4查询优化器003.png)
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/4查询优化器004.png)


* **查询优化器 代码阅读**
  * 代码目录: src\backend\optimizer\plan
    * 参考文章: 跟我一起读postgresql源码(五)——Planer(查询规划模块)
    * https://www.cnblogs.com/flying-tiger/p/6063709.html
    * 下图信息来源于这一文章和实际实验
  * 根据源码逻辑 优化主要在两个环节里进行
    * 1.预处理: 通过 向上提取(pull_up_) 减少逻辑中的浪费
    * 2.动态规划（或遗传算法）找到最优路径（代价最低和排序最优）
  * 最终返回优化了的 查询计划树
    * EXPLAIN在这个环节调取 查询计划树 （EXPLAIN并不真正执行）
    * 释放中间的过程树结构空间
    * 然后这个结果可以执行
    * EXPLAIN ANALYSIS用这个 查询计划实际执行 并返回时间
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/4查询优化器-代码阅读001.png)


* **对照实验：用File模拟**
  * 那么如果没有查询优化器呢？会怎么样呢？
  * 我按照本来的逻辑写了两段Java代码
```Java
@Override
public String findMovieByConstraintNationAndReleaseYear_usingGoodLogic(String nation, int year1, int year2) {
   ...
   while ((line = reader.readLine()) != null) {
      String[] movie = line.split(";");
      if (movie[2].equals(nation) && (Integer.parseInt(movie[3]) >= year1 && Integer.parseInt(movie[3]) <= year2)) {
          //System.out.println(line);
      }
   }
   ...
 }

 @Override
 public String findMovieByConstraintNationAndReleaseYear_usingBadLogic(String nation, int year1, int year2) {
     ...
     List<String> selectedContent = new ArrayList<>();
     while ((line = reader.readLine()) != null) {
         String[] movie = line.split(";");
         if (movie[2].equals(nation)) {
             selectedContent.add(line);
         }
     }
     for (String contentLine : selectedContent) {
         String[] movie = contentLine.split(";");
         if ((Integer.parseInt(movie[3]) >= year1 && Integer.parseInt(movie[3]) <= year2)) {
             //System.out.println(contentLine);
         }
     }
     ...
 }
```

* **实验与结果分析**
  * 差距并不大（就单这个查询来看）
  * 查询优化器本身需要花费时间
  * 这个优化的效果在数据更大时显然会更明显
  * 现实意义: 要有意识的审视自己逻辑是否为最佳 有些情况查询优化器可能不会考虑到
  * 改进: 可能需要更大的数据集 体现出查询优化器的价值
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/7d9e360b-c663-45ed-ad89-f8503d678dbc.png)
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b4323772-22c6-4247-8659-eeafeec68686.png)
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/24ac7dbc-cd09-47e6-af38-88def442d42c.png)





---

## Experiment04-使用工具-导入800万条记录

* **Step1：从Kaggle获取数据表8-Million-Lines的csv数据文件**
 ![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/c6ee14f3-1d33-4373-a2ff-e9a729ccbda0.png)


* **Step2：数据导入前的设置准备--设置内存和时间**
  * 避免导入失败
    * 要导出的数据大小为2GB+，默认的750MB填满后就会回滚。
    * 另外，运行时间超，也会回滚。
  * 所以我调整了：
    * 虚拟机供用内存大小、
    * 和时间上限大小（为0代表不设限）
 ![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/936fb094-781e-46fd-bcf3-a16508792e1f.png)


* **Step3：建如下四个table并载入数据**
  * 表1. 8K+
  * 表2. 80K+
  *	表3. 800K+
  *	表4. 8M+
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/5大批量实验001.png)


* **掌握导入四种方式 踩了不少坑**
  * 直接导入 如上图 要注意先创建有列的表格并一一对应导入 否则会出现失败回滚
  * [bash] \copy
  * [SQL] COPY
  * pg_dump 功能和普通Import不同，更快更稳定

```Java
mm=>
omm=> DROP TABLE IF EXISTS public.taxi_trips;
CREATE TABLE public.taxi_trips (
    vendor_name VARCHAR(10),
    Trip_Pickup_DateTime TIMESTAMP,
    Trip_Dropoff_DateTime TIMESTAMP,
    Passenger_Count INTEGER,
    Trip_Distance DECIMAL(8,2),
    Start_Lon DECIMAL(11,6),
    Start_Lat DECIMAL(11,6),
    Rate_Code INTEGER,
    store_and_forward VARCHAR(10),
    End_Lon DECIMAL(11,6),
    End_Lat DECIMAL(11,6),
    Payment_Type VARCHAR(10),
    Fare_Amt DECIMAL(8,2),
    surcharge DECIMAL(8,2),
    mta_tax DECIMAL(8,2),
    Tip_Amt DECIMAL(8,2),
    Tolls_Amt DECIMAL(8,2),
    Total_Amt DECIMAL(8,2)
);

...
    
omm=> \copy public.taxi_trips FROM '/tmp/yellow_final.csv' WITH (FORMAT csv, HEADER true, NULL '');
omm=> select * from public.taxi_trips where vendor_name = '%DD%';
 vendor_name | trip_pickup_datetime | trip_dropoff_datetime | passenger_count | trip_distance | start_lon | start_lat |
rate_code | store_and_forward | end_lon | end_lat | payment_type | fare_amt | surcharge | mta_tax | tip_amt | tolls_amt
| total_amt
-------------+----------------------+-----------------------+-----------------+---------------+-----------+-----------+-
----------+-------------------+---------+---------+--------------+----------+-----------+---------+---------+-----------
+-----------
(0 rows)
```
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/889d1125-c110-4f94-9081-96a1f851f697.png)




---

## Experiment05-大数据集-8千到800万条数据的查询、修改性能

* **8千到800万条数据的查询性能比较**
  * 运行实验
  * 实验结果
  * 柱状图对比
* **实验结果分析** 
  * 对于大型数据集 数据库的优秀表现是可以预见的
  * 通过[并行or批量处理]的优化（控制变量:缓存被及时清理限制了）越大的数据集越倾向于表现出优越的性能
  * 相比之下 JavaIO的表现并不差 较为稳定的每10倍时间增加3-5倍
  * 产生疑问: 在甄别是哪个导致了优化上产生了疑问
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b987d5fa-d430-449c-8749-9f327ab242c7.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/04aa3f6a-928d-4931-aebe-94c3605d0af0.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/ed832588-ab0e-4084-9195-4d4b810837d7.png)


* **8千到800万条数据的Update性能比较**
  * 运行实验
  * 实验结果
  * 柱状图对比
* **实验结果分析**
  * 对于高IO的情况是类似的
* **归因**
  * 批量操作会一次读写一批数据 减少IO时间
  * 并行操作会解放cpu 减少IO等待的浪费
  * 疑问: 这仍然不能区分两者
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/243fa742-07e6-4d20-b9d1-246435bcf539.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/689ff905-bff7-45b1-a4ca-b144038bfb7b.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b432fe03-d998-4ae9-899a-86dafbe893d7.png)



---

## Experiment06-并行-Postgresql并行执行


* **并行和批量操作**
  * 是截然不同的优化方式
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/5大批量实验003.png)

* **看文章 理解Postgresql并行执行**
  * 文章: Postgresql源码（109）并行框架实例与分析
  * https://cloud.tencent.com/developer/article/2339809
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/6并行实验001.png)

* **看代码 理解Postgresql并行执行**
  * 看源码[...\plan]
  * 并行计划的源代码（查询优化器会决定是否使用并行）
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/6并行实验-代码阅读001.png)

* **学习收获**
  * 检查并行参数 并行在实验中被支持（默认值）
  * 检查具体使用 Quiery Plan告诉我们并行没有被执行
  * 并行适用的场景
    * 使用并行有巨大的一次成本（硬件投入）
    * 大数据量&较复杂操作（多读写）时倾向于被使用
![image.png](https://raw.githubusercontent.com/shanghaiwenyu/goodc/refs/heads/main/6并行实验002.png)



---

## Experiment07-PgSQL与OpenGauss大数据集的读写性能比较


* **就我的感性认识而言，OpenGauss对格式/输入/操作质量的要求都远远更高**
  *	例如OpenGauss不允许使用su用户连接DataGrip
  *	例如OpenGauss强制要求su用户有高安全度密码
  *	例如OpenGauss提供了系统的日志文件和日志输出（驱动会报红）
  *	例如OpenGauss报错了psql不报错的.sql文件
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/bd369a80-bb67-4138-9487-f8d41fb98261.png)


* **进行了4组查询实验**
  * 数据量逐次增大，每轮实验后重连数据库/重启，确保缓存清理。
    * 组1. 8K+
    * 组2. 80K+
    * 组3. 800K+
    * 组4. 8M+
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/5db9eb81-3e5b-49c7-9086-f7a7722c8dd7.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/17e2dfb9-489b-4d83-9e3d-37abd953a7e3.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/161920ed-a7b0-42fd-92eb-19e3d105be6d.png)



* **进行了4组Update实验**
  * 数据量逐次增大，每轮实验后重连数据库/重启，确保缓存清理。
    * 组1. 8K+
    * 组2. 80K+
    * 组3. 800K+
    * 组4. 8M+
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/5b75dc53-8874-4c37-805b-3dbfbae40150.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/0c89c66f-c278-4e5d-a137-c597d60b145c.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/e5b6a174-965c-4b62-9ae9-f606ab1a8edd.png)


* **汇总分析**
  * 课上曾经提及:尽管OpenGauss优化了psql 但是其表现缺常常不如psql
  * 就这一实验来看OpenGauss的表现不如psql
  * 可能在更大的数据上OpenGauss会更好 但是我不打算尝试更大的数据集
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/797a7dbd-e70e-4ddd-bf92-9a12266837d4.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b8e8a1b5-ee55-4c14-b7fe-390084147688.png)



