1. 引用
   1. 引用的本质其实就是编译器自动给我们添加了一个指针常量
   2. 示例
      ```c++
      // 发现是引用，转换为 int* const ref = &a
      void test(int& ref) {
        ref = 100;      // ref是引用，转换为*ref = 100
      }
      
      int main() {
        int a = 10;
        int& ref = a;  // 自动转换为 int* const ref = &a  因为是常量所以引用必须初始化
        ref = 20;      // 编译器发现ref是引用，自动帮我们转换为：*ref = 20
        cout << "a:" << a << endl;
        cout << "ref:" << ref << endl;
        test(a);
        return EXIT_SUCCESS;
      }
      ```
      1. 为什么会有引用
      ``` C++
      void getMemory(PStudent* student) {
         *student = (PStudent)malloc(sizeof(Student));
         (*student)->age = 50;
         strcpy((*student)->name, "胡志明"));
      }
      // 上面的例子*太多，可以使用下面的函数代替
      void getMemory(PStudent& student) {
         student = (PStudent)malloc(sizeof(Student));
         student->age = 30;
         strcpy(student->name, "阮玲玉");
      }
      
      int main() {
         PStudent student = NULL;
      
         getMemory(&student);
         cout << student->name << student->age << endl;

         getMemory(student);
         cout << student->name << student->age << endl;
         return 0;
      }
      ```
   3. 常量的引用
      1. const int &ref = 100;
      2. 示例
      ```c++
      int &ref=100;          // 错误，不能直接给常量定义引用
      const int &ref = 100;  // 正确
      // 加了const之后等价于： int temp = 100; const int &ref = temp;
      // 虽然有const修饰，但是ref的值依然可以被修改
      int *p = (int*) &ref;
      *p = 200; // 可以修改ref的值
      ```
      3. 常量引用的使用场景
         1. 防止误操作去修改常量的值
      
2. 拷贝构造函数
   1. 用一个对象值创建另一个对象的方法---拷贝构造函数
   2. 例
   ```C++
   <类名>::<构造函数>(<类名>  &<引用名>) {
   
   }
   
   CTest::CTest(CTest &test) {
   
   }
   ```
   3. 如果一个类中没有定义拷贝构造函数，则系统自动生成一个默认拷贝构造函数，其功能只是简单的把形参的对应属性赋值给新对象。
   4. 使用场景：使用一个已存在的对象初始化另一个对象
   5. 例
   ```C++
   Student stu1;
   stu1.age = 10;
   stu1.sex = 'm';
   printf("stu1.age = %d\tstu1.sex=%c\n", stu1.age, stu1.sex);

   Student stu2(stu1);
   printf("stu2.age = %d\tstu2.sex=%c\n", stu2.age, stu2.sex);
   ```
3. 静态 static
   1. c语言中static的作用
      1. 作用域：当前文件
      2. 生存周期：整个程序开始到结束
      3. 静态的作用：static的静态变量，在进入函数的时候，还保留着上一次离开函数时候的值，因为他的生存周期是整个程序
      4. 隐藏作用：static用来修饰全局变量成员函数，该变量或者函数只能在本文件使用
   2. c++静态
      1. 静态成员
      2. 静态成员函数
4. 虚函数内存模型
   1. 不存在继承关系的虚函数内存模型
   2. 只要一个类中存在有虚函数，对象内就会产生一个虚表指针，指向虚函数表。虚函数表里存的是当前类内所有虚函数。
   3. 虚函数表只存在一个，保存在常量区
   4. 实例化出的对象的虚表指针指向的是同一个表
5. 进程间的通信
   1. COPY_DATA方式
      1. WM_COPYDATA是一种特殊的，专门用于传递数据的消息，这个消息可以携带一个大体积的消息参数，不同于其他只能携带两个固定参数的消息，
      2. 在发送WM_COPYDATA消息时，WPARAM应该保存有发送此消息的窗口句柄，LPARAM应该指向一个名为COPYDATASTURCT的结构体
      ``` c++
      typedef struct tagCOPYDATASTRUCT {
         ULONG_PIR dwData;
         DWORD     cbData;
         _Field_size_bytes_ (cbData) PVOID lpData;
      } COPYDATASTRUCT, *PCOPYDATASTRUCT;
      
      HWND hWnd = FindWindow(NULL, L"06.COPYDATA接收端");
      COPYDATASTRUCT cs;
      cs.dwData; // 发送过去一个随意的数据
      cs.cbData = 24; // 缓冲区大小
      cs.lpData = (LPVOID) L"hello world"; // 发送数据
      SendMesssage(hWnd, WM_COPYDATA, NULL, (LPARAM) &cs);
      ```
   2. 邮槽的方式
      1. 是windows系统中最简单的一种进程间的通信方式，一个进程可以创建一个又想，其他进程可以通过打开此邮箱与进程通信。
      2. 油槽的通讯方式是单向的，只邮服务器才能从油槽中读取消息，客户端只能写入，消息被写入后以队列的方式保存。
      3. 油槽除了可以在本机内进行通讯外，还可以在主机之间进行通讯（使用udp协议），想要通过网络间进行通讯必须知道服务器的主机名和域名
      ```aidl
      
      ```
