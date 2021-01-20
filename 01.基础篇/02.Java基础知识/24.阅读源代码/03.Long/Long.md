转载：https://blog.csdn.net/baidu_39299382/article/details/81910988

# 概况

Java的Long类主要的作用就是对基本类型long进行封装，提供了一些处理long类型的方法，比如long到String类型的转换方法或String类型到long类型的转换方法，当然也包含与其他类型之间的转换方法。

除此之外还有一些位相关的操作。

## 继承结构

```java
--java.lang.Object
  --java.lang.Number
    --java.lang.Long
```

## 主要属性

```java
public static final long MIN_VALUE = 0x8000000000000000L;
public static final long MAX_VALUE = 0x7fffffffffffffffL;
public static final int BYTES = SIZE / Byte.SIZE;
public static final int SIZE = 64;
public static final Class<Long>     TYPE = (Class<Long>) Class.getPrimitiveClass("long");
```

- MIN_VALUE静态变量表示long能取的最小值，为-2的63次方，被final修饰说明不可变。
- 类似的还有MAX_VALUE，表示long最大值为2的63次方减1。
- SIZE用来表示二进制补码形式的long值的比特数，值为64，静态变量且不可变。
- BYTES用来表示二进制补码形式的long值的字节数，值为SIZE除于Byte.SIZE，结果为8。
- TYPE的toString的值是long。
- Class的getPrimitiveClass是一个native方法，在Class.c中有个Java_java_lang_Class_getPrimitiveClass方法与之对应，所以JVM层面会通过JVM_FindPrimitiveClass函数根据”long”字符串获得jclass，最终到Java层则为Class\<Long\>。

```c++
JNIEXPORT jclass JNICALL
Java_java_lang_Class_getPrimitiveClass(JNIEnv *env,
                                       jclass cls,
                                       jstring name)
{
    const char *utfName;
    jclass result;
 
    if (name == NULL) {
        JNU_ThrowNullPointerException(env, 0);
        return NULL;
    }
 
    utfName = (*env)->GetStringUTFChars(env, name, 0);
    if (utfName == 0)
        return NULL;
 
    result = JVM_FindPrimitiveClass(env, utfName);
 
    (*env)->ReleaseStringUTFChars(env, name, utfName);
 
    return result;
}
```

当TYPE执行toString时，逻辑如下，则其实是getName函数决定其值，getName通过native方法getName0从JVM层获取名称

```java
public String toString() {
    return (isInterface() ? "interface " : (isPrimitive() ? "" : "class "))
        + getName();
}
```

getName0根据一个数组获得对应的名称，JVM根据Java层的Class可得到对应类型的数组下标，比如这里下标为11，则名称为”long”。

```c++
const char* type2name_tab[T_CONFLICT+1] = {
  NULL, NULL, NULL, NULL,
  "boolean",
  "char",
  "float",
  "double",
  "byte",
  "short",
  "int",
  "long",
  "object",
  "array",
  "void",
  "*address*",
  "*narrowoop*",
  "*conflict*"
};
```

## LongCache内部类

```java
private static class LongCache {
    private LongCache(){}

    static final Long cache[] = new Long[-(-128) + 127 + 1];

    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Long(i - 128);
    }
}
```

LongCache是Long的一个内部类，它包含了long可能值的Long数组，默认范围是[-128,127]，它不会像Byte类将所有可能值缓存起来，因为long类型范围很大，将它们全部缓存起来代价太高，而Byte类型就是从-128到127，一共才256个。

这里默认只实例化256个Long对象，当Long的值范围在[-128,127]时则直接从缓存中获取对应的Long对象，不必重新实例化。这些缓存值都是静态且final的，避免重复的实例化和回收。

# 主要方法

## parseLong方法

```java
public static long parseLong(String s) throws NumberFormatException {
    return parseLong(s, 10);
}

public static long parseLong(String s, int radix)
            throws NumberFormatException
{
    if (s == null) {
        throw new NumberFormatException("null");
    }

    if (radix < Character.MIN_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " less than Character.MIN_RADIX");
    }
    if (radix > Character.MAX_RADIX) {
        throw new NumberFormatException("radix " + radix +
                                        " greater than Character.MAX_RADIX");
    }

    long result = 0;
    boolean negative = false;
    int i = 0, len = s.length();
    long limit = -Long.MAX_VALUE;
    long multmin;
    int digit;

    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // Possible leading "+" or "-"
            if (firstChar == '-') {
                negative = true;
                limit = Long.MIN_VALUE;
            } else if (firstChar != '+')
                throw NumberFormatException.forInputString(s);

            if (len == 1) // Cannot have lone "+" or "-"
                throw NumberFormatException.forInputString(s);
            i++;
        }
        multmin = limit / radix;
        while (i < len) {
            // Accumulating negatively avoids surprises near MAX_VALUE
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {
                throw NumberFormatException.forInputString(s);
            }
            result *= radix;
            if (result < limit + digit) {
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;
        }
    } else {
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;
}
```

