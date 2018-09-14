---
layout: post
title: java代码实现一些特殊功能
date: 2017-06-09 08:36:45
tags: share
categoriets: share
---


### 打印矩形、三角形、菱形、空心菱形等

原理及实现

```java
/**
 * 打印星星
 */
public class PrintStar {

	public static void main(String[] args) {

		// 定义打印的个数
		int num = 5;

		System.out.println("矩形");
        //双重for循环即可实现
		for (int i = 1; i <= num; i++) {
			for (int j = 1; j <= num; j++) {
				System.out.print("*");
			}
			System.out.println();
		}
		System.out.println();

		System.out.println("直角三角形");
        //外从循环i从1到num，内层循环j从1到2*i-1
		for (int i = 1; i <= num; i++) {
			for (int j = 1; j <= 2 * i - 1; j++) {
				System.out.print("*");
			}
			System.out.println();
		}
		System.out.println();

		System.out.println("等腰三角形");
        //外从循环i从1到num，内从循环k从1到num-i(这里是打印*前面的空格),内层循环j从1到2*i-1
		for (int i = 1; i <= num; i++) {
			for (int k = 1; k <= num - i; k++) {
				System.out.print(" ");
			}
			for (int j = 1; j <= 2 * i - 1; j++) {
				System.out.print("*");
			}
			System.out.println();
		}
		System.out.println();

		System.out.println("菱形");
        //菱形由一个等腰三角形和一个倒三角组成，先打印等腰三角形，这样就打印了菱形的上半部分，
        //再把i的初始值与条件表达式值进行交换，在把i++变为i--，即可打印倒直角三角形，由于菱形中间只需一行中心，初始值前去1即可
		for (int i = 1; i <= num; i++) {
			for (int k = 1; k <= num - i; k++) {
				System.out.print(" ");
			}
			for (int j = 1; j <= 2 * i - 1; j++) {
				System.out.print("*");
			}
			System.out.println();
		}
		for (int i = num - 1; i >= 1; i--) {
			for (int k = 1; k <= num - i; k++) {
				System.out.print(" ");
			}
			for (int j = 1; j <= 2 * i - 1; j++) {
				System.out.print("*");
			}
			System.out.println();
		}
		System.out.println();

		System.out.println("三角形的两边");
        //在等腰三角形的基础上进行判断，如果j等于当前行的初始值或循环变量终止值时，
        //即(j == 1 || j == 2 * i - 1)成立，就代表是当前行的第一个星或者最后一个星，打印*,否则打印空格
		for (int i = 1; i <= num; i++) {
			for (int k = 1; k <= num - i; k++) {
				System.out.print(" ");
			}
			for (int j = 1; j <= 2 * i - 1; j++) {
				if (j == 1 || j == 2 * i - 1) {
					System.out.print("*");
				} else {
					System.out.print(" ");
				}
			}
			System.out.println();
		}
		System.out.println();

		System.out.println("空心三角型");
        //在打印三角形的两边的基础上，增加一个逻辑，外层循环i初始值等于i的终止值num时，代表最后一行，即(i == num),不是最后一行的正常打印，否则打印*
		for (int i = 1; i <= num; i++) {
			for (int k = 1; k <= num - i; k++) {
				System.out.print(" ");
			}
			for (int j = 1; j <= 2 * i - 1; j++) {
				if (!(i == num)) {
					if (j == 1 || j == 2 * i - 1) {
						System.out.print("*");
					} else {
						System.out.print(" ");
					}
				} else {
					System.out.print("*");
				}
			}
			System.out.println();
		}
		System.out.println();

		System.out.println("空心菱形");
        //打印空心菱形原理与通过三角形打印菱形原理一致，先打印三角形的两边，再打印少一行的倒三角形的两边即可。
		for (int i = 1; i <= num; i++) {
			for (int k = 1; k <= num - i; k++) {
				System.out.print(" ");
			}
			for (int j = 1; j <= 2 * i - 1; j++) {
				if (j == 1 || j == 2 * i - 1) {
					System.out.print("*");
				} else {
					System.out.print(" ");
				}
			}
			System.out.println();
		}
		for (int i = num - 1; i >= 1; i--) {
			for (int k = 1; k <= num - i; k++) {
				System.out.print(" ");
			}
			for (int j = 1; j <= 2 * i - 1; j++) {
				if (j == 1 || j == 2 * i - 1) {
					System.out.print("*");
				} else {
					System.out.print(" ");
				}
			}
			System.out.println();
		}
		System.out.println();
	}
}
```

<!-- more -->

### 打印水仙花数

>水仙花数是指一个 n 位数（n≥3 ），它的每个位上的数字的 n 次幂之和等于它本身（例如：1^3 + 5^3+ 3^3 = 153）

