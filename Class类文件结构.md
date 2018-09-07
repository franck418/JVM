这里的Class文件不只是类编译后的文件，接口编译后也会生成Class文件。

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，中间没有任何分隔符。当遇到需要占用8位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8位字节进行存储。

Java虚拟机规范规定，Class文件格式采用一种类似于C语言结构体的伪结构来存储数据，这种伪结构只有两种数据类型：**无符号数**和**表**
    
> 无符号数：基本的数据类型，以u1、u2、u4、u8来分别代表1个字节，2个字节、4个字节和8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编号构成的字符串。  
    
> 表：由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性地以“_info”结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表。
  
  整个Clss文件的格式如下：
  
  ```
1    ClassFile{
2        u4                 magic;                              // 魔数
3        u2                 minor_version;                      //次要版本号
4        u2                 major_version;                      //主要版本号
5        u2                 constant_pool_count;                //常量池数量
6        cp_info            constant_pool[constant_pool_count-1]//常量池
7        u2                 access_flags;                       //类或接口的访问信息       
8        u2                 this_class;                         //类索引
9        u2                 super_class;                        //父类索引
10       u2                 interface_count;                    //接口数量
11       u2                 interfaces[interfaces_count]        //接口表
12       u2                 fields_count;                       //实例变量和类变量的数量
13       field_info         fields[fields_count];               //字段表
14       u2                 methods_count;                      //方法表的数量
15       method_info        methods[methods_count];             //方法表
16       u2                 attributes_count;                   //属性表数量
17       attribute_info     attributes[attributes_count];       //属性表
        
    }
  ```
  
这么看着这个结构还是有些懵逼的。接下来，用一个实例来说明各个部分。先说明一下，第一列是这些属性的类型。第二列是这些属性的名称。还有注意的是，类似数组的结构，其实不是数组来的，中括号里面的是数量；例如attributes[attributes_count] 如果attributes_count的值是4，则是attributes[4]，意思是有4个属性表。

下面我们来看一个类：
```
1 package demo;
2
3 public class Test {
4
5    private int i;
6
7    public int test(){
8
9        return i+1;
10    }
11 }

```
就这么简单的11行代码。编译成Class文件后，用WinHex打开，就可以看到反编译代码了。

