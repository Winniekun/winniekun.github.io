# 拓扑排序

### 1.存在前提

​	当且仅当一个有向图为有向无环图（directed acyclic graph，或称DAG）时，才能得到对应于该图的拓扑排序。每一个有向无环图都至少存在一种拓扑排序。

​	该论断可以利用反证法被证明如下：

​	假设我们有一由`v_1`到`v_n`这n个结点构成的有向图，且图中v_1，v_2，...，v_n这些结点构成一个环。这即是说对于所有`1≤i，图中存在一条有向边从`v_i`指向`v_i+1`。同时还存在一条从`v_n`指向`v_1`的边。假设该图存在一个拓扑排序。那么基于这样一个有向图，显然我们可以得知对于所有`1≤i，`v_i`必须在`v_i+1`之前被遍历，也就是`v_1`必须在`v_n`之前被遍历。同时由于*还存在一条从`v_n`指向`v_1`的边*，`v_n`必须在`v_1`之前被遍历。这里出现了与我们的假设所冲突的结果。因此我们可以知道，该图存在拓扑排序的假设不成立。也就是说，对于非有向无环图而言，其拓扑排序不存在。

### 2.拓扑排序的算法和实现

​	拓扑排序的问题存在一个线性时间解。由于求出一种拓扑排序方法的最优时间复杂度为 O(n+m)，其中 n 和 m 分别是有向图 G的节点数和边数，或在`O(n+m)`时间内确定该图不是有向无环图，也就是说对应的拓扑排序不存在。

2.1广度优先遍历

​	广度优先遍历，指的是从图的一个未遍历的节点出发，先遍历这个节点的相邻节点，再依次遍历每个相邻节点的相邻节点。

![img](https://s2.51cto.com/oss/202004/16/29dc30c1546cc73846153359b0fad8fb.gif)

2.2 深度优先遍历

​	主要思路是从图中一个未访问的顶点 V 开始，沿着一条路一直走到底，然后从这条路尽头的节点回退到上一个节点，再从另一条路开始走到底...，不断递归重复此过程，直到所有的顶点都遍历完成，它的特点是不撞南墙不回头，先走完一条路，再换一条路继续走。

### 3.力扣例题

#### 例题一：207.[课程表](https://leetcode-cn.com/problems/course-schedule)

你这个学期必须选修 numCourse 门课程，记为 0 到 numCourse-1 。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 1 ，你需要先完成课程 0 ，我们用一个匹配来表示他们：[1,0]

给定课程总量以及它们的先决条件，请你判断是否可能完成所有课程的学习？

#####  方法一 ：广度优先遍历（队列）

每一次都从图中删除没有前驱的顶点，这里并不需要真正的做删除操作，我们可以设置一个入度数组，每一轮都输出入度为 0 的结点，并移除它、修改它指向的结点的入度（-1即可），依次得到的结点序列就是拓扑排序的结点序列。如果图中还有结点没有被移除，则说明“不能完成所有课程的学习”。

![](C:\Users\Administrator\Desktop\1.png)

​	![2](C:\Users\Administrator\Desktop\2.png)

![3](C:\Users\Administrator\Desktop\3.png)

![有向有环图](C:\Users\Administrator\Desktop\4.png)

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        //创建数组保存节点入度
        int[] indegrees = new int[numCourses];
        //创建数组保存每一个节点的邻接链表
        List<List<Integer>> adjacency = new ArrayList<>();
        //使用队列保存入度为0的点
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < numCourses; i++)
            adjacency.add(new ArrayList<>());
        //获取每一门课程的前置课程条件
        for(int[] cp : prerequisites) {
            //对应节点的入度+1
            indegrees[cp[0]]++;
            //添加到前置节点的邻接表中
            adjacency.get(cp[1]).add(cp[0]);
        }
        // 将入度为0的节点添加到队列中
        for(int i = 0; i < numCourses; i++)
            if(indegrees[i] == 0) queue.add(i);
        // BFS TopSort.
        while(!queue.isEmpty()) {
            int pre = queue.poll();
            //未被访问节点减一
            numCourses--;
            //该节点邻接表中节点的入度减一，当入度为0则添加到队列中
            for(int cur : adjacency.get(pre))
                if(--indegrees[cur] == 0) 
                    queue.add(cur);
        }
        //判断是否还有入度不为0的节点，返回结果
        return numCourses == 0;
    }
}
```

##### 方法二 ：深度优先遍历（栈）

第 1 步：构建邻接表；

第 2 步：建立访问标志

第 3 步：递归处理每一个还没有被访问的结点

![5](C:\Users\Administrator\Desktop\5.png)

![6](C:\Users\Administrator\Desktop\6.png)

```java
class Solution {
    //创建邻接表
    List<List<Integer>> edges;
    //保存节点的访问信息
    int[] visited;
    //是否可以完成学习
    boolean valid = true;

