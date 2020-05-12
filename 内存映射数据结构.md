## 内存映射数据结构<br>
### 1 整数集合<br>
1.1 整数集合(Intset) :
用于有序、无重复地保存多个整数值，能够根据元素的值（改变编码类型）自动选择该用什么长度的整形类型来保存。例如，一个Intset中最长的元素可以用16位来保存，那么这个intset中所有的元素都可以用16位来保存，当需要插入一个32位的整数时，Intset自动扩容到单个存储单元能够存储32位的整数。<br>
1.2 Intset是集合键的底层实现之一，如果一个集合只能存储整形数据，且数据数量不多，Redis就会用Intset去保存集合元素。<br>
1.3 创建Intset完毕以后，添加一个元素到Intset会有三种情况：<br>
&emsp;&emsp;&emsp;&emsp;(1) Intset中本身就有这个元素，不做任何操作。<br>
&emsp;&emsp;&emsp;&emsp;(2) Intset中没有这个元素，并且添加元素不需要对Intset进行升级。  <br>
&emsp;&emsp;&emsp;&emsp;(3) Intset中没有这个元素，但是添加元素需要对Intset进行升级。<br>
1.4 关于升级有两点需要说明：<br>
&emsp;&emsp;&emsp;&emsp;(1) 从短整型到长整型的转换不会改变元素的值（只改变类型位长度）。<br>
&emsp;&emsp;&emsp;&emsp;(2) 升级的编码格式由Intset中长度最大的那个值来决定。<br>
1.5 Intset是有序的，contents属性保证了Intset的有序性，所以Intset可以通过二分查找法来实现查找操作。<br>
###   2 压缩列表  
2.1 压缩列表（Ziplist）是由一系列特殊编码的内存块构成的列表（由连续的内存块构成），每一个压缩列表可以包含多个节点，每个节点可以存储一个长度受限的字符数组，或者整数。因为压缩列表节约内存，所以是哈希键、列表键和有序集合键的底层实现之一。<br>
2.2 压缩列表的域：<br>
&emsp;&emsp;&emsp;&emsp;(1) zlbytes：表示整个压缩列表占用内存的字节数，用于对压缩列表进行重新分配或者计算末端地址。<br>
&emsp;&emsp;&emsp;&emsp;(2) zlbytes：zltail:到达压缩列表尾节点的偏移量，能够在不遍历整个压缩列表的情况下弹出尾结点。<br>
&emsp;&emsp;&emsp;&emsp;(3) zllen：节点的数量。<br>
&emsp;&emsp;&emsp;&emsp;**zlbytes、zltail、zllen共同组成了压缩列表的header部分。**<br>
&emsp;&emsp;&emsp;&emsp;(4) entryX：压缩列表中保存的节点，长度视内容决定。<br>
&emsp;&emsp;&emsp;&emsp;(5) zlend：一个二进制的标识位，用于标识压缩列表的末端。<br>
2.3 节点的构成：<br>
&emsp;&emsp;压缩列表的节点有四个组成部分：pre_entry_length、encoding、length、content。<br>
&emsp;&emsp;&emsp;&emsp;(1) pre_entry_length：表示前一个节点的长度。<br>
&emsp;&emsp;&emsp;&emsp;(2) encoding：表示数据的类型。<br>
&emsp;&emsp;&emsp;&emsp;(3) length:表示节点的长度。<br>
&emsp;&emsp;&emsp;&emsp;(4) content:表示节点的内容。<br>
2.4 创建新的压缩列表：<br>
创建新的压缩列表首先需要建立一个表，然后往空表中插入节点，插入节点有两种方式：<br>
（1）将节点添加到压缩列表的末端，此时新节点后面没有任何节点。<br>
（2）将节点添加到某个/某些节点的前面，此时新节点后面至少有一个节点。<br>
除此之外还有删除节点、遍历压缩列表、查找元素等操作，大家可以参考数组类比理解。
