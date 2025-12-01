---
{"dg-publish":true,"permalink":"/algorithm/backtrace/"}
---


## N皇后

```Java
class Solution {  
    private int[][] chessBoard;  
    private int count;  
  
    public int totalNQueens(int n) {  
        chessBoard = new int[n][n];  
        count=0;  
        chess(chessBoard,0,n);  
        return count;  
    }  
  
    public void chess(int[][] chessBoard,int row,int n){  
        if(row>=n){  
            count++;  
            return;  
        }  
  
        for(int col=0;col<n;col++){  
            if(isSafe(chessBoard,row,col,n)){  
                chessBoard[row][col]=1;  
                chess(chessBoard,row+1,n);  
                chessBoard[row][col]=0;  
            }  
        }  
  
    }  
  
    private boolean isSafe(int[][] chessBoard,int row, int col,int n) {  
  
        for(int i=row-1,j=col-1;i>=0&&j>=0;i--,j--){  
            if(chessBoard[i][j]==1){  
                return false;  
            }  
        }  
        for(int i=row-1,j=col;i>=0;i--){  
            if(chessBoard[i][j]==1){  
                return false;  
            }  
        }  
        for(int i=row-1,j=col+1;i>=0&&j<n;i--,j++){  
            if(chessBoard[i][j]==1){  
                return false;  
            }  
        }  
        return true;  
  
    }  
  
}
```

## 组合总和

```Java
class Solution {  
  
    List<List<Integer>> res = new ArrayList<>();  
  
    public List<List<Integer>> combinationSum(int[] candidates, int target) {  
        Arrays.sort(candidates);  
        if(candidates==null||candidates.length==0||target<candidates[0]){  
            return res;  
        }  
        backtrace(0,0,candidates,new ArrayDeque<>(),target);  
        return res;  
    }  
  
    private void backtrace(int cur,int begin,int[] candidates,ArrayDeque<Integer> list,int target){  
        if(cur>target){  
            return;  
        }  
        if(cur==target){  
            res.add(new ArrayList<>(list));  
            return;  
        }  
        for(int i=begin;i<candidates.length;i++){  
            if(cur+candidates[i]>target){  
                break;  
            }  
            list.addLast(candidates[i]);  
            backtrace(cur+candidates[i],i,candidates,list,target);  
            list.removeLast();  
        }  
    }  
}
```


## 全排列

```Java
class Solution {  
    public List<List<Integer>> permute(int[] nums){  
        int len = nums.length;  
        List<List<Integer>> res = new ArrayList<>();  
        if(len==0){  
            return res;  
        }  
        Deque<Integer> path = new ArrayDeque<Integer>();  
        boolean[] used = new boolean[len];  
        dfs(nums,len,path,0,used,res);  
        return res;  
    }  
  
    private void dfs(int[] nums, int len, Deque<Integer> path, int depth, boolean[] used, List<List<Integer>> res) {  
        if(depth==len){  
            res.add(new ArrayList<>(path));  
            return;  
        }  
        for(int i=0;i<len;i++){  
            if(used[i]){  
                continue;  
            }  
            path.addLast(nums[i]);  
            used[i]=true;  
            dfs(nums, len, path, depth+1, used, res);  
            path.removeLast();  
            used[i]=false;  
        }  
    }  
}
```


## 组合

```Java
class Solution {  
  
    private List<List<Integer>> res;  
  
    public List<List<Integer>> combine(int n, int k) {  
        this.res = new ArrayList<>();  
        backtrace(n,k,1,new LinkedList<Integer>());  
        return res;  
    }  
  
    private void backtrace(int n,int k,int cur,LinkedList<Integer> path){  
        if(path.size() == k){  
            res.add(new ArrayList<>(path));  
            return;  
        }  
        for(int i=cur;i<=n;i++){  
            path.add(i);  
            backtrace(n,k,i+1,path);  
            path.removeLast();  
        }  
    }  
}
```