两个parseLong方法，主要看第二个即可，第一个参数是待转换的字符串，第二个参数表示进制数。怎么更好理解这个参数呢？举个例子，Long.parseLong("100",10)表示十进制的100，所以值为100，而Long.parseLong("100",2)表示二进制的100，所以值为4。

另外如果Long.parseLong("10000000000000000000",10)会抛出java.lang.NumberFormatException异常。

该方法的逻辑是首先判断字符串不为空且进制数在Character.MIN_RADIX和Character.MAX_RADIX之间，即2到36。

然后判断输入的字符串的长度必须大于0，再根据第一个字符可能为数字或负号或正号进行处理。

核心处理逻辑是字符串转换数字，n进制转成十进制办法基本大家都知道的了，假如357为8进制，则结果为$3*8^2+5*8^1+7*8^0 = 239$，假如357为十进制，则结果为$3*10^2+5*10^1+7*10^0 = 357$，上面的转换方法也差不多是根据此方法，只是稍微转变了思路，方式分别为$((3*8+5)*8+7) = 239$和$((3*10+5)*10+7)=357$。

从中可以推出规则了，从左到右遍历字符串的每个字符，然后乘以进制数，再加上下一个字符，接着再乘以进制数，再加上下个字符，不断重复，直到最后一个字符。

除此之外另外一个不同就是上面的转换不使用加法来做，全都转成负数来运算，其实可以看成是等价了，这个很好理解，而为什么要这么做就要归咎到long类型的范围了，因为负数Long.MIN_VALUE变化为正数时会导致数值溢出，所以全部都用负数来运算。

## 构造函数

```java
public Long(String s) throws NumberFormatException {
    this.value = parseLong(s, 10);
}
public Long(long value) {
    this.value = value;
}
```

包含两种构造函数，分别可以传入long和String类型。它是通过调用parseLong方法进行转换的，所以转换逻辑与上面的parseLong方法一样。

```java
static void getChars(long i, int index, char[] buf) {
    long q;
    int r;
    int charPos = index;
    char sign = 0;

    if (i < 0) {
        sign = '-';
        i = -i;
    }

    while (i > Integer.MAX_VALUE) {
        q = i / 100;
        // really: r = i - (q * 100);
        r = (int)(i - ((q << 6) + (q << 5) + (q << 2)));
        i = q;
        buf[--charPos] = Integer.DigitOnes[r];
        buf[--charPos] = Integer.DigitTens[r];
    }

    int q2;
    int i2 = (int)i;
    while (i2 >= 65536) {
        q2 = i2 / 100;
        r = i2 - ((q2 << 6) + (q2 << 5) + (q2 << 2));
        i2 = q2;
        buf[--charPos] = Integer.DigitOnes[r];
        buf[--charPos] = Integer.DigitTens[r];
    }

    for (;;) {
        q2 = (i2 * 52429) >>> (16+3);
        r = i2 - ((q2 << 3) + (q2 << 1));  
        buf[--charPos] = Integer.digits[r];
        i2 = q2;
        if (i2 == 0) break;
    }
    if (sign != 0) {
        buf[--charPos] = sign;
    }
}
```

该方法主要做的事情是将某个long型数值放到char数组里面，比如把357按顺序放到char数组中。这里面处理用了较多技巧，将long拆成高位4个字节和低位4个字节处理分开处理，while (i >= Integer.MAX_VALUE)部分就是处理高位的4个字节，每次处理2位数，这里有个特殊的地方((q << 6) + (q << 5) + (q << 2))其实等于q*100,Integer.DigitTens和Integer.DigitOnes数组在前面Integer文章中已经讲过它的作用了，用来获取十位和个位。

接着看怎么处理低4个字节，它继续将4个字节分为高位2个字节和低位2个字节，while (i >= 65536)部分就是处理高位的两个字节，每次处理2位数，处理逻辑与高位4个字节的处理逻辑一样。

