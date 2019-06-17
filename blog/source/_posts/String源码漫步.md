---
title: String源码漫步
date: 2019-05-27 
tag：[java,源码]
---



```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {}
```

<!--more-->

## 属性

```
private final char value[];
```

用于存储字符串的内容。所以，String的本质是char类型的数组。所以是不可变的。

```
private int hash;
```

用于缓存hashCode值，默认为0.

```
private static final long serialVersionUID = -6849794470754667710L;
 private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];
```

> 因为`String`实现了`Serializable`接口，所以支持序列化和反序列化支持。Java的序列化机制是通过在运行时判断类的`serialVersionUID`来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的`serialVersionUID`与本地相应实体（类）的`serialVersionUID`进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常(`InvalidCastException`)。

## 构造方法

```
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}


```

这里是直接赋值value和hash的，因为String类是不可变的，所以不会有赋值后对原值有修改的问题。



```
public String(char value[]) {
	this.value = Arrays.copyOf(value, value.length);
}
public String(char value[], int offset, int count) {
	if (offset < 0) {
		throw new StringIndexOutOfBoundsException(offset);
	}
	if (count <= 0) {
		if (count < 0) {
			throw new StringIndexOutOfBoundsException(count);
		}
		if (offset <= value.length) {
			this.value = "".value;
			return;
		}
	}
    // Note: offset or count might be near -1>>>1.
    if (offset > value.length - count) {
        throw new StringIndexOutOfBoundsException(offset + count);
    }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

copyOf() 里创建了一个新字符数组并赋值返回；因为是使用副本的，所以value[]参数后面修改也不会影响生成的String本身。



```
	public String(int[] codePoints, int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= codePoints.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > codePoints.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }

        final int end = offset + count;

        // Pass 1: Compute precise size of char[]
		// 计算字符[]的精确大小
        int n = count;
        for (int i = offset; i < end; i++) {
            int c = codePoints[i];
            //确定指定的字符(Unicode代码点)是否在Basic Multilingual Plane(BMP)中
            if (Character.isBmpCodePoint(c))
                continue;
            else if (Character.isValidCodePoint(c))//确定是否是有效的Unicode代码点值
                n++;
            else throw new IllegalArgumentException(Integer.toString(c));
        }

        // Pass 2: Allocate and fill in char[];分配并填充字符[]
        final char[] v = new char[n];

        for (int i = offset, j = 0; i < end; i++, j++) {
            int c = codePoints[i];
            if (Character.isBmpCodePoint(c))
                v[j] = (char)c;
            else
                Character.toSurrogates(c, v, j++);
        }
        this.value = v;
    }
```

传入一个int[]时先遍历int数组加入到字符数组中，再赋值给value。**这个方法中的验证准确长度，遍历加入到字符数组中的方法还需要深入研究，暂时不理解他的意思。**



```
	public String(byte bytes[], int offset, int length, String charsetName)
            throws UnsupportedEncodingException {
        if (charsetName == null)
            throw new NullPointerException("charsetName");
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(charsetName, bytes, offset, length);
    }
```

使用`StringCoding.decode`方法进行解码，使用的解码的字符集就是我们指定的`charsetName`或者`charset`。

下列方法同上，给了解码方式：

```
//注意解码参数和上面的String类型有区别
public String(byte bytes[], int offset, int length, Charset charset) {}
 public String(byte bytes[], String charsetName){}
  public String(byte bytes[], Charset charset) {}
```



如果没有指定编码格式则默认使用**ISO-8859-1**编码格式进行编码操作，如下。

```
 public String(byte bytes[], int offset, int length) {
        checkBounds(bytes, offset, length);
        this.value = StringCoding.decode(bytes, offset, length);
}

public String(byte bytes[]) {}
```

以下两个方法是先把StringBuffer转化成字符数组，再对value进行赋值。

注意StringBuffer是线程安全的，所以效率会比较低。

```
	public String(StringBuffer buffer) {
        synchronized(buffer) {
            this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
        }
    }
    public String(StringBuilder builder) {
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
    }
```

特殊的方法，限定为本包使用：

```
String(char[] value, boolean share) {
    // assert share : "unshared not supported";
    this.value = value;
}
```

没有使用的share参数是为了区分重载方法（String（char value[]），使用的是copOf() 方法）；直接把字符数组赋值给value；使用场景是Integer等包装类的toString()方法。

## 其他方法

```
public int length() 
public boolean isEmpty() 
```

```
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return value[index];
}


//返回指定索引处的字符（Unicode 代码点）。比如从ASCII 的 ‘A'转化成 65
//索引引用 char 值（Unicode代码单元），其范围从 0到 length() - 1。
public int codePointAt(int index) {
	if ((index < 0) || (index >= value.length)) {
       throw new StringIndexOutOfBoundsException(index);
    }
    return Character.codePointAtImpl(value, index, value.length);
}
//从1开始，类似于上面的方法
 public int codePointBefore(int index) {
        int i = index - 1;
        if ((i < 0) || (i >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        return Character.codePointBeforeImpl(value, index, 0);
    }
```



```
 public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
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

遍历的十分简介。用一个死循环没有使用for.



```
public boolean equalsIgnoreCase(String anotherString) {
        return (this == anotherString) ? true
                : (anotherString != null)
                && (anotherString.value.length == value.length)
                && regionMatches(true, 0, anotherString, 0, value.length);
    }
```





```
public boolean regionMatches(boolean ignoreCase, int toffset,
            String other, int ooffset, int len) {
        char ta[] = value;//本对象字符数组
        int to = toffset;//本字符串开始比较的位置
        char pa[] = other.value;//比较对象字符数组
        int po = ooffset;//参数字符串开始比较的位置
        // Note: toffset, ooffset, or len might be near -1>>>1.
        if ((ooffset < 0) || (toffset < 0)//如果两个位置小于0 直接返回错误
                || (toffset > (long)value.length - len)
                || (ooffset > (long)other.value.length - len)) {
            return false;
        }
        while (len-- > 0) { //遍历参数字符串的长度
            char c1 = ta[to++];//得到本对象开始比较后的第一个
            char c2 = pa[po++];//参数对象
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





```
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

注意new了一个对象，对新对象进行赋值后返回的。

```
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

新建一个数组并填充进去返回。损失了一些性能，可以避免造成内存泄漏。











[参考链接](https://www.hollischuang.com/archives/99)



















