jdk 8 版本。

## String类的作用

String 类代表字符串。Java 程序中的所有字符串字面值（如 “abc” ）都作为此类的实例实现。

字面量`String str="string";`，引用变量`String str=new String("string")`

## String类的类图

String类图没有属性和方法，因为在这里看类图主要是为了明白我们当前类所处的一个位置，以及他实现的接口。

![](images/Snipaste_2020-12-24_09-23-51.png)

该类实现了序列化接口，说明该类可以被序列化和反序列化（反序列化是构建对象的一种方式）

该类实现了比较器的接口，说明该类实现了默认的比较方式，在集合中的排序会根据这个比较的方式来进行排序。

该类实现了CharSequence 接口，CharSequence 是 char 值的一个可读序列。此接口对许多不同种类的 char 序列提供统一的只读访问。

## String类的重要方法源码解读

### 属性变量

```java
/** The value is used for character storage. */
//String的值都存在这个char数组中
private final char value[];

/** Cache the hash code for the string */
// String的hash值
private int hash; // Default to 0

/** use serialVersionUID from JDK 1.0.2 for interoperability */
private static final long serialVersionUID = -6849794470754667710L;
```

### 构造方法

从上面的属性变量可以知道，我们这个类中主要有char数组和hash值。

那么理所当然的，构造方法就是围绕着两个变量在展开的。

将全部的构造方法阅读一遍之后，发现除了参数检验之外的操作，主要是：

**Arrays.copyOf()** 和 **Arrays.copyOfRange()** 两个方法（如果涉及了编码问题会调用**StringCoding.decode()** 方法），将要组成String的值放入到char数组中。因此我们拿一个比较有代表性的来阅读即可。

```java
//通过使用平台的默认字符集解码指定的 byte 子数组，构造一个新的 String。
public String(char value[], int offset, int count) {
//检验offset 初始偏移量 的值，不能小于0
if (offset < 0) {
    throw new StringIndexOutOfBoundsException(offset);
}
//检验 count 裁剪的数量  不能小于0
if (count < 0) {
    throw new StringIndexOutOfBoundsException(count);
}
// Note: offset or count might be near -1>>>1.
//检验范围
if (offset > value.length - count) {
    throw new StringIndexOutOfBoundsException(offset + count);
}
this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

```java
//通过使用指定的 charset 解码指定的 byte 子数组，构造一个新的 String
public String(byte bytes[], int offset, int length, Charset charset) {
    if (charset == null)
        throw new NullPointerException("charset");
        //检验参数，如上
    checkBounds(bytes, offset, length);
    this.value =  StringCoding.decode(charset, bytes, offset, length);
}
private static void checkBounds(byte[] bytes, int offset, int length) {
    if (length < 0)
        throw new StringIndexOutOfBoundsException(length);
    if (offset < 0)
        throw new StringIndexOutOfBoundsException(offset);
    if (offset > bytes.length - length)
        throw new StringIndexOutOfBoundsException(offset + length);
}
```

### 常用方法

String 类包括的方法可用于检查序列的单个字符、比较字符串、搜索字符串、提取子字符串、创建字符串副本并将所有字符全部转换为大写或小写。大小写映射基于 Character 类指定的 Unicode 标准版。

#### 用于字符串的判断

1. public boolean equals(Object anObject)

```java
// 比较两个字符串是否想等。
public boolean equals(Object anObject) {
    //这里就表明了 String 中 equals 和 == 的作用是一样的
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String) anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                        return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

比较字符串是否相等，在底层就是将两个数组一个一个的对比。

2. public boolean equalsIgnoreCase(String anotherString)

