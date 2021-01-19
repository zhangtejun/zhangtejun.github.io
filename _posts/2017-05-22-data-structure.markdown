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
##### 函数的渐进增长：给定2个函数f(n)和g(n),如果存在一个整数N，使得对于所有的n>N，f(n)总是比g(n)大,那么我们就说f(n)的增长渐近快于g(n)。
#### 算法时间复杂度定义

1. 只关注循环执行次数最多的一段代码
2. 加法法则：总复杂度等于量级最大的那段代码的复杂度
3. 乘法法则：嵌套代码的复杂度等于嵌套内外代码复杂度的乘积

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

只要代码的执行时间不随 n 的增大而增长，这样代码的时间复杂度我们都记作 O(1)。
或者说，一般情况下，只要算法中不存在循环语句、递归语句，即使有成千上万行的代码，其时间复杂度也是Ο(1)。

```
 i=1;
 while (i <= n)  {
   i = i * 2;
 }
```
变量 i 的值从 1 开始取，每循环一次就乘以 2。当大于 n 时，循环结束。
通过 2^x=n 求解 x 这个问题。x=log2n，所以，这段代码的时间复杂度就是 O(log2n)。

```
 i=1;
 while (i <= n)  {
   i = i * 3;
 }
```
这段代码的时间复杂度为O(log3 n),log3 n 就等于 log3 2 * log2 n；
所以，O(log2n) 就等于 O(log3n)。因此，在对数阶时间复杂度的表示方法里，我们忽略对数的“底”，统一表示为 O(logn)。



#### 算法的空间复杂度
通过计算算法所需的存储空间实现，算法的空间复杂度的计算公式：S(n)=O(f(n)),其中n为问题的规模，f(n)为语句关于n所占存储空间的函数。

##### 最好情况时间复杂度（best case time complexity）

##### 最坏情况时间复杂度（worst case time complexity）

##### 平均情况时间复杂度（average case time complexity）

##### 均摊时间复杂度（amortized time complexity）


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
* 左/右斜树：所有结点都只有左/右子树的二叉树。
* 满二叉树：所有结点都有左右子树，并且所有叶子都在同一层上。
* 完全二叉树：对一棵右n个结点的二叉树按层序编号，如果编号为i(0<=i<=n)的结点
于同样深度的满二叉树中编号为i的结点在二叉树中的位置完全相同，则这棵二叉树称为完全二叉树。
即叶子结点只能出现在最下层或者次下层，并且最下层的叶子结点集中在树的左部。

二叉树性质：
* 在一棵非空二叉树中，第i层的结点总数不超过2^(i-1), i>=1；
* 在一棵树深度为h的二叉树最多有2^h - 1个结点(h>=1)，最少有h个结点；（结点层：根结点的层定义为1；根的孩子为第二层结点，依此类推；树的深度：树中最大的结点层）
*  对于任意一棵二叉树，如果其叶结点数为N0，而度数为2的结点总数为N2，则N0=N2+1；（结点的度：结点子树的个数; 树的度： 树中最大的结点度。）

二叉树的存储结构：
* 顺序存储结构：即用一组连续的存储单元存放二叉树中的节点，按照二叉树的结点从上到下，从左到右顺序存储。
![]({{ site.tree1 | prepend: site.baseurl }})
* 链式存储结构：即用链表结构来表示一棵二叉树，用指针指示其逻辑关系。
  在链式存储中，每个节点的结构如: `Lchild|data|Rchild`,即存储数据和2个指向孩子的指针域。也可以以带头结点的方式进行存放。
  ![]({{ site.tree2 | prepend: site.baseurl }})
  三叉链表存储，每个结点由4个域组成，具体为：data,Lchild,Rchild,parent，其中parent域指向该结点的双亲结点，这种存储方式即便于查找孩子结点，也便于查找
  双亲结点；都是相对于二叉存储结构，增加了存储空间。

