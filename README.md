# <p align="center">PDSProject1Report</p>

**<p align="center">Masen Wen</p>**
**<p align="center">2025-10-18</p>**


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



## 要求1 的思路整理过程


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

* **并发优势验证** 
  * 这依赖于足够大的数据量 以及足够大量的读写（我想update操作会体现出端倪）
* **缓存优势验证** 
  * 我会在进行实验之前对数据库进行"暖机"操作 即多次调用某一数据保证其被缓存（重连数据库清楚缓存可以体现其优化）
* **查询优化器验证**
  *  我们编写一些逻辑较差的查询来体现查询优化器的作用（课上讲过的两个例子）
* **索引机制研究**
  * PSQL和OpenGauss实现的索引机制


**整理思路过程中，我借助了AI查询知识点。例如我问AI：数据库的五大性能优势是什么（从大到小排列）？**
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/bc543f15-e937-485b-b52f-aba4e31918d2.png)
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/a0fd939d-769e-496a-868e-f143d90887c1.png)
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/054abeb9-5ad8-4af0-8623-efe167b1d051.png)


## 要求2 的思路整理过程

* **OpenGauss的优势是什么呢？要体现这种细微优势是有难度的**
  * 大型数据集
  * 超大量的并发
  * 复杂逻辑的查询
  * 索引机制的特化

**所以，我借助AI查询知识点。例如：**
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/f9dd8d31-463a-4bb4-954d-066c0ae79e50.png)
>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/58d2f423-f962-439e-9922-6ab3e271c7dd.png)


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

>![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/bf1ced30-a3d8-415e-be21-90bb299442ed.png)


## 实验程序设计：测试功能接口

* **DataManipulation接口设计**
  * 数据查询功能
    * 例如String findMovieByTitleLike(String like)
    * @return的字符串是方法名称标记和耗时(nm)
  * 数据修改功能
    * 例如String updateDistance_mt_d(int d, int group)
    * @return的字符串是方法名称标记和耗时(nm)
  * 缓存干扰功能
    * void bustCache()模拟大量访问不同Table的操作“冲掉”上个实验的缓存数据


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
    //
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

## 实验程序设计：计时功能

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

## 实验程序设计：DataManipulation接口的实现类

* **DataManipulation接口的实现类设计**
  * 对DB进行SELECT、UPDATE或INSERT等测试
  * 调用计时包装类MillisecondTimer
  * 返回结果中包含耗时值

```Java
@Override
    public String findMovieByTitleStrict(String title) {
        getConnection();
        MillisecondTimer timer = new MillisecondTimer();
        timer.start();
        String result = "";
        String sql = "select * from movies where title = " + "?";
        try {
            PreparedStatement preparedStatement = con.prepareStatement(sql);
            preparedStatement.setString(1, title);
            result = "Runtime of this [Database] findMovieByTitleStrict is " + timer.stop() + "ms";
            resultSet = preparedStatement.executeQuery();
            // 余下的打印极其耗时 而且不应该记在DBMS的时间内 因为数据已经打包传出了数据库
            while (resultSet.next()) {
                String str = "";
                str += resultSet.getString("movieid");
                str += ";";
                str += resultSet.getString("title");
                str += ";";
                str += resultSet.getString("country");
                str += ";";
                str += resultSet.getString("year_released");
                str += ";";
                str += resultSet.getString("runtime");
                str += ";";
                //System.out.println(str);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            closeConnection();
        }
        return result;
    }
```

# 基础实验实现和防缓存设计

## **实验1** SELECT * FROM movies WHERE title = 'Star Wars'; 和 SELECT * FROM movies WHERE title LIKE 'Star%';
## **实验方法**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/e6d521b9-5c5b-4446-b438-4731b9df871f.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/0fdc8c53-0609-4f77-a0e1-a77d18eca9b8.png)
## **实验结果**
## 1-1
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/f217db01-2f93-4a4f-a9fc-6de23f86b7d6.png)
## 1-2
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/9cc884ef-1470-4b98-82a1-53abfd2188b7.png)
## **分析**
### 在我最初的设想里，DBMS系统最大的性能优势应该是源于并发，而缓存(Cache)只是次之。但一些在movies和people上的小实验（这两个表都只有10000行左右的数据）和重复调试代码的过程让我注意到了**随着重复次数的增加**同一查询的执行时间明显依次递减。以找到"Star Wars"这一部电影的查询为例，缓存（重复执行三次）提升了效率**100多倍**，显然DBMS的缓存模块“记住”了这部电影。 
### 相应的，在找到"Star Wars"系列电影时，“记忆”就不那么强烈。
### 现实意义: 这样的例子提醒我们要善于利用Cache机制设计和运用数据库调用
### 该进: 我们也要意识到缓存在重复实验中的毁灭性影响 这是我问什么尝试实现了一个bustCache()函数