```java
// 忽略大小写比较两个字符串是否相等
public boolean equalsIgnoreCase(String anotherString) {
        return (this == anotherString) ? true
                : (anotherString != null)
                && (anotherString.value.length == value.length)
                && regionMatches(true, 0, anotherString, 0, value.length);
    }
//测试两个字符串区域是否相等。
/**
将此 String 对象的子字符串与参数 other 的子字符串进行比较。
如果这两个子字符串表示相同的字符序列，则结果为 true，
当且仅当 ignoreCase 为 true 时忽略大小写。
要比较的此 String 对象的子字符串从索引 toffset 处开始，长度为 len。
要比较的 other 的子字符串从索引 ooffset 处开始，长度为 len。
当且仅当下列至少一项为 true 时，结果才为 false：
**/ 
public boolean regionMatches(boolean ignoreCase, int toffset,String other, int ooffset, int len) {
    char ta[] = value;
    int to = toffset;
    char pa[] = other.value;
    int po = ooffset;
    // Note: toffset, ooffset, or len might be near -1>>>1.
    if ((ooffset < 0) || (toffset < 0)
            || (toffset > (long)value.length - len)
            || (ooffset > (long)other.value.length - len)) {
        return false;
    }
    while (len-- > 0) {
        char c1 = ta[to++];
        char c2 = pa[po++];
        if (c1 == c2) {
            continue;
        }
        if (ignoreCase) {
            // If characters don't match but case may be ignored,
            // try converting both characters to uppercase.
            // If the results match, then the comparison scan should
            // continue.
            char u1 = Character.toUpperCase(c1);
            char u2 = Character.toUpperCase(c2);
            if (u1 == u2) {
                continue;
            }
            // Unfortunately, conversion to uppercase does not work properly
            // for the Georgian alphabet, which has strange rules about case
            // conversion.  So we need to make one last check before
            // exiting.
            if (Character.toLowerCase(u1) == Character.toLowerCase(u2)) {
                continue;
            }
        }
        return false;
    }
    return true;
}
```

忽略大小写的方法就是在比较的时候，在加上比较将两个字符同时转换成大写，小写的比较。至于为什么转成大写了之后，还要转换成小写的在比较一次，我也不是很清楚。官方的解释是：如果只比较大写的话，有些情况是不能得到我们想要的答案的

3. public boolean startsWith(String prefix, int toffset)

```java
//测试此字符串从指定索引开始的子字符串是否以指定前缀开始。
public boolean startsWith(String prefix, int toffset) {
    char ta[] = value;
    int to = toffset;
    char pa[] = prefix.value;
    int po = 0;
    int pc = prefix.value.length;
    // Note: toffset might be near -1>>>1.
    if ((toffset < 0) || (toffset > value.length - pc)) {
        return false;
    }
    while (--pc >= 0) {
        if (ta[to++] != pa[po++]) {
            return false;
        }
    }
    return true;
}

public boolean startsWith(String prefix) {
    return startsWith(prefix, 0);
}
public boolean endsWith(String suffix) {
    return startsWith(suffix, value.length - suffix.value.length);
}
```

#### 获取字符串的方法

1. public char charAt(int index)

```java
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return value[index];
}
```

2. public int indexOf(String str)

```java
public int indexOf(String str) {
    return indexOf(str, 0);
}
public int indexOf(String str, int fromIndex) {
    return indexOf(value, 0, value.length,
            str.value, 0, str.value.length, fromIndex);
}

static int indexOf(char[] source, int sourceOffset, int sourceCount,
        char[] target, int targetOffset, int targetCount,
        int fromIndex) {
    if (fromIndex >= sourceCount) {
        return (targetCount == 0 ? sourceCount : -1);
    }
    if (fromIndex < 0) {
        fromIndex = 0;
    }
    if (targetCount == 0) {
        return fromIndex;
    }

    char first = target[targetOffset];
    int max = sourceOffset + (sourceCount - targetCount);

    for (int i = sourceOffset + fromIndex; i <= max; i++) {
        /* Look for first character. */
        // 寻找第一个字符相同的地方
        if (source[i] != first) {
            while (++i <= max && source[i] != first);
        }
        
        /* Found first character, now look at the rest of v2 */
        // 继续对比接下来的字符
        if (i <= max) {
            int j = i + 1;
            int end = j + targetCount - 1;
            for (int k = targetOffset + 1; j < end && source[j]
                    == target[k]; j++, k++);

            if (j == end) {
                /* Found whole string. */
                return i - sourceOffset;
            }
        }
    }
    return -1;
}
```

