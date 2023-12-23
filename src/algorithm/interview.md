# 常问leetcode 算法题目


## 从前序与中序遍历序列构造二叉树
前序遍历：根左右：[根，左边子树的前序，右边子树的前序]
中序遍历：左根右，[左边子树的中序，根，右边子树的中序]

给定两个整数数组 preorder 和 inorder ，其中 preorder 是二叉树的先序遍历， inorder 是同一棵树的中序遍历，请构造二叉树并返回其根节点。

输入: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
输出: [3,9,20,null,null,15,7]

```js
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {number[]} preorder
 * @param {number[]} inorder
 * @return {TreeNode}
 */
 // 前序：根左右
 // 中序：左根右
var buildTree = function(preorder, inorder) {
    let root = {};
    function loop(root, preorder, inorder) {
        const rootVal = preorder[0];
        root.val = rooVal;
        const rootIndex = inorder.findIndex((v) => v === rootVal);
        // 左边的中序
        const leftInorder = inorder.slice(0, rootIndex);
        const rightInorder = inorder.slice(rootIndex + 1);
        // 左边的前序
        const leftPreorder = preorder(1, leftInorder.length +1);
        const rightPreorder = preorder(leftInorder.length +1);
        if (!leftInorder.length) {
            root.left = null;
        } else {
            const left = {};
            root.left = left;
            loop(root.left, leftPreorder, leftInorder);
        }
        if (!rightInorder.length) {
            root.right = null;
        } else {
            const right = {};
            root.right = right;
            loop(root.right, rightPreorder, rightInorder);
        }

    }
    loop(root, preorder, inorder);
    return root;
}
```




