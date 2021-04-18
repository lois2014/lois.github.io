##### 插入排序
> 将一个记录插入到已经排好序的有序表中，从而一个新的、记录数增1的有序表
> 从前往后查询，每次查找前面已排好序的有序表中插入的位置，插入位置之后的元素都向后移动
1. 时间复杂度：O(n^2), 最好的O(n)
2. 稳定排序

> 如 4 6 5 3 7 4
> 第一轮：
> 1. 从下标为1的开始，记录下标为1的数字是6，跟前面下标为0的数字比较，6 > 4 不需要移动，所以当前的序列中前两个数字是有序的
> 
> 第二轮：
> 1. 从下标为2的数字开始，首先记录一下下标为2的数字是5， 跟前面下标为1的数字比较，5 < 6 所以6 移动到下一个位置，
> 即移动到下标为2的位置，记录插入下标为1；结果：
> 4 6 ***6** 3 7 4
> 2. 数字5跟下标为0的数字比较，5 > 4 所以不需要移动，
> 3. 最终结束之后，5插入到下标为1的位置上，结果：
> 3 ***5** 6 3 7 4
> 第二轮结果：3 5 6 3 7 4， 前三个数字是有序的
>
> 第三轮
> 1. 
> 4 **4 5 6 7** 4
>
> *3* 4 5 6 7 4
>
> 3 4 4 **5 6 7**
>
> 3 4 *4* 5 6 7


```java

import java.util.Arrays;

public class InsertionSort {

    public static void main(String[] args) {
        int[] array = {3, 5, 8, 3, 4};
        insertionSort(array);
        System.out.println(Arrays.toString(array));
    }

    private static void insertionSort(int[] array) {
        int n = array.length;
        for (int i = 1; i < n; i++) {
            int a = array[i];
            int j = i - 1;
            int k = i;
            while (j >= 0 && a < array[j]) {
                // 移动
                 array[j+1] = array[j];
                 k = j;
                 j--;
            }
            // 插入
            array[k] = a;
        }
    }

}


```