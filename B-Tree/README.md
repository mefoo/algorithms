B-Tree
=====

什么是B树？
---------------

From [维基百科](http://zh.wikipedia.org/wiki/B%E6%A0%91):B树（B-tree）是一种树状数据结构，它能够存储数据、对其进行排序并允许以O(log n)的时间复杂度运行进行查找、顺序读取、插入和删除的数据结构。B树，概括来说是一个节点可以拥有多于2个子节点的二叉查找树。与自平衡二叉查找树不同，B-树为系统最优化大块数据的读和写操作。B-tree算法减少定位记录时所经历的中间过程，从而加快存取速度。普遍运用在数据库和文件系统。

定义
--------
根据Knuth的定义，一棵m叉B树，需要同时满足以下条件：

1. 每个节点之多有m个孩子节点。
2. 除了叶子节点和根节点，其余节点至少有ceil(m / 2)个孩子节点。
3. 如果根节点不是叶子节点，则至少有两个孩子节点。
4. 非叶子节点如果有k个孩子，则有k-1个关键字。
5. 所有的叶子节点处于同一层。

查找
----

查找算法类似于二查搜索树查找，从根节点开始向下递归查找。

定义insertIndex为关键字不存在时应该插入的索引位置，index为关键字存在时的索引位置。

由二分查找算法
```java
int binarySearch(T key) {
	int low = 0;
	int high = size - 1;
	while (low <= high) {
		int mid = (low + high) >>> 1;
		int cmp = cmp(key, (T)values[mid]);
		if (cmp == 0)
			return mid;
		if (cmp < 0) {
			high = mid - 1;
		} else {
			low = mid + 1;
		}
	}
	return -(low + 1);
}
```
若关键字存在，返回关键字的位置，否则low即为需要插入的位置，根据返回的值x，-x-1即为insertIndex的值。

查找算法：

1. 初始化p = root, key
2. 若p是叶子节点，调用index = p.binarySearch(key), 若index >= 0, 返回true， 否则返回false。
3. p不是叶子节点，调用index = p.binarySearch(key)， 若index >= 0,返回true，否则转4步。
4. 求插入位置insertIndex = -index - 1; 令p = p.children[insertIndex], 转2.

插入
----

必须插入关键字在叶子节点上，不能直接插入到非叶子节点中。首先根据查找算法查找需要插入的叶子节点位置，若关键字已经存在，直接返回，否则插入到叶子中，然后判断叶子是否满了，若满了需要分裂节点。

插入算法：

1. 根据查找算法，找到插入的位置，若关键字已经存在，算法结束。否则转2。
2. 把关键字插入到叶子中，并保持关键字有序。若此时叶子节点关键字数量不超过最大数量，算法结束。否则转3。
3. 从叶子节点中抽出中间节点mid 。
4. 把叶子节点由mid平均分隔成两个节点，其中比mid小的关键字位于左节点，比mid大的位于右节点。
5. 更新左节点和右节点孩子节点的父亲节点，指向左节点或右节点。mid父亲节点指向父亲的父亲节点。
6. 若mid的父亲节点存在，把mid节点插入到父亲节点中，mid的两边孩子分别指向左节点和右节点。若不存在父亲节点，新建一个节点作为父亲节点，并把mid插入。
7. 若parent节点也满了，递归分裂parent节点，直到所有节点都满足条件。

删除
----

删除关键字的步骤是先定位关键字，然后删除，最后调整使其满足b树特征。

删除位置可能是叶子节点或者是内部节点。

1. 在叶子上删除。
  + 定位关键字，删除。
  + 如果该节点关键字数量少于最少数量，重新调整。
2. 在内部节点上删除。由于内部节点的每个关键字都作为两颗子树的分隔符，因此删除后，必须寻找新的分隔符。左子树的最大节点和右子树的最小节点都可以作为新的分隔符。
  + 从左子树中的叶子中找到最右关键字作为分隔符，若左子树不存在，寻找右子树最左关键字作为分隔符。将分隔符从叶子中移到删除关键字的位置。
  + 如果从叶子中移除了分隔符后，剩余关键字数量少于最少关键字数量，重新调整。

删除节点后调整
-------------
1. 如果贫困节点的右兄弟节点富裕，即多于最少节点，则左旋转。
  + 把父亲节点的分隔符关键字移到贫困节点末尾。此时贫困节点顺利脱贫。
  + 把右兄弟节点的第一个节点取代原来分隔符关键字位置。右兄弟节点依然富裕。
  + 树已经达到平衡，结束。
2. 否则若贫困节点的左兄弟富裕，则右旋转。
  + 把父亲节点的分隔符关键字插入到贫困节点的首位置。此时贫困节点顺利脱贫。
  + 把左兄弟的最后一个节点取代分隔符关键字原来的位置。
  + 树已经达到平衡，结束。
3. 否则，由于左兄弟和右兄弟都不富裕，需要合并操作。
  + 定义左节点为: 若存在右兄弟，则左节点为贫困节点，否则为贫困节点的左兄弟。
  + 定义右节点为：左节点的右兄弟。
  + 将分隔符关键字移到左节点（可能是贫困节点或者刚好等于最少数量的节点）的最后位置，父亲节点关键字数减一。
  + 把右节点的所有信息拷贝到左节点（包括孩子也要拷贝），此时左节点达到最大关键字数量，释放右节点。
  + 由于父亲节点关键字数少一：
    - 若父亲节点是root并且此时为空，释放该节点，并把root指向新合并的节点（左节点）。此时高度-1.
    - 若父亲节点是root，无需调整。
    - 否则若父节点关键字数量少于最少节点，递归调整父亲节点。

从容器中构建B-Tree
------------------

在实际引用中，一般会从一个已经存在的容器中构建B树，然后进行更新操作。在这种情况，从空树中逐一进行插入，效率并不高。而是先构造叶子节点，然后逐一向上构造内部节点，直到只有一个节点，作为根节点。这种构建B树的方法叫做[Bulkloading](http://en.wikipedia.org/wiki/B-tree#Deletion).假设我们需要构建m阶B树，算法从一个有序无重复列表中构建n个刚好有m个关键字的节点，和一个少于m的节点，作为初始化叶子节点（我们发现除了最后一个节点，所有节点都有一个多余的节点，因为m阶树关键字数量最多为m-1）。然后从叶子节点开始，依次取除最后一个叶子外的所有叶子的最后一个多余节点（不存在多余则不取），构造具有m个关键字的节点，这样能够保证除了最后一个节点，其余节点一定有m个节点。然后继续从些节点中使用相同的方法，直到只剩下一个节点。注意，第一次初始化叶子节点时，最后一个节点关键字数量必须满足小于m，如果关键字数量刚好整除m，则先拿出来一个关键字。算法为：

1. 初始化容器，排重和排序，返回一个有序无重复列表list
2. 若list.size % m == 0，拿出来一个节点，此时list.size % m != 0;
3. 构建叶子节点，L1, L2, L3, ... Ln-1, Ln, 其中L1 ... Ln-1有m个关键字, Ln有少于m个关键字。
4. 初始化nodes = L1, L2, L3, ... Ln-1, Ln。
5. 从nodes中找出关键字数量为m的节点k1,k2,...,kn， 如果不存在关键字数量为m的节点，转8.
6. 从k1, k2, ..., kn中依次取出最后一个节点，逐一组合节点N1， N2， N3， ..., Nn-1, Nn， 其中N1, N2, ..., Nn-1一定有m个关键字，Nn可能不足m个关键字。这些N节点作为原来的nodes节点的父亲节点。
7. nodes = N1, N2, ..., Nn，转5.
8. 设置root为nodes[0],算法结束。

比如有[1,2,3,4,5,6,7,8,9,10]节点，构建3阶b树，首先初始化叶子节点为

[1,2,3] [4,5,6] [7, 8, 9] [10]

然后第一层（即叶子）构建第二层节点，结果为：
       [3,6,9]
       
[1，2] [4,5] [7,8] [10]

然后继续构建第三层节点

[9]

[3,6]

[1,2] [4,5] [7,8] [10]

此时已经不存在关键字为3的节点，算法结束。[9]就是根节点，高度为3.

算法原理很简单，实现时需要注意指针，即更新孩子节点和父亲节点，当下层节点有少于m个节点关键字节点，则上层节点的最后一个孩子应该指向该节点，否则，应该取下层节点的最后一个孩子的孩子。

该算法java实现为：

```java
void initFromList(List<E> list) {
	// 构造叶子节点，要求最后一个叶子关键字必须少于order，其余叶子关键字数量必须等于order
	int n = (list.size() - 1 + order) / order;
	Node<E>[] nodes = new Node[n];
	for (int i = 0; i < n; ++i) {
		nodes[i] = new Node<E>();
		for (int j = 0; j < order; ++j) {
			if (i * order + j < list.size()) {
				nodes[i].values[j] = list.get(i * order + j);
				nodes[i].size++;
			}
		}
	}
	int height = 1;
	// make internal nodes;
	int lastN;
	while (true) {
		height ++;
		lastN = n;
		int m = 0;
		for (int i = 0; i < n; ++i) {
			if (nodes[i].size == order) {
				m++;
			}
		}
		if (m < 1)
			break;
		n = (m - 1 + order) / order;
		Node<E>[]p = new Node[n];
		for (int i = 0; i < n; ++i) {
			p[i] = new Node<E>();
			p[i].children = new Node[order + 1];
			p[i].isLeaf = false;
			for (int j = 0; j < order; ++j) {
				int index = i * order + j;
				if (index >= lastN)
					break;
				int size = nodes[index].size;
				if (size < order) {
					break;
				}
				p[i].values[j] = nodes[index].values[size - 1]; // 取最后一个节点作为上层节点的alue
				nodes[index].values[size - 1] = null;
				p[i].size++; // 上层节点数量+1
				nodes[index].size--; // 下层节点数量-1
				nodes[index].parent = p[i]; // 更新下层节点的父亲节点指向上层节点
				p[i].children[j] = nodes[index]; // 更新上层节点的的孩子节点指向内部节点
			}
		}
		// 更新最后一个节点，上层节点的最后一个孩子指向下层的最后一个节点，下层最后一个节点的parent指向上层最后一个节点
		Node<E> newLast = p[n - 1];
		Node<E> oldLast;
		//下层最后一个节点取决于最后一个节点关键字数量是否order
		if (newLast.children[newLast.size - 1] == nodes[lastN - 1]) {
			oldLast = nodes[lastN - 1].children[order];
		} else {
			oldLast = nodes[lastN - 1];
		}
		newLast.children[newLast.size] = oldLast;
		oldLast.parent = newLast;
		// 更新工作节点。
		nodes = p;
	}
	root = nodes[0];
}
```
java实现
--------

1. 实现了CURD操作，并支持批量插入、删除。
2. 支持迭代，迭代顺序为按关键字递增排列。
3. 支持自定义比较器，构造器可以传人自己的Comparator.

Demo：

```java
BTree<Integer> tree = new BTree<Integer>(3);
		for (int i = 1; i <= 100; i++) {
			tree.add(i);
		}
		tree.remove(50);
		for (Iterator<Integer> iter = tree.iterator(); iter.hasNext();) {
			int value = iter.next();
			if (value > 20 || (value & 1) == 0) {
				iter.remove();
				//System.out.println(value + " removed");
			}
		}
		System.out.println("height: " + tree.getHeight()); //3
		System.out.println("size: " + tree.size()); //10
		for (int i : tree) {
			System.out.print(i + " "); // 1 3 5 7 9 11 13 15 17 19
		}
		System.out.println();
	}
	```