原来判断字符串1是不是字符串2的子串，java的源码是使用暴力循环，并没有使用其他的算法。

3. public String substring(int beginIndex, int endIndex)

```java
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}
```

返回一个新的String对象,JDK6返回的是当前字符串的从指定位置开始的原始字符串，导致会出现溢出的情况

#### 字符串转换的方法

1. public char[] toCharArray()

```java
public char[] toCharArray() {
    // Cannot use Arrays.copyOf because of class initialization order issues
    char result[] = new char[value.length];
    System.arraycopy(value, 0, result, 0, value.length);
    return result;
}
```

在没有看源码之前，我以为是直接将对象里面的数组直接给出，现在看了代码之后发现不是，后来想一想发现，数组是引用类型，如果将数组的引用放出去，那么String的值也会相应的改变。如果发生这个，那么就会产生很大的灾难。

2. public String toLowerCase()

```java
public String toLowerCase() {
    return toLowerCase(Locale.getDefault());
}
// 使用给定 Locale 的规则将此 String 中的所有字符都转换为小写。
public String toLowerCase(Locale locale) {
    if (locale == null) {
        throw new NullPointerException();
    }

    int firstUpper;
    final int len = value.length;

    /* Now check if there are any characters that need to be changed. */
    scan: {
        for (firstUpper = 0 ; firstUpper < len; ) {
            char c = value[firstUpper];
            if ((c >= Character.MIN_HIGH_SURROGATE)
                    && (c <= Character.MAX_HIGH_SURROGATE)) {
                int supplChar = codePointAt(firstUpper);
                if (supplChar != Character.toLowerCase(supplChar)) {
                    break scan;
                }
                firstUpper += Character.charCount(supplChar);
            } else {
                if (c != Character.toLowerCase(c)) {
                    break scan;
                }
                firstUpper++;
            }
        }
        return this;
    }

    char[] result = new char[len];
    int resultOffset = 0;  /* result may grow, so i+resultOffset
                            * is the write location in result */

    /* Just copy the first few lowerCase characters. */
    System.arraycopy(value, 0, result, 0, firstUpper);

    String lang = locale.getLanguage();
    boolean localeDependent =
            (lang == "tr" || lang == "az" || lang == "lt");
    char[] lowerCharArray;
    int lowerChar;
    int srcChar;
    int srcCount;
    for (int i = firstUpper; i < len; i += srcCount) {
        srcChar = (int)value[i];
        if ((char)srcChar >= Character.MIN_HIGH_SURROGATE
                && (char)srcChar <= Character.MAX_HIGH_SURROGATE) {
            srcChar = codePointAt(i);
            srcCount = Character.charCount(srcChar);
        } else {
            srcCount = 1;
        }
        if (localeDependent || srcChar == '\u03A3') { // GREEK CAPITAL LETTER SIGMA
            lowerChar = ConditionalSpecialCasing.toLowerCaseEx(this, i, locale);
        } else if (srcChar == '\u0130') { // LATIN CAPITAL LETTER I DOT
            lowerChar = Character.ERROR;
        } else {
            lowerChar = Character.toLowerCase(srcChar);
        }
        if ((lowerChar == Character.ERROR)
                || (lowerChar >= Character.MIN_SUPPLEMENTARY_CODE_POINT)) {
            if (lowerChar == Character.ERROR) {
                if (!localeDependent && srcChar == '\u0130') {
                    lowerCharArray =
                            ConditionalSpecialCasing.toLowerCaseCharArray(this, i, Locale.ENGLISH);
                } else {
                    lowerCharArray =
                            ConditionalSpecialCasing.toLowerCaseCharArray(this, i, locale);
                }
            } else if (srcCount == 2) {
                resultOffset += Character.toChars(lowerChar, result, i + resultOffset) - srcCount;
                continue;
            } else {
                lowerCharArray = Character.toChars(lowerChar);
            }

            /* Grow result if needed */
            int mapLen = lowerCharArray.length;
            if (mapLen > srcCount) {
                char[] result2 = new char[result.length + mapLen - srcCount];
                System.arraycopy(result, 0, result2, 0, i + resultOffset);
                result = result2;
            }
            for (int x = 0; x < mapLen; ++x) {
                result[i + resultOffset + x] = lowerCharArray[x];
            }
            resultOffset += (mapLen - srcCount);
        } else {
            result[i + resultOffset] = (char)lowerChar;
        }
    }
    return new String(result, 0, len + resultOffset);
}
```