树的操作：
* Initiate(bt) 建立一棵空的二叉树
* Create(x,lbt,rbt) 生成一棵以x为跟结点的数据域信息，以二叉树lbt和rbt为左右子树的二叉树。
* InsertL(bt,x,parent) 将数据域信息x的结点插入到二叉树bt中作为结点的parent的左孩子结点。如果parent原有左孩子，则将原来的左孩子作为x的左孩子。
* InsertR(bt,x,parent) 同上
* DeleteL(bt,parent) 在二叉树bt中删除结点parent的左子树。
* DeleteR(bt,parent) 同上
* Search(bt,x) 在bt中查找元素x
* Traverse(bt) 遍历二叉树bt

二叉树遍历：
* DLR先序遍历： 1.访问根结点；2.先序遍历根结点左子树；3.先序遍历根结点右子树
* LDR中序遍历： 1.中序遍历根结点左子树；2.访问根结点；3.中序遍历根结点右子树
* LRD后序遍历： 1.后序遍历根结点左子树；2.后序遍历根结点右子树；3.访问根结点
* 层序遍历： 从第一层（根结点）开始遍历，同一层 由左到右进行访问。
```java
@Data
public class BinaryNode {
    Object element;
    BinaryNode left;
    BinaryNode right;

    public BinaryNode(Object element, BinaryNode left, BinaryNode right) {
        this.element = element;
        this.left = left;
        this.right = right;
    }

    /**
     * 前序遍历
     * @param binaryNode
     */
    static  void  preOrder(BinaryNode binaryNode){
        if (binaryNode == null) return;
            visit(binaryNode);// 访问结点数据
        preOrder(binaryNode.getLeft());// 递归左子树
        preOrder(binaryNode.getRight());// 递归右子树
    }

    static  void  inOrder(BinaryNode binaryNode){
        if(binaryNode == null) return;
        inOrder(binaryNode.getLeft());
        visit(binaryNode);
        inOrder(binaryNode.getRight());
    }
    static  void  postOrder(BinaryNode binaryNode){
        if(binaryNode == null) return;
        postOrder(binaryNode.getLeft());
        postOrder(binaryNode.getRight());
        visit(binaryNode);
    }

    private static void visit(BinaryNode binaryNode) {
        System.out.print(binaryNode.getElement());
    }

    public static void main(String[] args) {
        BinaryNode l = new BinaryNode("B",null,null);
        BinaryNode r = new BinaryNode("C",null,null);
        BinaryNode root = new BinaryNode("A",l,r);
        preOrder(root);
        System.out.println();
        inOrder(root);
        System.out.println();
        postOrder(root);
    }
}
```
##### 二叉查找树 Binary Search Tree，BST
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

##### 最优二叉树(哈夫曼树)
最优二叉树是指对于一组带有确定权值的叶结点，构造的具有`最小带权路径长度`的二叉树。
二叉树的`路径长度`则是指由根结点到所有叶结点的路径长度之和。
如果二叉树中的叶结点都具有一定的权值，则可将这一概念加以推广。
设二叉树具有n个带权值的叶结点，那么从根结点到各个叶结点的路径长度与相应结点权值的乘积之和叫做二叉树的带权路径长度，记为：`WPL= Wk·Lk`
其中Wk为第k个叶结点的权值，Lk 为第k个叶结点的路径长度。

由相同叶子结点所构成的二叉树有不同的形态和不同的带权路径长度，那如何找到最小带权路径长度？