```java
/**
 * 计算并三位数打印水仙花数
 */

public class NarcissisticNumber {
	public static void main(String[] args) {
		
		//方式一
		for (int i = 100; i <= 999; i++) {
			int bai = i / 100; //分解出百位数
			int shi = i / 10 % 10; //分解出十位数
			int ge = i % 10; //分解出个位数
			if (i == Math.pow(bai, 3) + Math.pow(shi, 3) + Math.pow(ge, 3))
			System.out.print(i + "\t");
		}
		
		System.out.println();
		
		//方式二
		for (int i = 1; i <= 9; i++)
			for (int j = 0; j <= 9; j++)
				for (int k = 0; k <= 9; k++)
					if (i * 100 + j * 10 + k == Math.pow(i, 3) + Math.pow(j, 3) + Math.pow(k, 3))
						System.out.print(i * 100 + j * 10 + k + "\t");	
	}
}
```

### 生成指定范围的随机整数

```java
import java.util.Random;

/**
 * 求指定范围的随机整数
 */

public class TestRandom {
	// 方法一：
	public static void RandomNumber(int min, int max) {
		double ran = Math.random() * (max - min + 1) + min;// 获取一个制定范围的随机小数
		int r = (int) Math.floor(ran);// 转换成整数
		System.out.println(r);
	}

	// 方法二：
	public static void RandomNumber2(int min, int max) {
		Random ra = new Random();
		int ran = ra.nextInt(max - min + 1) + min;// 获取一个制定范围的随机数
		System.out.println(ran);
	}

	public static void main(String[] args) {
		RandomNumber(1, 10);
		RandomNumber2(1, 10);
	}
}
```

### 生成指定范围内不重复的随机数

```java
import java.util.HashSet;
import java.util.Random;
import java.util.Set;

public class RandomNumber {
	/**
	 * 方法一：通过Set集合的特性，生成随机数
	 * 
	 * @param min
	 *            随机数的开始范围
	 * @param max
	 *            随机数的结束范围
	 * @param num
	 *            随机数的个数
	 */
	public static void makeRandomNumber(int min, int max, int num) {
		Random ra = new Random();
		Set<Integer> si = new HashSet<>();// 创建set集合，存放随机数

		for (int i = 0; i < num; i++) {
			int ranNum = ra.nextInt(max - min + 1) + min; // 获取开始与结束之间的随机数
			if (!si.add(ranNum)) // 把随机数存到集合中,如果添加失败，代表重复，i--，重新循环
				i--;
		}
		for (int i : si)// 遍历并打印集合
			System.out.print(i + "\t");
		System.out.println();
	}

	/**
	 * 方法二：通过新随机的数与前一个比较，如果当等，代表重复，i--，重新循环
	 * 
	 * @param min
	 *            随机数的开始范围
	 * @param max
	 *            随机数的结束范围
	 * @param num
	 *            随机数的个数
	 */
	public static void makeRandomNumber2(int min, int max, int num) {
		Random ra = new Random();
		int[] arr = new int[num]; // 存放随机数数的数组

		for (int i = 0; i < num; i++) {
			int ranNum = ra.nextInt(max - min + 1) + min; // 获取开始与结束之间的随机数
			arr[i] = ranNum;
			for (int j = 0; j <= i - 1; j++) {
				if (arr[i] == arr[j]) { // 当前随机数与前一个相等，i--，重新随机
					i--;
					break;
				}
			}
		}
		for (int i : arr)
			System.out.print(i + "\t");
		System.out.println();

	}

	/**
	 * 方法三：通过有效个数和标志来判断是否重复
	 * 
	 * @param min
	 *            随机数的开始范围
	 * @param max
	 *            随机数的结束范围
	 * @param num
	 *            随机数的个数
	 */
	public static void makeRandomNumber3(int min, int max, int num) {
		Random ra = new Random();
		int[] arr = new int[num]; // 存放随机数数的数组

		int count = 0; // 计算有效个数

		while (count < arr.length) {
			int ranNum = ra.nextInt(max - min + 1) + min; // 获取开始与结束之间的随机数
			boolean flag = true; // 定一个标志，用来判断是否重复
			for (int i = 0; i < arr.length; i++) {
				if (ranNum == arr[i]) {
					flag = false;
					break;
				}
			}
			if (flag) {
				arr[count] = ranNum;
				count++; // 有效个数加一
			}
		}
		for (int i : arr)
			System.out.print(i + "\t");
		System.out.println();
	}

	public static void main(String[] args) {
		System.out.println("方法一：");
		RandomNumber.makeRandomNumber(10, 20, 5);
		System.out.println("方法二：");
		RandomNumber.makeRandomNumber2(10, 20, 5);
		System.out.println("方法三：");
		RandomNumber.makeRandomNumber3(10, 20, 5);
	}
}
```


### 正整数的逆序输出

