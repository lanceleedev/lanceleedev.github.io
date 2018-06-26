## 一、概述
这个数列前两项为1，从第3项开始，每一项都等于前两项之和。

属于面试经常问的一个算法题，简单那是因为考虑的少，如果你跑一下程序，你就知道为什么你觉得简单了，下面进行分析及测试，以及为什么说不简单的原因。
## 二、编码及测试
### 1、常见的递归写法
``` markdown 
/**
 * 计算斐波那契数列项
 * @param i 第n项
 * @return 第n项对应结果
 */
private static Long fib(int i) {
    if(1 == i || 2 == i){
        return 1L;
    }
    return fib(i-1) + fib(i-2);
}
```
#### 测试结果[`机器为16G，4核`]：
19及以下：1ms  
20：2ms  
45：4900ms  
50：53000ms  11位结果
#### 存在问题：
（1）存储：long只能存储2^63-1，19位，只能算到92  
（2）效率：每次都要把之前的计算一次，呈指数级的增长
### 2、改进性能，使用缓存  
``` markdown
/** 记录计算好的斐波那契数列值 */
private static Map<Integer, Long> record = new HashMap<Integer, Long>();
/**
 * 计算斐波那契数列项
 * @param i 第n项
 * @return 第n项对应结果
 */
private static Long fibUseCache(int i) {
    if(1 == i || 2 == i){
	    return 1L;
    }
    Long result = record.get(i);
    if(null == result){
	    result = fibUseCache(i-1) + fibUseCache(i-2);
    }
    record.put(i, result);
    return result;
}
```
#### 测试结果[`机器为16G，4核`]：
91及以下：1-2ms  
92：1-2ms
#### 存在问题：
（1）存储，只能算到92
### 3、改变计算方式
``` markdown
/**
 * 计算斐波那契数列项
 * @param i 第n项
 * @return 第n项对应结果
 */
private static Long fibChangeArith(int i) {
    if(1 == i || 2 == i){
	    return 1L;
    }
    long a1 = 1;
    long a2 = 1;
    while(i-- > 1){
        long temp = a1;
        a1 = a1 + a2;
        a2 = temp;
    }
    return a2;
}
```
#### 测试结果[`机器为16G，4核`]：
92：0ms，结果在毫秒级以内
#### 存在问题：
（1）存储，只能算到92
### 4、利用String存储计算结果
``` markdown
/**
 * 计算斐波那契数列项,无i的限制
 * @param i 第n项
 * @return 第n项对应结果
 */
private static String fibFinal(int i) {
    String a1 = "1";
    String a2 = "1";
    while(i-- > 1){
    	String temp = a1;
        a1 = addLongStr(a1, a2);
        a2 = temp;
    }
    return a2;
}

/**
 * 10进制,long字符串相加.<br>
 * 思路:每18位计算一次,[long支持19位,防止最高位为9的情况],先将长度相等的一段计算出来,然后将剩下的最高位加上.<br>
 * note:进位问题
 * @param str1 参数1
 * @param str2 参数2
 * @return 两字符串相加后的结果
 */
private static String addLongStr(String str1, String str2){
    StringBuilder result = new StringBuilder();
    int len1 = str1.length();
    int len2 = str2.length();
    /** 截取的长度,每18位计算一次*/
    int cutLength = 18;
    int len = len1 < len2 ? len1 : len2;
    int count = len / cutLength ;
    int remainder = len % cutLength;
    if(remainder > 0){
    	count ++;
    }
    
    /** 相加后的进位*/
    long up_position = 0;
    for(int i = count;i > 0;i--){
    	long temp1 = Long.valueOf(str1.substring(subIndex(len1, cutLength), len1));
    	long temp2 = Long.valueOf(str2.substring(subIndex(len2, cutLength), len2));
    	len1 = subIndex(len1, cutLength);
    	len2 = subIndex(len2, cutLength);
    	/** 每次相加后的结果*/
    	temp2 = temp1 + temp2 + up_position;
    	String temp = temp2 + "";
    	if(temp.length() > cutLength){
    		up_position = Integer.valueOf(temp.substring(0, 1));
    		result.insert(0, temp.substring(1, temp.length()));
    	} else {
    		up_position = 0;
    		result.insert(0, temp);
    	}
    }
    
    /** 计算剩下的最高位 */
    long maxSegment = up_position;
    if(len1 > len2){
    	maxSegment += Long.valueOf(str1.substring(0, len1));
    } else if(len1 < len2){
    	maxSegment += Long.valueOf(str2.substring(0, len2));			
    }
    if(maxSegment > 0){
    	result.insert(0, maxSegment + "");
    }
    return result.toString();
}

/**
 * 对下标做减法，保证没有下标越界
 * @param index 下标
 * @param sub 移动长度
 * @return 移动后的结果
 */
private static int subIndex(int index, int sub){
    int temp = index - sub;
    return temp < 0 ? 0 : temp;
}
```
#### 测试结果[`机器为16G，4核`]：
1000：1ms
***
最后，你知道斐波那契数列的1000位有多大吗?207位