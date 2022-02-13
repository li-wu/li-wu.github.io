---
title: 二叉树的遍历（前序遍历，中序遍历和后序遍历）
date: 2013-12-29 11:00:36 +0800
categories: algorithm
---

二叉树是数据结构中一种非常重要的数据结构，对于二叉树的遍历也在面试以及实际应用之中非常广泛，下面总结一下对于二叉树的三种遍历方法的递归和非递归实现。

## 数据结构
这里是采用Java来实现的，首先给出树节点的数据结构：
```
public class TreeNode {
      int val;
      TreeNode left;
      TreeNode right;
      TreeNode(int x) { val = x; }
}
```
以下实现方法都是返回一个遍历的结果序列。
## 先序遍历
先序遍历是先遍历根节点，然后先序方式遍历左子树，最后先序遍历右子树。先序遍历的递归实现：
```
    public ArrayList<Integer> preorderTraversal(TreeNode root) {
        ArrayList<Integer> result=new ArrayList<Integer>();
        if(root==null){
        	return result;
        }
        result.add(root.val);
        if(root.left!=null){
        	result.addAll(preorderTraversal(root.left));
        }
        if(root.right!=null){
        	result.addAll(preorderTraversal(root.right));
        }
        return result;
    }
```
先序遍历的非递归实现：先序遍历的非递归实现需要借助一个栈结构来保存遍历过程中的信息。先将根节点入栈，如果栈非空，取出栈顶元素，加入结果序列，然后将右子树入栈，将左子树入栈（栈的特点是先进后出，所以要先将右子树入栈）。
```
    public ArrayList<Integer> preorderTraversalIterative(TreeNode root) {
    	ArrayList<Integer> result=new ArrayList<Integer>();
    	Stack<TreeNode> s=new Stack<TreeNode>();
    	if(root==null){
    		return result;
    	}
    	s.push(root);
    	while(!s.isEmpty()){
    		TreeNode cur=s.pop();
    		result.add(cur.val);
    		if(cur.right!=null){
    			s.push(cur.right);
    		}
    		if(cur.left!=null){
    			s.push(cur.left);
    		}
    	}
    	return result;
    }
```
## 中序遍历
递归实现：
```
    public ArrayList<Integer> inorderTraversal(TreeNode root) {
    	ArrayList<Integer> result=new ArrayList<Integer>();
    	if(root==null){
    		return result;
    	}
    	if(root.left!=null){
    		result.addAll(inorderTraversal(root.left));
    	}
    	result.add(root.val);
    	if(root.right!=null){
    		result.addAll(inorderTraversal(root.right));
    	}
        return result;
    }
```
中序遍历的非递归实现:先寻找最左边的点，把经过的节点都入栈，第一个被弹出来的是最左节点，访问其右子树，对右子树也像之前一样遍历，整个过程和递归一样。
```
	public ArrayList<Integer> inorderTraversalIterative(TreeNode root) {
		ArrayList<Integer> result=new ArrayList<Integer>();
		Stack<TreeNode> s=new Stack<TreeNode>();
		while(root!=null||!s.isEmpty()){
			while(root!=null){
				s.push(root);
				root=root.left;
			}
			if(!s.isEmpty()){
				root=s.pop();
				result.add(root.val);
				root=root.right;
			}
		}
		return result;
	}
```
## 后序遍历
递归实现：
```
    public ArrayList<Integer> postTraversal(TreeNode root) {
    	ArrayList<Integer> result=new ArrayList<Integer>();
    	if(root==null){
    		return result;
    	}
    	if(root.left!=null){
    		result.addAll(postTraversal(root.left));
    	}
    	if(root.right!=null){
    		result.addAll(postTraversal(root.right));
    	}
		result.add(root.val);
       return result;
    }
```
后序遍历的非递归实现是想对要复杂一点点，如果不更改之前的树节点的数据结构，那么需要一个额外的节点来记录当前访问节点的子节点是否被访问。
```
    public ArrayList<Integer> postorderTraversalIterative(TreeNode root) {
    	ArrayList<Integer> result=new ArrayList<Integer>();
    	Stack<TreeNode> s=new Stack<TreeNode>();
    	if(root==null){
    		return result;
    	}
    	TreeNode pre=null;
    	TreeNode cur;
    	s.push(root);
    	while(!s.isEmpty()){
    		cur=s.peek();
    		//if the current node has no children or the children nodes have been visited
    		if((cur.left==null&&cur.right==null)||(pre!=null&&(pre==cur.left||pre==cur.right))){
    			result.add(cur.val);
    			s.pop();
    			pre=cur;
    		}else{
    			if(cur.right!=null){
    				s.push(cur.right);
    			}
    			if(cur.left!=null){
    				s.push(cur.left);
    			}
    		}
    	}
        return result;
    }
```








    