一棵二叉树要使其WPL值最小，必须使权值越大的叶子结点越靠近根结点，权值越小越远离根结点。
基本思想是：
* （1）由给定的n个权值{W1，W2，…，Wn}构造n棵只有一个叶结点的二叉树，从而得到一个二叉树的集合F={T1，T2，…，Tn}；
* （2）在F中选取根结点的权值最小和次小的两棵二叉树作为左、右子树构造一棵新的二叉树，这棵新的二叉树根结点的权值为其左、右子树根结点权值之和；
* （3）在集合F中删除作为左、右子树的两棵二叉树，并将新建立的二叉树加入到集合F中；
* 重复（2）（3）两步，当F中只剩下一棵二叉树时，这棵二叉树便是所要建立的哈夫曼树。
```java

@Data
public class HuffmanTree<T extends Comparable<T>> {
    @Data
    class Node<T>{
        private T data;// 数据域
        private int weight;// 权重
        private Node<T> left;// 左
        private Node<T> right;

        public Node(T data, int weight, Node<T> left, Node<T> right) {
            this.data = data;
            this.weight = weight;
            this.left = left;
            this.right = right;
        }
    }

    /**
     * 创建树
     * @param nodeList
     * @param <T>
     * @return
     */
    public  <T> Node<T> createTree(List<Node<T>> nodeList){
        while (nodeList.size() > 1){
            nodeList.sort(Comparator.comparing(Node::getWeight));// 排序
            Node<T> left = nodeList.get(0);// 最小集合
            Node<T> right = nodeList.get(1);// 次小集合
            Node<T> parent = new Node<>(null,left.getWeight() + right.getWeight(),left,right);// 构造node
            nodeList.remove(left);
            nodeList.remove(right);// 移除集合
            nodeList.add(parent);// 加入新集合
        }
        return nodeList.get(0);
    }

    public List<Node<String>> test(){
        List<Node<String>> nodes = new ArrayList<>();
        nodes.add(new Node<>("b", 5,null,null));
        nodes.add(new Node<>("a", 7,null,null));
        nodes.add(new Node<>("c", 2,null,null));
        nodes.add(new Node<>("d", 4,null,null));
        return nodes;
    }


    public static void main(String[] args) {
        HuffmanTree huffmanTree = new HuffmanTree();
        List test = huffmanTree.test();
        HuffmanTree.Node tree = huffmanTree.createTree(test);
        System.out.println(tree);
    }
}
```


##### 查找
* 静态查找：只在数据元素集合中查找是否存在关键字等于某个给定的关键字的数据元素。
* 动态查找：除包含静态查找外，还包括在查找过程中同上插入或删除元素，即在查找的过程中数据集合可能会改变。
* 哈希查找：以上2种查找方法中 数据元素的关键字和数据元素的存储位置之间没有关系，因此查找过程是一系列的比较。
            如果构造一种存储结构，使得数据元素的关键字和数据元素的存储位置之间存在某种对应关系，则就可以通过元素的关键字直接得到该元素的存储位置，
            即哈希表或哈希存储结构。

>静态查找
>>静态查找的存储结构主要有 顺序表，有序顺序表，索引顺序表。
>>顺序表：从顺序表的一端开始，用给定数据元素的关键字逐个和顺序表中的各元素对比，来确定是否可以查找成功。
>>有序顺序表：主要有顺序查找和折半查找。因为表已经有序，查找效率会更高。
>>
>>顺序查找：和顺序表的查找方法相同，当查找失败时，不需要对比所有元素即可得到结果。
>>
>>折半查找：这种查找每一次比较都使查找范围缩小一半。
>>
>>索引顺序表：当数据量很大时，在以上的查找方法中，都需要很长一段时间，此时提高查询效率的一个常见方法就是在顺序表上建索引表。
>>索引表和书的目录类似，把要在其上建立索引表的顺序表称为主表，主表存放数据元素的所有信息，索引表存放主表要查找的元素的主关键字和索引信息。
![]({{ site.tree3 | prepend: site.baseurl }})
>>要使索引表的查找效率高，索引表必须有序，当主表的元素关键字不一定要有序。
>>建立多个索引的一般方法是：先在主表上建立一个和主表项完全相同，但只包含索引关键字和该数据在主表中位置信息的索引表，再在这个索引表上建立索引表。
>动态查找
>>静态查找的存储结构主要有 二叉树结构和树结构。二叉树分为二叉排序树，平衡二叉树等，树结构分为B_树，B+树等。
>>二叉排序树：（1）若左子树不空，则左子树上所有节点的值均小于它的根节点的值；（2）若右子树不空，则右子树上所有节点的值均大于或等于它的根节点的值；
>>（3）左、右子树也分别为二叉排序树；
>>查找步骤：若根结点的关键字值等于查找的关键字，成功。否则，若小于根结点的关键字值，递归查左子树。若大于根结点的关键字值，递归查右子树。若子树为空，查找不成功。
>>插入：树的结构通常不是一次生成的，而是在查找过程中，当树中不存在关键字等于给定值的结点时再进行插入。新插入的结点一定是一个新添加的叶子结点，并且是查找不成功时查找路径上访问的最后一个结点的左孩子或右孩子结点。
>>二叉树的插入过程首先是一个查找过程。
>>为了防止二叉排序树的最坏情况出现，可以将其改为`平衡二叉树`。