#### 其他方法

1. public String trim()

```java
// 返回字符串的副本，忽略前导空白和尾部空白。
public String trim() {
    int len = value.length;
    int st = 0;
    char[] val = value;    /* avoid getfield opcode */

    while ((st < len) && (val[st] <= ' ')) {
        st++;
    }
    while ((st < len) && (val[len - 1] <= ' ')) {
        len--;
    }
    return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
}
```
2. public String[] split(String regex, int limit)

```java
public String[] split(String regex, int limit) {
    /* fastpath if the regex is a
        (1)one-char String and this character is not one of the
        RegEx's meta characters ".$|()[{^?*+\\", or
        (2)two-char String and the first char is the backslash and
        the second is not the ascii digit or ascii letter.
        */
    char ch = 0;
    if (((regex.value.length == 1 &&
            ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
            (regex.length() == 2 &&
            regex.charAt(0) == '\\' &&
            (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
            ((ch-'a')|('z'-ch)) < 0 &&
            ((ch-'A')|('Z'-ch)) < 0)) &&
        (ch < Character.MIN_HIGH_SURROGATE ||
            ch > Character.MAX_LOW_SURROGATE))
    {
        int off = 0;
        int next = 0;
        boolean limited = limit > 0;
        ArrayList<String> list = new ArrayList<>();
        while ((next = indexOf(ch, off)) != -1) {
            if (!limited || list.size() < limit - 1) {
                list.add(substring(off, next));
                off = next + 1;
            } else {    // last one
                //assert (list.size() == limit - 1);
                list.add(substring(off, value.length));
                off = value.length;
                break;
            }
        }
        // If no match was found, return this
        if (off == 0)
            return new String[]{this};

        // Add remaining segment
        if (!limited || list.size() < limit)
            list.add(substring(off, value.length));

        // Construct result
        int resultSize = list.size();
        if (limit == 0)
            while (resultSize > 0 && list.get(resultSize - 1).length() == 0)
                resultSize--;
        String[] result = new String[resultSize];
        return list.subList(0, resultSize).toArray(result);
    }
    return Pattern.compile(regex).split(this, limit);
}
```

内部使用arrayList集合来收集被分割的子串，子串是通过substring（）方法来得到的，参数是通过indexof方法来获取的。

3. public String concat(String str)

```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
```

## String类涉及的设计模式

### 享元模式

模式解释：

一个系统中如果有多处用到了相同的一个元素，那么我们应该只存储一份此元素，而让所有地方都引用这一个元素

String类的体现：

因为String太过常用，JAVA类库的设计者在实现时做了个小小的变化，即采用了享元模式,每当生成一个新内容的字符串时，他们都被添加到一个共享池中，当第二次再次生成同样内容的字符串实例时，就共享此对象，而不是创建一个新对象，但是这样的做法仅仅适合于通过=符号进行的初始化。

## 阅读String类的感想

String类是我们最常使用的一个类，我们也都知道String类是不可改变的，每当改变一个变量的值时候，地址也会随之改变。基于这种不变的特性，出现了享元模式，让系统中的多个相同的String共享一个地址，避免了开销。

String类的底层是char数组，所以我们看到String的方法（对字符串的操作）底层都是操作数组。其实我们看到方法之后，了解底层是数组，对方法的一些大概操作是可以预知的，在读完源码之后，会更加清晰明了。

## 结尾