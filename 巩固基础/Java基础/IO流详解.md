# IO流体系：

## File

- **相对路径：**不带盘符的路径信息，相对于当前项目文件而言。
- **绝对路径：**带盘符的路径。

**File类对象，指向的其实就是一个路径。**这个路径可以是**存在**的，也可以是**不存在**的；可以是***相对的***也可以是***绝对的***。

### 创建对象

1. **根据文件路径创建文件对象。**

   ```Java
           // 1.public File(String pathname)
           File file = new File("D:\\test\\test.txt");
   ```

2. 根据父级路径名 字符串和子路径名 字符串创建

   ```Java
           //2.public File(String parent, String child)
           File file = new File("D:\\test", "test.txt");
   ```

3. 根据父路径对应文件对象和子路径名 字符串创建文件对象

   ```Java
           // 3.public File(File parent, String child)
           File fileFather = new File("D:\\test");
           File file = new File(fileFather, "test.txt");
   ```

**总结：**

- File对象表示路径，可以是文件、也可以是文件夹。这个路径可以存在，也可以不存在。
- 绝对路径表示带盘符的路径信息；相对路径表示不带盘符的路径，基本是默认是在当前项目下进行。

### 常见成员方法

#### 判断、获取：

1. 判断路径名表示的是否是**文件夹**

   ```Java
           //1.判断是否是文件夹,如果是文件夹返回true，否则返回false
           boolean isDirectory = file.isDirectory();
   ```

2. 判断路径名表示的是否是**文件**

   ```Java
           //2.判断是否是文件,如果是文件返回true，否则返回false
           boolean isFile = file.isFile();
   ```

3. 判断路径名表示的File对象**是否存在**

   ```Java
           //3.判断是否存在,存在返回true，否则返回false
           boolean exists = file.exists();
   ```

4. 获取文件的大小（字节数量）细节：**如果当前File对象是文件夹，那么当前的方法返回值没有意义**

   ```Java
           //4.获取文件的大小（字节数量）
           long length = file.length();
   ```

5. 获取文件的绝对路径

   ```Java
           //5.获取文件的绝对路径
           String absolutePath = file.getAbsolutePath();
   ```

6. 获取定义文件时使用的路径

   ```Java
           //6.获取定义文件时的路径
           String path = file.getPath();
   ```

7. 获取文件的名称、带后缀名

   ```Java
           //7.获取文件的名称、带后缀
           String name = file.getName();
   ```

8. 获取文件最后修改时间（时间毫秒值）

   ```Java
           //8.获取文件最后的修改时间（时间毫秒值）
           long lastTime = file.lastModified();
   ```

#### 创建、删除：

1. 创建一个新的空文件，当前文件存在则创建失败，但是系统不会报错。如果当前文件的父级路径不存在则报错。该方法创建的一定是一个文件，不会创建文件夹。

   ```Java
           //1.创建一个新的空文件
           boolean createNewFileBoolean = file.createNewFile();
   ```

2. 创建单级文件夹（一个文件夹）

   ```Java
           //2.创建单级文件夹
           boolean mkdir = file.mkdir();
   ```

3. 创建多级文件夹，可以创建单层文件夹，也可以创建多层文件夹

   ```java
           //3.创建多级文件夹
           boolean mkdirs = file.mkdirs();
   ```

4. 删除文件、空文件夹；**直接从系统中删除，不会进入回收站**

   ```java
           //4.删除文件、空文件夹
           boolean delete = file.delete();
   ```

#### 获取、遍历：

- **获取当前路径下的所有内容。（返回File数组）**

  - 调用者File表示的路径不存在时，返回null
  - 调用者File表示的路径是文件时，返回null
  - 调用者File表示的路径是空文件夹时，返回一个长度为0的数组
  - 调用者File表示的路径是一个有隐藏文件的文件夹时，会将隐藏文件放进数组中返回
  - 调用者File表示的路径是需要权限才能访问的文件夹时，返回null

  ```Java
          //获取当前路径下的所有内容
          File[] files = file.listFiles();
  ```

- **遍历**

  ```Java
          for (File file1 : files) {
              System.out.println(file1.getName());
          }
  ```
  
- **补充**

  1. 获取当前系统的所有盘符

     ```Java
             //获取当前系统的所有盘符
             File[] roots = File.listRoots();
     ```

  2. 获取当前路径下的所有内容（返回String数组）

     ```Java
             //获取当前路径下的所有内容（字符串数组）
             String[] listStr = file.list();
     ```

  3. 利用文件名过滤器获取当前文件下的所有内容（返回String数组）

     ```java
             //利用文件名过滤器获取当前路径下的所有内容（字符串数组）
             file.list((dir, name) ->false);
     ```

  4. 利用文件名/文件路径过滤器获取当前文件下的所有内容（返回File数组）

     ```Java
             //利用文件名过滤器获取当前路径下的所有内容（文件数组）
             file.listFiles(new FilenameFilter() {
                 @Override
                 public boolean accept(File dir, String name) {
                     return false;
                 }
             });
             //利用文件过滤器获取当前路径下的所有内容（文件数组）
             file.listFiles(new FileFilter() {
                 @Override
                 public boolean accept(File pathname) {
                     return false;
                 }
             });
     ```