**平衡查找树**
平衡二叉树（Balanced Binary Tree）具有以下性质：它是一 棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。

平衡二叉树的构造时间要大于二叉排序树。

对一棵查找树（search tree）进行查询/新增/删除 等动作, 所花的时间与树的高度h 成比例, 并不与树的容量 n 成比例。

平衡查找树的数据结构，能够保证在最差的情况下也能达到 lgN 的效率，要实现这一目标我们需要保证树在插入完成之后始终保持平衡状态，
这就是平衡查找树(Balanced Search Tree)。在一棵具有 N 个节点的树中，我们希望该树的高度能够维持在 lgN 左右，这样我们就能保证
只需要lgN次比较操作就可以查找到想要的值。

**2-3树**
2-3树是最简单的B-树（或-树）结构，其每个非叶节点都有两个或三个子女，而且所有叶都在统一层上。2-3树不是二叉树，其节点可拥有3个孩子。
不过，2-3树与满二叉树相似。高为h的2-3树包含的节点数大于等于高度为h的满二叉树的节点数，即至少有2^h-1个节点。

* 2-节点有两个孩子，必含一个数据项，其查找关键字大于左孩子的查找关键字，而小于右孩子的查找关键字。
* 3-节点有三个孩子 ，必含两个数据项，其查找关键字S和L满足下列关系：S大于左孩子的查找关键字，而小于中孩子的查找关键字；L大于中孩子的查找关键字，而小于右孩子的查找关键字。
* 叶子可以包含一个或两个数据项。

2-3树是平衡的3路查找树，其中2（2-node）是指拥有两个分支的节点，3（3-node）是指拥有三个分支的节点。

![]({{ site.tree4 | prepend: site.baseurl }})



**B树**
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

B-树是一种平衡的多路查找树,概括来说是一个节点可以拥有多于2个子节点的二叉查找树。
平衡是指所有节点都在同一层，从而避免二叉排序分支退化。
**B+树**
B+树是对B树的一种变形树,B树主要用于动态查找问题，而B+树主要用于文件系统。
它与B树的差异在于：
* ① 在B_树中，有n棵子树的结点中有n-1个关键字；而在B+树中，有n棵子树的结点中有n个关键字。
* ② B+树比B_树多一层叶子结点，B+树在这层增加的叶子结点中包含了每个数据元素的所有信息，并且所有叶子结点从左到右依次链接，这样刚好构成一个每个叶子结点都包含若干个有序关键字的有序单链表。
* ③ 在B_树中，每个非叶子结点中的关键字值大于相邻左指针所指子树中所有结点的关键字值，小于相邻右指针所指子树中所有结点的关键字值；而在B+树中，每个非叶子结点中的一个关键字与一个指针对应，
这些关键字与对应的指针满足要求：该关键字值是对应指针所指子树中所有关键字的最大值。这样，B+树的叶子结点可以看作是一个个的文件，而B+树的所有非叶子结点刚好就是这些文件的索引文件。

为什么使用B-Tree（B+Tree）？
红黑树等数据结构也可以用来实现索引，但是文件系统及数据库系统普遍采用B/+Tree作为索引结构。

