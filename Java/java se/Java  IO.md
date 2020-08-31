[toc]

## Java  IO

### 1. 概述

java 的 io 根据数据的流向分为：**输入流**和**输出流**。

* **输入流** ：把数据从`其他设备`上读取到`内存`中的流。 
* **输出流** ：把数据从`内存` 中写出到`其他设备`上的流。

根据数据的类型分为：**字节流**和**字符流**。

* **字节流** ：以字节为单位，读写数据的流。
* **字符流** ：以字符为单位，读写数据的流。

java.io 的 抽象父类

|            |           **输入流**           |             输出流              |
| :--------: | :----------------------------: | :-----------------------------: |
| **字节流** | 字节输入流<br/>**InputStream** | 字节输出流<br/>**OutputStream** |
| **字符流** |   字符输入流<br/>**Reader**    |    字符输出流<br/>**Writer**    |



### 2. 字节输出流

#### 2.1 OutputStream

`java.io.OutputStream `抽象类是表示字节输出流的所有类的超类，定义了字节输出流的基本功能方法

``` java
public abstract class OutputStream implements Closeable, Flushable {

    public abstract void write(int b) throws IOException;

    public void write(byte b[]) throws IOException {
        write(b, 0, b.length);
    }

    public void write(byte b[], int off, int len) throws IOException {
        if (b == null) {
            throw new NullPointerException();
        } else if ((off < 0) || (off > b.length) || (len < 0) ||
                   ((off + len) > b.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }
        for (int i = 0 ; i < len ; i++) {
            write(b[off + i]);
        }
    }

    // 刷新输出流并强制任何缓冲的输出字节被写出
    public void flush() throws IOException {
    }

    // 关闭输出流并释放系统资源
    public void close() throws IOException {
    }

}
```

#### 2.2 FileOutputStream

`java.io.FileOutputStream `是文件输出流，用于将数据写出到文件

使用方式：

- 通过构造方法创建流
- 调用  `write` 写入文件
- 调用 `close` 释放资源

##### 2.2.1 创建流

* `public FileOutputStream(File file)`：创建文件输出流以写入由指定的 File对象表示的文件。 
* `public FileOutputStream(String name)`： 创建文件输出流以指定的名称写入文件。  

``` java
// 方式一: 使用File对象创建流对象
File file = new File("a.txt");
FileOutputStream fos = new FileOutputStream(file);
      
// 方式二: 使用文件名称创建流对象
FileOutputStream fos = new FileOutputStream("b.txt");
```

##### 2.2.2 写入数据

**写入单个字节**：会向下转型为 byte，并根据 ASCII 码写入

``` java
OutputStream fos = new FileOutputStream("file/a.txt");
fos.write(97); // 写入字符'a', ASCII码为97
fos.write(98); // 写入字符'b'
fos.write(99); // 写入字符'c'
fos.close();
```

**写入多个字节**

``` java
byte[] b = "bzzb".getBytes();
fos.write(b);
```

**追加写入**：构造函数中指定 `append` 为 true

``` java
OutputStream fos = new FileOutputStream("file/a.txt", true);
byte[] b = "bzzb".getBytes();
fos.write(b);
fos.close();
```



### 3. 字节输入流

#### 3.1 InputStream

`java.io.InputStream ` 是表示字节输入流的所有类的超类，基本方法如下：

- `public void close()` ：关闭此输入流并释放与此流相关联的任何系统资源。    
- `public abstract int read()`： 从输入流读取数据的下一个字节，读到文件末尾返回 -1 。
- `public int read(byte[] b)`： 从输入流中读取一些字节数，并将它们存储到字节数组 b中 。

#### 3.2 FileInputStream

`java.io.FileInputStream `是文件输入流，使用方法与 FileInputStream 类似

##### 3.2.1 读入数据

```java
InputStream fis = new FileInputStream("file/a.txt");
int len = 0;
byte[] b = new byte[6];
while ((len = fis.read(b)) != -1) {
   System.out.println(new String(b, 0, len));
}
fis.close();
```

##### 3.2.2 文件复制

``` java
        InputStream is = null;
        OutputStream os = null;
        try {
            is = new FileInputStream("file/test.jpg");
            os = new FileOutputStream("file/copy.jpg");

            byte[] b = new byte[1024];
            int len = 0;
            while ((len = is.read(b)) != -1) {
                os.write(b,0, len);
            }
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (is != null) {
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            try {
                if (os != null) {
                    os.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```



