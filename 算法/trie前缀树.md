前缀树（Prefix Tree），也称为字典树（Trie），是一种特殊的树形数据结构，用于高效地存储和检索字符串集合。它的主要应用场景包括：

1. 字符串检索：前缀树可以快速地检索字符串集合中的特定字符串，尤其适用于前缀匹配、单词补全和自动完成等场景。例如，在搜索引擎中实现搜索关键词的提示功能。
2. 单词频率统计：通过将字符串存储在前缀树中，并在每个节点上记录字符串的出现次数，可以方便地统计某个单词的频率。这在文本处理和信息检索中非常有用。
3. 字符串排序：前缀树可以被用作字符串的排序工具。通过遍历整个前缀树，可以获得按照字典序排列的字符串集合。

下面是一个使用Java实现前缀树的测试代码示例：

```java
// 前缀树节点类
class TrieNode {
    private TrieNode[] children;
    private boolean isEndOfWord;

    public TrieNode() {
        children = new TrieNode[26]; // 假设只包含小写字母
        isEndOfWord = false;
    }

    public TrieNode getChild(char ch) {
        return children[ch - 'a'];
    }

    public void setChild(char ch, TrieNode node) {
        children[ch - 'a'] = node;
    }

    public boolean isEndOfWord() {
        return isEndOfWord;
    }

    public void setEndOfWord(boolean endOfWord) {
        isEndOfWord = endOfWord;
    }
}

// 前缀树类
class Trie {
    private TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    public void insert(String word) {
        TrieNode current = root;
        for (int i = 0; i < word.length(); i++) {
            char ch = word.charAt(i);
            TrieNode node = current.getChild(ch);
            if (node == null) {
                node = new TrieNode();
                current.setChild(ch, node);
            }
            current = node;
        }
        current.setEndOfWord(true);
    }

    public boolean search(String word) {
        TrieNode current = root;
        for (int i = 0; i < word.length(); i++) {
            char ch = word.charAt(i);
            TrieNode node = current.getChild(ch);
            if (node == null) {
                return false;
            }
            current = node;
        }
        return current.isEndOfWord();
    }

    public boolean startsWith(String prefix) {
        TrieNode current = root;
        for (int i = 0; i < prefix.length(); i++) {
            char ch = prefix.charAt(i);
            TrieNode node = current.getChild(ch);
            if (node == null) {
                return false;
            }
            current = node;
        }
        return true;
    }
}

// 测试代码
public class TrieTest {
    public static void main(String[] args) {
        Trie trie = new Trie();

        // 插入测试
        trie.insert("apple");
        trie.insert("banana");
        trie.insert("orange");

        // 搜索测试
        System.out.println(trie.search("apple")); // 输出: true
        System.out.println(trie.search("banana")); // 输出: true
        System.out.println(trie.search("orange")); // 输出: true
        System.out.println(trie.search("pear")); // 输出: false

        // 前缀搜索测试
        System.out.println(trie.startsWith("app")); // 输出: true
        System.out.println(trie.startsWith("ban")); // 输出: true
        System.out.println(trie.startsWith("ora")); // 输出: true
        System.out.println(trie.startsWith("pea")); // 输出: false
    }
}
```

以上代码演示了如何使用Java实现一个简单的前缀树，并对其进行插入、搜索和前缀搜索的测试。你可以根据具体的使用场景和需求进行进一步的扩展和优化。

# 示意图
下面是一个使用Markdown绘制的前缀树（Trie）的示意图：

```
          (root)
          /    \
         a      b
        / \      \
       p   p      a
      /     \      \
     l       e      n
    / \      |      |
   e   p     e      a
  /     \    |      |
 p*      l*  *e*    *n*
```

在上面的示意图中，根节点表示空字符（即根节点没有实际存储的字符），每个节点都可能包含一个字符。在节点之间的连线表示字符的连接关系。叶子节点上的星号 (*) 表示一个单词的结束。在这个示例中，前缀树包含了单词 "apple"、"app"、"banana" 和 "bean"。

请注意，前缀树的实际形状会根据插入的字符串集合而有所变化。上述示意图仅为了提供一个基本的可视化示例，实际使用时可能会有更多的节点和连线。
