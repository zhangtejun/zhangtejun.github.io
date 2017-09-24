---
layout: post
title:  "算法"
date:   2017-05-22 15:22:43
author: zhangtj
categories: zhangtj
---
##### 算法的定义：算法是解决特定问题求解步骤的描述，在计算机中为指令的有限序列，并且每条指令表示一个或多个操作。
##### 算法的特性：有穷性，确定性，可行性，输入，输出
##### 算法设计的要求：正确性，可读性，健壮性，高效率和低存储量的需求。
##### 函数的渐进增长：给定2个函数f(n)和g(0),如果存在一个整数N，使得对于所有的n>N，f(n)总是比g(n)大,那么我们就说f(n)的增长渐近快于g(n)。
#### 算法时间复杂度定义
##### 在进行算法分析时，语句总的执行次数T(n)是关于问题规模n的函数，进而分析T(n)随n的变化情况并确定T(n)的数量级。算法的时间复杂度，也就是算法的时间复杂度，记作：T(0)=O(f(n))。它表示随问题规模n的增大，算法执行时间的增长率和f(n)的增长率相同，
称作算法的渐进时间复杂度，简称为时间复杂度。
其中f(n)是问题规模n的某个函数。
这样用写大O()来体现的时间复杂度的记法，我们称之为大O记法。
一般情况下，随n增大，T(0)增长最慢的算法为最优算法。
显然由时间复杂度的定义可知，算法的时间复杂度分别为O(1),O(n),O(n²)，用非官方的名称来叫它们，O(1)常数阶，O(n)线性阶，O(n²)平方阶，当然还有一些其他的阶。
#### 推导大O阶的方法
1. 用常数1取代运行时间中所有的加法常数。
2. 在修改后的运行次数函数中，只保留最高阶项。
3. 如果最高阶项存在且不是1，则去除与这个项相乘的常熟。
4. 得到的结果就是大O阶。
常用的时间复杂度所耗费的时间从小到大依次是：
O(1)<O(logn)<O(n)<O(nlogn)<O(n²)<O(n³)<O(2^n)<O(n!)<O(n^n)
#### 算法的空间复杂度
通过计算算法所需的存储空间实现，算法的空间复杂度的计算公式：S(n)=O(f(n)),其中n为问题的规模，f(n)为语句关于n所占存储空间的函数。

### 线性表(List)
0个或多个数据元素的有限队列。
* 首先是一个序列，即元素间是有顺序的，如果存在多个元素，则第一个无前驱，最后一个无后继，其他元素仅有一个前驱和一个后继。
##### 两种存储结构
* 顺序存储结构（存取时间复杂度均为O(1),通常把具有这类特性的存储结构称为随机存储结构）  用一段地址连续的储存单元依次存储线性表的数据元素。
  * 查找元素 根据已知元素的位置i,可以根据下标直接返回。
  * 插入元素 从最后元素开始遍历到第i位置，将其后面的元素向后移动一位，将新元素插入位置i，表长加1.
  * 删除元素 从删除元素位置遍历到最后一个元素，将他们向前移动一位，表长减1。
  * 插入和删除时，时间复杂度为O(n)。最后情况，插入和删除位置为最后一个，此时为O(1)。最坏情况，插入和删除位置为第一个，此时为O(n)。
  平均时间复杂度O((n-i)/2) = O(n)。

**优点：** 无需为表中元素之间的逻辑关系而增加额外的存储空间。可以快速的存取表中任何位置的元素。

**缺点:**  插入和删除需要移动大量元素。当表变化较大时，难以确认存储空间。造成存储空间碎片。

* 链式存储结构  用一组任意的存储单元来存储元素。这组存储单元可以是连续的，也可以是不连续的。可以看出在链式存储结构中，除了要存储数据元素外，还需要存储后继元素的位置。
把存储数据元素的域称为数据域，把存储后继结点位置的域称为指针域。由这2部分信息组成的数据元素a的存储映像，称为结点（Node）。

n个结点（元素的存储映像）链接成一个链表，即线性表的链式存储结构，该Node只存储了一个指针域，故称为单链表。
    
 * 查找元素 和遍历的位置有关，最坏情况为O(n)。
 * 插入和删除元素 可以看成2步，①遍历查找i的位置，②插入和删除。即第一次查找时，时间复杂度为O(n),之后仅需要通过复制移动指针，时间复杂度均为O(1).
    显然对于插入和删除频繁的操作，单链表效率比顺序存在结构高。
    
    在单链表中，查找下一个结点的时间复杂度为O(1),如何查找上一个结点，又需要重新遍历。为解决这问题设计出了双向链表，即双向链表中有2个指针域。

