---
layout: post
title: 一些常见的算法java实现
date: 2017-06-09 08:52:28
tags: share
categoriets: share
---


### 冒泡排序算法

```java
/**
 * 冒泡排序算法
 */
public class BubbleSort {
	/**
	 * 方法一
	 * 假如是5个数字相比较，那么只需比较4趟，所以外层循环是长度-1次
	 * 内层循环j比i大1，循环比较，arr[i]大于arr[j]则交换
	 * 这样，第一次 下标为0的位置就是最小的，
	 * 第二次比较i为1，比较到最后，就能选出第二小的，放在下标1的位置
	 */
	public static void sort(int[] arr){
		for(int i=0; i<arr.length-1; i++){
			for(int j=i+1; j<arr.length; j++){
				if(arr[i]>arr[j]){
					int temp = arr[i];
					arr[i] = arr[j];
					arr[j] = temp;
				}
			}
		}
	}
	
	/**
	 * 方法二，本质上和方法一一样
	 * 外层循环控制比较次数，也是数组长度-1
	 * 内层循环下表j从0开始，和下标j+1进行比较
	 * 也就是每两个相邻的比较，这样大的就会放在最后
	 * 下一次循环长度-i,就把最后一位排除了
	 */
	public static void sort2(int[] arr){
		for(int i=0; i<arr.length-1; i++){
			for(int j=0; j<arr.length-i-1; j++){
				if(arr[j]>arr[j+1]){
					int temp = arr[j];
					arr[j] = arr[j+1];
					arr[j+1] = temp;
				}
			}
		}
	}
	
	public static void main(String args[]){
		int[] arr = {10,8,25,7,30,24,33,7,5,7,2,8,24,-1};
		System.out.println("排序前");
		for(int a : arr)
			System.out.print(a + " ");
		
		sort2(arr);
		
		System.out.println();
		System.out.println("排序后");
		for(int a : arr)
			System.out.print(a + " ");
	}
}

```

<!-- more -->

### 选择排序算法

```java
import java.util.Arrays;

/**
 * 选择排序算法
 */
public class SelectionSort {
	/**
	 * 外层循环循环长度-1次
	 * 每次内层循环寻找最小值，并记录最小值小标
	 * 循环结束，把通过下标把最小值所在位置与i所在的位置进行交换
	 * 这样，每次都能找到剩余数组中的最小值，放在前面
	 * @param arr 要进行排序的数组
	 */
	public void sort(int[] arr){
		for(int i=0; i<arr.length-1; i++){
			int minIndex = i;
			for(int j=i+1; j<arr.length; j++){
				if(arr[minIndex] > arr[j]){
					minIndex = j;
				}
			}
			int temp = arr[minIndex];
			arr[minIndex] = arr[i];
			arr[i] = temp;
		}
	}
	
	public static void main(String[] args) {
		int[] arr = new int[]{3,24,2,55,65,67,73,3,48};
		SelectionSort b = new SelectionSort();
		b.sort(arr);
		System.out.println(Arrays.toString(arr));
	}
}

```

### 直接插入排序算法

```java
import java.util.Arrays;

/**
 * 直接插入排序算法
 */
public class InsertSort {
	
	public void sort(int[] arr){
		for(int i=1; i<arr.length; i++){
			int temp = arr[i];//带插入元素
			int j;
			for(j=i-1; j>=0; j--){
				if(temp < arr[j]){
					arr[j+1] = arr[j];//将大于temp的往后移一位
				}else{
					break;
				}
			}
			arr[j+1] = temp;//插入进来
		}
	}
	
	public static void main(String[] args) {
		int[] arr = new int[]{3,24,2,55,65,67,73,3,48};
		InsertSort i = new InsertSort();
		i.sort(arr);
		System.out.println(Arrays.toString(arr));
	}
}
```

### 二分法插入排序算法

```java
/**
 * 二分法插入排序
 */
public class BinaryInsertSort {
	
	public static void sort(int[] arr){
		for(int i=1; i<arr.length; i++){
			int temp = arr[i];
			int left = 0;
			int right = i - 1;
			int mid = 0;
			//确定要插入的位置
			while(left<=right){
				//先获取中间的位置
				mid = (left+right)/2;
				if(temp>arr[mid]){
					//如果值比中间值大，让left右移到中间下标+1
					left = mid + 1;
				}else{
					//如果值比中间值小，让right右移到中间下标+1
					right = mid - 1;
				}
			}
			for(int j=i-1; j>=left; j--){
				//以下表为准，在左位置前插入该数据，左及左后边全部后移
				arr[j+1] = arr[j];
			}
			if(arr[left]!=temp){
				//左位置插入该数据
				arr[left] = temp;
			}
		}
	}
	
	public static void main(String args[]){
		int[] arr = {10,8,25,7,30,24,33,7,5,7,2,8,24,-1};
		System.out.println("排序前");
		for(int a : arr)
			System.out.print(a + " ");
		
		sort(arr);
		
		System.out.println();
		System.out.println("排序后");
		for(int a : arr)
			System.out.print(a + " ");
	}
}
```

### 快速排序算法