```java
import java.util.Scanner;

/**
 * 实现用户输入正整数的逆序输出
 */

public class NumberReverse {

	/**
	 * 把一个数字进行逆序并返回
	 * 
	 * @param num 要进行逆序的数字
	 * @return 返回逆序后的数字
	 */
	public static int numRev(int num) {
		int res = 0;
		while (num > 0) {
			res = res * 10 + num % 10; // 取出最后一位并存入结果
			num /= 10; // 去掉最后以为
		}
		return res;
	}

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		System.out.println("请输入一个正整数：");
		int num = sc.nextInt();
		System.out.println(num + "逆序后：" + numRev(num));
		sc.close();
	}
}
```

### 统计数字出现次数
```java
import java.util.Scanner;

/**
 * 用户输入，统计数字出现重复次数
 */

public class NumberRepeat {
	/**
	 * 计算数字出现次数
	 * 
	 * @param num 用户输入的数字
	 * @return 统计重复次数后返回的数组
	 */
	public static int[] numberRepeatCount(int num) {
		int arr[] = new int[10]; //定义存放数字出现次数的数组
		while (num > 0) {
			arr[num % 10]++; //取出最后一位数字当作数组下表，并加1
			num /= 10; //删除最后一位数
		}
		return arr; //返回统计后的数组
	}

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		System.out.println("请输入任意整数：");
		int num = sc.nextInt();
		int arr[] = numberRepeatCount(num);
		for (int i = 0; i < arr.length; i++)
			if (arr[i] != 0)
				System.out.println(i + "出现了" + arr[i] + "次");
		sc.close();
	}

}
```

### 计算阶乘 

```java
/**
 * 计算阶乘 
 */

public class Factorial {
	
	/**
	 * @param num 要计算阶乘的数字
	 * @return 返回计算阶乘后的数字
	 */
	public static int countFactorial(int num){
		int res = 1;
		for(int i=1; i<=num; i++)
			res *=i;
		return res;
	}
	
	/**
	 * @param num 要计算阶乘的数字
	 * @return 返回计算阶乘后的数字
	 */
	public static int countFactorial2(int num){
		if(num == 1)
			return 1;
		return num * countFactorial2(num -1);
	}
	
	public static void main(String[] args) {
		int num = 5;
		System.out.println("递推计算" + num + "的阶乘为：" + countFactorial(num));
		System.out.println("递归计算" + num + "的阶乘为：" + countFactorial2(num));
	}

}
```

### 计算第n位的斐波那契数列的值

>斐波那契数列指的是这样一个数列 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233，377，610，987，1597........
这个数列从第3项开始，每一项都等于前两项之和。

```java
/*
 * 实现求第n位的斐波那契数列的值
 */

public class FibonacciSequence {

	/**
	 * 递归求斐波那契数列
	 * 
	 * @param num 要求的第n项
	 * @return 斐波那契数列第n项的值
	 */
	public static int getFib(int n) {
		if (n <= 2) 
			return 1;
		return getFib(n - 1) + getFib(n - 2); 
	}

	/**
	 * 递推求斐波那契数列
	 * 
	 * @param num 要求的第n项
	 * @return 斐波那契数列第n项的值
	 */
	public static int getFib2(int n) {
		if (n <= 2)
			return 1;
		int a = 1; //数列第一项的值位1
		int b = 1; //数列第二项的值位1
		int c = 0; //数列第三项的值
		for (int i = 3; i <= n; i++) {
			c = a + b;
			a = b;
			b = c;
		}
		return c;
	}

	public static void main(String[] args) {
		int n = 10;
		for (int i = 1; i <= n; i++) {
			System.out.print(getFib(i) + " ");
		}
		System.out.println();
		for (int i = 1; i <= n; i++) {
			System.out.print(getFib2(i) + " ");
		}
	}

}
```

### 统计字符串中每组数字出现的次数

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * 统计字符串中每组数字出现的次数
 */

public class CountStringNum {
	/**
	 * 统计字符串中数字出现的次数，并返回Map集合
	 * 
	 * @param str 要统计的字符串
	 * @return 返回统计后的字符串集合
	 */
	public static Map<String, Integer> count(String str) {
		// 用","把字符串分割并存放在数组里
		String[] arr = str.split(",");
		// 创建Map集合来存放统计结果，键为要统计的字符，值位出现的次数
		Map<String, Integer> map = new HashMap<>();
		// 循环遍历来进行统计
		for (int i = 0; i < arr.length; i++)
			// 判断当前键中是否存在要统计的数，存在取出当前已统计次数，加1；
			//不存在就代表第一次出现，把要统计的数当作键存入集合，值为1
			if (map.containsKey(arr[i]))
				map.put(arr[i], map.get(arr[i]) + 1);
			else
				map.put(arr[i], 1);
		return map; // 最后返回统计好的集合
	}

	public static void main(String[] args) {
		// 要统计的字符串
		String str = "123,456,123,456,789,678,567,456";
		// 调用统计方法并接受统计好的数据
		Map<String, Integer> map = count(str);
		// 打印Set集合
		Set<Map.Entry<String, Integer>> sme = map.entrySet();
		for (Map.Entry<String, Integer> me : sme)
			System.out.println(me.getKey() + " 出现了 " + me.getValue() + " 次");
	}

}
```