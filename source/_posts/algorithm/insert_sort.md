---
categories: 算法
date: 2017-06-2 16:12
title: 插入排序
---



## 1. 算法原理

- 设有一组关键字｛K1， K2，…， Kn｝；排序开始就认为 K1 是一个有序序列；让 K2 插入上述表长为 1 的有序序列，使之成为一个表长为 2 的有序序列；然后让 K3 插入上述表长为 2 的有序序列，使之成为一个表长为 3 的有序序列；依次类推，最后让 Kn 插入上述表长为 n-1 的有序序列，得一个表长为 n 的有序序列。


- 具体算法描述如下：
1. 从第一个元素开始，该元素可以认为已经被排序
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置
4. 重复步骤 3，直到找到已排序的元素小于或者等于新元素的位置
5. 将新元素插入到该位置后（注：或者新元素本身就是最小的，插入到数组第一个位置）
6. 重复步骤 2~5

<!-- more -->



## 2. 算法优化

如果比较操作的代价比交换操作大的话，可以采用`二分查找法`减少比较操作的数目。

该算法可以认为是插入排序的一个变种，称为**`二分查找排序`**。



## 3. 算法实现

```java
public class InsertSort {

      public static void main(String[] args) {

        int array[] = {15,6,6,10,6,5,6,3,1,6,7,6,2,8,6};
        System.out.println("排序之前：");
        for(int element : array){
            System.out.print(element+" ");
        }

        insertSort(array);
        // 改进的二分查找插入排序
        //binarySort(array);

        System.out.println("\n排序之后：");
        for(int element : array){
            System.out.print(element+" ");
        }

    }
  
  	/**
     * 直接插入排序
     * @param array
     */
    public static void insertSort(int[] array) {
        int length = array.length;
        int i,j,temp;

        for (i = 1; i < length; i++) {

            temp = array[i];                  // 从第二个元素开始排序
            for (j = i; j > 0; j--) {          // 从待排序元素开始往前寻找小于等于该元素的元素
                if (array[j - 1] > temp) {
                    array[j] = array[j - 1];
                } else {
                    array[j] = temp;            // 插入到合适的位置
                    break;
                }
            }
            if(j==0)
                array[j] = temp;
        }
    }
    
    /**
     * 二分搜索查找插入点
     * @param array
     * @param start
     * @param end
     * @param temp
     * @return
     */
    public static int binarySearch(int[] array, int start, int end, int temp) {
        int middle;
        while (start < end) {
            middle = (int) (start + end) / 2;
            if (temp >= array[middle]) {                                // 大于等于中间值
                if (middle + 1<= end && temp < array[middle + 1]) {     // 小于中间值后面的值
                    return middle + 1;
                } else {
                    start = middle + 1;
                }
            } else {                                                    // 小于中间值
                if (middle - 1 >= start && temp >= array[middle - 1]) { // 大于等于中间值前面的值
                    return middle;
                } else {
                    end = middle - 1;
                }
            }
        }

        return start;
    }

    /**
     * 二分查找插入排序
     * @param array
     */
    public static void binarySort(int[] array) {
        int length = array.length;
        int i,j,k,temp;
        for (i = 1; i < length; i++) {
            temp = array[i];
            if (temp >= array[i - 1]) {     // 待排序元素大于等于已排序的数组
                k = i;
            } else {
                k = binarySearch(array, 0, i - 1, temp);    // 二分查找插入点
                for (j = i; j > k; j--) {
                    array[j] = array[j - 1];
                }
                array[k] = temp;
            }
        }
    }
}

```


参考：[常见排序算法 - 插入排序 (Insertion Sort)](http://bubkoo.com/2014/01/14/sort-algorithm/insertion-sort/)