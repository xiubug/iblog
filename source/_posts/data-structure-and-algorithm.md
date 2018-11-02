---
title: 数据结构与算法
date: 2018-04-18 21:22:38
tags: 
  - note
categories:
  - data structure
---
## 数组
数组的标准定义是：一个存储元素的线性集合（collection），元素可以通过索引来任意存取，索引通常是数字，用来计算元素之间存储位置的偏移量。几乎所有的编程语言都有类似的数据结构。然而 JavaScript 的数组却略有不同。
JavaScript 中的数组是一种特殊的对象，用来表示偏移量的索引是该对象的属性，索引可能是整数。然而，这些数字索引在内部被转换为字符串类型，这是因为 JavaScript 对象中的属性名必须是字符串。数组在 JavaScript 中只是一种特殊的对象，所以效率上不如其他语言中的数组高。
JavaScript 中的数组，严格来说应该称作对象，是特殊的 JavaScript 对象，在内部被归类为数组。由于 Array 在 JavaScript 中被当作对象，因此它有许多属性和方法可以在编程时使用。
## 双向链表
双向链表属于链表的一种，也叫双链表，双向即是说它的链接方向是双向的，它由若干个节点组成，每个节点都包含下一个节点和上一个节点的指针，所以从双向链表的任意节点开始，都能很方便访问它的前驱结点和后继节点。

### 特点
**1、**创建双链表时无需指定链表的长度。
**2、**比起单链表，双链表需要多一个指针用于指向前驱节点，所以需要存储空间比单链表多一点。
**3、**双链表的插入和删除需要同时维护 next 和 prev 两个指针。
**4、**双链表中的元素访问需要通过顺序访问，即要通过遍历的方式来寻找元素。

### 执行过程图解
#### 创建
创建一个空链表：
![img1.png](/images/data-structure-and-algorithm/img1.png)

#### 插入链尾
将`the monster is coming`这些单词按顺序分别插入尾部，创建“the”节点：
![img2.png](/images/data-structure-and-algorithm/img2.png)

连接起来：
![img3.png](/images/data-structure-and-algorithm/img3.png)

创建“monster”节点：
![img4.png](/images/data-structure-and-algorithm/img4.png)

再连接起来：
![img5.png](/images/data-structure-and-algorithm/img5.png)

以此类推，将剩下的节点全部创建并连接起来：
![img6.png](/images/data-structure-and-algorithm/img6.png)

![img7.png](/images/data-structure-and-algorithm/img7.png)

#### 创建迭代器
迭代器的 current 指针初始指向head：
![img8.png](/images/data-structure-and-algorithm/img8.png)

执行两次 next 操作， current 指针指向索引为2的节点：
![img9.png](/images/data-structure-and-algorithm/img9.png)

此时的节点值为：
![img10.png](/images/data-structure-and-algorithm/img10.png)

设置 current 指针指向索引为3的节点：
![img11.png](/images/data-structure-and-algorithm/img11.png)

#### 插入节点
在索引1后面插入“big”节点。先将 current 指针指向索引为1的节点，创建一个"big"新节点：
![img12.png](/images/data-structure-and-algorithm/img12.png)

插入到 current 指向位置：
![img13.png](/images/data-structure-and-algorithm/img13.png)

#### 删除节点
将“big”节点删除，移动当前指针 current 到“big”节点位置：
![img14.png](/images/data-structure-and-algorithm/img14.png)

执行删除操作，断掉“big”节点与前后两节点的 next 和 prev 指针，然后将“the”节点与“monster”节点关联起来：
![img15.png](/images/data-structure-and-algorithm/img15.png)
![img16.png](/images/data-structure-and-algorithm/img16.png)

#### 双向循环链表
前面的双向链表的 head 节点和链尾没有连接关系，所以如果要访问最后一个节点的话需要从头开始遍历，直到最后一个节点。在双向链表基础上改进一下，把 header 节点的 prev 指针指向最后一个节点，而最后一个节点的 next 指针指向 header 节点，于是便构成双向循环链表。
![img17.png](/images/data-structure-and-algorithm/img17.png)

### 实现细节
我们的链表将包括两个构造函数：Node 和 DoublyList。看看它们是怎样运作的。

