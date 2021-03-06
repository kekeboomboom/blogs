## 找到二叉树中的最大搜索二叉子树

```java
/**
 * 找到二叉树中的最大搜索二叉子树
 *
 * @author keboom
 * @date 2021/5/9
 */
public class FindLargestTree {

    class ReturnType {
        public Node maxBSTHead;
        public int maxBSTSize;
        public int min;
        public int max;

        public ReturnType(Node maxBSTHead, int maxBSTSize, int min, int max) {
            this.maxBSTHead = maxBSTHead;
            this.maxBSTSize = maxBSTSize;
            this.min = min;
            this.max = max;
        }
    }


    public ReturnType process(Node X) {
        // base case：如果子树是空树
        if (X == null) {
            return new ReturnType(null, 0, Integer.MAX_VALUE, Integer.MIN_VALUE);
        }
        // 默认直接得到左树全部信息
        ReturnType lData = process(X.left);
        // 默认直接得到右树全部信息
        ReturnType rData = process(X.right);
        // 信息整合
        // 求最小值：X，左子树，右子树
        int min = Math.min(X.value, Math.min(lData.min, rData.min));
        // 求最大值：X，左子树，右子树
        int max = Math.max(X.value, Math.max(lData.max, rData.max));
        // 如果只考虑可能性一和二，也就是最大搜索二叉树是X的左子树或者右子树
        int maxBSTSize = Math.max(lData.maxBSTSize, rData.maxBSTSize);
        // 如果只考虑可能性一和二，也就是最大搜索二叉树是X的左子树或者右子树
        Node maxBSTHead = lData.maxBSTSize >= rData.maxBSTSize ? lData.maxBSTHead : rData.maxBSTHead;
        // 利用收集的信息，可以判断是否存在第三种可能
        if (lData.maxBSTHead == X.left && rData.maxBSTHead == X.right
                && X.value > lData.max && X.value < rData.min) {
            maxBSTSize = lData.maxBSTSize + rData.maxBSTSize + 1;
            maxBSTHead = X;
        }
        return new ReturnType(maxBSTHead, maxBSTSize, min, max);
    }

    public Node getMaxBST(Node head) {
        return process(head).maxBSTHead;
    }

}
```



## 判断二叉树是否为平衡二叉树

```java
/**
 * @author keboom
 * @date 2021/5/17
 */
public class IsBalanceTree {
    class ReturnType {
        public boolean isBalanced;
        public int height;

        public ReturnType(boolean isBalanced, int height) {
            this.isBalanced = isBalanced;
            this.height = height;
        }
    }

    public boolean isBalanced(Node head) {
        return process(head).isBalanced;
    }

    private ReturnType process(Node head) {
        if (head == null) {
            return new ReturnType(true, 0);
        }
        ReturnType leftData = process(head.left);
        ReturnType rightData = process(head.right);
        int height = Math.max(leftData.height, rightData.height) + 1;
        boolean isBalanced = leftData.isBalanced && rightData.isBalanced
                && Math.abs(leftData.height - rightData.height) < 2;
        return new ReturnType(isBalanced, height);
    }
}
```



## 套路

1. 分析有几种可能
2. 需要哪些信息
3. 汇总信息，构造`ReturnType`
4. 构造递归函数