局部性原理与磁盘预读：
由于存储介质的特性，磁盘本身存取就比主存慢很多，再加上机械运动耗费，磁盘的存取速度往往是主存的几百分分之一，因此为了提高效率，要尽量减少磁盘I/O。为了达到这个目的，磁盘往往不是严格按需读取，
而是每次都会预读，即使只需要一个字节，磁盘也会从这个位置开始，顺序向后读取一定长度的数据放入内存。这样做的理论依据是计算机科学中著名的局部性原理：

当一个数据被用到时，其附近的数据也通常会马上被使用。

程序运行期间所需要的数据通常比较集中。

由于磁盘顺序读取的效率很高（不需要寻道时间，只需很少的旋转时间），因此对于具有局部性的程序来说，预读可以提高I/O效率。

预读的长度一般为页（page）的整倍数。页是计算机管理存储器的逻辑块，硬件及操作系统往往将主存和磁盘存储区分割为连续的大小相等的块，每个存储块称为一页（在许多操作系统中，页得大小通常为4k），
主存和磁盘以页为单位交换数据。当程序要读取的数据不在主存中时，会触发一个缺页异常，此时系统会向磁盘发出读盘信号，磁盘会找到数据的起始位置并向后连续读取一页或几页载入内存中，然后异常返回，程序继续运行。

B-/+Tree索引的性能分析

一般使用磁盘I/O次数评价索引结构的优劣。先从B-Tree分析，根据B-Tree的定义，可知检索一次最多需要访问h个节点。数据库系统的设计者巧妙利用了磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次I/O就可以完全载入。为了达到这个目的，在实际实现B-Tree还需要使用如下技巧：

每次新建节点时，直接申请一个页的空间，这样就保证一个节点物理上也存储在一个页里，加之计算机存储分配都是按页对齐的，就实现了一个node只需一次I/O。

B-Tree中一次检索最多需要h-1次I/O（根节点常驻内存），渐进复杂度为O(h)=O(logdN)。一般实际应用中，出度d是非常大的数字，通常超过100，因此h非常小（通常不超过3）。

综上所述，用B-Tree作为索引结构效率是非常高的。

而红黑树这种结构，h明显要深的多。由于逻辑上很近的节点（父子）物理上可能很远，无法利用局部性，所以红黑树的I/O渐进复杂度也为O(h)，效率明显比B-Tree差很多。

上文还说过，B+Tree更适合外存索引，原因和内节点出度d有关。从上面分析可以看到，d越大索引的性能越好，而出度的上限取决于节点内key和data的大小：

dmax=floor(pagesize/(keysize+datasize+pointsize))
floor表示向下取整。由于B+Tree内节点去掉了data域，因此可以拥有更大的出度，拥有更好的性能。

**R-B树**
R-B Tree，全称是Red-Black Tree，又称为“红黑树”，它一种特殊的二叉查找树。红黑树的每个节点上都有存储位表示节点的颜色，可以是红(Red)或黑(Black)。
红黑树的特性:
* （1）每个节点或者是黑色，或者是红色。
* （2）根节点是黑色。
* （3）每个叶子节点（NIL）是黑色。 [注意：这里叶子节点，是指为空(NIL或NULL)的叶子节点！]
* （4）如果一个节点是红色的，则它的子节点必须是黑色的。
* （5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点。
红黑树和AVL树类似，都是在进行插入和删除操作时通过特定操作保持二叉查找树的平衡，从而获得较高的查找性能。

##### 哈希表
选取某个函数，依该函数按关键码计算元素的存储位置，并按此存放；查找时，由同一个函数对给定值kx计算地址，将kx和地址中的关键码进行比较，
确定查找是否成功。即哈希方法。哈希方法中使用的函数称为哈希函数（散列函数）。