```java
import java.util.Arrays;

/**
 * 快速排序算法
 */
public class QuickSort {
	
	private static void quickSort(int[] arr, int left, int right){
		//1.计算基准值下标，并以该元素做为基准值单独保存
		int i = left;
		int j = right;
		int p = (i+j)/2; //计算中间值下标
		int temp = arr[p]; //备份中间值
		//2.分别使用左两边的元素依次与基准值比较大小，将所有比基准值大的元素放在左边，
		//将所有比基准值大于或等于的元素放在右边
		while(i<j){
			while(i<p && arr[i]<temp){ //左边比基准值的小，i++
				i++;
			}
			if(i<p){ //上面的循环结束，代表arr[i]比基准值大
				arr[p] = arr[i]; //把arr[i]赋给基准值arr[p]所在的位置
				p = i; //p指向i的位置
			}
			while(p<j && arr[j]>=temp){ //右边的值大于等于基准值，j--
				j--;
			}
			if(p<j){ //上面的循环结束，代表arr[j]小于或等于基准值
				arr[p] = arr[j]; //把arr[j]赋给基准值arr[p]所在的位置
				p = j; //p指向i的位置
			}
		}
		//3.上面循环结束，代表左右两边下标重合时，就将基准值放在重合的位置
		arr[p] = temp;
		//4.分别对左右两边的分组进行再次分组，使用递归处理
		if(p - left > 1){
			quickSort(arr, left, p-1);
		}
		if(right - p > 1){
			quickSort(arr, p+1, right);
		}
	}
	
	public static void sort(int[] arr){
		quickSort(arr,0,arr.length-1);
	}
	
	public static void main(String[] args) {
		int[] arr = {49,40,30,1,65,97,76,13,27,49,8,6,2};
		System.out.println("排序前");
		System.out.println(Arrays.toString(arr));
		
		sort(arr);
		
		System.out.println("排序后");
		System.out.println(Arrays.toString(arr));
	}
}
```

### 基数排序算法

```java
import java.util.ArrayList;
import java.util.Arrays;

/**
 * 基数排序算法
 */
public class BasicSort {
	public void sort(int[] arr) {
		//1.获取最大值
		int max = 0;
		for(int i=0; i<arr.length; i++){
			if(max<arr[i])
				max = arr[i];
		}
		//2.获取最大值位数
		int times = 0;
		while(max>0){
			max /= 10;
			times++;
		}
		//3.创建二维数组存放数据
		ArrayList<ArrayList<Integer>> queue = new ArrayList<>();
		for(int i=0; i<10; i++){
			ArrayList<Integer> q = new ArrayList<>();
			queue.add(q); 
		}
		//4.外层循环遍历times次，内层循环遍历每位数字
		for(int i=0; i<times; i++){
			for(int j=0; j<arr.length; j++){
				//获取对应位的值，（i为0是各位，1是十位，2是百位）
				int x = arr[j] % (int)Math.pow(10, i+1) / (int)Math.pow(10, i);
				ArrayList<Integer> q = queue.get(x);
				q.add(arr[j]);//把元素添加到对应元素数组
			}
			//收集元素
			int c = 0;
			for(int j=0; j<10; j++){
				ArrayList<Integer> q = queue.get(j);
				while(q.size()>0){
					arr[c] = q.get(0);
					q.remove(0);
					c++;
				}
			}
		}
	}
	
	public static void main(String[] args) {
		int[] arr = {6,13,27,495,78,34,12,64,17,353,685,269,107,8,25,7};
		System.out.println("排序前");
		System.out.println(Arrays.toString(arr));
		
		new BasicSort().sort(arr);
		
		System.out.println("排序后");
		System.out.println(Arrays.toString(arr));
	}
}
```

### 二分查找算法

```java
/**
 * 二分法查找算法
 */
public class BinarySearch {
	
	/**
	 * 递归方法
	 * @param elem 要查找到的元素
	 * @param arr 要查找的数组
	 * @param left 数组起始下标
	 * @param right 数组末尾下标
	 * @return 找到返回对应下表，没找到返回-1
	 */
	public int binarySearch(int elem, int[] arr, int left, int right){
		//如果开始下标大于结束下标，代表没找到，直接返回-1
		if(left>right){
			return -1;
		}
		int middle = (left+right)/2;//计算中间元素下表
		if(arr[middle] > elem){
			//中间值大于要找的元素，去左边找
			return binarySearch(elem, arr, left, middle-1);
		}
		if(arr[middle] < elem){
			//中间值大于要找的元素，去右边找
			return binarySearch(elem, arr, middle+1, right);
		}
		if(arr[middle] == elem){
			//找到了，返回下标
			return middle;
		}
		return -1;
	}
	
	/**
	 * 非递归方法
	 * @param elem 要查找到的元素
	 * @param arr 要查找到的数组
	 * @return 到返回对应下表，没找到返回-1
	 */
	public int directBinarySearch(int elem, int[] arr){
		int left = 0;
		int right = arr.length-1;
		while(left<=right){
			int middle = (left+right)/2;
			if(arr[middle] > elem){
				//中间值大于要找的元素，去左边找
				right = middle - 1;
			}else if(arr[middle] < elem){
				//中间值大于要找的元素，去右边找
				left = middle + 1;
			}else{
				//找到了，返回下标
				return middle;
			}
		}
		return -1;
	}
	
	public static void main(String[] args) {
		int[] arr = new int[]{2,4,5,7,9,10,43,56,60,70,78,79,345};
		BinarySearch bs = new BinarySearch();
		int index = bs.binarySearch(79, arr, 0, arr.length-1);
		int index2 = bs.directBinarySearch(79, arr);
		if(index != -1)
			System.out.println("找到元素了，下标为：" + index);
		else
			System.out.println("没找到元素");
		if(index2 != -1)
			System.out.println("找到元素了，下标为：" + index);
		else
			System.out.println("没找到元素");
	}
}
```