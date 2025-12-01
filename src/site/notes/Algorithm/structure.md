---
{"dg-publish":true,"permalink":"/algorithm/structure/"}
---


## LRU缓存

```Java
import java.util.HashMap;  
import java.util.Map;  
  
class LRUCache {  
  
    private Map<Integer,Node> map;  
    private DoubleList cache;  
    private int capacity;  
  
    public LRUCache(int capacity){  
        map = new HashMap<>();  
        cache = new DoubleList();  
        this.capacity = capacity;  
    }  
  
    public int get(int key){  
        if(!map.containsKey(key)){  
            return -1;  
        }  
        int val = map.get(key).val;  
        //利用put方法将数据提前  
        put(key,val);  
        return val;  
    }  
  
    public void put(int key,int val){  
        Node node = new Node(key,val);  
        if(map.containsKey(key)){  
            cache.remove(map.get(key));  
            cache.addFirst(node);  
            //更新map中对应的数据  
            map.put(key,node);  
        }else{  
            if(capacity==cache.size()){  
                Node last = cache.removeLast();  
                map.remove(last.key);  
            }  
            cache.addFirst(node);  
            map.put(key,node);  
        }  
  
    }  
}  
/**双向链表节点类*/  
class Node{  
    int key;  
    int val;  
    Node prev;  
    Node next;  
    public Node(int key,int val){  
        this.key=key;  
        this.val=val;  
    }  
}  
  
/**  
 * 双向链表的简单操作实现  
 * @author 奥特曼打小怪兽  
 */  
class DoubleList {  
  
    private Node head;  
    private Node tail;  
    private int size;  
  
    public DoubleList(){  
        head = new Node(0,0);  
        tail = new Node(0,0);  
        head.next=tail;  
        tail.prev=head;  
        size=0;  
    }  
  
    public void addFirst(Node node){  
        node.next = head.next;  
        node.prev = head;  
        head.next.prev = node;  
        head.next = node;  
        size++;  
    }  
  
    public void remove(Node node){  
        node.prev.next = node.next;  
        node.next.prev = node.prev;  
        size--;  
    }  
  
    /**删除最后一个节点，并返回该节点*/  
    public Node removeLast(){  
        if(tail.prev==head){  
            return null;  
        }  
        Node last = tail.prev;  
        remove(last);  
        return last;  
    }  
  
    public int size(){  
        return size;  
    }  
}
```



## LFU

