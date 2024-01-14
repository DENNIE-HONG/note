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


```js









## 给定一个二叉树, 找到该树中两个指定节点间的最短距离

```js
const shortestDistance = function(root, p, q) {
    // 最近公共祖先
    let lowestCA = lowestCommonAncestor(root, p, q)
    // 分别求出公共祖先到两个节点的路经
    let pDis = [], qDis = []
    getPath(lowestCA, p, pDis)
    getPath(lowestCA, q, qDis)
    // 返回路径之和
    return (pDis.length + qDis.length)
}

// 最近公共祖先
const lowestCommonAncestor = function(root, p, q) {
    if(root === null || root === p || root === q) return root
    const left = lowestCommonAncestor(root.left, p, q)
    const right = lowestCommonAncestor(root.right, p, q)
    if(left === null) return right
    if(right === null) return left
    return root
}

const getPath = function(root, p, paths) {
    // 找到节点，返回 true
    if(root === p) return true
    // 当前节点加入路径中
    paths.push(root)
    let hasFound = false
    // 先找左子树
    if (root.left !== null)
        hasFound = getPath(root.left, p, paths)
    // 左子树没有找到，再找右子树
    if (!hasFound && root.right !== null)
        hasFound = getPath(root.right, p, paths)
    // 没有找到，说明不在这个节点下面，则弹出
    if (!hasFound)
        paths.pop()
    return hasFound
}
```
