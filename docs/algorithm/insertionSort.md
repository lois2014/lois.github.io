##### 插入排序
> 将一个记录插入到已经排好序的有序表中，从而一个新的、记录数增1的有序表
> 从前往后查询，每次查找前面已排好序的有序表中插入的位置，插入位置之后的元素都向后移动
1. 时间复杂度：O(n^2), 最好的O(n)
2. 稳定排序


如 4 6 5 3 7 4

> 4 6 5 3 7 4
> 4 6 5 3 7 4
> 4 6 **6** 3 7 4
> 4 *5* 6 3 7 4
> 4 **4 5 6 7** 4
> *3* 4 5 6 7 4
> 3 4 4 **5 6 7**
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