    public boolean canFinish(int numCourses, int[][] prerequisites) {
        edges = new ArrayList<List<Integer>>();
        for (int i = 0; i < numCourses; ++i) {
            //为每一个节点创建链表保存前置节点信息
            edges.add(new ArrayList<Integer>());
        }
        visited = new int[numCourses];
        for (int[] info : prerequisites) {
            //info[1]为学习课程info[0]的前置条件
            edges.get(info[1]).add(info[0]);
        }
        for (int i = 0; i < numCourses && valid; ++i) {
            if (visited[i] == 0) {
                dfs(i);
            }
        }
        return valid;
    }

    public void dfs(int u) {
        //将该节点置为已访问
        visited[u] = 1;
        //遍历该节点的邻接节点
        for (int v: edges.get(u)) {
            if (visited[v] == 0) {
                //如果邻接节点存在未访问节点，则继续递归遍历
                dfs(v);
                if (!valid) {
                    return;
                }
            } else if (visited[v] == 1) {
                valid = false;
                return;
            }
        }
        //节点没有邻接节点。则加入栈中，并把标志置为2，意为已完成搜索·
        visited[u] = 2;
    }
}
```

#### 例题二 [课程表II](https://leetcode-cn.com/problems/course-schedule-ii)

现在你总共有 n 门课需要选，记为 0 到 n-1。

在选修某些课程之前需要一些先修课程。 例如，想要学习课程 0 ，你需要先完成课程 1 ，我们用一个匹配来表示他们: [0,1]

给定课程总量以及它们的先决条件，返回你为了学完所有课程所安排的学习顺序。

可能会有多个正确的顺序，你只要返回一种就可以了。如果不可能完成所有课程，返回一个空数组。

##### 方法一：广度优先遍历

```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        //保存课程完成顺序
        List<Integer> list = new ArrayList<>();
        //创建数组保存节点入度
        int[] indegrees = new int[numCourses];
        //创建数组保存每一个节点的邻接链表
        List<List<Integer>> adjacency = new ArrayList<>();
        //使用队列保存入度为0的点
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < numCourses; i++)
            adjacency.add(new ArrayList<>());
        //获取每一门课程的前置课程条件
        for(int[] cp : prerequisites) {
            //对应节点的入度+1
            indegrees[cp[0]]++;
            //添加到前置节点的邻接表中
            adjacency.get(cp[1]).add(cp[0]);
        }
        // 将入度为0的节点添加到队列中
        for(int i = 0; i < numCourses; i++)
            if(indegrees[i] == 0) queue.add(i);
        // BFS TopSort.
        while(!queue.isEmpty()) {
            int pre = queue.poll();
            list.add(pre);
            //未被访问节点减一
            numCourses--;
            //该节点邻接表中节点的入度减一，当入度为0则添加到队列中
            for(int cur : adjacency.get(pre))
                if(--indegrees[cur] == 0) 
                    queue.add(cur);
        }
        //判断是否还有入度不为0的节点，返回结果
        return numCourses == 0 ? list.stream().mapToInt(Integer::valueOf).toArray():new int[0];
    }
}
```

##### 方法二： 深度优先遍历

```java
    // 方法 2：邻接矩阵 + DFS   由于用的数组，每次都要遍历，效率比较低
    class Solution {
    // 存储有向图
    List<List<Integer>> edges;
    // 标记每个节点的状态：0=未搜索，1=搜索中，2=已完成
    int[] visited;
    // 用数组来模拟栈，下标 n-1 为栈底，0 为栈顶
    int[] result;
    // 判断有向图中是否有环
    boolean valid = true;
    // 栈下标
    int index;

    public int[] findOrder(int numCourses, int[][] prerequisites) {
        edges = new ArrayList<List<Integer>>();
        for (int i = 0; i < numCourses; ++i) {
            edges.add(new ArrayList<Integer>());
        }
        visited = new int[numCourses];
        result = new int[numCourses];
        index = numCourses - 1;
        for (int[] info : prerequisites) {
            edges.get(info[1]).add(info[0]);
        }
        // 每次挑选一个「未搜索」的节点，开始进行深度优先搜索
        for (int i = 0; i < numCourses && valid; ++i) {
            if (visited[i] == 0) {
                dfs(i);
            }
        }
        if (!valid) {
            return new int[0];
        }
        // 如果没有环，那么就有拓扑排序
        return result;
    }

    public void dfs(int u) {
        // 将节点标记为「搜索中」
        visited[u] = 1;
        // 搜索其相邻节点
        // 只要发现有环，立刻停止搜索
        for (int v: edges.get(u)) {
            // 如果「未搜索」那么搜索相邻节点
            if (visited[v] == 0) {
                dfs(v);
                if (!valid) {
                    return;
                }
            }
            // 如果「搜索中」说明找到了环
            else if (visited[v] == 1) {
                valid = false;
                return;
            }
        }
        // 将节点标记为「已完成」
        visited[u] = 2;
        // 将节点入栈
        result[index--] = u;
    }
}