```Java
class Node{  
    int key;  
    int value;  
    int freq=1;  
  
    public Node(){}  
  
    public Node(int key,int value){  
        this.key=key;  
        this.value = value;  
    }  
}  
class LFUCache {  
  
    /**内容缓存*/  
    private Map<Integer,Node> cache;  
    /**存储每个频次对应的双向链表*/  
    private Map<Integer, LinkedHashSet<Node>> freqMap;  
    /**缓存容量*/  
    private int capacity;  
    /**记录最小频次*/  
    private int minFreq;  
    /**记录缓存中的数据个数*/  
    private int size;  
  
    public LFUCache(int capacity) {  
        cache = new HashMap<>(capacity);  
        freqMap = new HashMap<>();  
        this.capacity = capacity;  
    }  
  
    /**  
     * 获得key对应的值  
     * @param key  
     * @return  
     */  
    public int get(int key) {  
        Node node = cache.get(key);  
        if(node==null){  
            return -1;  
        }  
        freqInc(node);  
        return node.value;  
    }  
  
    /**  
     * 存入新的键值对  
     * 如果键已存在，则变更其值；如果键不存在，请插入键值对。  
     * 当缓存达到其容量时。则应该在插入新项之前，使最不经常使用的项无效。  
     * 在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除最久未使用的键。  
     * @param key  
     * @param value  
     */  
    public void put(int key, int value) {  
        if(capacity==0){  
            return;  
        }  
        Node node = cache.get(key);  
        if(node!=null){  
            node.value = value;  
            freqInc(node);  
        }else{  
            if(size==capacity){  
                Node deadNode = deleteNode();  
                cache.remove(deadNode.key);  
                size--;  
            }  
            Node newNode = new Node(key,value);  
            cache.put(key,newNode);  
            addNode(newNode);  
            size++;  
        }  
  
    }  
  
    /**  
     * 更新频次以及最小值  
     * @param node  
     */  
    private void freqInc(Node node){  
        //从原freq对应的set中移除掉node，并更新minFreq  
        int freq = node.freq;  
        LinkedHashSet<Node> set = freqMap.get(freq);  
        set.remove(node);  
        if(freq==minFreq&&set.size()==0){  
            minFreq = freq+1;  
        }  
        //加入新的freq对应的双向链表  
        node.freq++;  
        LinkedHashSet<Node> newSet = freqMap.get(freq+1);  
        if(newSet==null){  
            newSet = new LinkedHashSet<>();  
            freqMap.put(freq+1,newSet);  
        }  
        newSet.add(node);  
    }  
  
    /**  
     * 添加一个节点  
     * @param node  
     */  
    private void addNode(Node node){  
        LinkedHashSet<Node> set = freqMap.get(1);  
        if(set==null){  
            set = new LinkedHashSet<>();  
            freqMap.put(1,set);  
        }  
        set.add(node);  
        minFreq = 1;  
    }  
  
    /**  
     * 删除一个节点，即清除掉一个最久未使用数据  
     * @return  
     */  
    private Node deleteNode(){  
        LinkedHashSet<Node> set = freqMap.get(minFreq);  
        Node deadNode = set.iterator().next();  
        set.remove(deadNode);  
        return deadNode;  
    }  
}
```
## skipList

```Java
import java.util.Random;

class SkipListNode {
    int value;
    SkipListNode[] next;

    public SkipListNode(int value, int level) {
        this.value = value;
        this.next = new SkipListNode[level];
    }
}

public class SkipList {
    private static final int MAX_LEVEL = 16;
    private static final double PROBABILITY = 0.5;
    private int levelCount = 1;
    private SkipListNode head = new SkipListNode(-1, MAX_LEVEL);
    private Random random = new Random();

    public SkipListNode find(int value) {
        SkipListNode p = head;
        for (int i = levelCount - 1; i >= 0; i--) {
            while (p.next[i] != null && p.next[i].value < value) {
                p = p.next[i];
            }
        }
        if (p.next[0] != null && p.next[0].value == value) {
            return p.next[0];
        }
        return null;
    }

    public void insert(int value) {
        int level = randomLevel();
        SkipListNode newNode = new SkipListNode(value, level);
        SkipListNode[] update = new SkipListNode[level];
        SkipListNode p = head;
        for (int i = level - 1; i >= 0; i--) {
            while (p.next[i] != null && p.next[i].value < value) {
                p = p.next[i];
            }
            update[i] = p;
        }
        for (int i = level - 1; i >= 0; i--) {
            newNode.next[i] = update[i].next[i];
            update[i].next[i] = newNode;
        }
        if (levelCount < level) {
            levelCount = level;
        }
    }

    public void delete(int value) {
        SkipListNode[] update = new SkipListNode[levelCount];
        SkipListNode p = head;
        for (int i = levelCount - 1; i >= 0; i--) {
            while (p.next[i] != null && p.next[i].value < value) {
                p = p.next[i];
            
            }
            update[i] = p;
        }
        if (p.next[0] != null && p.next[0].value == value) {
            for (int i = levelCount - 1; i >= 0; i--) {
                if (update[i].next[i] != null && update[i].next[i].value == value) {
                    update[i].next[i] = update[i].next[i].next[i];
                }
            }
        }
    }

    private int randomLevel() {
        int level = 1;
        while (random.nextDouble() < PROBABILITY && level < MAX_LEVEL) {
            level++;
        }
        return level;
    }
}

```


## 从前序和中序遍历序列构建二叉树