## **bustCache()函数**

### 实验往往需要多次取平均，但多次重复操作会让缓存高度“注意”其相关的行和列，一个立刻会有的想法是分散这种“注意力”。
### 我实现了一个bustCache()函数，轮番(做个几十次)对几个不在实验中使用的数据库进行查询，这里具体指，以下是这个函数实现的代码:


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

### 效果是有的 但并不明显
### 这里的8%与上图的18%对应
### 对于数据库的复杂性来说这并不足以说明太多（即使这可以复现）
### 这是因为数据库有很大的缓存池 (Cache Pool)
## **Cache Pool**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/459aa05e-5838-40fd-a409-b5fe10aa3433.png)
## 设定缓存池的原则
### DataGrip中的缓存池
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/e8fe23fb-5879-4bb7-8fab-d5c60bd6f558.png)
## 数据库缓存池的争抢和Cache()函数的实际使用情况
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b6227a87-6a65-4a90-b18e-c55805005640.png)
### 在现实使用中我们往往调用几个表的组合（如movies, people, credits等的组合） 而一个项目里也会有不相关的多个组合同时运作 争抢的程度取决于工程的大小
### Cache函数倒是可以不错的模拟数据库争抢缓存的一般情况
## **Cache()的实际效果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/f2d8ae04-dc94-45d8-a458-8870a26240a1.png)

## 更加稳妥的方案:结合以下四个操作
### 1.断开数据库清除数据库缓存或者执行等价的DataGrip操作


```Java
sudo systemctl restart postgresql
```

![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/777cdac4-b6cc-4516-8d04-2236d3d710a7.png)

### 2.重启 尤其对于JavaIO 由于Windows不支持缓存清除 必须重启

### 3.使用 bustCache() 添加使用场景下的噪声
### 4.设定数据库的缓存池大小（可能不安全）

## **实验2**  UPDATE people SET first_name = replace(first_name,'TTOO','to') WHERE first_name LIKE '%TTOO%';

## **实验方法**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/4833ea4f-87cf-467f-8da4-9149159ef61d.png)
## **实验结果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/a5e8e85b-ef10-450f-9b3c-aff68fa89ba6.png)

## **分析**
### 相较于先前实验，比Java单线程实现提速更多
### 并发操作大大减少了IO等待浪费的时间
### 并发的效果在数据量越大时越明显
### 现实意义: 要有意识的使用多线程读写 这对cpu也没有过多的要求
### 改进: 可能需要更大的数据集 仅仅几毫秒的运行时间 即使有成倍的关系也不足以令人信服

## **实验3** 对比等价逻辑的不同查询
### 这两个查询逻辑被认为在执行时是会等价的
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/0dc9d443-f2e0-4717-8cf6-3a4bcfde0b63.png)
### 让我们来尝试验证一下
### 鉴于后者的逻辑被认为”本应该“更好，命名为GoodLogic，另一个是BadLogic

![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/c282a1a2-cdac-451a-91a1-52d64987b6a5.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/bd4e018e-0672-4f77-b35b-fabe04d7e4ea.png)

### 总的来看是差不多的（毕竟他们调用的实际算法是一样的）
## **EXPLAIN [Execution Plan][Query Plan]**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/99a52ed8-0a71-4f87-9aa8-8e2f85c3f958.png)
## **EXPLAIN ANALYSIS [Execution Plan][Query Plan]**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/c7a920df-54d2-4726-8274-2b56a4973a59.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/8c833a4a-3a0c-4daf-bdd5-d3b74b28ed01.png)
## 排除缓存影响
### 这里分开操作 操作间断开数据库链接并清理缓存
### 否则由于缓存问题先执行的必定慢很多
## **实验方法**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/1ef718bb-31b8-4960-8703-f451d838aa99.png)
## **实验结果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/e3ccadb8-09e6-4afd-8a82-611f9bd583a1.png)

### 那么如果没有查询优化器呢？会怎么样呢？
### 我按照本来的逻辑写了两段Java代码


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

## **实验方法**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/7d9e360b-c663-45ed-ad89-f8503d678dbc.png)
### 同样也必须分开执行 清理缓存
## **实验结果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b4323772-22c6-4247-8659-eeafeec68686.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/24ac7dbc-cd09-47e6-af38-88def442d42c.png)

## **分析**
### 差距并不大（就单这个查询来看）
### 查询优化器本身需要花费时间
### 这个优化的效果在数据更大时显然会更明显
### 现实意义: 要有意识的审视自己逻辑是否为最佳 有些情况查询优化器可能不会考虑到
### 改进: 可能需要更大的数据集 体现出查询优化器的价值
#

# 实现了更大数据集的实验

#### **后续的实验都是整批完成 实现了缓存清理（每轮重连数据库/重启）**
### 从Kaggle获取数据表8-Million-Lines

