---
categories: 算法
date: 2017-9-29 16:00
title: 鸡尾酒排序
---



## 算法原理

为什么叫鸡尾酒排序？其实我也不知道，知道的小伙伴请告诉我。

其实它还有很多**奇怪**的名称，比如双向冒泡排序 (Bidirectional Bubble Sort)、波浪排序 (Ripple Sort)、摇曳排序 (Shuffle Sort)、飞梭排序 (Shuttle Sort) 和欢乐时光排序 (Happy Hour Sort)。本文中就以鸡尾酒排序来称呼它。

鸡尾酒排序是[冒泡排序](http://bubkoo.com/2014/01/12/sort-algorithm/bubble-sort/)的轻微变形。不同的地方在于，鸡尾酒排序是从低到高然后从高到低来回排序，而冒泡排序则仅从低到高去比较序列里的每个元素。他可比冒泡排序的效率稍微好一点，原因是冒泡排序只从一个方向进行比对(由低到高)，每次循环只移动一个项目。

以序列(2,3,4,5,1)为例，鸡尾酒排序只需要访问一次序列就可以完成排序，但如果使用冒泡排序则需要四次。但是在乱数序列状态下，鸡尾酒排序与冒泡排序的效率都很差劲，优点只有原理简单这一点。

排序过程：

1. 先对数组从左到右进行冒泡排序（升序），则最大的元素去到最右端
2. 再对数组从右到左进行冒泡排序（降序），则最小的元素去到最左端
3. 以此类推，依次改变冒泡的方向，并不断缩小未排序元素的范围，直到最后一个元素结束



## 算法实现

```java

public class CocktailSort {

	public static void main(String[] args) {
		 int array[] = {15,6,6,10,6,5,6,3,1,6,7,6,2,8,6};
	        System.out.println("排序之前：");
	        for(int element : array){
	            System.out.print(element+" ");
	        }

	        cocktailSort(array);

	        System.out.println("\n排序之后：");
	        for(int element : array){
	            System.out.print(element+" ");
	        }
	}
	
	public static void cocktailSort(int[] array) {
	    
	    int length = array.length,
	        left = 0,
	        right = length - 1,
	        lastSwappedLeft = left,
	        lastSwappedRight = right,
	        i,
	        j;
	    while (left < right) {
	        // 从左到右
	        lastSwappedRight = 0;
	        for (i = left; i < right; i++) {
	            if (array[i] > array[i + 1]) {
	                swap(array, i, i + 1);
	                lastSwappedRight = i;
	            }
	        }
	        right = lastSwappedRight;
	        // 从右到左
	        lastSwappedLeft = length - 1;
	        for (j = right; left < j; j--) {
	            if (array[j - 1] > array[j]) {
	                swap(array, j - 1, j);
	                lastSwappedLeft = j;
	            }
	        }
	        left = lastSwappedLeft;
	    }	
	}
	
	public static void swap(int[] array, int i, int j) {
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
}


```



[常见排序算法 - 鸡尾酒排序 (Cocktail Sort/Shaker Sort)](http://bubkoo.com/2014/01/15/sort-algorithm/shaker-sort/)