---
title: trie
date: 2020-07-07 19:58:58
tags: trie
categories: 算法
thumbnail:
---



## 前缀树原理

<!--more-->

Trie, 由`Edward Fredkin`提出, 来自英文单词`retrieval`, Trie也称为前缀树, 用于保存[关联数组](https://zh.wikipedia.org/wiki/关联数组)，其中的键通常是[字符串](https://zh.wikipedia.org/wiki/字符串)。与[二叉查找树](https://zh.wikipedia.org/wiki/二叉查找树)不同，键不是直接保存在节点中，而是由节点在树中的位置决定。一个节点的所有子孙都有相同的[前缀](https://zh.wikipedia.org/wiki/前缀)，也就是这个节点对应的字符串，而根节点对应[空字符串](https://zh.wikipedia.org/wiki/空字符串)。一般情况下，不是所有的节点都有对应的值，只有叶子节点和部分内部节点所对应的键才有相关的值。 

对于字符Trie而言, 如果使用二叉树那样的两个分支明显是不够的. 举个例子, 英语中一共有26个字母, 每个字母还区分大小写, 如果忽略大小写的话, 那么可以使用简单的限定分支(子树)个数为26. 如下图所示为一个字符串Trie

![Trie](https://i.loli.net/2020/07/07/eMrDF8pubcj5kth.png)

> 一个保存了8个键的trie结构，"A", "to", "tea", "ted", "ten", "i", "in", "inn

 这只是针对于不区分大小写的英语字母, 若是区分大小写, 同时还有标点等, 那么分之的数量会更加的庞大且不确定, 这是我们可以使用散列表来解决动态数量的分支.

Trie的出现, 解决了散列表无法解决的字符串数集问题:

- 找到具有同一前缀的全部键值。
- 按词典序枚举字符串的数据集。

同时Trie由散列表的另一方面是: 随着数据量的增大, 散列表的查找效率可能会降低到$O(n)$, 与哈希表相比, Trie 树在存储多个具有相同前缀的键时可以使用较少的空间. 此时Trie只需要$O(m)$的时间复杂度，其中$m$为键长。



## 前缀树的实现

Trie 树是一个有根的树，其结点具有以下字段：。

* 最多$R$个指向子结点的链接，其中每个链接对应字母表数据集中的一个字母, 简单起见, 假定$R$为 26, 小写英语字母数量
* 布尔字段, 以指定节点是对应键的结尾还是只是键前缀.



### 节点构造

在构造之前, 先展示包含三个单词"sea","sells","she"的 Trie的样子, 加深理解

![](https://i.loli.net/2020/07/07/A16vF79kB23HYKl.png)



```java
class TrieNode {
    private TrieNode[] links;
    private boolean isEnd;
    private final int R = 26;
    public TrieNode() {
        links = new TrieNode[R];
    }
    public boolean containsKey(char ch) {
        return links[ch - 'a'] != null;
    }
    public void put(char ch, TrieNode node) {
        links[ch - 'a'] = node;
    }
    public TrieNode get(char ch) {
        return links[ch - 'a'];
    }
    public void setEnd() {
        isEnd = true;
    }
    public boolean isEnd() {
        return isEnd;
    }
}
```

### 插入

```java
/**
 * 插入
 */
public void insert(String word) {
    // 从根节点开始, 根节点为空
    TrieNode node = root;
    for (int i = 0; i < word.length(); i++) {
        char current = word.charAt(i);
        if (!node.containsKey(current)) {
            node.put(current, new TrieNode());
        }
        node = node.get(current);
    }
    node.setEnd();
}
```



### 查找

每个键在 trie 中表示为从根到内部节点或叶的路径。我们用第一个键字符从根开始，。检查当前节点中与键字符对应的链接。有两种情况：

1. 存在链接。我们移动到该链接后面路径中的下一个节点，并继续搜索下一个键字符。
2. 不存在链接。若已无键字符，且当前结点标记为 isEnd，则返回 true。否则有两种可能，均返回 false :
   1. 还有键字符剩余，但无法跟随 Trie 树的键路径，找不到键。
   2. 没有键字符剩余，但当前结点没有标记为 isEnd。也就是说，待查找键只是Trie树中另一个键的前缀。

```java
private TrieNode searchPrefix(String word) {
    TrieNode node = root;
    for (int i = 0; i < word.length(); i++) {
        char current = word.charAt(i);
        if (node.containsKey(current)) {
            node = node.get(current);
        } else {
            return null;
        }
    }
    return node;
}

public boolean search(String word) {
    TrieNode node = searchPrefix(word);
    return node != null && node.isEnd();
}

/**
 * 是否存在输入前缀的字符
 */
public boolean startsWith(String prefix) {
    TrieNode node = searchPrefix(prefix);
    return node != null;
}
```

### 整体

```java
class Trie {

    private TrieNode root;

    /**
     * 初始化
     */
    public Trie() {
        root = new TrieNode();
    }

    /**
     * 插入
     */
    public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char current = word.charAt(i);
            if (!node.containsKey(current)) {
                node.put(current, new TrieNode());
            }
            node = node.get(current);
        }
        node.setEnd();
    }

    /**
     * 删除
     */
    public boolean search(String word) {
        TrieNode node = searchPrefix(word);
        return node != null && node.isEnd();
    }

    private TrieNode searchPrefix(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            char current = word.charAt(i);
            if (node.containsKey(current)) {
                node = node.get(current);
            } else {
                return null;
            }
        }
        return node;
    }

    /**
     * 是否存在输入前缀的字符
     */
    public boolean startsWith(String prefix) {
        TrieNode node = searchPrefix(prefix);
        return node != null;
    }

    class TrieNode {
        private TrieNode[] links;
        private boolean isEnd;
        private final int R = 26;

        public TrieNode() {
            links = new TrieNode[R];
        }

        public boolean containsKey(char ch) {
            return links[ch - 'a'] != null;
        }

        public void put(char ch, TrieNode node) {
            links[ch - 'a'] = node;
        }

        public TrieNode get(char ch) {
            return links[ch - 'a'];
        }

        public void setEnd() {
            isEnd = true;
        }

        public boolean isEnd() {
            return isEnd;
        }

    }
}
```



## 敏感词过滤的前缀树实现

因为敏感词除了英文字母之外, 还有其他的字符, 所以使用散列表来实现, 

```java
public class Trie{
    // 根节点
	private TrieNode rootNode = new TrieNode();
    
    // 将一个敏感词添加到前缀树中
    private void addKeyword(String keyword) {
        TrieNode tempNode = rootNode;
        for (int i = 0; i < keyword.length(); i++) {
            char c = keyword.charAt(i);
            TrieNode subNode = tempNode.getSubNode(c);
            if(!tempNode.containsKey(c)){
                tempNode.addSubNode(c, new TrieNode());
            }
            tempNode = tempNode.getSubNode(c);
        }
        tempNode.setKeywordEnd(true);
    }

    // 前缀树
    private class TrieNode {
        // 关键词结束标识
        private boolean isKeywordEnd = false;
        // 子节点(key是下级字符,value是下级节点)
        private Map<Character, TrieNode> subNodes = new HashMap<>();
        public boolean isKeywordEnd() {
            return isKeywordEnd;
        }
        public void setKeywordEnd(boolean keywordEnd) {
            isKeywordEnd = keywordEnd;
        }
        // 添加子节点
        public void addSubNode(Character c, TrieNode node) {
            subNodes.put(c, node);
        }
        // 获取子节点
        public TrieNode getSubNode(Character c) {
            return subNodes.get(c);
        }
    }
}
```



### 实例讲解



### 最终实现

```java
public class SensitiveFilter {
    
    // 替换符
    private static final String REPLACEMENT = "***";
    // 根节点
    private TrieNode rootNode = new TrieNode();
    // 加载敏感字符, 只加载一次
    @PostConstruct
    public void init() {
        try (
                InputStream is = this.getClass().getClassLoader().getResourceAsStream("sensitive-word.txt");
                BufferedReader reader = new BufferedReader(new InputStreamReader(is));
        ) {
            String keyword;
            while ((keyword = reader.readLine()) != null) {
                // 添加到前缀树
                this.addKeyword(keyword);
            }
        } catch (IOException e) {
            logger.error("加载敏感词文件失败: " + e.getMessage());
        }
    }
    // 将一个敏感词添加到前缀树中
    private void addKeyword(String keyword) {
        TrieNode tempNode = rootNode;
        for (int i = 0; i < keyword.length(); i++) {
            char c = keyword.charAt(i);
            TrieNode subNode = tempNode.getSubNode(c);
            if(!tempNode.containsKey(c)){
                tempNode.addSubNode(c, new TrieNode());
            }
            tempNode = tempNode.getSubNode(c);
        }
        tempNode.setKeywordEnd(true);
    }
    /**
     * 过滤敏感词
     *
     * @param text 待过滤的文本
     * @return 过滤后的文本
     */
    public String filter(String text) {
        if (StringUtils.isBlank(text)) {
            return null;
        }
        // 指针1
        TrieNode tempNode = rootNode;
        // 指针2
        int begin = 0;
        // 指针3
        int position = 0;
        // 结果
        StringBuilder sb = new StringBuilder();
        while (position < text.length()) {
            char c = text.charAt(position);
            // 跳过符号
            if (isSymbol(c)) {
                // 若指针1处于根节点,将此符号计入结果,让指针2向下走一步
                if (tempNode == rootNode) {
                    sb.append(c);
                    begin++;
                }
                // 无论符号在开头或中间,指针3都向下走一步
                position++;
                continue;
            }
            // 检查下级节点
            tempNode = tempNode.getSubNode(c);
            if (tempNode == null) {
                // 以begin开头的字符串不是敏感词
                sb.append(text.charAt(begin));
                // 进入下一个位置
                position = ++begin;
                // 重新指向根节点
                tempNode = rootNode;
            } else if (tempNode.isKeywordEnd()) {
                // 发现敏感词,将begin~position字符串替换掉
                sb.append(REPLACEMENT);
                // 进入下一个位置
                begin = ++position;
                // 重新指向根节点
                tempNode = rootNode;
            } else {
                // 检查下一个字符
                position++;
            }
        }
        // 将最后一批字符计入结果
        sb.append(text.substring(begin));
        return sb.toString();
    }
    // 判断是否为符号
    private boolean isSymbol(Character c) {
        // 0x2E80~0x9FFF 是东亚文字范围
        return !CharUtils.isAsciiAlphanumeric(c) && (c < 0x2E80 || c > 0x9FFF);
    }
    // 前缀树
    private class TrieNode {
        // 关键词结束标识
        private boolean isKeywordEnd = false;
        // 子节点(key是下级字符,value是下级节点)
        private Map<Character, TrieNode> subNodes = new HashMap<>();
        public boolean isKeywordEnd() {
            return isKeywordEnd;
        }
        public void setKeywordEnd(boolean keywordEnd) {
            isKeywordEnd = keywordEnd;
        }
        // 添加子节点
        public void addSubNode(Character c, TrieNode node) {
            subNodes.put(c, node);
        }
        // 获取子节点
        public TrieNode getSubNode(Character c) {
            return subNodes.get(c);
        }
        public boolean containsKey(Character c){
            return subNodes.containsKey(c);
        }
    }
}
```



## References

* 算法新解

* [实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/solution/shi-xian-trie-qian-zhui-shu-by-leetcode/)
* [问答社区-敏感词过滤](https://blog.nowcoder.net/n/543c682c844f4fcaab6be5d22b0530d4)
* [基于前缀树图文详解敏感词过滤](https://490.github.io/基于前缀树图文详解敏感词过滤/)
* [Trie Tree 的实现 (适合初学者)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/solution/trie-tree-de-shi-xian-gua-he-chu-xue-zhe-by-huwt/)   

*  [面试被虐 说说游戏中的敏感词过滤是如何实现的？](https://www.cnblogs.com/kubidemanong/p/10834993.html)