### 4. 字符输入流

当使用字节流读取文本遇到中文字符时，可能不会显示完整的字符，因为一个中文字符可能占用多个字节存储。Java 提供字符流类，以字符为单位读写数据，处理文本文件

#### 4.1 Reader

`java.io.Reader` 是字符输入流的抽象超类，基本功能：

- `public void close()` ：关闭此流并释放系统资源。    
- `public int read()`： 从输入流读取一个字符。 
- `public int read(char[] cbuf)`： 从输入流中读取一些字符，并将它们存储到字符数组 cbuf中 ，读到文件末尾返回 -1 。

#### 4.2 FileReader

`java.io.FileReader `  用于读取字符文件，构造时使用系统默认的字符编码和字节缓冲区

##### 4.2.1 创建流

- `FileReader(File file)`： 给定要读取的File对象。   
- `FileReader(String fileName)`： 给定要读取的文件的名称。 

##### 4.2.2 读取字符

``` java
        Reader reader = null;
        try {
            reader = new FileReader("file/b.txt");
            int len = 0;
            char[] buff = new char[1024];
            while ((len = reader.read(buff)) != -1) {
                System.out.println(new String(buff, 0, len));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```



### 5. 字符输出流

#### 5.1 Writer

`java.io.Writer  `字符输出流的抽象超类，主要功能：

- `void write(int c)` ：写入单个字符。
- `void write(char[] cbuf) `：写入字符数组。 
- `abstract void write(char[] cbuf, int off, int len) `
- `void write(String str) `：写入字符串。 
- `void write(String str, int off, int len)`
- `void flush() `：将该流的内存缓冲写入文件。  
- `void close()` ：刷新缓存并关闭此流。 

#### 5.2 FileWriter

##### 5.2.1 写入字符

``` java
        Writer fw = null;
        try {
            fw = new FileWriter("file/fw.txt");
            char[] chars = "bzzb永远的神".toCharArray();
            fw.write(chars);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fw != null) {
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

##### 5.2.2 追加写

构造函数中指定 `append` 为 true 即可开启追加写功能

``` java
fw = new FileWriter("file/fw.txt", true);
```



### 6. Properties

#### 6.1 概述

`java.util.Properties ` 继承于` Hashtable` ，使用键值结构存储数据。该类也被许多Java类使用，比如获取系统属性时，`System.getProperties` 方法就是返回一个`Properties`对象。

#### 6.2 使用

**构造方法**

- `public Properties()` :创建一个空的属性列表。

**基本的存储方法**

- `public Object setProperty(String key, String value)` ： 保存一对属性。  
- `public String getProperty(String key) ` ：使用此属性列表中指定的键搜索属性值。
- `public Set<String> stringPropertyNames() ` ：所有键的名称的集合。

```java
public class ProDemo {
    public static void main(String[] args) throws FileNotFoundException {
        // 创建属性集对象
        Properties properties = new Properties();
        // 添加键值对元素
        properties.setProperty("filename", "a.txt");
        properties.setProperty("length", "209385038");
        properties.setProperty("location", "D:\\a.txt");
        // 打印属性集对象
        System.out.println(properties);
        // 通过键,获取属性值
        System.out.println(properties.getProperty("filename"));
        System.out.println(properties.getProperty("length"));
        System.out.println(properties.getProperty("location"));
    }
}
```

#### 与流相关的方法

- `public void load(InputStream inStream)`： 从字节输入流中读取键值对。 

参数中使用了字节输入流，通过流对象，可以关联到某文件上，加载文本中的数据

```java
public class ProDemo2 {
    public static void main(String[] args) throws FileNotFoundException {
        // 创建属性集对象
        Properties pro = new Properties();
        // 加载文本中信息到属性集
        pro.load(new FileInputStream("read.txt"));
        // 遍历集合并打印
        Set<String> strings = pro.stringPropertyNames();
        for (String key : strings ) {
          	System.out.println(key+" -- "+pro.getProperty(key));
        }
     }
}
输出结果：
filename -- a.txt
length -- 209385038
location -- D:\a.txt
```