再看接下去的低位的两个字节怎么处理，其实本质也是求余思想，但又用了一些技巧，比如(i * 52429) >>> (16+3)其实约等于i/10，((q << 3) + (q << 1))其实等于q*10，然后再通过Integer.digits数组获取到对应的字符。可以看到低位处理时它尽量避开了除法，取而代之的是用乘法和右移来实现，可见除法是一个比较耗时的操作，比起乘法和移位。另外也可以看到能用移位和加法来实现乘法的地方也尽量不用乘法，这也说明乘法比起它们更加耗时。而高位处理时没有用移位是因为做乘法后可能会溢出。

## toString方法

```java
public static String toString(long i) {
    if (i == Long.MIN_VALUE)
        return "-9223372036854775808";
    int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
    char[] buf = new char[size];
    getChars(i, size, buf);
    return new String(buf, true);
}

public String toString() {
    return toString(value);
}

public static String toString(long i, int radix) {
    if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
        radix = 10;
    if (radix == 10)
        return toString(i);
    char[] buf = new char[65];
    int charPos = 64;
    boolean negative = (i < 0);

    if (!negative) {
        i = -i;
    }

    while (i <= -radix) {
        buf[charPos--] = Integer.digits[(int)(-(i % radix))];
        i = i / radix;
    }
    buf[charPos] = Integer.digits[(int)(-i)];

    if (negative) {
        buf[--charPos] = '-';
    }

    return new String(buf, charPos, (65 - charPos));
}
```

一共有3个toString方法，两个静态方法一个是非静态方法，第一个toString方法很简单，就是先用stringSize得到数字是多少位，再用getChars获取数字对应的char数组，最后返回一个String类型。

第二个toString调用第一个toString，没啥好说。

第三个toString方法是带了进制信息的，它会转换成对应进制的字符串。凡是不在2到36进制范围之间的都会被处理成10进制，我们都知道从十进制转成其他进制时就是不断地除于进制数得到余数，然后把余数反过来串起来就是最后结果，所以这里其实也是这样子做的，得到余数后通过digits数组获取到对应的字符，而且这里是用负数的形式来运算的。

## valueOf方法

```java
public static Long valueOf(long l) {
    final int offset = 128;
    if (l >= -128 && l <= 127) { // will cache
        return LongCache.cache[(int)l + offset];
    }
    return new Long(l);
}

public static Long valueOf(String s) throws NumberFormatException
{
    return Long.valueOf(parseLong(s, 10));
}

public static Long valueOf(String s, int radix) throws NumberFormatException {
    return Long.valueOf(parseLong(s, radix));
}
```

有三个valueOf方法，核心逻辑在第一个valueOf方法中，因为LongCache缓存了[-128,127]值的Long对象，对于在范围内的直接从LongCache的数组中获取对应的Long对象即可，而在范围外的则需要重新实例化了。

## decode方法

```java
public static Long decode(String nm) throws NumberFormatException {
    int radix = 10;
    int index = 0;
    boolean negative = false;
    Long result;

    if (nm.length() == 0)
        throw new NumberFormatException("Zero length string");
    char firstChar = nm.charAt(0);
    if (firstChar == '-') {
        negative = true;
        index++;
    } else if (firstChar == '+')
        index++;
    if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
        index += 2;
        radix = 16;
    }
    else if (nm.startsWith("#", index)) {
        index ++;
        radix = 16;
    }d
    else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
        index ++;
        radix = 8;
    }

    if (nm.startsWith("-", index) || nm.startsWith("+", index))
        throw new NumberFormatException("Sign character in wrong position");

    try {
        result = Long.valueOf(nm.substring(index), radix);
        result = negative ? Long.valueOf(-result.longValue()) : result;
    } catch (NumberFormatException e) {
        String constant = negative ? ("-" + nm.substring(index))
                                    : nm.substring(index);
        result = Long.valueOf(constant, radix);
    }
    return result;
}
```

decode方法主要作用是解码字符串转成Long型，比如Long.decode("11")的结果为11；Long.decode("0x11")和Long.decode("#11")结果都为17，因为0x和#开头的会被处理成十六进制；Long.decode("011")结果为9，因为0开头会被处理成8进制。

## xxxValue方法

```java

```

## hashCode方法

## hashCode方法

## compare方法

## 无符号转换

## bitCount方法

## highestOneBit方法

## lowestOneBit方法

## numberOfLeadingZeros方法

## numberOfTrailingZeros方法

## reverse方法

## toHexString和toOctalString方法
 