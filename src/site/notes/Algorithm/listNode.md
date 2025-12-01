---
{"dg-publish":true,"permalink":"/algorithm/list-node/"}
---



## [ K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)

```Java
class Solution {  
    public ListNode reverseKGroup(ListNode head, int k) {  
        if(head==null){  
            return null;  
        }  
        return helper(head,k);  
    }  
  
  
    private ListNode helper(ListNode head,int k){  
        if(head == null){  
            return null;  
        }  
        ListNode start = head;  
        ListNode end = head;  
        int idx = 1;  
        for(;idx<k;idx++){  
            if(end == null){  
                break;  
            }  
            end = end.next;  
        }  
        if(idx != k || end == null){  
            return start;  
        }  
        ListNode node = end == null ? null : end.next;  
        end.next = null;  
        start = reverse(start);  
        start.next = helper(node,k);  
        return end;  
    }  
  
    private ListNode reverse(ListNode node){  
        ListNode pre = node;  
        ListNode next = node.next;  
        ListNode tmp = next;  
        while(next != null){  
            tmp = next.next;  
            next.next = pre;  
            pre = next;  
            next = tmp;  
        }  
        return node;  
    }  
  
}
```


### [反转链表 II](https://leetcode.cn/problems/reverse-linked-list-ii/)

```Java
class Solution {  
    public ListNode reverseBetween(ListNode head, int m, int n) {  
        ListNode dummy = new ListNode(-1);  
        dummy.next = head;  
        ListNode pre = dummy;  
        for(int i=0;i<m-1;i++){  
            pre = pre.next;  
        }  
        ListNode rightNode = pre;  
        for(int i=0;i<n-m+1;i++){  
            rightNode = rightNode.next;  
        }  
        ListNode leftNode = pre.next;  
        ListNode cur = rightNode.next;  
  
        pre.next = null;  
        rightNode.next = null;  
        reverse(leftNode);  
        pre.next = rightNode;  
        leftNode.next = cur;  
        return dummy.next;  
    }  
  
    private void reverse(ListNode node){  
        ListNode pre = null;  
        ListNode cur = node;  
        while(cur!=null){  
            ListNode tmp = cur.next;  
            cur.next = pre;  
            pre = cur;  
            cur = tmp;  
        }  
    }  
}
```


###  [删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)
```Java
class Solution {  
    public ListNode removeNthFromEnd(ListNode head, int n) {  
        if(head==null){  
            return null;  
        }  
        ListNode dummy = new ListNode(0,head);  
        ListNode first = head;  
        ListNode sec = dummy;  
        for(int i=0;i<n;i++){  
            first = first.next;  
        }  
        while(first != null){  
            first = first.next;  
            sec = sec.next;  
        }  
        sec.next = sec.next.next;  
        return dummy.next;  
    }  
}
```