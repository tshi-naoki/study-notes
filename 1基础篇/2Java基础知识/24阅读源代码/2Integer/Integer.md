# 类定义

```java
public final class Integer extends Number implements Comparable<Integer>{
   ......
}
```

我们可以了解到以下信息：

- Integer不可以被继承；
- 继承Number类，所以有intValue()、longValue()、floatVaule()、doubleValue()、byteValue()和shortValue()方法；
- 实现了Comparable接口，可以使用compareTo方法，并且只能和Integer进行比较。

# 属性

```java
private final int value;
@Native public static final int   MIN_VALUE = 0x80000000;//-2147483648
@Native public static final int   MAX_VALUE = 0x7fffffff;//2147483647
//表示基本类型 int 的 Class 实例。
public static final Class  TYPE = (Class) Class.getPrimitiveClass("int");
//用来以二进制补码形式表示 int 值的比特位数。
public static final int SIZE = 32;
//用来以二进制补码形式表示 int 值的字节数。1.8以后才有
public static final int BYTES = SIZE / Byte.SIZE;
```

Integer的值存在value中，并且我们可以发现value是不可更改的。那么为什么我们初始化一个Integer对象后还可以改变它的值呢？其实本质上是新创建了一个对象，将原对象指向新对象而已。

```java
Integer i = new Integer(1);
System.out.println(i);//1
i = 5;//底层==>i = Integer.valueOf(5);
System.out.println(i);//5
```

# 构造方法

直接传入int类型的值创建Integer对象：

```java
public Integer(int value) {
    this.value = value;
}
```

传入String类型，使用parseInt()函数将字符串转为十进制的数字赋值给value。转换出错则抛出NumberFormatException：

```java
public Integer(String s) throws NumberFormatException {
     this.value = parseInt(s, 10);
}
```

# valueOf(int i)

```java
public static Integer valueOf(int i) {
	if (i >= IntegerCache.low &amp;&amp; i <= IntegerCache.high)//判断i是否在缓存的数字范围内
		return IntegerCache.cache[i + (-IntegerCache.low)];//直接返回已经缓存的对象
	return new Integer(i);//不再缓存的数字范围，直接新建对象返回
}
```

在默认情况向low=-128，high=127，但是high是可以通过配置进行修改的。IntegerCache代码如下：

```java
private static class IntegerCache {
	static final int low = -128;
	static final int high;
	static final Integer cache[];

	static {
		// high value may be configured by property//说明了high的值是可以修改的
		int h = 127;
		String integerCacheHighPropValue =
			sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");//配置java.lang.Integer.IntegerCache.high即可修改high的值了
		if (integerCacheHighPropValue != null) {
			try {
				int i = parseInt(integerCacheHighPropValue);
				i = Math.max(i, 127);
				// Maximum array size is Integer.MAX_VALUE
				h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
			} catch( NumberFormatException nfe) {
				// If the property cannot be parsed into an int, ignore it.
			}
		}
		high = h;

		cache = new Integer[(high - low) + 1];
		int j = low;
		for(int k = 0; k < cache.length; k++)
			cache[k] = new Integer(j++);//所谓的缓存就是一个数组，里面已经创建了[-128，,127]的所有实例

		// range [-128, 127] must be interned (JLS7 5.1.7)
		assert IntegerCache.high >= 127;
	}

	private IntegerCache() {}
}
```

当把一个int变量转成Integer的时候（或者新建一个Integer的时候），建议使用valueOf方法来代替构造函数。或者直接使用Integer i = 100;编译器会转成Integer s = Integer.valueOf(10000)。可以重用IntegerCache中的缓存。

```java
Integer i = new Integer(100);
Integer j = new Integer(100);
System.out.println(i==j);//false
Integer m = Integer.valueOf(100);
Integer n = Integer.valueOf(100);
System.out.println(m==n);//true
```

# String转int相关

## parseInt(String s, int radix)

将字符串s转为radix进制的数字，支持的进制范围为[2,36]，超过范围抛出NumberFormatException。如果转换错误也抛出NumberFormatException。需要注意的是，**parseInt返回的是int类型**。

