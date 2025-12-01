---
{"dg-publish":true,"permalink":"/algorithm/math-and-bfs-and-dfs/"}
---


## [基本计算器](https://leetcode.cn/problems/basic-calculator/)
给你一个字符串表达式 `s` ，请你实现一个基本计算器来计算并返回它的值。
**题解：**
```Java
class Solution {  
    public int calculate(String s) {  
        Deque<Integer> stack = new ArrayDeque<>();  
        stack.push(1);  
        int sign = 1;  
        int ret = 0;  
        int n = s.length();  
        int i=0;  
        while(i<n){  
            char ch = s.charAt(i);  
            if(ch == ' '){  
                i++;  
            }else if(ch == '+'){  
                sign = stack.peek();  
                i++;  
            }else if(ch == '-'){  
                sign = -stack.peek();  
                i++;  
            }else if(ch == '('){  
                stack.push(sign);  
                i++;  
            }else if(ch == ')'){  
                stack.pop();  
                i++;  
            }else{  
                long num = 0;  
                while(i<n && Character.isDigit(s.charAt(i))){  
                    num = num*10+s.charAt(i)-'0';  
                    i++;  
                }  
                ret += sign*num;  
            }  
        }  
        return ret;  
    }  
}
```

## [位1的个数](https://leetcode.cn/problems/number-of-1-bits/)


```Java
public class Solution {  
    // you need to treat n as an unsigned value  
    public int hammingWeight(int n) {  
        int ret = 0;  
        while(n!=0){  
            n &= n-1;  
            ret++;  
        }  
        return ret;  
    }  
}
```


## [单词接龙](https://leetcode.cn/problems/word-ladder/)

**BFS**
```Java
class Solution {  
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {  
        HashSet<String> set = new HashSet<>(wordList);  
        if(set.size()==0 || !set.contains(endWord)){  
            return 0;  
        }  
        set.remove(beginWord);  
  
        //队列  
        Queue<String> queue = new LinkedList<>();  
        queue.offer(beginWord);  
        //记录已访问节点  
        Set<String> visited = new HashSet<>();  
        visited.add(beginWord);  
  
        //结果  
        int step = 1;  
        while(!queue.isEmpty()){  
            int size = queue.size();  
            for(int i=0;i<size;i++){  
                String str = queue.poll();  
                if(changeOneLetter2End(str,endWord,queue,visited,set)){  
                    return step+1;  
                }  
            }  
            step++;  
        }  
        return 0;  
    }  
  
    private boolean changeOneLetter2End(String curWord,String endWord,Queue<String> queue,Set<String> visited,HashSet<String> set){  
        char[] charArray = curWord.toCharArray();  
        for(int i=0;i<endWord.length();i++){  
            char originChar = charArray[i];  
            for(char ch='a';ch<='z';ch++){  
                if(ch == originChar){  
                    continue;  
                }  
                charArray[i] = ch;  
                String str = String.valueOf(charArray);  
                if(set.contains(str)){  
                    if(str.equals(endWord)){  
                        return true;  
                    }  
                    if(!visited.contains(str)){  
                        queue.offer(str);  
                        visited.add(str);  
                    }  
                }  
            }  
            charArray[i]=originChar;  
        }  
        return false;  
    }  
}
```

## [插入区间](https://leetcode.cn/problems/insert-interval/)

```Java
class Solution {  
    public int[][] insert(int[][] intervals, int[] newInterval) {  
        int left = newInterval[0];  
        int right = newInterval[1];  
        boolean placed = false;  
        List<int[]> ansList = new ArrayList<>();  
        for(int[] interval : intervals){  
            if(interval[0] > right){  
                if(!placed){  
                    ansList.add(new int[]{left,right});  
                    placed = true;  
                }  
                ansList.add(interval);  
            }else if(interval[1]<left){  
                ansList.add(interval);  
            }else{  
                left = Math.min(left,interval[0]);  
                right = Math.max(right,interval[1]);  
            }  
        }  
        if(!placed){  
            ansList.add(new int[]{left,right});  
        }  
        int[][] ans = new int[ansList.size()][2];  
        for(int i =0;i<ansList.size();i++){  
            ans[i] = ansList.get(i);  
        }  
        return ans;  
    }  
}
```

## [单词搜索 II](https://leetcode.cn/problems/word-search-ii/)

**DFS+字典树**
```Java
class Solution {  
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};  
    public List<String> findWords(char[][] board, String[] words) {  
        Trie trie = new Trie();  
        for(String word : words){  
            trie.insert(word);  
        }  
        Set<String> ans = new HashSet<>();  
        for(int i=0;i<board.length;i++){  
            for(int j=0;j<board[0].length;j++){  
                dfs(board,trie,i,j,ans);  
            }  
        }  
        return new ArrayList<String>(ans);  
    }  
  
    private void dfs(char[][] board,Trie trie,int i,int j,Set<String> set){  
        if(!trie.children.containsKey(board[i][j])){  
            return;  
        }  
        char ch = board[i][j];  
        trie = trie.children.get(ch);  
        if(!"".equals(trie.word)){  
            set.add(trie.word);  
        }  
        board[i][j] = '#';  
        for(int[] dir : dirs){  
            int row = i+dir[0];  
            int col = j+dir[1];  
            if(row>=0 && row < board.length && col >= 0 && col < board[0].length){  
                dfs(board,trie,row,col,set);  
            }  
        }  
        board[i][j] = ch;  
    }  
}  
  
class Trie{  
    String word;  
    Map<Character,Trie> children;  
    boolean isWord;  
  
    public Trie(){  
        this.word="";  
        this.children = new HashMap<Character,Trie>();  
    }  
  
    public void insert(String word){  
        Trie cur = this;  
        for(int i=0;i<word.length();i++){  
            char ch = word.charAt(i);  
            if(!cur.children.containsKey(ch)){  
                cur.children.put(ch,new Trie());  
            }  
            cur = cur.children.get(ch);  
        }  
        cur.word = word;  
    }  
}
```