#### Node
**data：**存储数据。
**next：**指向链表中下一个节点的指针。
**previous：**指向链表中前一个节点的指针。

#### DoublyList
**_length：**保存链表中节点的个数。
**head：**指定一个节点作为链表的头节点。
**head：**tail 指定一个节点作为链表的尾节点。
**add(value)：**向链表中添加一个节点。
**searchNodeAt(position)：**找到在列表中指定位置 n 上的节点。
**remove(position)：**删除链表中指定位置上的节点。

#### 节点模板
在实现中，将会创建一个名为`Node`的构造函数：
``` js
function Node(value) {
  this.data = value;
  this.previous = null;
  this.next = null;
}
```
想要实现双向链表的双向遍历，我们需要指向链表两个方向的属性。这些属性被命名为`previous`和`next`。

接下来，我们需要实现`DoublyList`并添加三个属性：`_length`，`head`和`tail`。
与单链表不同，双向链表包含对链表开头和结尾节点的引用。 由于`DoublyList`刚被实例化时并不包含任何节点，所以`head`和`tail`的默认值都被设置为null。
``` js
function DoublyList() {
  this._length = 0;
  this.head = null;
  this.tail = null;
}
```

#### 操作节点
接下来我们讨论以下方法：`add(value)`，`remove(position)`和`searchNodeAt(position)`。所有这些方法都用于单链表，然而，它们必须被重写为可以双向遍历。

##### 方法1/3 `add(value)`
```js
DoublyList.prototype.add = function(value) {
  var node = new Node(value);

  if (this._length) {
    this.tail.next = node;
    node.previous = this.tail;
    this.tail = node;
  } else {
    this.head = node;
    this.tail = node;
  }

  this._length++;
    
  return node;
};
```
在这个方法中，存在两种可能。首先，如果链表是空的，则给它的`head`和`tail`分配节点。其次，如果链表中已经存在节点，则查找链表的尾部并把新节点分配给`tail.next`；同样，我们需要配置新的尾部以供进行双向遍历。换句话说，我们需要把`tail.previous`设置为原来的尾部。

##### 方法2/3: `searchNodeAt(position)`
创建一个名为`searchNodeAt(position)`的方法，它接受一个名为`position`的参数。这个参数是个整数，用来表示链表中的位置`n`。
``` js
DoublyList.prototype.searchNodeAt = function(position) {
  var currentNode = this.head,
    length = this._length,
    count = 1,
    message = {failure: 'Failure: non-existent node in this list.'};

  // 1st use-case: an invalid position 
  if (length === 0 || position < 1 || position > length) {
    throw new Error(message.failure);
  }

  // 2nd use-case: a valid position 
  while (count < position) {
    currentNode = currentNode.next;
    count++;
  }

  return currentNode;
};
```
在`if`中检查第一种情况：参数非法。如果传给`searchNodeAt(position)`的索引是有效的，那么我们执行第二种情况 —— `while`循环。 在`while`的每次循环中，指向头的`currentNode`被重新指向链表中的下一个节点。这个循环不断执行，一直到`count`等于`position`。

