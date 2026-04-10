### 文件基本操作
#### 创建文件流
```java
package FILE;  
import java.io.File;  
import java.io.IOException;  
  
public class file {  
    public static void main(String[] args) {  
        File f1=new File("E:\\java_code\\文件实验\\java.txt");  
        File f2=new File("E:\\java_code\\文件实验\\bean\\ji");  
        //1.在目录下创建一个文件,哎，不能直接写哎  
        try {  
            if (f1.createNewFile()) {  
                System.out.println("文件创建成功: " + f1.getAbsolutePath());  
            } else {  
                System.out.println("文件已存在");  
            }  
        } catch (IOException e) {  
            System.err.println("创建文件失败: " + e.getMessage());  
        }  
        //2.在目录下创建一个多级目录,这个不需要检查异常,mkdirs只能创建一个目录  
        f2.mkdirs();  
  
        System.out.println(f2.isDirectory());//测试是否为目录  
        System.out.println(f2.isFile());//测试是否为文件  
        System.out.println(f2.exists());//测试不管是文件还是目录吧，存在不存在  
        System.out.println(f2.getAbsolutePath()+" "+f1.getAbsolutePath());//返回绝对路径名，String形式 E:\java_code\文件实验\bean\ji  
        System.out.println(f2.getPath()+" "+f1.getPath());  
        System.out.println(f2.getName()+" "+f1.getName());  
  
        File f3=new File("E:\\java_code\\文件实验");  
        String[] a1=f3.list();//生成的是子目录的文件名（用字符串表示），返回一个String[]数组  
        //public File[] listFiles()  一堆返回一个File数组  
        for(String s:a1) {  
            System.out.println(s);  
        }  
        File[] fileArray=f3.listFiles();  
        for(File f:fileArray) {  
            System.out.println(f);  
        }  
    }  
}  
  
//文件已存在  
//true  
//false  
//true  
//E:\java_code\文件实验\bean\ji E:\java_code\文件实验\java.txt  
//E:\java_code\文件实验\bean\ji E:\java_code\文件实验\java.txt  
//ji java.txt  
//bean  
//java.txt  
//E:\java_code\文件实验\bean  
//E:\java_code\文件实验\java.txt
```
#### 写入文件
```java
import java.io.File;  
import java.io.FileNotFoundException;  
import java.io.FileOutputStream;  
import java.io.IOException;  
//void write(int b);  
  
public class file {  
    public static void main(String[] args) throws IOException {  
        File f1=new File("E:\\java_code\\文件实验\\java.txt");  
        File f2=new File("E:\\java_code\\文件实验\\bean\\ji");  
        //1.在目录下创建一个文件,哎，不能直接写哎  
        try {  
            if (f1.createNewFile()) {  
                System.out.println("文件创建成功: " + f1.getAbsolutePath());  
            } else {  
                System.out.println("文件已存在");  
            }  
        } catch (IOException e) {  
            System.err.println("创建文件失败: " + e.getMessage());  
        }  
        //写文件  
//        FileOutputStream fos=new FileOutputStream("E:\\java_code\\文件实验\\java1.txt");  
        //创建了一个文件,这个函数参数可以是String，相当于创建了一个File文件，或者File类型的  
        //Alt+Enter 选第一个抛出异常  
        //方法一：  
//        fos.write('9');//会从文件开始处写入  
        //方法二  
        byte[] arr="\nhghjfdsjkl".getBytes();//可实现换行  
//        fos.write(arr);  
//  
//        fos.write(arr,5,2);//从数组的第五个元素开始写入两个字节  
  
        //如何追加写入  
        FileOutputStream fos1=null;  
        try{  
        fos1=new FileOutputStream("E:\\java_code\\文件实验\\java1.txt",true);  
        fos1.write(arr);  
        }catch(IOException e){  
            e.printStackTrace();  
        }finally{  
            if(fos1!=null){  
                fos1.close();  
            }  
        }  
    }  
}
```

#### 读取文件
字节流和字符流   可以读懂的就叫字符流
```java
public class file {  
    public static void main(String[] args) throws IOException {  
        File f1=new File("E:\\java_code\\文件实验\\java.txt");  
        File f2=new File("E:\\java_code\\文件实验\\bean\\ji");  
        //创建输入流对象  
        FileInputStream fis=new FileInputStream(f1);  
//读取一个字节数据
        int by=fis.read();  
        System.out.println((char)by);  
  
        while((by=fis.read())!=-1){//如果到达文件末尾  
            System.out.print((char)by);  
        }  
        fis.close();  
}}

   //读取多个字节数据  
        System.out.println("+");  
        byte[] a= new byte[4];  
        int d=fis.read(a);//返回类型为int  
        System.out.println(new String(a));  
        fis.close();  
//        fos.close();
```

```java
//复制图片
        FileInputStream fis = new FileInputStream("F:\\画\\friends.png");//读取文件流  
        FileOutputStream fos=new FileOutputStream("F:\\画\\copy.png");  
        byte[] buf=new byte[1024];  
        int len;  //存储每次实际读取的字节数
        while((len=fis.read(buf))!=-1){  
            fos.write(buf,0,len);  
        }  
        fis.close();  
        fos.close();  
}
```

#### 字符流
字符流=字节流+编码表
用字节流复制带中文的文本文件时，底层操作会自动进行字节拼接成中文
```java
String s="中国";
byte[] bys=s.getBytes("GBK")//解码方式按照GBK来解码

//字符流  
        FileOutputStream fos=new FileOutputStream("E:\\java_code\\文件实验\\java.txt",true);//输出文件流  
        OutputStreamWriter osw = new OutputStreamWriter(fos);  
// 写方法  
        // 第一种  
        osw.write('中');//写文件,必须要最后刷新一下  
        osw.write('几');  
        //第二种  
        String a= "方式";  
        osw.write(a);  
        //第三种  
        char[] ch={'在','发'};  
        osw.write(ch);  
        osw.flush();  
//读数据
```