通常关键码的集合比哈希地址集合大得多，因为通过哈希函数变换后，可能会将不同的关键码映射到同一个哈希地址上（即哈希冲突），冲突不可避免，只能尽可能减小。

把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

哈希方法需要解决以下问题：
* 如何构造哈希函数
* 如何处理冲突

构造哈希函数原则：
* 函数本身要便于计算，尽可能的简单，用来提高转换速度
* 对关键码计算出的地址要大致均匀分布，以尽可能的减少冲突

* 1.直接寻址法 取关键字或关键字的某个线性函数值为散列地址。即H(key)=key或H(key) = a·key + b，其中a和b为常数（这种散列函数叫做自身函数）。这类函数是
一一对应函数，不会产生冲突，要求地址集合和关键码集合大小一致。
* 2. 数字分析法：分析一组数据，比如一组员工的出生年月日，这时我们发现出生年月日的前几位数字大体相同，这样的话，出现冲突的几率就会很大，但是我们发现年月
日的后几位表示月份和具体日期的数字差别很大，如果用后面的数字来构成散列地址，则冲突的几率会明显降低。因此数字分析法就是找出数字的规律，尽可能利用这些数据来构造冲突几率较低的散列地址。
* 3. 平方取中法：当无法确定关键字中哪几位分布较均匀时，可以先求出关键字的平方值，然后按需要取平方值的中间几位作为哈希地址。
这是因为：平方后中间几位和关键字中每一位都相关，故不同关键字会以较高的概率产生不同的哈希地址。
* 4. 折叠法：将关键字分割成位数相同的几部分，最后一部分位数可以不同，然后取这几部分的叠加和（去除进位）作为散列地址。数位叠加可以有移位叠加和间界叠加两种方法。移位叠加是将分割后的每
一部分的最低位对齐，然后相加；间界叠加是从一端向另一端沿分割界来回折叠，然后对齐相加。
* 5. 随机数法：选择一随机函数，取关键字的随机值作为散列地址，通常用于关键字长度不同的场合。
* 6. 除留余数法：取关键字被某个不大于散列表表长m的数p除后所得的余数为散列地址。即 H(key) = key MOD p,p<=m。不仅可以对关键字直接取模，也可在折叠、平方取中等运算之后取模。对p的选择很重要
，一般取素数或m，若p选的不好，容易产生同义词。

处理冲突:

