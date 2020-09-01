[toc]

# Java  IO

## 第一部分：IO 基础

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

- `FileReader(File file)`： 给定要读取的 File 对象。   
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



## 第二部分：高级流

### 1. 缓冲流

#### 1.1 概述

缓冲流是对4个基本流的增强，按照数据类型分类：

* **字节缓冲流**：`BufferedInputStream`，`BufferedOutputStream` 
* **字符缓冲流**：`BufferedReader`，`BufferedWriter`

缓冲流的基本原理，是创建了一个内置的默认大小的缓冲区数组，一次读写一个缓冲区的大小，减少系统IO次数，从而提高读写的效率。

#### 1.2 字节缓冲流

##### 1.2.1 构造函数

``` java
// 创建字节缓冲输入流
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("x.txt"));
// 创建字节缓冲输出流
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("x.txt"));
```

##### 1.2.2 使用

基本 API 与字节流相同

``` java
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;

        try {
            bis = new BufferedInputStream(new FileInputStream("file/DockerToolbox-18.03.0-ce.exe"));
            bos = new BufferedOutputStream(new FileOutputStream("D:/test.exe"));

            int len = 0;
            byte[] buffer= new byte[8192]; // 此处设置的数组长度与缓冲区相同
            while ((len = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, len);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 省略关闭资源的代码
            ...
        }
    }
```

#### 1.3 字符缓冲流

##### 1.3.1 构造方法

``` java
// 创建字符缓冲输入流
BufferedReader br = new BufferedReader(new FileReader("x.txt"));
// 创建字符缓冲输出流
BufferedWriter bw = new BufferedWriter(new FileWriter("x.txt"));
```

##### 1.3.2 特有方法

字符缓冲流除了普通字符流的方法外，还有以下特有方法：

* BufferedReader：`public String readLine()`: 读一行文字，读到文件末尾返回 null 。 
* BufferedWriter：`public void newLine()`: 写一空行，由系统属性定义符号。 

##### 1.3.3 使用

```java
public static void main(String[] args) throws IOException {
    
    BufferedReader br = new BufferedReader(new FileReader("file/a.txt"));
    String line  = null;
  	// 循环读取,读取到最后返回null
    while ((line = br.readLine())!=null) {
        System.out.print(line);
        System.out.println("------");
    }
	// 释放资源
    br.close();
}
```


### 2. 转换流

#### 2.1 概述

使用`FileReader` 读取文本文件，当编码不统一时，将出现乱码

 Java 引入转换流实现 **字节流** 和 **字符流** 的转换，转换流类型：

- `InputStreamReader`：实现字节流到字符流的转换
- `OutputStreamWriter`：实现字符流到字节流的转换

<img src="img/转换流.jpg"/>

#### 2.2 InputStreamReader

转换流`java.io.InputStreamReader`，是Reader的子类，FileReader 的父类，是从字节流到字符流的桥梁。它读取字节，并使用指定的字符集将其解码为字符。

##### 2.2.1 构造方法

``` java
// 创建使用默认字符集的字符流
InputStreamReader isr = new InputStreamReader(new FileInputStream("in.txt"));
// 创建使用指定字符集的字符流
InputStreamReader isr2 = new InputStreamReader(new FileInputStream("in.txt") , "GBK");
```

##### 2.2.2 使用

``` java
        String FileName = "file/c.txt";
        int read;

        // 创建流对象,指定编码
        InputStreamReader isr = new InputStreamReader
            (new FileInputStream(FileName) , "GBK");
        
        while ((read = isr.read()) != -1) {
            System.out.print((char)read);
        }
        isr.close();
```

#### 2.3 OutputStreamWriter

转换流`java.io.OutputStreamWriter` ，是Writer的子类，FileWriter 的父类，是从字符流到字节流的桥梁。使用指定的字符集将字符编码为字节。 

##### 2.3.1 构造方法

```java
// 创建使用默认字符集的字符流
OutputStreamWriter isr = new OutputStreamWriter(new FileOutputStream("x.txt"));
// 创建使用指定字符集的字符流
OutputStreamWriter isr2 = new OutputStreamWriter(new FileOutputStream("out.txt") , "GBK");
```

##### 2.3.2 使用

``` java
        String FileName = "x.txt";
     	// 创建流对象,指定编码
        OutputStreamWriter osw = new OutputStreamWriter(new             		FileOutputStream(FileName),"GBK");
      	osw2.write("你好"); // 保存为4个字节
        osw2.close();
```



### 3. 序列化与反序列化

#### 3.1 概述

Java 提供了一种对象**序列化**的机制，用一个字节序列可以表示一个对象，该字节序列包含该对象的数据、类型和属性，相当于文件中**持久保存**了一个对象的信息。 

反之，该字节序列还可以从文件中读取回来，重构对象，对它进行**反序列化**。

<img src="img/序列化与反序列化.jpg"/>

#### 3.2 序列化

一个对象要想序列化，必须满足两个条件:

* 该类必须实现`java.io.Serializable ` 接口，`Serializable` 是一个标记接口，不实现此接口的类将不会使任何状态序列化或反序列化，会抛出`NotSerializableException` 。
* 该类的所有属性必须是可序列化的。如果有一个属性不需要可序列化的，需要使用`transient` 关键字修饰。

序列化实现

``` java
    	Person e = new Person();
        e.setXXX; // 设置属性
    	try {
      		// 创建序列化流对象
          ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("x.txt"));
        	// 写出对象
        	out.writeObject(e);
        } catch(IOException e)   {
            e.printStackTrace();
        } finally {
            out.close();
        }
```

#### 3.3 反序列化

反序列的要求：

- 对于可以反序列化的对象，JVM 需要能够找到 class 文件，否则抛出`ClassNotFoundException` 异常。
- class 文件在序列化对象后没有发生修改，否则反序列化将失败，抛出 `InvalidClassException`异常。`Serializable` 接口给需要序列化的类，提供了一个序列版本号 `serialVersionUID`，用于验证序列化的对象和对应类是否匹配

反序列化实现：

``` java
        Person e = null;
        try {		
             // 创建反序列化流
             FileInputStream fileIn = new FileInputStream("x.txt");
             ObjectInputStream in = new ObjectInputStream(fileIn);
             // 读取一个对象
             e = (Person) in.readObject();
        } catch (IOException e) {
             e.printStackTrace();
        } catch (ClassNotFoundException e)  {
             e.printStackTrace();
        } finally {
            // 释放资源
            ... 
        }
```

