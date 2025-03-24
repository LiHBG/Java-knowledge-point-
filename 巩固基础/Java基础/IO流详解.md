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



  