1. 开放寻址法：Hi=(H(key) + di) MOD m,i=1,2，…，k(k<=m-1），其中H(key）为散列函数，m为散列表长，di为增量序列，可有下列三种取法：
* 1.1. di=1,2,3，…，m-1，称线性探测再散列；
* 1.2. di=1^2,-1^2,2^2,-2^2，⑶^2，…，±（k)^2,(k<=m/2）称二次探测再散列；
* 1.3. di=伪随机数序列，称伪随机探测再散列。
```
以线性探索为例
例如关键码{47，7，29，11，16，92，22，8，3}，哈希表长为11，Hash(key) = key mod 11
结果如果：{3， 7， 7， 0， 5， 4， 0，8，3}
47 7 11 16 92 8 没有冲突直接存入
Hash(29) = 7 地址冲突，需要找下一个地址,H1 = (Hash(29)+1) mod 11 = 8 仍然冲突，继续找H2 = (Hash(29)+2) mod 11 = 9
存入位置9
```
线性探索可能使得第i个哈希地址的同义词存入第i+1个哈希地址....，因此可能出现很多元素在相邻的哈希地址上”堆积“，大大降低查找效率。

二次探测再散列和随机探测再散列可以改善”堆积“。

2. 链地址法（拉链法）
哈希函数得到的哈希地址域在区间`[0,m-1]`上，以每个哈希地址作为一个指针，指向一个链，即分配数组`E *eptr[m]`，建立m个空链表，由哈希函数对关键码转换后
,映射到同一个哈希地址i的同义词均加入到`*eptr[i]`指向的链表中。
3. 建立一个公共溢出区  建立2个表，一个基本表，一个溢出表（发生冲突存入次表）

查找性能
散列表的查找过程基本上和造表过程相同。一些关键码可通过散列函数转换的地址直接找到，
另一些关键码在散列函数得到的地址上产生了冲突，需要按处理冲突的方法进行查找。
查找过程中，关键码的比较次数，取决于产生冲突的多少，产生的冲突少，查找效率就高，产生的冲突多，查找效率就低。
影响产生冲突多少有以下三个因素：
1. 散列函数是否均匀；
2. 处理冲突的方法；
3. 散列表的装填因子。
散列表的装填因子定义为：α= 填入表中的元素个数 / 散列表的长度



############
可以先在磁器口逛吃逛吃，之后前往李子坝轻轨站亲身体验轻轨穿楼而过的景观！

       鹅岭二厂是一个充满传奇故事的地方，文青必来打卡的地方，其中的一面涂鸦墙还还原了电影《从你的为全世界路过》中的场景，满足了老夫的少女心~
洪崖洞、解放碑、八一路好吃街、磁器口古镇、李子坝轻轨、观音桥步行街、长江索道,两江夜游等..


* 第一天
* 上午到江北机场/重庆北站
* 中午吃点东西+休息
* 下午14点后，观音桥步行街+吃饭 预计2-3小时
* 预计18:30【人民大礼堂外景】逛20-30分钟
* 距离3km(20分钟 19:00)到【洪崖洞之洪崖滴翠】玩30分钟后
* 朝天门广场
* 随后去往两江的交汇处朝天门，建议打的前往（约3公里），这里是重庆的水码头，可乘坐游船欣赏两江仿若人间星河般绚烂的夜景。
* 八一路好吃街

* 第二天
* 吃完早餐后，做轻轨李子坝轻轨站瞎逛
* 涂鸦一条街 【到达方式】①公共交通：继续从李子坝站乘坐轻轨2号线经5站至杨家坪站下车，后从杨九路公交站乘坐441路或233路公交经5站到达黄桷坪正街站下即达涂鸦一条街（用时约45分钟）；沿黄桷坪正街步行约三百米可达四川美院。
                         ②的士：从李子坝打的前往约12公里，用时约25分钟。
* 四川美院
* 两路口皇冠大扶梯（从四川美院到达约1小时）吃晚饭
* 【南滨路】+ 【海棠烟雨公园】到晚上时，前往南滨路的海棠烟雨公园（距离约10公里，从两路口站乘坐3号线至南坪站下车，
  4出口出站，随后在南坪东原东东摩站乘坐375或338公交经12站到海棠烟雨公园站下即达，亦可打的前往）
* 【南山一棵树】
* 【长江索道】40分钟(排队加游玩)，方向（新华路到龙门路）下索道即到达位于解放碑中心地带的龙门路
* 【解放碑】

* 第三天
* 渣滓洞
* 白公馆
* 磁器口古镇

* 第4天
* 大足石刻1日游(大巴车)
```
07:50 集合 乘车前往大足；
10:00 游玩 【昌州古城】（30分钟，可车观，也可下车游览）；
10:30 游玩 【北山石刻】约1.5小时
12:00 午餐:享用当地特色餐，不含酒水;
13:00 游览 【大足博物馆】
此为赠送项目，每周一全天闭馆。如若未游览，自然加长其他景点游览时间。
14:00 游览【大足宝顶山石刻】约1.5小时
15:30 集合返程，返回主城区渝中较场口附近，导游安排送回酒店或者自由活动
```
* 第5天
* 重庆_成都（约2小时）
* 锦里、春熙路、宽窄巷子
```
【18:50】宽窄巷子B口集合
【19:00】宽窄巷子
【20:30】锦里古街
【21:00】琴台路
【21:10】玉林小酒馆
【21:30】爱情斑马线
【21:40】九眼桥*合江亭
【22:00】九眼桥廊桥散团
```
* 第6天
* 都江堰景区或者青城山

* 第7天
* 回




