---
title: Java容器：ArrayList源码分析
date: 2020-7-26 10:48
urlname: 2020072601
categories: Java
tags:
  - Java
author: foochane
toc: true
mathjax: true
top: false
top_img: /images/banner/0.jpg
cover: /images/cover/5.jpg

---



> 注意以下源码分析基于JDK 1.8



## 1 基本定义及构造方法

- ArrayList实现List接口
- RandomAccess接口表示表示ArrayList支持快速随机访问
- 使用动态数组存储数据

```JAVA
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * 默认初始化容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 可共享的空数组实例，用于空实例对象的创建
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
	 *
     * 可共享的空数组实例对象，用于默认容量的空实例对象的创建
     * 将其与EMPTY_ELEMENTDATA区分开来，以便知道添加第一个元素时需要扩容多少。
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 存储ArrayList元素的数组，ArraList的容量是数组的长度.
     * 当elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA时，会在添加第一个元素的时候，
     * 将容量扩大到DEFAULT_CAPACITY
     * 
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList的容量(包含的元素数量)。
     *
     * @serial
     */
    private int size;
    
    /**
     * 构造一个具有指定初始容量的空ArrayList。
     * @param  initialCapacity  初始化容量
     * @throws IllegalArgumentException 如果指定的初始化容量为空
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * 构造一个空的ArrayList，初始化容量为10（）
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
	 * 
     * 构造一个包含指定集合元素的ArrayList
     *
     * @param c 将其元素放置在此列表中的集合
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
    
    //其他方法
    //略
    
}
```



根据上面的定义，ArrayList有如下三种创建方法。

1. 构造空的ArrayList

   ```java
   //创建一个空的ArrayList，默认容量为10
   ArrayList<Integer> arrayList1 = new ArrayList<>();
   ```

2. 指定ArrayList的容量

   ```java
   //创建一个空的ArrayList，初始容量为20
   ArrayList<Integer> arrayList2 = new ArrayList<>(20);
   ```

3. 传入指定集合元素

   ```java
   //创建一个ArrayList，其内容为collection里面的元素
   //如果collection为空会默认创建一个，会创建一个容量为空的ArrayList
   ArrayList<Integer> arrayList3 = new ArrayList<>(collection);
   ```



## 2 add方法



```java
    /**
     * 在list后面插入一个元素
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    /**
  	 * 指定插入位置（下标）插入元素
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```



- add(E e)方法
  1. 先会先检查elementData数组的容量，如果容量不够会先扩容，**扩容为原来的1.5倍**
  2. 在list后面插入元素

![](https://foochane.cn/images/2020/102.png)



- add(int index, E element)
  1. 会先检查index是否在elementData数组的区间中如果不在就后抛出异常
  2. 先检查elementData数组的容量，如果容量不够会先扩容，**扩容为原来的1.5倍**（和上面一样）
  3. 把index后面的元素后移
  4. 在index处插入要插入的元素



![](https://foochane.cn/images/2020/103.png)

## 3 数组扩容

扩容的代码如下：

```java
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

   //计算容量
   private static int calculateCapacity(Object[] elementData, int minCapacity) {
        //如果当前的list为空，则将list的容量设置为默认值（10）
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureExplicitCapacity(int minCapacity) {
        
        //每修改一次list，这个数就+1
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;


    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        
        //新的容量 = 旧的容量加+旧的容量的一半
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        
        //如果新的容量超过Integer.MAX_VALUE就会溢出，抛出异常
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```



##  4 set方法

set方法用于替换指定位置的元素

```java
    /**
     * 替换指定位置的元素
     *
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        // 先检查index
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```



```java
    /**
     * 检查给定的索引是否在目前数组的范围内，不在就抛出异常
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```



## 5 get方法

同样先检查index，如果符合条件，直接放回数组中index对应的数据

```java
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```



## 6 remove方法

```java
public E remove(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

    modCount++;
    // 返回被删除的元素值
    E oldValue = (E) elementData[index];

    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将 index + 1 及之后的元素向前移动一位，覆盖被删除值
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 将最后一个元素置空，并将 size 值减 1     
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

E elementData(int index) {
    return (E) elementData[index];
}

/删除指定元素，若元素重复，则只删除下标最小的元素
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        // 遍历数组，查找要删除元素的位置
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

//快速删除，不做边界检查，也不返回删除的元素值
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

```



删除一个元素步骤如下：

1. 获取指定位置 index 处的元素值
2. 将 index + 1 及之后的元素向前移动一位
3. 将最后一个元素置空，并将 size 值减 1
4. 返回被删除值，完成删除操作



![](https://foochane.cn/images/2020/104.png)



## 7 trimToSize方法

现在，考虑这样一种情况。我们往 ArrayList 插入大量元素后，又删除很多元素，此时底层数组会空闲处大量的空间。因为 ArrayList 没有自动缩容机制，导致底层数组大量的空闲空间不能被释放，造成浪费。对于这种情况，ArrayList 也提供了相应的处理方法，如下：

```java
/** 将数组容量缩小至元素数量 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```



通过上面的方法，我们可以手动触发 ArrayList 的缩容机制。这样就可以释放多余的空间，提高空间利用率。

![](https://foochane.cn/images/2020/105.png)



## 8 clear方法

clear 的逻辑很简单，就是遍历一下将所有的元素设置为空。

```java
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```



## 9 方法复杂度分析

- add(E e)方法：添加元素到末尾，平均时间复杂度为O(1)。
- add(int index, E element)方法：添加元素到指定位置，平均时间复杂度为O(n)。
- get(int index)方法：获取指定索引位置的元素，时间复杂度为O(1)。
- remove(int index)方法：删除指定索引位置的元素，时间复杂度为O(n)。
- remove(Object o)方法：删除指定元素值的元素，时间复杂度为O(n)。



> ArrayList适用于有序插入，读取频繁的场景

## 10 ArrayList和Vector的区别

这两个类都实现了 List 接口（List 接口继承了 Collection 接口），他们都是有序集合

- 线程安全：**Vector 使用了 Synchronized 来实现线程同步，是线程安全的，而 ArrayList 是非线程安全的。**
- 性能：ArrayList 在性能方面要优于 Vector。
- 扩容：ArrayList 和 Vector 都会根据实际的需要动态的调整容量，只不过在 Vector 扩容为原来的2倍，而 ArrayList 扩容为原来的1.5倍。
- Vector类的所有方法都是同步的。可以由两个线程安全地访问一个Vector对象、但是一个线程访问Vector的话代码要在同步操作上耗费大量的时间。
- Arraylist不是同步的，所以在不需要保证线程安全时时建议使用Arraylist。



> Vector的开销比ArrayList大，访问速度更慢，在实际开发中最好使用ArrayList，而且同步操作完全可以由程序员来控制。



## 11 多线程场景下如何使用 ArrayList？



ArrayList 不是线程安全的，如果遇到多线程场景，可以通过 Collections 的 synchronizedList 方法将其转换成线程安全的容器后再使用。例如像下面这样：



```
List<String> synchronizedList = Collections.synchronizedList(new ArrayList<Object>);
synchronizedList.add("aaa");
synchronizedList.add("bbb");

for (int i = 0; i < synchronizedList.size(); i++) {
    System.out.println(synchronizedList.get(i));
}
```