### 栈(Stack)和队列(Queue)
栈 (Stack)是一种后进先出(last in first off，LIFO)的数据结构，而队列(Queue)则是一种先进先出 (fisrt in first out，FIFO)的结构。

##### Stack的实现
* 数组
  ```java
  import java.util.ArrayList;
  
  /**
   * Created by 10539 on 2017/9/23.
   */
  public class StackByArray {
      public static void main(String[] args) {
          StackByArray stackByArray = new StackByArray(2);
          stackByArray.Push(Integer.valueOf("1"));
          stackByArray.Push(Integer.valueOf("2"));
          stackByArray.Push("3");
          for (int i = 0;i<stackByArray.num;i++){
              System.out.print(stackByArray.arrayStack[i]);
          }
          stackByArray.Pop();
          System.out.println(stackByArray.arrayStack.length);
          for (int i = 0;i<stackByArray.num;i++){
              System.out.print(stackByArray.arrayStack[i]);
          }
      }
  
      Object[] arrayStack;
      private int num = 0;//stack元素总数
  
      public StackByArray(int  capacity)
      {
          arrayStack = new Object[capacity];
      }
  
  
      public void Push(Object value)
      {
          //检查数组是否已占满，是则扩容
          if (num == arrayStack.length){
              Resize(2 * arrayStack.length);
          }
          arrayStack[num++] = value;
      }
  
      /**
       * Push的时候，当元素的个数达到数组的Capacity的时候，我们开辟2倍于当前元素的新数组，然后将原数组中的元素拷贝到新数组中。
       * Pop的时候，当元素的个数小于当前容量的1/4的时候，我们将原数组的大小容量减少1/2。
       * @param capacity
       */
      private void Resize(int capacity) {
          //新建一个数组
          Object[] temp = new Object[capacity];
          //将原数组的元素插入新数组
          for (int i = 0 ;i<arrayStack.length;i++){
              temp[i] = arrayStack[i];
          }
          //将新数组返回给arrayStack
          arrayStack = temp;
      }
      public Object Pop()
      {
          //出栈最近新增的元素
          Object temp = arrayStack[--num];
          arrayStack[num] = null;
          if (num > 0 && num == arrayStack.length / 4) Resize(arrayStack.length / 2);
          return temp;
      }
  }

  ```
  
  
* 链表
  * 定义一个内部类来保存每个链表的节点。
  
  ```java
  import java.util.HashMap;
  
  /**
   * Created by 10539 on 2017/9/23.
   */
  public class Stack<T> {
      public static void main(String[] args) {
          Stack stack = new Stack();
          stack.Push(Integer.valueOf("1"));
          stack.Push(Integer.valueOf("2"));
          stack.Push(new HashMap());
          System.out.println(stack.toString());
          stack.Pop();
          System.out.println(stack.toString());
      }
  
      private Node firstNode = null;
      private int num = 0;//stack元素总数
  
      /**
       * 向栈顶压入一个元素
       * @param node
       */
      void Push(T node)
      {
          //1.首先保存原先的位于栈顶的元素
          Node stackTop = firstNode;
          //2.新建一个新的栈顶元素
          firstNode = new Node();
          firstNode.setValue(node);
          //3.然后将该元素的下一个指向原先的栈顶元素,即新元素成为栈顶元素。
          firstNode.setNext(stackTop);
          num++;
      }
  
      /**
       * 移除并返回最近添加的元素
       * @return
       */
      T Pop()
      {
          //1. 保存栈顶元素的值
          T item = (T) firstNode.getValue();
          //2. 删除栈顶元素
          firstNode = firstNode.getNext();
          num--;
          return item;
      }
  
      @Override
      public String toString() {
          return firstNode.toString();
      }
  
      static class Node<T>{//保存链表的节点信息，内部类。
           Node next;
           T value;
  
          public Node getNext() {
              return next;
          }
  
          public void setNext(Node next) {
              this.next = next;
          }
  
          public void setValue(T value) {
              this.value = value;
          }
  
          public T getValue() {
              return value;
          }
  
          @Override
          public String toString() {
              return "next: "+getNext() + " value: "+getValue();
          }
      }
  }

  ```
  