```java
public static int parseInt(String s, int radix) throws NumberFormatException{
	//省略参数校验部分代码
	int result = 0;
	boolean negative = false;
	int i = 0, len = s.length();
	int limit = -Integer.MAX_VALUE;
	int multmin;
	int digit;

	if (len > 0) {
		char firstChar = s.charAt(0);//判断第一个字符是不是'-'或'+'
		if (firstChar < '0') { // Possible leading "+" or "-"
			if (firstChar == '-') {
				negative = true;//首字符为'-'，说明是负数
				limit = Integer.MIN_VALUE;//最小值为Integer.MIN_VALUE
			} else if (firstChar != '+')
				throw NumberFormatException.forInputString(s);//首字符不是'-'或'+',直接抛出异常

			if (len == 1) // Cannot have lone "+" or "-"
				throw NumberFormatException.forInputString(s);//不能单独一个'-'或'+'
			i++;
		}
		multmin = limit / radix;
		//假设最小范围为?-2147483647?，那么在等同于-?214748364*10-digit，
		//在加最后一位前如果发现result<-?214748364，那么最后结果一定是超出范围了?
		while (i < len) {
			// Accumulating negatively avoids surprises near MAX_VALUE
			digit = Character.digit(s.charAt(i++),radix);//获取字符对应进制的数字
			if (digit < 0) {
				throw NumberFormatException.forInputString(s);
			}
			if (result < multmin) {
				throw NumberFormatException.forInputString(s);//超出Integer范围
			}
			result *= radix;
			if (result < limit + digit) {
				throw NumberFormatException.forInputString(s);//超出Integer范围
			}
			result -= digit;
		}
	} else {
		throw NumberFormatException.forInputString(s);
	}
	return negative ? result : -result;//负数直接返回result，正数取反
}
```

## getInteger(String nm,Integer val)

使用指定的名字获取系统属性值。如果没有对应名字的属性且val没有指定默认值则返回null，否则返回系统属性值或val。getInteger返回的是Integer类型。

```java
Properties p = System.getProperties();
p.put("key1","100");
System.out.println(Integer.getInteger("key1"));//100
System.out.println(Integer.getInteger("key2"));//null
System.out.println(Integer.getInteger("key2",500));//500
p.put("key3",300);//值为int类型，不是String类型
System.out.println(Integer.getInteger("key3"));//null
```

# int转String相关

## toString(int i)

```java
public static String toString(int i) {
	if (i == Integer.MIN_VALUE)//由于2147483648大于Integer.MAX_VALUE,所以需要单独处理
		return "-2147483648";
	int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);//确定申请char[]的大小
	char[] buf = new char[size];
	getChars(i, size, buf);
	return new String(buf, true);
}

final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
								  99999999, 999999999, Integer.MAX_VALUE };

//确定需要多少位
static int stringSize(int x) {
	for (int i=0; ; i++)
		if (x <= sizeTable[i])
			return i+1;
}

static void getChars(int i, int index, char[] buf) {
	int q, r;
	int charPos = index;
	char sign = 0;

	if (i < 0) {
		sign = '-';//是负数，首位为'-'号
		i = -i;
	}

	// 每一次需要会产生最后两位数字
	while (i >= 65536) {
		q = i / 100;
		// really: r = i - (q * 100);
		r = i - ((q << 6) + (q << 5) + (q << 2));//移位的效率比直接乘除的效率要高
		i = q;
		buf [--charPos] = DigitOnes[r];//获取个位上的数字
		buf [--charPos] = DigitTens[r];//获取十位上的数字
	}

	// Fall thru to fast mode for smaller numbers
	// assert(i <= 65536, i);
	for (;;) {
		q = (i * 52429) >>> (16+3);//这里其实就是除以10//移位的效率比直接乘除的效率要高//乘法效率比除法高
		r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
		buf [--charPos] = digits [r];
		i = q;
		if (i == 0) break;
	}
	if (sign != 0) {
		buf [--charPos] = sign;
	}
}
```

在第二个循环中i*52429>>>19,等同于i * 52429 / 524288，即 i * 0.1。

## hashCode()

```java
public int hashCode() {
	return Integer.hashCode(value);
}
public static int hashCode(int value) {
	return value;
}
```

Integer的hash值就是value的值。

