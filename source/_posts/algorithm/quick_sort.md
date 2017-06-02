---
categories: 算法
date: 2017-05-31 00:22
title: 快速排序
---

## 1. 算法原理

快速排序是图灵奖得主[ C. R. A. Hoare](http://zh.wikipedia.org/wiki/%E6%9D%B1%E5%B0%BC%C2%B7%E9%9C%8D%E7%88%BE) 于 1960 年提出的一种划分交换排序。它采用了一种**分治**的策略，通常称其为[分治法(Divide-and-ConquerMethod)](http://en.wikipedia.org/wiki/Quicksort)。

分治法的基本思想是：将原问题分解为若干个规模更小但结构与原问题相似的子问题。递归地解这些子问题，然后将这些子问题的解组合为原问题的解。

利用**分治法**可将快速排序的分为三步：

1. 在数据集之中，选择一个元素作为”基准”（pivot）。
2. 所有小于”基准”的元素，都移到”基准”的左边；所有大于”基准”的元素，都移到”基准”的右边。这个操作称为[分区 (partition) 操作](http://en.wikipedia.org/wiki/Quicksort)，分区操作结束后，基准元素所处的位置就是最终排序后它的位置。
3. 对”基准”左边和右边的两个子集，不断重复第一步和第二步，直到所有子集只剩下一个元素为止。

**快排最重要的一步就是划分了。划分的过程用通俗的语言讲就是“挖坑”和“填坑”。**

<!-- more -->



## 2.算法优化

- 优化策略1：三者取中。降低待排序数组本身已接近有序的影响
- 优化策略2：聚集枢轴相同元素。降低待排序数组存在大量相同元素的影响
- 优化策略3：混合使用插入排序。当待排序子集缩小到一定范围后，使用插入排序效率更高。





## 3. 算法实现

```java
/**
 * Created by pc on 2017/5/24.
 */
public class QuickSort {

    public static void quickSort(int arr[],int low,int high){
        int left = low;
        int right = high;
        int temp = 0;

        //待排序的元素至少有两个的情况
        if(left <= right){

            // 三者取中寻找基准元素
            selectPivotMidOfThree(arr,low,high);

            temp = arr[left];       //待排序的第一个元素作为基准元素
            while(left != right){   //从左右两边交替扫描，直到left = right

                while(right > left && arr[right] >= temp)
                    right --;               //从右往左扫描，找到第一个比基准元素小的元素
                arr[left] = arr[right];     //找到这种元素arr[right]后与arr[left]交换

                while(left < right && arr[left] <= temp)
                    left ++;                //从左往右扫描，找到第一个比基准元素大的元素
                arr[right] = arr[left];     //找到这种元素arr[left]后，与arr[right]交换

            }
            arr[right] = temp;      //基准元素归位

            // 将与基准元素相等元素聚集到枢轴周围
            gather(arr,low,high,right);

            // 递归处理枢轴左右子集
            quickSort(arr,low,left-1);  //对基准元素左边的元素进行递归排序
            quickSort(arr, right+1,high);  //对基准元素右边的进行递归排序
        }
    }

    /**
     * 优化策略1：三者取中
     * 作用：降低待排序数组本身已接近有序的影响
     *
     * @param arr
     * @param low
     * @param high
     * @return
     */
    public static int selectPivotMidOfThree(int arr[], int low, int high) {

        int mid = (low + high)/2;
        //使用三数取中法选择枢轴
        if (arr[mid] > arr[high])//目标: arr[mid] <= arr[high]
        {
            swap(arr, mid, high);
        }
        if (arr[low] > arr[high])//目标: arr[low] <= arr[high]
        {
            swap(arr, low, high);
        }
        if (arr[mid] > arr[low]) //目标: arr[low] >= arr[mid]
        {
            swap(arr, mid, low);
        }
        //此时，arr[mid] <= arr[low] <= arr[high]
        return arr[low];
        //low的位置上保存这三个位置中间的值
        //分割时可以直接使用low位置的元素作为枢轴，而不用改变分割函数了
    }


    /**
     * 优化策略2：聚集枢轴相同元素
     * 作用：降低待排序数组存在大量相同元素的影响
     *
     * @param arr
     * @param low
     * @param high
     * @param mid
     */
    public static void gather(int arr[], int low, int high,int mid){

        int left = low;
        int right = mid-1;

        if(left < right) {
            while(left < right ) {

                // 在右边找到一个比枢轴小的元素
                while(left < right && arr[right] == arr[mid]){
                    right--;
                }

                // 在左边找到一个和枢轴相等的元素
                while(left < right && arr[left] != arr[mid]){
                    left++;
                }

                // 交换两个元素，使相同元素聚集到枢轴周围
                if(left!=right) {
                    swap(arr,left,right);
                }
            }
        }


        left = mid+1;
        right = high;

        if(left < right) {
            while(left < right ) {

                while(left < right && arr[left] == arr[mid]){
                    left++;
                }

                while(left < right && arr[right] != arr[mid]){
                    right--;
                }

                if(left!=right) {
                    swap(arr,left,right);
                }
            }
        }
    }

    // 优化策略3：当待排序子集缩小到一定范围后，使用插入排序效率更高


    public static void swap(int arr[], int a, int b) {
        int tmp = arr[a];
        arr[a] = arr[b];
        arr[b] = tmp;
    }

    public static void main(String[] args) {
        int array[] = {15,6,6,10,6,5,6,3,1,6,7,6,2,8,6};
        System.out.println("排序之前：");
        for(int element : array){
            System.out.print(element+" ");
        }

        quickSort(array,0,array.length-1);

        System.out.println("\n排序之后：");
        for(int element : array){
            System.out.print(element+" ");
        }

    }

}

```



参考：

[常见排序算法 - 快速排序 (Quick Sort)](http://bubkoo.com/2014/01/12/sort-algorithm/quick-sort/)

[图解快速排序](http://www.cnblogs.com/MOBIN/p/4681369.html)