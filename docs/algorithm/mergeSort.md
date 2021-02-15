##### 归并排序
> 拆分成最小1个元素的的子序列，然后合并成有序的序列，最后就是一个完全有序序列
> 如 1 7 3 6 5 5 6
> 拆分最小序列  {1} {7} {3} {6} {5} {5} {6}
>
> 然后比较并合并相邻的两个序列到新的序列中
>
> 如下A{1 7}, B{3 6}, C{5 5}, D{6}
>
> 再同样合并如下
> 由于子序列都是有序的，只需要从前往后线性比较两个序列的元素，合并到新序列即可
> 如A{1} < B{3} 所以A{1}放入新序列第一个，A序列继续下一个 A{7} > B{3}, 所以B{3}放入新序列第二个，
> B序列继续下一个 A{7} > B{6} 所以 B{6} 放入新序列第三个，最后只剩下A{7} 放入新序列最后一个, 新序列如下
>
> A{1 3 6 7}, C{5 5 6}
>
> 后面继续合并即可得到最后有序的序列
>
> {1 3 5 5 6 6 7}
>

平均时间复杂度O(nlogn)
空间复杂度O(n)
稳定排序


```java
import java.util.Arrays;

public class MergeSort {

    public static void main(String[] args) {
        int[] arraya = {1, 7, 3, 6, 5};
        int[] arrayb = arraya.clone();
        sort(arraya, arrayb, 0, arraya.length - 1);
        System.out.println(Arrays.toString(arrayb));
    }

    private static void sort(int[] array, int[] target, int l, int r) {
        int mid = (l + r) >>> 1;
        int high = r;
        int low = l;
        if (l >= r) {
            return;
        }
       sort(target, array, l, mid);
       sort(target, array, mid+1, r);

        for(int k = low, i = l, j = mid + 1; k <= high ; k++) {
            if (j > r || i <= mid && array[i] <= array[j]) {
                target[k] = array[i];
                i++;
            }else {
               target[k] = array[j];
               j++;
            }
        }
        return ;
    }

}

```