## IO 

**IO流：**来操作文件本身内容的操作，由程序来进行文件内容的读写。

### IO流的分类

- 按照流的方向分：输入流--输出流
- 按照操作文件类型分：字节流--字符流（纯文本文件）

**纯文本文件：Windows自带的记事本能够打开，并且能够读懂的文件。**

### IO流的常见使用

- 字节输入流（从文件->程序）：inputStream（抽象类）

  - 文件字节输入流：FileInputSteam

    **细节：**

    1. 如果文件不存在直接抛出异常

    2. 读取到文件的末尾并且读取不到数据了，那么read()方法的返回值为-1

    3. 循环读取（必须使用中间变量来接收读取的数据，如果直接使用read，在代码中出现一次就会移动一次读取的光标，这样会造成数据的丢失）

       ```java
               int b;
               while ((b = fis.read()) != -1){
                   System.out.print((char) b);
               }
       ```

    4. read(byte[] buffer)，当前方法的返回值是读取了多少个字节，并且会将读取到的内容放入传递的数组中。但是在多次读取中，将读取到的数据装入数组中时并不会将原本数组中的内容清空，而是从头开始逐次进行覆盖。

- 字节输出流（从程序->文件）：outputStream（抽象类）

  - 文件字节输出流：FileOutputStream

    **细节：**

    1. 创建输出流通道的时候，指定的参数可以是字符串也可以是文件File对象

       ```java
           //传输的参数是字符串时：
       	public FileOutputStream(String name, boolean append)
               throws FileNotFoundException
           {
               this(name != null ? new File(name) : null, append);
           }
       	//传输的参数是文件（并且传输的参数是字符串时，底层调用的也是该方法）
           public FileOutputStream(File file, boolean append)
               throws FileNotFoundException
       ```

    2. 在File对象指定的路径中，文件本身可以不存在，但是文件的父级路径必须存在，不然就会报错。

    3. 如果文件本身存在，那么如果在创建流的通道之后，但是不进行输出就直接关闭了通道，那么文件本身的内容也会被清空。（建议：在创建通道的时候，添加能够续写的参数）

    4. 如果在写入内容的时候需要进行换行写，那么在输出中写入一个换行符即可。Windows：/r/n ；liunx：/n ；mac：/r也可以使用一个方法

    ```java
    FileOutputStream fos = new FileOutputStream("D:\\workpace\\文件字节输出流.txt",true);//不会清空原有的内容
    
    fos.write(bytes, 0, bytes.length);//参数一：字节数组，参数二：开始位置（索引），参数三：写入字节的个数
    
    //文件字节输出流
            
            //1.获取字节输出流对象，并指定需要操作的文件的路径
            //FileOutputStream fos = new FileOutputStream("D:\\workpace\\文件字节输出流.txt");//当前文件路径不存在，指当前文件(文件字节输入流.txt)不存在
            //FileOutputStream fos = new FileOutputStream("D:\\workpace\\文件字节输出流.txt");//当前电脑不存在E盘
            FileOutputStream fos = new FileOutputStream("D:\\workpaceTest\\文件字节输出流.txt");//当前电脑不存在这个文件夹和文件
            //  思考：如果指定的路径没有这个文件会发生什么？
            //  结论：如果父级路径准确，那么就会创建一个这样的文件，并且会将需要写入的内容写入到文件中
            //  思考：如果没有指定的父级路径会发生什么？
            //  结论：如果父级路径不正确，那么就会抛出异常（E:\workpace\文件字节输出流.txt (系统找不到指定的路径。)
            //  思考：如果指定的盘符存在，但是在文件前的文件夹不存在，会发生什么？
            //  结论：会抛出异常，并不会创建会存储该文件的文件夹（D:\workpaceTest\文件字节输出流.txt (系统找不到指定的路径。)）
            String str = "hello world";
            byte[] bytes = str.getBytes(StandardCharsets.UTF_8);//尽量在进行转化的时候使用需要指定编码格式的方法，否则有可能出现乱码
            //2.写出数据
            fos.write(bytes);
            //3.释放资源
            fos.close();
    ```

  **文件拷贝**

  ```java
  // 文件拷贝
  FileInputStream fis = null;
  FileOutputStream fos = null;
  
  try {
      fis = new FileInputStream("D:\\workpace\\文件字节输入流.txt");
      fos = new FileOutputStream("D:\\workpace\\文件字节输出流.txt");
  
      byte[] bytes = new byte[1024 * 1024 * 5];
      
      int len;
      while ((len = fis.read(bytes)) != -1) {
          fos.write(bytes, 0, len);
      }
  } catch (Exception e) {
      e.printStackTrace();
  } finally {
      try {
          fis.close();
          fos.close();
      } catch (Exception e) {
          e.printStackTrace();
      }
  }
  ```