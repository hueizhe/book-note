## 1. 集合中如何删除元素

~~~java  
  
import java.util.ArrayList;  
import java.util.List;  
  
/** 
 *   集合中如何删除元素 
 */  
public class Demo2 {  
    /** 
     * 下面我们以list集合为例 
     */  
    public static  void main(String []args){  
        List list=new ArrayList();  
        list.add(1);  
        list.add(2);  
        list.add(2);  
        list.add(3);  
        //执行如下删除操作，会有什么问题？  
        for(int i=0;i<list.size();i++){  
            if(2==list.get(i)){  
                list.remove(i);  
            }  
        }  
        //执行结果  
        for(int i=0;i<list.size();i++){  
            System.out.println(list.get(i));  
        }  
    }  
}
~~~  
 输出结果:
~~~
原集合：1 2 2 3  
删除后集合：1 2 3
~~~
>这是因为，删除时改变了list的长度。删除第一个2后，长度变为了3，这时list.get(2)为3，不再是2了，不能删除第2个2, 利用for循环删除时索引没有回溯，导致漏删元素  

#### 那么怎么样才能正确删除集合中的元素呢？

- 第一种删除方式（利用for循环删除）：
~~~java
for(int i=0;i<list.size();i++){  
           if(2==list.get(i)){  
               list.remove(i--);// 索引回溯    
           }  
       } 
~~~
 运行结果
~~~
原集合：1 2 2 3  
删除后集合：1 3  
~~~

- 第二种删除方法（利用迭代器的remove()方法删除）

~~~java
ListIterator listIterator = list.listIterator();  
        while(listIterator.hasNext()){  
            int  value = (Integer)listIterator.next();  
            if (2==value) {  
                //aList.remove(str);   // 集合自身的remove()方法删除  
                listIterator.remove(); //迭代器的remove() 方法删除  
            }  
        }
~~~
运行结果
~~~
原集合：1 2 2 3  
删除后集合：1 3 
~~~

> 注意事项

- 利用for循环删除时索引没有回溯，导致漏删元素
- 使用迭代器循环删除元素时，没有利用迭代器remove方法删除元素而是利用集合自身的remove方法删除元素，这样会导致“并发修改异常错误”

~~~java
ListIterator listIterator = list.listIterator();  
        while(listIterator.hasNext()){  
            int  value = (Integer)listIterator.next();  
            if (2==value) {  
                list.remove(value);   // 集合自身的remove()方法删除  
               // listIterator.remove(); //迭代器的remove() 方法删除  
            }  
        }  
~~~
 运行结果：

~~~
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$listIterator.checkForComodification(Unknown Source)
	at java.util.ArrayList$listIterator.next(Unknown Source)
~~~
并发修改异常错误的产生是因为在生成迭代器后迭代器就已经确定了集合的长度size了， 而后集合删除元素后集合的size变小， 但是迭代器任然记录的是之前的size数据在迭代过程中产生并发修改异常 ConcurrentModificationException，

但是如果是使用迭代器的remove()方法来删除元素的话则不会产出这个问题，因为迭代器中的cursor能够自动适应元素删除后集合大小的变化；

所以在删除集合元素时，如果适应迭代器来循环集合元素一定要使用迭代器自身的remove()方法来删除元素
