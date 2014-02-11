---
layout: post
title: "拓扑排序及其应用"
date: 2014-01-30 17:13
comments: true
categories: 
---
概念介绍
--------

在生活中，我们做每件事情都是有一定步骤的，比如洗衣服要经过`放衣服->加水->洗涤->脱水`几个步骤，做蛋炒饭要`打鸡蛋->炒鸡蛋->炒米饭`。每个步骤的开始依赖于前一步骤的完成，顺序是不能乱的。

对于一件事情先做哪步后做哪步一目了然，可是如果告诉你：
> 把衣服泡上、写篇读书笔记、把衣服洗了、学个算法、把文章读了、作业让家长检查、
> 做个算法练习题、把作业写完……`

恐怕先做哪个后做哪个还得好好琢磨一下吧。

拓扑排序就是用来解决这个问题的。我们给每个任务一个编号，如果一个任务依赖于另一个任务，我们就用一个箭头从前面的任务指向后面的任务，比如图中的`7->11`的箭头说明要做`11`这件事必须先把`7`完成了。这样我们就得到一个有向无环图（directed acyclic graph，DAG），我们可以根据这个图来确定完成任务的顺序。
> 无环是因为我们无法对相互依赖的任务给出一个完成顺序，就像我们不知道先有鸡还是先有蛋一样
<!--more-->
![有向无环图][1]

将任务转换成DAG之后，我们可以用**拓扑排序**（Topological sorting）算法得到满足依赖关系的任务完成顺序。在介绍拓扑排序之前，我们先定义几个概念：

* 入度：其他节点指向当前节点的箭头数目
* 出度：一个节点指向其他节点的箭头数目

那么要得到一组拓扑排序，我们可以先选择所有入度为0的节点来完成，然后把已完成的节点和他们所有指向其他节点的箭头都移除，得到一个子图，再从子图中选择入度为0的节点，再进行移除……直到所有节点都被选中为止。这种算法采用了贪心的思想，伪代码如下：

    L ← 按顺序保存选择过的节点，初始为空
    S ← 所有入度为0的节点的集合
    while S非空：
        从S中取出一个节点n
        将n加入到L的尾部
        对所有被节点n通过边e指向的节点m：
            将边e移除
            if 节点m的入度变为了0：
                把m加入集合S
    if 还有边没有移除：
        return error （该图至少有一个环，不是DAG）
    else：
        return L （保存了拓扑排序的结果）

这个算法需要对图进行修改，程序执行完图也被破坏了。
另外还有一个基于深度优先搜索（depth-first search，DFS）的算法可以在保证图的完整性的情况下求得DAG的拓扑排序：

    L ← 按顺序保存选择过的节点，初始为空
    while 存在未访问过的节点：
        选择一个未访问过的节点n
        visit(n) 
        
    function visit(node n)
        if 节点n的访问状态为-1（在当前DFS中已被访问过）：
            return error  （该图至少有一个环，不是DAG）
        if 节点n的访问状态为0（从未被访问过）
            将节点n的访问状态置为-1
            对所有被节点n通过边e指向的节点m：
                visit(m)
            将节点n的访问状态置为1（节点已访问过，且访问过程未出错）
            把节点n加到L的头部（加到头部是因为该操作在后继节点访问完毕后执行）

这个算法不要求从入度为0的节点开始，因此一条完整依赖链的拓扑排序可能不是由一次递归调用得到的。究竟使用哪种算法应该根据具体的应用场景进行选择。

对于相同优先级的节点，选择是不受约束的，因此一个DAG的拓扑排序可能有多个结果，比如图示的拓扑排序可能是：

* 7, 5, 3, 11, 8, 2, 9, 10 (visual left-to-right, top-to-bottom)
* 3, 5, 7, 8, 11, 2, 9, 10 (smallest-numbered available vertex first)
* 3, 7, 8, 5, 11, 10, 2, 9 (because we can)
* 5, 7, 3, 8, 11, 10, 9, 2 (fewest edges first)
* 7, 5, 11, 3, 10, 8, 9, 2 (largest-numbered available vertex first)
* 7, 5, 11, 2, 3, 8, 9, 10 (attempting top-to-bottom, left-to-right)

但是所有结果都是满足依赖关系约束的。

程序编写
--------
贪心算法：

    const int MAXN = 1005;

    int nr_nodes; // 节点数目
    deque<int> result; // 排序结果

    int graph[MAXN][MAXN]; // 边（邻接矩阵）
    int degree[MAXN]; // 入度
    bool toposort()
    {
        deque<int> seeds;
        for (int u = 0; u < nr_nodes; u++) {
            if (degree[u] == 0)
                seeds.push_back(u);
        }
        while (!seeds.empty()) {
            int u = seeds.front();
            seeds.pop_front();
            result.push_back(u);
            for (int v = 0; v < nr_nodes; v++) {
                if (graph[u][v] == 1) {
                    graph[u][v] = 0;
                    degree[v] --;
                    if (degree[v] == 0)
                        seeds.push_back(v);
                }
            }
        }
        for (int u = 0; u < nr_nodes; u++) {
            if (degree[u] != 0)
                return false;
        }
        return true;
    }

DFS算法：
    
    const int MAXN = 1005;

    int nr_nodes;
    deque<int> result;

    int graph[MAXN][MAXN];
    int status[MAXN];
    bool visit_dfs(int node)
    {
        if (status[node] == -1)
            return false;
        if (status[node] == 0) {
            status[node] = -1;
            for (int v = 0; v < nr_nodes; v++) {
                if (graph[node][v])
                    visit_dfs(v);
            }
            status[node] = 1;
            result.push_front(node);
        }
        return true;
    }

    bool toposort_dfs()
    {
        for (int u = 0; u < nr_nodes; u++) {
            if ((status[u] == 0) && !visit_dfs(u))
                return false;
        }
        return true;
    }


例题讲解
--------
NOIP2013P4.车站分级
> 一条单向的铁路线上，依次有编号为1, 2, …, n的n个火车站。每个火车站都有一个级别，最低为1级。现有若干趟车次在这条线路上行驶，每一趟都满足如下要求：如果这趟车次停靠了火车站x，则始发站、终点站之间所有级别大于等于火车站x的都必须停靠。（注意：起始站和终点站自然也算作事先已知需要停靠的站点）
例如，下表是5趟车次的运行情况。其中，前4趟车次均满足要求，而第5趟车次由于停靠了3号火车站（2级）却未停靠途经的6号火车站（亦为2级）而不满足要求。     
> ![enter image description here][2]     
> 现有m趟车次的运行情况（全部满足要求），试推算这n个火车站至少分为几个不同的级别。

输入：

> 第一行包含2个正整数n, m，用一个空格隔开。
第i+1行（1≤i≤m）中，首先是一个正整数s_i（2≤s_i≤n），表示第i趟车次有s_i个停靠站；接下来有s_i个正整数，表示所有停靠站的编号，从小到大排列。每两个数之间用一个空格隔开。输入保证所有的车次都满足要求。

输出：

> 输出只有一行，包含一个正整数，即n个火车站最少划分的级别数。

样例：

    【输入样例1】
        9 2
        4 1 3 5 6
        3 3 5 6
    【输入样例2】
        9 3
        4 1 3 5 6
        3 3 5 6
        3 1 5 9
    【输出样例1】
        2
    【输出样例2】
        3

备注：

> 对于20%的数据，1 ≤ n, m ≤ 10；
对于50%的数据，1 ≤ n, m ≤ 100；
对于100%的数据，1 ≤ n, m ≤ 1000。

思路：

> 在介绍拓扑排序的时候，我们是以任务的依赖关系为例讲解的，实际上依赖关系也可以抽象为优先级关系，先做的任务优先级高，后做的任务优先级低。对于题目中的每一趟列车，其经过的车站可以分为两个优先级，高级的要停靠，低级的不停靠。每趟列车相当于给出一组高低优先级的约束，比如一辆车的停靠站点是`1、3、5`，那么`1、3、5`属于高级站点，`2、4`属于低级站点；另外一辆车停靠`1、3、6`，那么`1、3、6`属于高级站点，`2、4、5`属于低级站点，其中站点`5`既在前车的高级站点里又在后车的低级站点里，那么说明两个分级是不够的，需要再增加一个分级才能满足要求。对于每辆车的约束，我们可以用一个DAG图表示。比如上面的两个约束可以分别表示为下图：    
> ![enter image description here][3]   
> 将两个约束合并可以得到一个新的DAG：   
> ![enter image description here][4]    
> 而站点分级的数目就等于从入度为0的节点（可能有多个）到出度为0的节点（可能有多个）的所有路径中最长（经过节点最多）的那条的节点数目。
因此，我们可以使用拓扑排序解决这个问题。可以在贪心版本的基础上稍加修改得到最终的算法。这个任务就交给你啦，写好后可以在下面的网址提交代码来在线验证程序的正确性。

[在线测试](http://oj.luogu.org:8888/problemshow?pid=1983).

自测练习
--------
[Sorting It All Out](http://poj.org/problem?id=1094).   
[Following Orders](http://poj.org/problem?id=1270).    
[Window Pains](http://poj.org/problem?id=2585).    
[Frame Stacking](http://poj.org/problem?id=1128).    
[Genealogical tree](http://poj.org/problem?id=2367).   
[Cow Contest](http://poj.org/problem?id=3660).   


> Written with [StackEdit](https://stackedit.io/).


  [1]: https://lh5.googleusercontent.com/-UbpA_iTD2mk/Uuopjzy3_VI/AAAAAAAAAHs/hddLajfvSmM/s0/180px-Directed_acyclic_graph.png "有向无环图"
  [2]: https://lh3.googleusercontent.com/nV-iq3qxZ2N5rl3bfNT-HFdbf_xLBfjkTU9zfgCuXi0=s550 "车站分级"
  [3]: https://lh4.googleusercontent.com/-WuTaT998gts/Uuy-dxFgU0I/AAAAAAAAAIU/owTgnm98T3w/s0/1.png "1.png"
  [4]: https://lh5.googleusercontent.com/-nzjW-qLq0C8/Uuy--M8Qc1I/AAAAAAAAAIg/oKPNQgX5SvE/s0/2.png "2.png"
