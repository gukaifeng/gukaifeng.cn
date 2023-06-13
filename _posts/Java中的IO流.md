
## 1. `java.io.File` 类的使用

File 类能新建、删除、重命名文件和目录，但 File 不能访问文件内容本身。

如果要访问文件内容本身，要用输入/输出流。
<!--more-->
```java
File f = new File("./file/abc.txt");

// 访问文件名
f.getName(); // 获取当前文件名或文件夹名（就是路径的最后一部分）
f.getPath(); // 获取当前路径（所有部分，就是 new File 传的参数）
f.getAbsolutePath(); // 获取绝对路径
f.getAbsoluteFile(); // 返回一个由当前文件绝对路径构建的 File 类
f.getParent(); // 返回当前文件或文件夹的父级路径
f.renameTo(); // 给文件或文件夹重命名


// 文件检测
f.exist(); // 返回文件或文件夹是否存在
f.canWrite(); // 返回是否可写
f.canRead(); // 返回是否可读
f.isFile(); // 返回是否为文件
f.isDirectory(); // 返回是否为文件夹


// 获取常规文件信息
f.lastModified(); // 获取文件最后的修改时间，为自 1970年1月1日0:00 以来的毫秒数。没有已知的最后修改时间则会返回当前时间。
f.length(); // 返回文件的长度（字节数）


// 文件操作相关
f.createNewFile(); // 创建 f 文件，返回文件是否创建成功，需要捕获异常
// createNewFile()  代码演示
if(!f.exists()) {
    try {
        System.out.println(f.createNewFile());;
    } catch (IOException e) {
        e.printStackTrace();
    }
}

f.delete(); // 删除 f 文件或文件夹，返回是否成功删除

// 目录操作相关
f.mkDir(); // 创建 f 文件夹，返回是否创建成功，可以创建多级目录
f.list(); // 返回目录下的内容的名称，返回一个 String[]
f.listFiles(); // 返回目录下的内容的 File 对象，返回一个 File[]，即为目录下所有内容各创建一个 File 类
```

注：IDEA中相对路径从项目根目录下开始找，不是包目录也不是当前 class 文件目录。



## 2. IO原理及流的分类

### 2.1. IO原理

IO 流用来处理设备之间的数据传输。Java程序中，对数据的输入/输出操作以流（Stream）的方式进行。java.io 包下提供了各种流类和接口，用于获取不同种类的数据，并通过标准的方法输入或输出数据。

### 2.2. 流的分类

1. 按操作数据单位分类：字节流（8 bit）、字符流（16 bit）；
2. 按数据流的流向分类：输入流、输出流；
3. 按流的角色分类：节点流、、处理流。

| 抽象基类 |    字节流    | 字符流 |
| :------: | :----------: | :----: |
|  输入流  | InputStream  | Reader |
|  输出流  | OutputStream | Writer |

Java 的 IO 流共涉及 40 多个类，实际上非常规则，都是从 4 个抽象基类派生的。由这四个类派生出来的子类名称都是以其父类名作为子类名后缀的。

IO 流体系如下，只需掌握深色背景部分。