##### 队列(Queue)的实现

Queue是一种先进先出的数据结构，和Stack一样，他也有链表和数组两种实现。[参考链接 ](http://blog.jobbole.com/79267/)
* 数组 和Stack的实现方式不同，在Queue中，我们定义了head和tail来记录头元素和尾元素。当enqueue的时候，tial加1，将元素放在尾部，当dequeue的时候，head减1，并返回。
 
 ```java
 public void Enqueue(T _item)
 {
     if ((head - tail + 1) == item.Length) Resize(2 * item.Length);
     item[tail++] = _item;
 }
  
 public T Dequeue()
 {
     T temp = item[--head];
     item[head] = default(T);
     if (head > 0 && (tail - head + 1) == item.Length / 4) Resize(item.Length / 2);
     return temp;
 }
  
 private void Resize(int capacity)
 {
     T[] temp = new T[capacity];
     int index = 0;
     for (int i = head; i < tail; i++)
     {
         temp[++index] = item[i];
     }
     item = temp;
 }
 ```
* 链表

```java
/**
 * Created by 10539 on 2017/9/23.
 */
public class queue<T> {
    private Node firstNode = null;//队头
    private Node lastNode = null;//队尾
    private int num = 0;//queue元素总数


    /**
     * 往队列中添加一个新的元素
     * @return 返回链表中的第一个元素
     */
    public T Dequeue()
    {
        T temp = (T) firstNode.getValue();
        firstNode = firstNode.getNext();
        num--;
        if (IsEmpety())
            lastNode = null;
        return temp;
    }

    /**
     * 移除队列中最早添加的元素
     * @param value
     */
    public void Enqueue(T value)
    {
        //保存队尾元素
        Node oldLast = lastNode;
        //在链表的末尾增加新的元素
        lastNode = new Node();
        lastNode.setValue(value);
        if (IsEmpety())
        {
            firstNode = lastNode;
        }
        else
        {
            oldLast.setNext(lastNode);
        }
        num++;
    }


    private boolean IsEmpety() {
        if (num == 0){
            return true;
        }
        return  false;
    }

    @Override
    public String toString() {
        return firstNode.toString();
    }

    static class Node<T>{//保存链表的节点信息，内部类。
        Node next;
        T value;

        public Node getNext() {
            return next;
        }

        public void setNext(Node next) {
            this.next = next;
        }

        public void setValue(T value) {
            this.value = value;
        }

        public T getValue() {
            return value;
        }

        @Override
        public String toString() {
            return "next: "+getNext() + " value: "+getValue();
        }
    }
}

```

### 树(Tree)
##### 树是n(n>>0)个结点的有限集。
n=0时称为空树。在任意一棵非空树中：(1)有且仅有一个特定的称为根(Root)的结点；
(2)当n>1时，其余结点可分为m(m>0)个互不相交的有限集T1,T2,...,Tm,其中每个集合本身又是一棵树，并且称为根的子树(SubTree)。
结点拥有子树数称为结点的度(degree)。树的度是树内各结点的度的最大值。
##### 二叉树 Binary Tree
在计算机科学中，二叉树是每个结点最多有两个子树的有序树。通常子树的根被称作“左子树”（left subtree）和“右子树”（right subtree）。
左/右斜树：所有结点都只有左/右子树的二叉树。
满二叉树：所有结点都有左右子树，并且所有叶子都在同一层上。
完全二叉树：对一棵右n个结点的二叉树按层序编号，如果编号为i(0<=i<=n)的结点
于同样深度的满二叉树中编号为i的结点在二叉树中的位置完全相同，则这棵二叉树称为完全二叉树。

#####二叉查找树Binary Search Tree，BST
二叉查找树（Binary Search Tree），也称有序二叉树（ordered binary tree）,排序二叉树（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

1. 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2. 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
3. 任意节点的左、右子树也分别为二叉查找树。
4. 没有键值相等的节点（no duplicate nodes）。

* 查找
1. 迭代的方式

```java
public class  BinarySearchTree<BSTKey extends Comparable<BSTKey>,BSTValue>{

    public static void main(String[] args) {

//        BinarySearchTree bst = new BinarySearchTree();
//        bst.put("12",12);
//        bst.put(new aaa(),14);
//        bst.put("13",13);
//        bst.getKey("12");
        }


    private Node root; //root of BST

    /**
     * 查找key
     * @param key
     * @return Returns the value associated with the given key.
     */
    public  BSTValue getKey(BSTKey key){
        return get(root, key);
    }
    private BSTValue get(Node x, BSTKey key) {
        if (key == null) throw new IllegalArgumentException("called get() with a null key");
        if (x == null) return null;
        int cmp = key.compareTo(x.key);//比较查找key和root大小
        if(cmp < 0) return get(x.left, key);//小于root,则在左子树查找
        else if (cmp > 0) return get(x.right, key);//大于root,则在左子树查找
        else  return x.value;
    }

    public void put(BSTKey key, BSTValue val) {
        //如果key为null
        if (key == null) throw new IllegalArgumentException("called put() with a null key");
        if (val == null) {
//            delete(key);
            return;
        }
        root = put(root, key, val);
//        assert check();//Check integrity of BST data structure.
    }
    private Node put(Node x, BSTKey key, BSTValue val) {
        //如果树为空，则新建一个
        if (x == null) return new Node(key,val,1);
        int cmp = key.compareTo(x.key);//比较查找key和root大小
        if(cmp < 0) x.left = put(x.left,key,val);//小于root,则在左子树插入
        else if (cmp > 0) x.right = put(x.right, key, val);
        else  x.value   = val;
        x.size = 1 + size(x.left) + size(x.right);
        return x;
    }

    //内部类Node
    private class Node {
        //左右节点的Node
        public Node left,right;

        //一个用于排序的Key
        public  BSTKey key;

        //该节点包含的值Value
        public  BSTValue value;

        //子节点个数
        public int size;//number of nodes in subtree

       Node(BSTKey key,BSTValue value ,int size){
            this.key = key;
           this.value = value;
           this.size = size;
       }

    }

    /**
     * Initializes an empty symbol table.
     */
    public BinarySearchTree() {
    }

    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * Returns the number of key-value pairs in this symbol table.
     * @return
     */
    public int size() {
        return size(root);
    }

    /**
     * return number of key-value pairs in BinarySearchTree rooted at x
     * @param x
     * @return
     */
    private int size(Node x) {
        if (x == null) return 0;
        else return x.size;
    }
}

```

2. 递归的方式

```java
    private BSTValue get2Value(Node root, BSTKey key) {
        if (key == null) throw new IllegalArgumentException("called get() with a null key");
        if (root == null) return null;
        Node node = root;
        while (node != null){
            if (key.compareTo(node.key) > 0){
                node = node.right;
            }else if(key.compareTo(node.key) < 0){
                node = node.left;
            }else {
                return node.value;
            }
        }
        return node.value;
    }
```

[http://algs4.cs.princeton.edu/32bst/BST.java.html](http://algs4.cs.princeton.edu/32bst/BST.java.html)

它和二分查找一样，插入和查找的时间复杂度均为lgN，但是在最坏的情况下仍然会有N的时间复杂度。原因在于插入和删除元素的时候，树没有保持平衡

##### B树
**树**是由n(n>=0)个结点组成的有限集合，其中当n=0时，它是一颗空树，空树是树的特例。

一棵m阶B树(balanced tree of order m)是一棵平衡的m路搜索树。它或者是空树，或者是满足下列性质的树：
* 根结点至少有两个子女；
* 每个非根节点所包含的关键字个数 j 满足：┌m/2┐ - 1 <= j <= m - 1；(┌m/2┐ 向上取整)
* 除根结点以外的所有结点（不包括叶子结点）的度数正好是关键字总数加1，故内部子树个数 k 满足：┌m/2┐ <= k <= m ；
* 所有的叶子结点都位于同一层。

**树的“阶”**定义为一个节点拥有子节点数目的最大值。 
对于一棵m阶B-tree，每个结点至多可以拥有m个子结点。各结点的关键字和可以拥有的子结点数都有限制，
规定m阶B-tree中，根结点至少有2个子结点，除非根结点为叶子节点，相应的，根结点中关键字的个数为`1~m-1`，比节点数目少一个；
非根结点至少有`[m/2]`（`[]`，向上取整）个子结点，相应的，关键字个数为`[m/2]-1~m-1`。

概括来说是一个节点可以拥有多于2个子节点的二叉查找树。