![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/c6ee14f3-1d33-4373-a2ff-e9a729ccbda0.png)

### 导入csv
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/95ca68df-4ebd-407a-bdbc-5f453ecca779.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/54788a15-b45d-48fd-8e2f-aef339c23ea9.png)
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/1698896b-0894-42db-8a7c-549d1c34a829.png)

## **\copy COPY 和 pg_dump IMPORT** (在这里走了很多弯路)

## 导入可以使用四种方式
### 直接导入 如上图 要注意先创建有列的表格并一一对应导入 否则会出现失败回滚
### **[bash] \copy** 


```Java
\copy movies FROM '______.csv' WITH CSV HEADER
```

### 具体实现是复杂但可行的


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

### **[SQL] COPY**


```Java
COPY movies FROM '______.csv' WITH CSV HEADER;
```

### **pg_dump**
### 功能和普通Import不同
### 更快更稳定
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/889d1125-c110-4f94-9081-96a1f851f697.png)

### **关于内存和时间的调整**

![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/936fb094-781e-46fd-bcf3-a16508792e1f.png)

### 要导入的数据大小为2GB+
### 默认的750MB填满后就会回滚
### 除了调整内存还需要调整虚拟机供用内存
### 对于运行时间 超时会回滚
### 保证运行时间上限为0（不设限）

## **大批量实验**

## **实验方法**
## 查询实验
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/04aa3f6a-928d-4931-aebe-94c3605d0af0.png)

## **实验结果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/ed832588-ab0e-4084-9195-4d4b810837d7.png)

## **可视化**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b987d5fa-d430-449c-8749-9f327ab242c7.png)

## **分析**
### 对于大型数据集 数据库的优秀表现是可以预见的
### 通过并发的优化（控制变量:缓存被及时清理限制了）越大的数据集越倾向于表现出优越的性能
### 相比之下 JavaIO的表现并不差 较为稳定的每10倍时间增加3-5倍
### **并发处理**

## **实验方法**
## 更新实验
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/689ff905-bff7-45b1-a4ca-b144038bfb7b.png)
## **实验结果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b432fe03-d998-4ae9-899a-86dafbe893d7.png)

## **可视化**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/243fa742-07e6-4d20-b9d1-246435bcf539.png)

## **分析**
### 对于高IO的情况是类似的
## **归因**

![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/8c22da37-f76e-42fe-8c0e-4dc6180c023f.png)

### **分区和索引**

![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/03ef22f9-24f5-4c8f-bdbc-f4f978b6b6e1.png)

### **分区处理的源代码**
#

# OpenGauss和PostgreSQL的大数据集性能对比

## 一些猜测
### 对OpenGauss了解很少
### OpenGauss对格式/输入/操作质量的要求都远远更高
### 例如OpenGauss不允许使用su用户连接DataGrip
### 例如OpenGauss强制要求su用户有高安全度密码
### 例如OpenGauss提供了系统的日志文件和日志输出（驱动会报红）
### 例如OpenGauss报错了psql不报错的.sql文件

![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/bd369a80-bb67-4138-9487-f8d41fb98261.png)

## **实验方法**
## 查询实验
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/5db9eb81-3e5b-49c7-9086-f7a7722c8dd7.png)
## **实验结果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/17e2dfb9-489b-4d83-9e3d-37abd953a7e3.png)
## **可视化**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/161920ed-a7b0-42fd-92eb-19e3d105be6d.png)
## **实验方法**
## 更新实验
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/5b75dc53-8874-4c37-805b-3dbfbae40150.png)
## **实验结果**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/0c89c66f-c278-4e5d-a137-c597d60b145c.png)
## **可视化**
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/e5b6a174-965c-4b62-9ae9-f606ab1a8edd.png)
## **分析**
### 课上曾经提及:尽管OpenGauss优化了psql 但是其表现缺常常不如psql
### 就这一实验来看OpenGauss的表现不如psql
### 可能在**更大**的数据上OpenGauss会更好 但是我不打算尝试**更大**的数据集
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/797a7dbd-e70e-4ddd-bf92-9a12266837d4.png)
### 在8000万行数据附近 psql还在随着数据集变大而更加胜出
### 这意味着所谓**更大**可能比较遥远
![image.png](https://raw.githubusercontent.com/MasenWen/My-Objects/refs/heads/main/PDSProject1Report/b8e8a1b5-ee55-4c14-b7fe-390084147688.png)
### 我对AI的分析持保留意见
### 我倾向于认为OpenGauss的优化不是在这个场景和方向上努力的 否则它无疑是失败的

# End of BassLine
#### 这一部分耗时5天 导入表花费了我一整天 设计实验和编写代码花费了较多的时间 处理缓存影响的几个尝试无疑是重要的（否则实验无法进行）

# Extra Attempts