##### 方法3/3: `remove(position)`
理解这个方法是最具挑战性的。我先写出代码，然后再解释它。
``` js
DoublyList.prototype.remove = function(position) {
    var currentNode = this.head,
        length = this._length,
        count = 1,
        message = {failure: 'Failure: non-existent node in this list.'},
        beforeNodeToDelete = null,
        nodeToDelete = null,
        deletedNode = null;
 
    // 1st use-case: an invalid position
    if (length === 0 || position < 1 || position > length) {
        throw new Error(message.failure);
    }
 
    // 2nd use-case: the first node is removed
    if (position === 1) {
        this.head = currentNode.next;
 
        // 2nd use-case: there is a second node
        if (!this.head) {
            this.head.previous = null;
        // 2nd use-case: there is no second node
        } else {
            this.tail = null;
        }
 
    // 3rd use-case: the last node is removed
    } else if (position === this._length) {
        this.tail = this.tail.previous;
        this.tail.next = null;
    // 4th use-case: a middle node is removed
    } else {
        while (count < position) {
            currentNode = currentNode.next;
            count++;
        }
 
        beforeNodeToDelete = currentNode.previous;
        nodeToDelete = currentNode;
        afterNodeToDelete = currentNode.next;
 
        beforeNodeToDelete.next = afterNodeToDelete;
        afterNodeToDelete.previous = beforeNodeToDelete;
        deletedNode = nodeToDelete;
        nodeToDelete = null;
    }
 
    this._length--;
    return message.success;
};
```
`remove(position)`处理以下四种情况：
**1、**如果`remove(position)`的参数传递的位置存在, 将会抛出一个错误。
**2、**如果remove(position)的参数传递的位置是链表的第一个节点（head），将把head赋值给deletedNode ，然后把head重新分配到链表中的下一个节点。 此时，我们必须考虑链表中否存在多个节点。 如果答案为否，头部将被分配为null，之后进入if-else语句的if部分。 在if的代码中，还必须将tail设置为null —— 换句话说，我们返回到一个空的双向链表的初始状态。如果删除列表中的第一个节点，并且链表中存在多个节点，那么我们输入if-else语句的else部分。 在这种情况下，我们必须正确地将head的previous属性设置为null —— 在链表的头前面是没有节点的。
**3、**如果remove(position)的参数传递的位置是链表的尾部，首先把tail赋值给deletedNode，然后tail被重新赋值为尾部之前的那个节点，最后新尾部后面没有其他节点，需要将其next值设置为null。
**4、**这里发生了很多事情，所以我将重点关注逻辑，而不是每一行代码。 一旦CurrentNode指向的节点是将要被remove(position)删除的节点时，就退出while循环。这时我们把nodeToDelete之后的节点重新赋值给beforeNodeToDelete.next。相应的，
把nodeToDelete之前的节点重新赋值给afterNodeToDelete.previous。——换句话说，我们把指向已删除节点的指针，改为指向正确的节点。最后，把nodeToDelete 赋值为null。

最后，把链表的长度减1，返回deletedNode。

#### 完整代码实现
``` js
function Node(value) {
    this.data = value;
    this.previous = null;
    this.next = null;
}
 
function DoublyList() {
    this._length = 0;
    this.head = null;
    this.tail = null;
}
 
DoublyList.prototype.add = function(value) {
    var node = new Node(value);
 
    if (this._length) {
        this.tail.next = node;
        node.previous = this.tail;
        this.tail = node;
    } else {
        this.head = node;
        this.tail = node;
    }
 
    this._length++;
 
    return node;
};
 
DoublyList.prototype.searchNodeAt = function(position) {
    var currentNode = this.head,
        length = this._length,
        count = 1,
        message = {failure: 'Failure: non-existent node in this list.'};
 
    // 1st use-case: an invalid position
    if (length === 0 || position < 1 || position > length) {
        throw new Error(message.failure);
    }
 
    // 2nd use-case: a valid position
    while (count < position) {
        currentNode = currentNode.next;
        count++;
    }
 
    return currentNode;
};
 
DoublyList.prototype.remove = function(position) {
    var currentNode = this.head,
        length = this._length,
        count = 1,
        message = {failure: 'Failure: non-existent node in this list.'},
        beforeNodeToDelete = null,
        nodeToDelete = null,
        deletedNode = null;
 
    // 1st use-case: an invalid position
    if (length === 0 || position < 1 || position > length) {
        throw new Error(message.failure);
    }
 
    // 2nd use-case: the first node is removed
    if (position === 1) {
        this.head = currentNode.next;
 
        // 2nd use-case: there is a second node
        if (!this.head) {
            this.head.previous = null;
        // 2nd use-case: there is no second node
        } else {
            this.tail = null;
        }
 
    // 3rd use-case: the last node is removed
    } else if (position === this._length) {
        this.tail = this.tail.previous;
        this.tail.next = null;
    // 4th use-case: a middle node is removed
    } else {
        while (count < position) {
            currentNode = currentNode.next;
            count++;
        }
 
        beforeNodeToDelete = currentNode.previous;
        nodeToDelete = currentNode;
        afterNodeToDelete = currentNode.next;
 
        beforeNodeToDelete.next = afterNodeToDelete;
        afterNodeToDelete.previous = beforeNodeToDelete;
        deletedNode = nodeToDelete;
        nodeToDelete = null;
    }
 
    this._length--;
 
    return message.success;
};
```