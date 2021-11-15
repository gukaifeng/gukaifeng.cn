---
title: 'LeetCode: 98. 验证二叉搜索树'
mathjax: true
reward: false
date: 2019-04-26 13:06:44
updated: 2019-04-26 13:06:44
tags: [LeetCode, 树, 深度优先搜索]
categories: [算法题解]
toc: true
---

原题链接（英文）：https://leetcode.com/problems/validate-binary-search-tree/
原题链接（中文）：https://leetcode-cn.com/problems/validate-binary-search-tree/

这个题主要有两种解法：
一、利用二叉搜索树的性质：中序遍历序列严格单调递增。
二、递归法逐一判断每个结点（子树）是否满足二叉搜索树的特征。
<!--more-->
方法一：

我们只需要对此二叉树进行一次中序遍历，查看其中序遍历序列是否是严格单调递增的即可。

我们并不需要将这个树的中序遍历序列记录下来，也可能不需要遍历完整棵树。当我们遍历到某结点时，只需要比较一下看看这个结点的值是否大于上一个遍历的结点的值。因此我们只需要记录上一个遍历的结点的值 $inorder\\_last\\_val$，并每当遍历过一个结点后，更新该值。如果遍历到某个结点时，发现当前结点的值比 $inorder\\_last\\_val$ 小，说明这不是一个二叉搜索树，直接中断遍历并返回 $false$；如果遍历完整棵树都没有出现这种情况，那么这就是一个二叉搜索树，返回 $true$。

需要特殊处理第一个结点，有两种方法：一是令初始的 $inorder\\_last\\_val=LONG\\_MIN$，即初始值足够小（保证一定比中序遍历到的第一个结点的值要小）；二是设置变量 $first\\_node\\_flag$ 用来记录当前的结点是不是第一个结点。下面的代码使用了第二种方法。

```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */


int inorder_traveral(struct TreeNode* root, int* first_flag, int* last_val)
{
    if (root == NULL) return 1;
    
    int judge_left, judge_current, judge_right;
    
    judge_left = inorder_traveral(root->left, first_flag, last_val);
    
    if (*first_flag == 1)
    {
        *first_flag = 0;
        *last_val = root->val;
        judge_current = 1;
    }
    else
    {
        judge_current = root->val > *last_val;
        *last_val = root->val;
    }

    judge_right = inorder_traveral(root->right, first_flag, last_val);
    
    return judge_left && judge_current && judge_right;
}

bool isValidBST(struct TreeNode* root)
{
    int inorder_last_val;
    int first_node_flag = 1;
    
    return inorder_traveral(root, &first_node_flag, &inorder_last_val);
}
```


方法二：

判断二叉树是否满足一下几个特征：
1\. 结点的左子树只包含小于当前结点的数。
2\. 结点的右子树只包含大于当前结点的数。
3\. 所有左子树和右子树自身必须也是二叉搜索树。

当然我们也不需要直接去按着特征比较，那样太过繁琐。
我们采用分块的策略，设此树的所有结点值应大于 $left\\_val$，小于 $right\\_val$，当前结点值为 $root$->$val$。
那么以根结点为例，该结点值应满足 $left\\_val < root$->$val < right\\_val$，其左子树结点的值 $root$->$left$->$val$ 应满足 $left\\_val < root$->$left$->$val < root$->$val$，其右子树结点的值 $root$->$right$->$val$ 应满足 $root$->$val < root$->$right$->$val < right\\_val$。

递归下去，若所有结点均满足上述条件，则该树为二叉搜索树，反之亦然。其中关于 $left\\_val$ 和 $right\\_val$ 初始值的选择，只需要前者比所有结点的值小，后者比所有结点的值大就行。因为题目结点的值为 int 类型，所以下面的代码为了方便就取了 long long 类型的最大和最小值。

下面两个代码基本一样，第二个只是把那些判断条件写成了一行，zb用哈。

```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */


bool judge_valid_bst(struct TreeNode* root, long long left_val, long long right_val)
{
    if (root == NULL)
        return true;
    if (root->val <= left_val || root->val >= right_val)
        return false;
    
    return judge_valid_bst(root->left, left_val, root->val) && judge_valid_bst(root->right, root->val, right_val);
}

bool isValidBST(struct TreeNode* root)
{
    return judge_valid_bst(root, LONG_MIN, LONG_MAX);
}
```
```C
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
bool judge_valid_bst(struct TreeNode* root, long long left_val, long long right_val)
{
    return root ? (root->val > left_val && root->val < right_val ? judge_valid_bst(root->left, left_val, root->val) && judge_valid_bst(root->right, root->val, right_val) : false) : true;
}
bool isValidBST(struct TreeNode* root) {
    return judge_valid_bst(root, LONG_MIN, LONG_MAX);
}
```