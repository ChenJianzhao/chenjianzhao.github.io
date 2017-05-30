---
categories: 算法
date: 2017-05-31 00:22
title: 快速排序
---

## 1. 快速排序的思路

待补充。

<!-- more -->

## 2. 快速排序的实现

```java
/**
 * Created by pc on 2017/5/24.
 */
public class QuickSort {

    public static void quickSort(int arr[],int low,int high){
        int left = low;
        int right = high;
        int temp = 0;
        if(left <= right){   //待排序的元素至少有两个的情况

            selectPivotMidOfThree(arr,low,high);

            temp = arr[left];  //待排序的第一个元素作为基准元素
            while(left != right){   //从左右两边交替扫描，直到left = right

                while(right > left && arr[right] >= temp)
                    right --;        //从右往左扫描，找到第一个比基准元素小的元素
                arr[left] = arr[right];  //找到这种元素arr[right]后与arr[left]交换

                while(left < right && arr[left] <= temp)
                    left ++;         //从左往右扫描，找到第一个比基准元素大的元素
                arr[right] = arr[left];  //找到这种元素arr[left]后，与arr[right]交换

            }
            arr[right] = temp;    //基准元素归位

            agg(arr,low,high,right);

            quickSort(arr,low,left-1);  //对基准元素左边的元素进行递归排序
            quickSort(arr, right+1,high);  //对基准元素右边的进行递归排序
        }
    }

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

    public static void swap(int arr[], int a, int b) {
        int tmp = arr[a];
        arr[a] = arr[b];
        arr[b] = tmp;
    }

    public static void agg(int arr[], int low, int high,int mid){
        int left = low;
        int right = mid-1;

        if(left < right) {
            while(left < right ) {

                while(left < right && arr[right] == arr[mid]){
                    right--;
                }


                while(left < right && arr[left] != arr[mid]){
                    left++;
                }
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