```Java
class Solution {  
    private int[] preorder;  
    private int[] inorder;  
    private HashMap<Integer,Integer> idx_map;  
    private int pre_idx=0;  
  
    public TreeNode buildTree(int[] preorder, int[] inorder) {  
        this.preorder = preorder;  
        this.inorder = inorder;  
        idx_map = new HashMap<>();  
        int idx = 0;  
        for(Integer val : inorder){  
            idx_map.put(val,idx++);  
        }  
        return helper(0,inorder.length);  
    }  
  
    private TreeNode helper(int start,int end){  
        if(start==end){  
            return null;  
        }  
        int rootVal = preorder[pre_idx];  
        TreeNode root = new TreeNode(rootVal);  
        int index = idx_map.get(rootVal);  
        pre_idx++;  
        root.left = helper(start,index);  
        root.right = helper(index+1,end);  
        return root;  
    }  
}
```

## 从中序和后序遍历序列构建二叉树

```Java
class Solution {  
  
    private int[] inorder;  
    private int[] postorder;  
    private Map<Integer,Integer> map;  
    private int idx;  
  
    public TreeNode buildTree(int[] inorder, int[] postorder) {  
        this.inorder = inorder;  
        this.postorder = postorder;  
        this.map = new HashMap<>();  
        this.idx = inorder.length-1;  
        int idx_map = idx;  
        for(Integer val : inorder){  
            map.put(val,idx_map--);  
        }  
        return helper(0,idx+1);  
    }  
  
    private TreeNode helper(int start,int end){  
        if(start==end){  
            return null;  
        }  
        int rootVal = postorder[idx];  
        TreeNode root = new TreeNode(rootVal);  
        int index = map.get(rootVal);  
        idx--;  
        root.right = helper(start,index);  
        root.left = helper(index+1,end);  
        return root;  
    }  
}
```

## 字典树
```Java
class TrieNode {  
    // 存储节点的孩子节点  
    TrieNode[] children;  
    // 标记该节点是否为一个单词的结束  
    boolean isEndOfWord;  
  
    // 构造函数  
    public TrieNode() {  
        // 初始化孩子节点数组，大小为26，表示26个英文字母  
        children = new TrieNode[26];  
        isEndOfWord = false;  
    }  
}  
  
class Trie {  
    private TrieNode root;  
  
    // 构造函数  
    public Trie() {  
        root = new TrieNode();  
    }  
  
    // 插入一个单词到字典树中  
    public void insert(String word) {  
        TrieNode current = root;  
        // 遍历单词中的每一个字符  
        for (int i = 0; i < word.length(); i++) {  
            char ch = word.charAt(i);  
            int index = ch - 'a';  
            // 如果当前字符的节点不存在，创建一个新的节点  
            if (current.children[index] == null) {  
                current.children[index] = new TrieNode();  
            }  
            // 移动到下一个节点  
            current = current.children[index];  
        }  
        // 标记当前节点为一个单词的结束  
        current.isEndOfWord = true;  
    }  
  
    // 搜索一个单词是否在字典树中  
    public boolean search(String word) {  
        TrieNode current = root;  
        // 遍历单词中的每一个字符  
        for (int i = 0; i < word.length(); i++) {  
            char ch = word.charAt(i);  
            int index = ch - 'a';  
            // 如果当前字符的节点不存在，返回false  
            if (current.children[index] == null) {  
                return false;  
            }  
            // 移动到下一个节点  
            current = current.children[index];  
        }  
        // 检查最后一个节点是否标记为一个单词的结束  
        return current != null && current.isEndOfWord;  
    }  
  
    // 判断是否存在以给定前缀开头的单词  
    public boolean startsWith(String prefix) {  
        TrieNode current = root;  
        // 遍历前缀中的每一个字符  
        for (int i = 0; i < prefix.length(); i++) {  
            char ch = prefix.charAt(i);  
            int index = ch - 'a';  
            // 如果当前字符的节点不存在，返回false  
            if (current.children[index] == null) {  
                return false;  
            }  
            // 移动到下一个节点  
            current = current.children[index];  
        }  
        return true;  
    }  
}
```