```

#### 例题三 [矩阵中的最长链接路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix/)

​	给定一个整数矩阵，找出最长递增路径的长度。

​	对于每个单元格，你可以往上，下，左，右四个方向移动。 你不能在对角线方向上移动或移动到边界外（即不允许环绕）。

输入: nums = 
[
  [9,9,4],
  [6,6,8],
  [2,1,1]
] 
输出: 4 
解释: 最长递增路径为 [1, 2, 6, 9]。

```java
class Solution {
    public int longestIncreasingPath(int[][] matrix) {
        if(matrix.length == 0){
            return 0;
        }
        //visited有两个作用：1.判断是否访问过，2.存储当前格子的最长递增长度
        int[][] visited = new int[matrix.length][matrix[0].length];
        int max = 0;
        for(int i=0; i<matrix.length; i++){
            for(int j=0; j<matrix[0].length; j++){
                if(visited[i][j] == 0){
                    //这里先做一次比较找出max，可以避免最后再去遍历一个visited数组
                    max = Math.max(max, dfs(i, j, matrix, visited));
                }
                //max = Math.max(max, visited[i][j]);
                
            }
        }
        return max;
    }
    public int dfs(int i, int j, int[][] matrix, int[][] visited){
        if(i < 0 || i >= matrix.length || j < 0 || j >= matrix[0].length){
            return 0;
        }
        if(visited[i][j] > 0){
            return visited[i][j];
        }
        int max = 0;
        //这里分别去判断4周是否比当前数小，然后去递归遍历
        if(i - 1 >= 0 && matrix[i-1][j] < matrix[i][j]){
            max = Math.max(max, dfs(i-1, j, matrix, visited));
        }
        if(i + 1 < matrix.length && matrix[i+1][j] < matrix[i][j]){
            max = Math.max(max, dfs(i+1, j, matrix, visited));
        }
        if(j - 1 >= 0 && matrix[i][j-1] < matrix[i][j]){
            max = Math.max(max, dfs(i, j-1, matrix, visited));
        }
        if(j + 1 < matrix[0].length && matrix[i][j+1] < matrix[i][j]){
            max = Math.max(max, dfs(i, j+1, matrix, visited));
        }
        
        visited[i][j] = max+1;
        return max+1;

    }
}

```
