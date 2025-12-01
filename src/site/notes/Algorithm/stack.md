---
{"dg-publish":true,"permalink":"/algorithm/stack/"}
---


## [简化路径](https://leetcode.cn/problems/simplify-path/)

```Java
class Solution {  
    public String simplifyPath(String path) {  
        String[] names = path.split("/");  
        Deque<String> stack = new ArrayDeque<>();  
        for(String name : names){  
            if("..".equals(name)){  
                if(!stack.isEmpty()){  
                    stack.pollLast();  
                }  
            }else if(name.length() > 0 && !".".equals(name)){  
                stack.offerLast(name);  
            }  
        }  
        StringBuffer sb = new StringBuffer();  
        if(stack.isEmpty()){  
            sb.append("/");  
        }else{  
            while(!stack.isEmpty()){  
                sb.append("/");  
                sb.append(stack.pollFirst());  
            }  
        }  
        return sb.toString();  
    }  
}
```


## [接雨水](https://leetcode.cn/problems/trapping-rain-water/)


```Java
class Solution {  
    public int trap(int[] height) {  
        if (height == null) {  
            return 0;  
        }  
        int n = height.length;  
        int[] left = new int[n];  
        int[] right = new int[n];  
        for(int i=1;i<n;i++){  
            left[i] = Math.max(left[i-1],height[i-1]);  
        }  
        for(int i=n-2;i>=0;i--){  
            right[i] = Math.max(right[i+1],height[i+1]);  
        }  
        int sum = 0;  
        for(int i=1;i<n;i++){  
            int min = Math.min(left[i],right[i]);  
            if(min > height[i]){  
                sum += (min - height[i]);  
            }  
        }  
        return sum;  
    }  
}
```


