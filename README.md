# discrete-math-lab-fdzc

 Labs of discrete mathematics from Fuzhou University Zhicheng College
## 实验思路
### 1. 问题建模

    二元关系表示
    将用户之间的关注关系看作一个二元关系 R ⊆ U × U，其中 U 是用户集合。如果用户 A 关注用户 B，则记为 (A, B) ∈ R。

### 2. 利用二元关系的知识

    对称性
    在实际社交网络中关注关系通常是单向的，但为了生成社交圈（即等价类），我们需要将二元关系取对称闭包。
    对称闭包定义为：对于所有 (A, B) ∈ R，如果 (B, A) 不在 R 中，则将 (B, A) 加入，从而得到对称关系 R'。

    传递性
    传递性是判断两个用户是否属于同一个社交圈的关键：如果 A 与 B 有关系，B 与 C 有关系，则 A 与 C 存在间接关系。
    通过计算传递闭包，可以将所有间接连接的用户归为一个等价类，即同一个社交圈。
    传递闭包可以使用 Warshall 算法来计算。

    朋友推荐
    对于好友推荐，利用“朋友的朋友”原理：
    如果用户 A 关注了 B，B 又关注了 C，则可以为 A 推荐 C。但需要排除 A 自身及 A 已经关注的用户。

### 3. 算法伪代码

下面给出生成社交圈的伪代码（GenerateCircles）：
```
Input: 用户集合 U, 关注关系 R（以关系字典或边列表表示）
Output: 社交圈列表

1. 将关系 R 转换为边列表 LR = {(A, B) | A 关注 B}
2. 计算对称闭包 SYM：
      SYM = LR ∪ {(B, A) for each (A, B) in LR if (B, A) not in LR}
3. 计算传递闭包 TRANS：
      使用 Warshall 算法对 SYM 进行传递闭包计算
4. 将传递闭包 TRANS 转换为字典映射（等价类表示） eq_dict：
      对于每个 (A, B) ∈ TRANS，将 B 加入到 A 的等价类中
5. 对于 U 中的每个用户 user：
      如果 user 在 eq_dict 中，则其社交圈 = eq_dict[user] ∪ {user}
      否则，其社交圈为 {user}（单独圈）
6. 对所有社交圈去重并排序，返回社交圈列表
```

### 4. 流程图说明
```
      开始
        │
        ▼
  输入用户数量 n
        │
        ▼
  输入 n 个用户名 → 构成集合 U
        │
        ▼
  输入关注关系数量 m
        │
        ▼
  输入 m 条关注关系 → 构成关系 R
        │
        ▼
  调用 Relationship.follow() 建立关注关系
        │
        ▼
  读取操作命令：
    ┌─────────────────────────┐
    │ query_following /       │
    │ recommend_friends /     │
    │ generate_circles        │
    └─────────────────────────┘
        │
        ▼
    根据命令调用相应函数：
        ├─────────────┐
        │             │
  查询关注列表     好友推荐      生成社交圈（调用 GenerateCircles）
        │             │                │
        ▼             ▼                ▼
  输出结果       输出结果         输出每个社交圈信息
        │             │                │
        ▼             ▼                ▼
      结束            结束             结束
```
### 代码解释

1. 整体结构

- 主程序（main 函数）
    主程序负责接收命令行输入，包括用户信息、关注关系和操作命令，然后调用相应的功能函数执行操作并输出结果。

- 辅助函数 generate_circles
    利用二元关系的对称闭包和传递闭包来生成社交圈（等价类），该函数不使用 DFS，而是依赖于关系模块中的函数来完成二元关系的闭包计算。

- 模块导入
    import relationship 导入了本项目自定义的关系模块，模块中包含了对关注关系的管理及二元关系相关的方法，如 dr2lr、symmetric、transitive、lr2dr、followings_of、followed_by、recommend 等。

2. 头文件/库的作用
```
relationship 模块
│── 函数/方法
│   │── follow(target_user, followings)  # 记录 target_user 关注 followings 中的每个用户。
│   │── followings_of(target_user)  # 返回 target_user 关注的用户列表。
│   │── followed_by(target_user)  # 返回关注 target_user 的用户列表。
│   │── recommend(target_user)  # 基于“朋友的朋友”原理，返回推荐好友列表。
│   │── dr2lr(dictionary)  # 将关系字典转换为边列表（list of pairs）。
│   │── symmetric(lr)  # 计算边列表的对称闭包。
│   │── transitive(lr)  # 使用 Warshall 算法计算边列表的传递闭包。
│   │── lr2dr(lr)  # 将边列表转换回关系字典。
│
│── 作用
│   │── 该模块封装了与用户关注关系相关的所有逻辑。
│   │── 支持对二元关系的基本操作和闭包计算。
│   │── 为主程序提供数据处理和计算支持。

```
3. 全局变量和类
```
全局变量  
│── 在本代码中未定义全局变量，所有数据都在 main() 内部通过局部变量传递。  

Relationship 类  
│── 定义在 relationship 模块中，用于存储和操作用户之间的关注关系。  
│  
│── 作用  
│   │── 存储关注数据  
│   │── 实现关注关系的更新、查询和推荐逻辑  
│   │── 提供二元关系操作的辅助函数（如转换为边列表、闭包计算等）  

```

### 函数或方法的详细介绍

4.1 generate_circles(rel, users)

- 参数

        rel：Relationship 类的实例，包含所有用户的关注关系（类型：Relationship）。

        users：所有用户名的列表（类型：List[str]）。

- 返回值

        返回一个社交圈列表，每个社交圈用一个集合表示（类型：List[Set[str]]）。

- 关键代码说明

        lr = rel.dr2lr(rel.relationship)：将关系字典转换为边列表。

        sym_lr = rel.symmetric(lr)：计算对称闭包，使得关注关系变为无向关系。

        trans_lr = rel.transitive(sym_lr)：计算传递闭包，得到所有间接连接的用户对。

        eq_dict = rel.lr2dr(trans_lr)：将闭包结果转换回字典，方便后续查找等价类。

        遍历每个用户，构造其社交圈，并通过集合比较去重，最终排序输出。

4.2 main()

- 功能

        读取用户数量、用户名、关注关系数量和具体的关注关系。

        根据用户输入的操作命令调用不同的功能：

            query_following：调用 rel.followings_of(target_user) 查询并输出目标用户关注列表。

            recommend_friends：调用 rel.recommend(target_user) 获取并输出好友推荐列表。

            generate_circles：调用 generate_circles(rel, users) 生成社交圈，并按指定格式输出每个圈子。

- 输入输出说明

        输入采用命令行交互方式，输出根据操作类型采用不同格式显示结果。

- 关键代码说明

        通过 input() 读取用户和关系数据，然后依次调用 Relationship 类中的方法建立关注关系。

        依据操作命令进行分支判断，调用相应的方法输出结果。

### 总结

本实验利用离散数学中二元关系的对称性和传递性闭包理论来处理社交网络中用户之间的关注关系，并生成社交圈（等价类）。实验思路明确了如何将关注关系建模为二元关系，再通过闭包计算得到所有用户的等价类。代码结构上，主程序（main()）负责输入输出和调用相关函数，辅助函数 generate_circles 通过关系模块中提供的函数完成社交圈的生成。