![image](https://camo.githubusercontent.com/81ef7120cc3b94bab834b86d173c56d98f89837a/68747470733a2f2f6e6f74652e796f7564616f2e636f6d2f7977732f7075626c69632f7265736f757263652f39633531313463623933303861396164626639383839303430363132353463342f786d6c6e6f74652f34354331453233413233383434303234424544433832353038353933333639432f32393034)


下面按照Class文件结构进行分析：

#### 1 ) u4 magic  首先是4个字节长度魔数
    存放在地址 00000000~00000003 里面，值是0xCAFABABE 。魔数的左右是确定这个文件是被虚拟机接受的Class文件。而不是jpg，gif其他文件。  
    其他文件有其他文件的魔数。使用魔数而不使用后缀名来确定文件是因为后缀名可以随意改动。魔数是文件格式的制定者随意取的。 只要不是被广泛使
    用导致容易重复就可以了。


<br/>

#### 2 ) u2 minor_version 次要版本号
     接下来的两个字节 00000004~00000005 存放的是次要版本号。这里的值是0x0000

<br/>

#### 3 ) u2 major_version 主要版本号
    接下来的两个字节 00000006~00000007 存放的是主要版本号。值为 0x0033 转成十进制后就是：51。Java的版本号是从45开始的。每个JDK大版本发布，
    主版本号就加一。高版本JDK能向下兼容以前版本的Class文件，但不能运行以后版本的Class文件。下面是一些JDK对应版本号的列表。
    
    J2SE 7 = 51 (0x33 hex),  
    J2SE 6.0 = 50 (0x32 hex),  
    J2SE 5.0 = 49 (0x31 hex),  
    JDK 1.4 = 48 (0x30 hex),  
    JDK 1.3 = 47 (0x2F hex),  
    JDK 1.2 = 46 (0x2E hex),  
    JDK 1.1 = 45 (0x2D hex).  

#### 4 )  u2 constant_pool_count 常量池数量。
    存放在地址00000008 ~ 00000009中。值是0x0016，转成十进制之后就是22。这就代表常量池中有21个常量，索引值为1~21。索引0表示“不引用任何一个常量池项目”。

#### 5 ) cp_info constant_pool[constant_pool_count-1] 常量池
    常量池中存在不同种类的常量。每一种常量都是一个表，都有自己的结构，并且每一个常量都用一个公共部分u1 类型的tag来表示是哪种类型的常量。
    JDK1.7之前有11种常量，JDK1.7为了更好地支持动态语言调用，有增加了3种。下面是这些常量的具体含义。
    

类型 | 标志位(tag) | 描述  
---|---|---
CONSTANT_Utf8_info | 1 | UTF-8编码的字符串
CONSTANT_Integer_info | 3 | 整形字面量
CONSTANT_Float_info | 4 | 浮点型字面量
CONSTANT_Integer_info | 5 | UTF-8编码的字符串
CONSTANT_Double_info | 6| 双精度字面量
CONSTANT_Class_info  |  7| 类或接口的符号引用
CONSTANT_String_info |   8|字符串类型的字面量
CONSTANT_Fieldref_info | 9 |字段的符号引用
CONSTANT_Methodref_info | 10|类中方法的符号引用
CONSTANT_InterfaceMethodref_info| 11 |接口中方法的符号引用
CONSTANT_NameAndType_info| 12|字段和方法的名称以及类型的符号引用
CONSTANT_MethodHandle_info| 15|表示方法句柄
CONSTANT_MethodType_info| 16|标识方法类型
CONSTANT_InvokeDynamic_info| 18|表示一个动态方法调用点


    接下来看00000000AH这个偏移地址的值：0x0A ，转换成二进制为：10。 这个就是第1个常量的第1个tag。根据上表tag为10的常量类型是CONSTANT_Methodref_info。
    CONSTANT_Methodref_info的结构如下：
```
CONSTANT_Methodref_info{
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```
    class_index指向常量池中类型为CONSTANT_Class_info的常量。tag值后面跟着的就是class_index的值，即地址0000000BH~0000000CH单元的值：0x0004，
    十进制也是4，意思是指向第4个常量。
    接下来name_and_type_index指向常量池中类型为CONSTANT_NameAndType_info的常量，地址0000000DH ~ 0000000EH 的值为0x0012 即是十进制的 18，
    意思是指向第18个常量。
    
    到此，分析完了第1个常量。那么接下来的就是第2个常量。
    
    第2个常量的第1个字节是 0000000FH,值是 0x09 ，十进制也是09。每一个常量的第一个字节都是tag标志位。查看常量类型表发现tag为9的常量类型是
    CONSTANT_Fieldref_info。
    CONSTANT_Fieldref_inf的结构如下：
```
CONSTANT_Fieldref_info{
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```
    class_index与name_and_type_index和之前的一样。
    class_index的值保存在 00000010H ~ 00000011H中，值为0x0003，意思是指向第3个常量。
    name_and_type_index的值保存在 00000012H ~ 00000013H中，值为0x0013，十进制为：19，意思是指向第19个常量。
    
    接下来看第3个常量。根据第2个常量的class_index表明，第3个常量的类型是CONSTANT_Class_info。根据class文件，第3个常量的第一个字节存的是tag值，
    即地址 00000014H 的值是0x07，十进制也是07。根据常量类型表查找tag值7的常量类型是CONSTANT_Class_info。和第二个常量的class_index的指向一致。
    CONSTANT_Class_info类型常量的结构如下:
```
CONSTANT_Class_info{
    u1 tag;
    u2 name_index;
}
```
    紧接tag之后的是name_index,即地址00000015H ~ 00000016H的值0x0014，十进制为：20。意思是指向第20个常量。接下来的常量也可以按照这个方法去分析。
    我们也可以通过jdk提的工具javap来查看所有常量池中的常量。使用javap -verbose class文件地址 命令就可以输出class文件的字节码内容。如下图：
    
![image](https://note.youdao.com/yws/public/resource/9c5114cb9308a9adbf988904061254c4/xmlnote/4FF252E5B6AA474F8967B5937642C083/3430)
    
    通过查看工具输出的结果可以看到分析是对的。 第1个常量指向第4和第18个常量，第4个常量又指向第21个常量，第21个常量的值为“java/lang/Object”;
    第18关常量又指向第7和第8个常量，值为"<init>  ()V" ; 
    
    按照这个方式，就可以分析完剩下的常量了。常量一直到地址000000C2。
    
#### 6 ) u2 access_flags 常量池结束后，接下来是2个字节的访问标志。

2个字节长的访问标志一共有16位，目前只有8位是被定义了，还有8位未定义，留着使用。访问标志用于标志出这个class是类还是接口，是否是abstract，是否fina等。这16位的具体定义如下：


    

    
![image](https://note.youdao.com/yws/public/resource/0e91087b1c0fef4ca83554e26c9572c1/xmlnote/1BE7B5DD8A0541C7A52863B8EB15E37B/4190)
    
    
下表是对应值的说明：
    
标志名称　|	标志值	| 含义
---|---|---
ACC_PUBLIC |	0x00 01 |	是否为Public类型 
ACC_FINAL |		0x00 10 |		是否被声明为final，只有类可以设置
ACC_SUPER |		0x00 20 |		是否允许使用invokespecial字节码指令的新语义．
ACC_INTERFACE |		0x02 00 |		标志这是一个接口
ACC_ABSTRACT |		0x04 00 |		是否为abstract类型，对于接口或者抽象类来说，次标志值为真，其他类型为假
ACC_SYNTHETIC |		0x10 00	 |	标志这个类并非由用户代码产生
ACC_ANNOTATION |		0x20 00	 |	标志这是一个注解
ACC_ENUM |	0x40 00 |		标志这是一个枚举


    将访问标志位进行或运算，就可以得到访问标志的值了。这个class是public类来的，并且是JDK1.2之后编译的。所以它的访问标志的值为：0x0020 | 0x0001 = 0x0021。 
    或者我们可以直接根据上面的标志示意图来推算访问标志位的值： 这个class的第11位和16位为真，其余位为假，对应的二进制数为 00000000 00100001，转换为十六进制为0x0021

    地址000000C3H ~ 000000C4H的值是0x0021，和之前的推算一致。
    
    
更加具体的标志位的解析[请点击这里查看](https://blog.csdn.net/luanlouis/article/details/41039269)
    
#### 7 ) u2 this_class 类索引，用于确定这个类的全限定名
    
    访问标志位后的2个字节是类索引，即地址：000000C5H ~ 000000C6H 。值为0x0003 ，对应的十进制值是3。表示指向第3个常量。通过上面的常量分析可知，第3个常量指向
    第20个常量。第20个常量的值为“demo/Test”。和源码所写的一致，这个类的名称是Test，放在demo包下面。
    
    
#### 8 ) u2 super_class 父类索引，用于确定这个类的父类的全限定名

    紧接下来的2个字节是父类索引，即地址：000000C7H ~ 000000C8H 。值为0x0004 ，对应的十进制值是4。表示指向第4个常量。通过上面的常量分析可知，第4个常量指向
    第21个常量。第21个常量的值为“java/lang/Object”。
    
    
#### 9 ) u2 interface_count 接口计数器，表示这个class实现的接口
    
    接下来的2个字节是接口的计数器。如果没有实现任何接口，则计数器的值为0。地址000000C9H ~ 000000CAH的值为0。这个class没有实现任何接口。

#### 10 ) u2 interfaces[interfaces_count 接口索引表
    接口计数器后面的是接口索引表，每一个接口索引占有两个字节。接口索引和类索引和父类索引一样，其内的值存储的是指向了常量池中的常量池项的索引，
    表示着这个接口的完全限定名。如果接口计数器的值为0，则接口索引表不占用任何字节。这里，接口索引表不占用任何字节。
    

#### 11 ) u2 fields_count 实例变量和类变量的数量
    接下来的2个字节表示的是实例变量和类变量(static变量)的数量。地址 000000CBH ~ 000000CCH 的值为 0x0001，转换为十进制值为1，表示这个class有1个实例变量或类变量。不包含父类继承过来的字段。
    

#### 12 ) field_info 字段表
    字段数量后接着的是字段表。字段表的结构如下：

  
  ```
1    field_info{
2        u2     access_flags;   //访问标志
3        u2     name_index;     //名称索引
4        u2     descriptor;     //描述索引
5        u2     attributes_count;//属性数量
6        attrbutes_info     attributes[attributes_count]; //属性表
7    }
  ```
  
  对于一个字段的描述和字段接口的关系如下图所示：
  ![image](https://note.youdao.com/yws/public/resource/e3e299d368a9c2c3b9e825b2b70b5542/xmlnote/79F279D208E641029F1F2E24476CB4B5/4361)
  
  

    首先看字段的第一个信息，2个字节长度的访问标志。这个访问标志和类的访问标志相似，用于标明这个字段是否是public的，是否是final的，是否static的等。
    访问标志的具体每一位的含义如下图所示：

![image](https://note.youdao.com/yws/public/resource/e3e299d368a9c2c3b9e825b2b70b5542/xmlnote/9E3BC5FCD2064A7990F63C01E61CBD9D/4370)


    根据地址000000CDH ~ 000000CEH的值0x0002分析。二进制值为10。那就是表明标志位的第15位为真以外，其他位都为0。第15位为真，表示该字段被privite修饰符修饰。
    根据原码来看，第一个字段“i”也是只有被private修饰符修饰。
    
    接下来的是2个字节长度的的name_index。存放在地址000000CFH ~ 000000D0H 中。值为0x0005 转换为十进制值也是5。表明改字段的名称存放在第5个常量中。
    第5个常量的值是“i”。和源码一致。
    
    接下来的是2个字节长度的描述符索引。也就是字段的类型索引。根据描述符规则，基本数据类型(byte、char、double、float、int、long、short、boolean)都用
    一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示。对于数组类型，每一个维度用一个前置的“[”来表示。
    具体每个类型的描述符对应如下图所示。
    
![image](https://note.youdao.com/yws/public/resource/e3e299d368a9c2c3b9e825b2b70b5542/xmlnote/F29FF943B6E541F9BD5558AA65E6C26B/4406)
    
    例如二维数组int[][]  Class文件中的类型表示则为 [[I


    接下来的2个字节的地址 000000D1H ~ 000000D2H 的值为0x0006，转换为十进制也是6。指向常量池中的第6个常量，第6个常量的值为"I"，表明这个字段是int类型。和源码一致。
    
    
    后面的2个字节表示附加信息，其他属性的数量。地址 000000D3H ~ 000000D4H 的值为0，表示没有附加信息。



#### 13 ) u2 methods_count  方法表的数量
    地址 000000D3H ~ 000000D4H 这两个字节表示方法表的数量，值为0x0002。转换为十进制也是2。表示有2个方法:编译器添加的实例构造器<init>和源码定义的方法test()
    
#### 14 ) method_info   methods[methods_count] 方法表
    接下来的是方法表，方法表的结构和字段表的结构差不多。方法表不包含父类的方法。方法表的具体的结构如下：
    
![image](https://note.youdao.com/yws/public/resource/e3e299d368a9c2c3b9e825b2b70b5542/xmlnote/92F7DE615E7C43CC98170DB9F7317A50/4448)


接下来分析第一项，2个字节长的访问标志。方法的访问标志和字段的访问标志很相似，但是也存在一些不同。具体的访问标志位如下所示：

![image](https://note.youdao.com/yws/public/resource/e3e299d368a9c2c3b9e825b2b70b5542/xmlnote/6DE06C67746745D4A7D0A35D39F5C13D/4454)

    地址000000D7H ~ 000000D8H 存的是第1个方法的访问标志。值是0x0001，转换为二进制也是1。表明这个方法是public的。接下来的2个字节存放的是方法名称的索引。
    地址000000D9H ~ 000000DAH 的值是0x0007。转换为十进制也是7 。表明指向第7个常量， 第7个常量的值是“<init>”。这表明第一个方法的名称是"<init>"，
    这个方法是编译器自动生成的构造方法。
    
    接下来的是2个字节长度的方法描述索引值，这个索引值也是指向常量池的常量。地址为000000DBH ~ 000000DCH，值为0x0008，转换为十进制值也是8。
    方法描述符记录的是方法的参数类型和返回值，形式为“(方法参数数据类型描述列表)+返回值数据类型描述符”。据称他的数据类型对应的描述符，在上面已经介绍过了。
    现在看第8个常量对应的值是 “()V” ，意思是方法没有参数，返回值为void。


#### 15 ) u2 attributes_count   属性表数量

    接下来的2个字节长度的是属性表个数，存放在地址000000DDH ~ 000000DEH 中，值为0x0001，转换为10进制也值为1。表明这个方法有1个属性表。
    属性表记录了某个方法的一些属性信息，这些信息包括：
    1) 这个方法的代码实现，即方法的可执行的机器指令
    2) 这个方法声明的要抛出的异常信息
    3) 这个方法是否被@deprecated注解表示
    4) 这个方法是否是编译器自动生成的

    对于属性表，不同的属性表结构是不一样的，但前面2个字段都是一样的，缩略结构如下：
```
1    attribute_info_table {
2        u2	    attribute_name_index	    //名称索引
3        u4	    attribute_length            //属性长度
4        ···
5        ···
5        ···
5    }
```




#### 16 ) attribute_info    attributes[attributes_count]    属性表
    


    清楚了结构，继续分析，那么接着属性表长度后面的是第一个属性表的名称索引，长度为2个字节，地址是 000000DFH ~ 000000E0H。 值为0x0009，转换为十进制为9。
    第9个常量的值为“code”。说明这个属性名称是code。
    
    接下来的4个字节长度是属性的长度，地址为000000E1F ~ 000000E4F ，值为0x0000002F，转换为十进制，值为47。表明，code这个属性有47个字节长度。
    这个长度不包括attribute_name_index和attribute_length的6个字节的长度。
    
    
    code属性的结构如下：
    
```
1    Code_attribute {
2        u2	    attribute_name_index;	    //名称索引
3        u4	    attribute_length;           //属性长度
4        u2     max_stack;                  //最大栈深度
5        u2     max_locals;                 //局部变量表所需要的存储空间
6        u4     code_length;                //字节码指令的数量
7        u1     code[code_length];          //字节码指令
8        u2     exception_table_length;     //异常表长度
9        {
10          u2 start_pc;
11          u2 end_pc;
12          u2 handler_pc;
13          u2 catch_type;
14       } exception_table[exception_table_length];//异常表
15       u2 attributes_count;               //属性表数量
16       attribute_info attributes[attributes_count];//属性表
17    }
```

    名称索引和属性长度已经分析了。接下来的是2个字节长度的最大栈深度。地址为00000E5H ~ 000000E6H。值为0x0001，转换为十进制也是1。
    虚拟机在运行时根据这个值来分配栈帧中操作栈深度。具体的意思，需要了解方法调用栈帧结构。
    
    接下来的是局部变量所需要的存储空间。长度为2个字节。地址为000000E7H ~ 000000E8H 。值为0x0001，转换为十进制也是1。max_locals的单位是Slot。
    Slot是虚拟机为局部变量分配内存的最小单元，在运行时，对于不超过32位类型的数据类型，比如 byte,char,int等占用1个slot，而double和Long这种64位的数据类型则需要
    分配2个slot，另外max_locals的值并不是所有局部变量所需要的内存数量之和，因为slot是可以重用的，当局部变量超过了它的作用域以后，局部变量所占用的slot就会被重用。
    
    接下来带是4个字节长度的字节码指令数量。地址为000000E9H ~ 000000ECH 。值为0x005，转换为十进制也是5。表示这个方法的字节码指令占了5个字节。
    
    接下来的5个字节，表示的是5个字节码指令。1个字节对应1个指令。一个字节取值范围是0 ~ 255。 目前虚拟机规范已经定义了200多条指令。具体的指令含可以查阅“字节码指令表”。
    当前看到的3个指令为： 2A(aload_0),B7(invokespecial),0001(invokespecial的参数),B1(return)。具体的指令分析，不在本文。
    
    
    接下来的2个字节是异常表的数量，地址是000000F2H ~ 000000F3H 。值为0x0000。这个方法没有异常表。
    
    所以接下来的就是属性表数量，长度为2个字节地址是000000F4H ~ 000000F5H 。值为0x0002,转换为十进制也是2。表明这个方法有2个属性表。第一个属性表的
    名称变量索引为2个字节长度。地址是000000F6H ~ 000000F7H 。值为0x000A，转换为十进制是10。第10个常量的值是“LineNumberTable”。后面接着的4个字节就是
    长度，地址是000000F8H ~ 000000FBH 。值为0x00000006，转换为十进制也是6，表示本属性表为6个字节长度。
  
    
LineNumberTable属性表的结构如下：
```
1 LineNumberTable_attribute{
2       u2 attribute_name_index;
3       u4 attributelength;
4       u2 line_number_table_length;
5       {
6           u2 start_pc;        //字节码行号
7           u2 line_number;     //源码行号
8       } line_number_table[line_number_table_length]
9}
```

    LineNumberTable用于描述java源代码的行号和字节码行号的对应关系，它不是运行时必需的属性，如果通 过-g:none的编译器参数来取消生成这项信息的话，
    最大的影响就是异常发生的时候，堆栈中不能显示出出错的行号，调试的时候也不能按照源代码来设置断点。
    
    接下来的是2个字节长度的行号属性表长度，地址是000000FBH ~ 000000FCH。 值为0x0001,转换为十进制也是1。表明有1个line_number_table结构。
    接下来的是2个字节长的字节码行号,地址是 000000FEH ~ 000000FFH ，值是0x0000。表示字节码行号为0。地址00000100H ~ 00000101H ，2个字节长度。值为0x0003,
    转换为十进制也是3。表示对应的源码行号是3。

    第1个属性表分析完了，紧接的是第2个属性表。地址00000102H ~ 00000103H，值为0x000B。转换成10进制是11。第11个常量的值是“LocalVariableTable”。
    接下来的4个字节存储的是这个属性表的长度。地址00000104H ~ 00000107H。值为0x0000000c。转换为十进制为12。表示这个属性表的长度是12个字节。
    
LocalVariableTable属性表的结构如下。
```
1 LocalVariableTable_attribute{
2       u2 attribute_name_index;
3       u4 attributelength;
4       u2 local_variable_table_length; //本地变量表长度
5       {
6           u2 start_pc;        
7           u2 length; 
8           u2 name_index;
9           u2 descriptor_index;
10          u2 index;
8       } local_variable_table[local_variable_table_length]
9}
```
    
    LocalVariableTable属性用于描述战阵中局部变量表中的变量与Java源码中定义的变量之间的的关系，它也不是运行时必须的属性，但默认会生成到Class文件中。
    可以在Javac中分别使用-g:none 或 -g:vasr选项来取消或要求生成这项信息。如果没有生成这两属性，最大的影响就是当其他人引用这个方法时，所有的参数名称都会丢失，IDE将会使用arg0，arg1这样的占位符替代原有的参数名。在调试期间无法根据参数名称从上下文中获得参数值。
    
    属性名称和长度已经分析了，接下来的是本地变量表长度。地址是00000108H ~ 00000109H 值为0x0001，转换为十进制也是1。表明这个属性有1个local_variable_table。
    
    接下来的是local_variable_table结构的第1个字段 start_pc ，长度为2个字节，地址是0000010AH ~ 0000010BH ，值为0x00。表明这个局部变量的生命周期开始的字节码偏移量
    是0.
    
    接下来的2个字节长度的是这个局部变量的作用范围覆盖的长度，存在地址 0000010CH ~ 0000010DH中。值是0x0005,转换为十进制也是5。表明作用范围长度是5。
    这两个字段结合起来就是这个局部变量在字节码之中的作用范围。
    
    接下来的2个字节保存的是变量的名称的索引。保存在地址0000010EH ~0000010FH中，值是0x000C,转换为十进制是12。常量池中的第12个常量的值是"this"。
    接下来的2个字节保存的是变量的描述符索引。保存在地址00000110H ~00000111H中，值是0x000D,转换为十进制是13。常量池中的第13个常量的值是"LDemo/Test"。
    表明这个变量的类型是对象类型，类型是Demo.Test类型。
    
    接下来的2个字节保存的是index。index是这个局部变量在栈帧局部变量表中Slot的位置，当这个变量数据类型是64位类型是(double 和 long)，
    它占用的Slot为index和index+1两个。index保存在地址00000112H ~ 00000113H中。值是0x0000。
    
    到此，第一个方法已经分析完了。紧接着的就是第2个方法的访问标志。长度为2个字节，保存在00000114H ~ 00000115H中，值为0x001。所以这个方法是public的。
    接下来的是方法的名称索引。值为0x000E，转换为十进制是14。第14个常量的值是“test”。这个方法就是源码写的test()方法。可以按照上面的分析方法来分析这个方法。
    
    这个方法的整个地址范围是 00000114H ~ 00000152H 。
    
    最后就剩下Class文件的属性表了。2个字节长度的属性表长度，地址是00000153H ~ 00000154H.值是0x0001，转换为十进制也是1,表明这个类有1个属性表。
    接下来的2个字节是属性表的名称索引，00000155H ~ 00000156H.值是0x0010，转换为十进制是16。第16个常量的值是"SourceFile".

SourceFile属性的结构如下。

```
1 SourceFile_attribute{
2       u2 attribute_name_index;
3       u4 attributelength;
4       u2 sourcefile_index;
5}
```
 
    那么这4个字节保存的是属性的长度 00000157H ~ 0000015AH。 值为0x02，转换为十进制也是2，表明这个属性长2个字节。
    那么接下来的2个字节是sourcefile_index，保存的是源码文件的文件名。 0000015BH ~ 0000015CH.值是0x0011,转换为十进制是17.第17个常量的值是“Test.java”.
    这个就是源码的文件名称。
    
    可以看到属性是很灵活的，可以放在字段，方法，类，甚至是属性里面。
    
    
    到此为止，整个class文件就分析完毕了。
    

https://blog.csdn.net/luanlouis/article/details/41046443