![](https://gukaifeng.cn/posts/java-zhong-de-io-liu/Java%E4%B8%AD%E7%9A%84IO%E6%B5%81_1.png)

## 3. 文件字节流

### 3.1. 文件字节输入流（FileInputSteam）

通过字节流的方式来读取一个文件

```java
// read() 方法按字节读取，返回读取的长度
// 受 f.read(b); 中 b 的长度影响，可能读不下，如果读不下，就把 b 读满，返回读取的长度
// f 中的内容只能读取一次，如果读完了，再读就没了，read() 返回 -1
// 如果前面的 read() 没有读完 f 中的内容，下次 read() 继续读
// 对同一个 byte[]，后面的 read()，会覆盖 byte[] 原有的内容，只覆盖到新读数据的长度

try {
    FileInputStream f = new FileInputStream("file/abc32.txt");
    byte[] b = new byte[100];
    int a = f.read(b);
    if(a != -1)
        System.out.println(new String(b, 0, a));
    f.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

### 3.2. 文件字节输出流（FileOutputStream）

```java
try {
    FileOutputStream f = new FileOutputStream("file/abc.txt"); // 文件存在就清空，不存在就创建一个
    String s = new String("Qixueting 520");
    f.write(s.getBytes()); // 输出到内存，这步已经写完了，因为是字节流输出，要把 String 转换为 btye[]
    f.flush();  // flush是清空缓存用的，把缓存区的内容刷到硬盘
//    这个是缓冲区的问题.
//    java在使用流时,都会有一个缓冲区,按一种它认为比较高效的方法来发数据:把要发的数据先放到缓冲区,缓冲区放满以后再一次性发过去,而不是分开一次一次地发.
//    而flush()表示强制将缓冲区中的数据发送出去,不必等到缓冲区满.
//    所以如果在用流的时候,没有用flush()这个方法,很多情况下会出现流的另一边读不到数据的问题,特别是在数据特别小的情况下.
//    举个例子
//    就是你的buffer里面有一大堆东西，每write一个就增加一个，如果不用flush可能面临缓存溢出或者其他问题，一般write()方法后必须跟一个flush()以释放buffer
    f.close();
} catch(IOException e) {
    e.printStackTrace();
}
```

### 3.3 编程：文件字节流复制文件

```JAVA
String in = "file/abc32.txt";
String out = "file/abc44/abc32.txt";
try {
    FileInputStream fin = new FileInputStream(in);
    byte[] b = new byte[10];
    int len = 0;
    FileOutputStream fout = new FileOutputStream(out);
    while((len = fin.read(b)) != -1) 
        fout.write(b, 0, len);
    fout.flush();
    fin.close();
    fout.close();
} catch(IOException e) {
    e.printStackTrace();
}
```

注：字节流非常通用，可以用来操作字符的文档，也可以操作任何其他类型的文件（包括图片，压缩包等），因为字节流直接使用二进制。



## 4. 文件字符流

### 4.1. 文件字符输入流

与字节流操作类似，见 4.3 编程。

### 4.2. 文件字符输出流

与字节流操作类似，见 4.3 编程。

### 4.3. 编程：文件字符流复制文件

```JAVA
public void copy(String inputPath, String outputPath) {
    try {
        FileReader fr = new FileReader(inputPath);
        FileWriter fw = new FileWriter((outputPath));
        char[] ch = new char[100];
        int len = 0;
        while((len = fr.read(ch)) != -1) {
            fw.write(ch, 0, len);
        }
        fw.flush();
        fr.close();
        fw.close();
    } catch(Exception e) {
        e.printStackTrace();
    }
}
```

注：字符流仅使用内容是字符的文档。

## 5. 处理流

### 5.1. 缓冲流

文件流是内存与外存上进行的io操作，比较慢，因此有了缓冲流。

`BufferedInputStream`/`BufferedOutputStream` 对应 `FileInputStream`/`FileOutputStream`

`BufferedReader`/`BufferedWriter` 对应 `FileReader`/`FileWriter`

缓冲流就是先把数据缓冲到内存里，在内存中进行io操作。

对于输出的缓冲流，写出的数据会现在内存中缓存，使用 flush() 将会使内存中的数据立刻写出。

#### 5.1.1 缓冲字节流

与之前的操作很类似，参照复制文件代码

```Java
public void BufferedByteStreamCopy(String inputPath, String outputPath) {
    try {
        FileInputStream fin = new FileInputStream(inputPath);
        BufferedInputStream br = new BufferedInputStream(fin);
        FileOutputStream fout = new FileOutputStream(outputPath);
        BufferedOutputStream bw = new BufferedOutputStream(fout);

        byte[] b = new byte[10];
        int len = 0;
        while((len = br.read(b)) != -1)
            bw.write(b, 0, len);
        bw.flush();

        bw.close();
        fout.close();
        br.close();
        fin.close();
    } catch(IOException e) {
        e.printStackTrace();
    }
}
```

#### 5.1.2. 缓冲字符流

与之前的操作很类似，参照复制文件代码

```java
public void BufferedCharStreamCopy(String inputPath, String outputPath) {
    try {
        FileReader fr = new FileReader(inputPath);
        BufferedReader br = new BufferedReader(fr);
        FileWriter fw = new FileWriter((outputPath));
        BufferedWriter bw = new BufferedWriter(fw);
        char[] ch = new char[100];
        int len = 0;
        while((len = br.read(ch)) != -1)
            bw.write(ch, 0, len);
        bw.flush();

        bw.close();
        fw.close();
        br.close();
        fr.close();
    } catch(Exception e) {
        e.printStackTrace();
    }
}
```



### 5.2. 转换流

转换流提供了在字节流和字符流之间的转换

`InputStreamReader` 和 `OutputStreamWriter`

当字节流指中的数据都是字符时，转换成字符流操作更高效。

`InputStreamReader`用于将字节流中读取到的字符按指定字符集解码成字符，需要和 InputStream “套接”。

具体使用方法见下面复制文件代码，注意编码选择要匹配。

```java
public void StreamCopy(String inputPath, String outputPath) throws Exception{
    FileInputStream fr = new FileInputStream(inputPath);
    InputStreamReader tr = new InputStreamReader(fr, StandardCharsets.UTF_8);
    FileOutputStream fw = new FileOutputStream(outputPath);
    OutputStreamWriter tw = new OutputStreamWriter(fw, StandardCharsets.UTF_8);

    char[] ch = new char[100];
    int len = 0;

    while((len = tr.read(ch)) != -1)
        tw.write(ch, 0, len);
    tw.flush();

    tw.close();
    fw.close();
    tr.close();
    fr.close();
}
```





### 5.3. 标准输出输出流

`System.in` 和 `System.out` 分别代表了系统标准的输入和输出设备。

默认输入设备是键盘，输出设备是显示器。

System.In 的类型是 InputStream

System.out 的类型是 PrintStream，其是 OutputStream 的子类 FilterOutputStream 的子类

练习：把控制台输入的内容写到指定的 txt 文件中，读到 over 结束。

```java
public void write(String outputPath) throws Exception{

    InputStreamReader isr = new InputStreamReader(System.in);
    BufferedReader br = new BufferedReader(isr);
    FileWriter fw = new FileWriter(outputPath);
    BufferedWriter bw = new BufferedWriter(fw);

    String in = ""; // 用于接收输入
    while((in = br.readLine()) != null) {
        if(in.equals("over"))
            break;
        bw.write(in);
    }
    bw.flush();

    bw.close();
    fw.close();
    br.close();
    isr.close();
}
```





### 5.4. 对象流

`ObjectInputStream` 和 `ObjectOutputStream`

用于存储和读取对象的处理流。它的强大之处就是可以把 Java 中的对象写到数据源中，也能把对象从数据源中还原回来。

1. 序列化（Serialize）：用 `ObjectOutputStream` 类将一个 Java 对象写入 IO 流中。
2. 反序列化（Deserialize）：用 `ObjectInputStream` 类从 IO 流中恢复该 Java 对象。

`ObjectInputStream` 和 `ObjectOutputStream` 不能序列化 `static` 和 `transient` 修饰的成员变量。

序列化和反序列化，针对的是对象的属性，不包括类的属性。

正式因为要保存对象到硬盘（对象的持久化）和对象的网络编程，就产生了对象的输入与输出流。对象序列化机制允许把内存中的 Java 对象转换成平台无关的二进制流，从而把正宗二进制流持久地保存在磁盘上，或者通过网络将这种二进制流传输到另一个网络结点。当其他程序获取了这种二进制流，就可以恢复成原来的 Java 对象。

序列化的好处在于可以将任何实现了 `Serializable` 接口的对象转化为字节数据，使其在保存和传输时可被还原。

序列化是 RMI (Remote Method Invoke - 远程方法调用) 过程的参数和返回值都必须实现的机制，而 RMI 是 JavaEE 的基础，因此序列化机制是 JavaEE 平台的基础。

如果要让某个对象支持序列化机制，则必须让其类是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一：

1. **`Serializable`**
2. `Externalizable(不常用)`

凡是实现 `Serializable`接口的类都有一个表示序列化版本标识符的静态变量：

`private static final long serialVersionUID;`

serivalVersionUID 用来表名类的不同版本间的兼容性

如果类没有显式地定义这个静态变量，它的值是 Java 运行时环境根据类的内部细节自动生成的。若类的源代码做了修改，serialVersionUID 可能发生变化。故建议显示声明。

显式定义 serialVersionUID 的用途。若希望类的不同版本对序列化兼容，就需要确保类的不同版本具有相同的 serivalVersionUID。若不希望类的不同版本对序列化兼容，就需要保证不同版本具有不同的 serivalVersionUID。

**对象序列化和反序列化代码范例**

```java
// 可序列化对象
class Person implements Serializable {
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    private static final long serialVersionUID = 1L;
    public String name;
    public int age;
}
```

```java
// 序列化和反序列化
class testSerialize {
    public void serialize(Person obj, String outputPath) throws Exception {
        FileOutputStream fos = new FileOutputStream(outputPath);
        BufferedOutputStream bos = new BufferedOutputStream(fos);
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(obj);
        oos.flush();
        oos.close();
        bos.close();
        fos.close();
    }
    public Person deserialize(String inputPath) throws Exception {
        FileInputStream fis = new FileInputStream(inputPath);
        BufferedInputStream bis = new BufferedInputStream(fis);
        ObjectInputStream ois = new ObjectInputStream(bis);
        Object obj = ois.readObject();
        return (Person)obj;
    }
}
```

```java
// 测试
public static void main(String[] args) {
    String outputpath = "file/abc/obj";
    Person p = new Person("gukaifeng", 22);
    try {
        new testSerialize().serialize(p, outputpath);
        Person p0 = new testSerialize().deserialize(outputpath);
        System.out.println(p0.name);
        System.out.println(p0.age);

    } catch (Exception e) {
        e.printStackTrace();
    }
}

// 输出：
// gukaifeng
// 22
```

注：

1. 如果某个类的字段不是基本数据类型或 String 类型，而是另一个引用类型，那么这个引用类型必须是可序列化的，否则拥有该类型 Field 的类也不能序列化。
2. 对象的序列化和反序列化使用的类要严格一致，包名、类名、结构等。



## 6. `RandomAccessFile` 类

`RandomAccessFile` 类支持随机存取。支持只访问文件部分内容，可以向已存在的文件后追加内容。

`RandomAccessFile` 对象包含一个记录指针，用来标识当前读写处指针。

`RandomAccessFile`对象可以自由移动记录指针：

* long getFilePointer() : 获取文件记录指针的当前位置；
* void seek(long pos)：将文件记录指针定位到 pos 位置。

构造器：

* `public RandomAccessFile(File file, String mode)`
* `public RandomAccessFile(String name, String mode)`

创建 `RandomAccessFile` 类实例需要指定一个 `mode` 参数，该参数指定 `RandomAccessFile` 的访问模式：

1. r：以只读方式打开；
2. rw：以读写方式打开；
3. rwd：以读写方式打开、同步文件内容的更新；
4. rws：打开以便读取和写入、同步文件内容和元数据的更新。

代码测试：

```
// 文本内容
QiXueting
520
yayaya
傻子老婆我爱你咿呀咿呀呦
```

1. r 测试

    ```java
    public void testr(String path) throws Exception {
        RandomAccessFile raf = new RandomAccessFile(path, "r");
        raf.seek(10); // 设置记录指针位置，每行后还有换行符
        byte[] b = new byte[1024];
        int len = 0;
        while((len = raf.read(b)) != -1) {
            System.out.println(new String(b, 0, len));
        }
        raf.close();
    }
    // 520
    // yayaya
    ///傻子老婆我爱你咿呀咿呀呦
    ```

2. rw 测试

    ```java
    public void testrw(String path) throws Exception {
        RandomAccessFile raf = new RandomAccessFile(path, "rw");
        raf.seek(0); // 把记录指针移到起点
        byte[] b = new byte[1024];
        int len = 0;
        while((len = raf.read(b)) != -1)
            System.out.println(new String(b, 0, len));
    
        raf.seek(0); // 把记录指针移到起点
        raf.write("h顾hh".getBytes()); // 在文档开头写数据
    
        raf.seek(raf.length()); // 把记录指针移到末尾
        raf.write("g\ngggg".getBytes()); // 在文档末尾写
    }
    
    // System.out.println(new String(b, 0, len)); 输出内容
    // Qixueting
    // 520
    // yayaya
    // 傻子老婆我爱你咿呀咿呀呦
    
    // 文本内容变更为
    // hh顾hing
    // 520
    // yayaya
    // 傻子老婆我爱你咿呀咿呀呦g
    // gggg
    ```

    注：

    1. 在非末尾的位置写，会覆盖掉等长的原数据，其中汉字长度是字符两倍。
    2. 文件读写数据都会把记录指针移到读到数据的末尾和写完数据的末尾。比如在末尾写完后指针依然在末尾。若要有其他操作需要重新 seek()。
    3. `RandomAccessFile` 的写操作，无需 flush();