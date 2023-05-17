1. 什么是可执行文件
   * windows：PE (Portable Executalble) 文件结构  
   * ELF(Executable and Linking Format) 文件结构
2. 如何识别pe文件
    1. Pe文件特征：分别打开exe dll sys 等文件，观察特征前2个字节 
       1. MZ开头 
       2. 查找3c位置
       2. 找到3c位置的地址，如果是pe, 则是pe文件。
    2. 不要仅仅通过文件的后缀来判断。
3. Pe的主要结构

   |结构体|大小(十进制)|
   |---|---|
   |IMAGE_DOS_HEADER|64字节|
   |DOS STUB|不确定|
   |IMAGE_FILE_HEADER|20字节|
   |IMAGE_OPTIONAL_HEADER|224字节，不固定|
   |IMAGE_SECTION_HEADER|每个结构体成员40个字节
   1. dos部分 
      1. IMAGE_DOS_HEADER, dos mz文件头
   
    ```c++
     // DOS EXE header 64个字节
     typedef struct _IMAGE_DOS_HEADER {
         WORD e_magic;   // Magic number，2个字节
                     ... ...               
         LONG e_lfanew;  // File address of new exe header，4个字节，指向了pe开头，这其实就是3c位置
     }             
     // 此结构体是给16位程序使用，只有上面2个字段有用
     实验，用ultraEidt打开一个exe文件，删掉中间内容，看看程序能否正常运行。 
   ```
      2. Dos Stub, dos块，IMAGE_DOS_HEADER后到e_lfanew指向的pe头中间的部分，这部分是由连接器添加，所以可以随意修改。
      实验：修改dos块的内容，看程序是否能够正常打开。

   2. pe文件头
      1. IMAGE_NT_HEADERS, pe头
      ```c++                               
      // pe头结构体
      typedef struct IMAGE_NT_HEADERS {
          DWORD Signature;                         // pe标识，操作系统在启动一个程序的时候会检测这个标识，4个字节
          IMAGE_FILE_HEADER FileHeader;            // 标准pe头，20个字节
          IMAGE_OPTIONAL_HEADER32 OptionalHeader;  // 扩展pe头，默认224个字节，标注pe头种的sizeofoptionalheader定义
      } IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;  
      
      // 标准pe头结构体
      typedef struct IMAGE_FILE_HEADER {   
          WORD  Machine;                // 可以运行在什么样的cpu，任意：0，intel386及后续：14c， x64：8664
          WORD  NumberOfSections;       // 节的数量
          DWORD TimeDateStamp;          // 时间戳，编译器填写，与文件属性里面（创建时间、修改时间）无关
          DWORD PointerToSymbolTable;   // 调试相关
          DWORD NumberOfSymbol;         // 调试相关 
          WORD  SizeOfOptionalHeader;   // 扩展pe头的大小，32位pe：0xE0 64位pe：0xF0
          WORD  Characteristics;        // 文件属性
      } IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER
      ```
      重要属性：
      * <font color=red>NumberOfSections 节数量</font>
      * <font color=red>SizeOfOptionalHeader 扩展pe大小</font>
      
       Characteristics 16位 每位的定义如下：

       | 数据位 | 常量符号 |为1时的含义|
       |---|---|---|
       | 0   | IMAGE_FILE_RELOCS_STRIPPED         |文件中不存在重定位信息|
       | 1   | IMAGE_FILE_EXECUTABLE_IMAGE        |文件是可执行文件|
       | 2   | IMAGE_FILE_LINE_NUMS_STRIPPED      |不存在行信息| 
       | 3   | IMAGE_FILE_LOCAL_SYMS_STRIPPED     |不存在符号信息
       | 4   | IMAGE_FILE_AGGRESSIVE_WS_TRIM      |调整工作集
       | 5   | IMAGE_FILE_LARGE_ADDRESS_AWARE     |应用程序可处理大于2gb的地址
       | 6   |                                    |保留位
       | 7   | IMAGE_FILE_BYTES_REVERSED_LO       |小尾方式 
       | 8   | IMAGE_FILE_32BIT_MACHINE           |只在32位平台上运行
       | 9   | IMAGE_FILE_DEBUG_STRIPPED          |不包含调试信息    
       | 10  | IMAGE_FILE_REMOVABLE_RUN_FROM_SWAP |不能从可移动盘运行
       | 11  | IMAGE_FILE_NET_RUN_FROM_SWAP       |不能从网络运行 
       | 12  | IMAGE_FILE_SYSTEM                  |系统文件(如驱动程序)，不能直接运行 
       | 13  | IMAGE_FILE_DLL                     |这是一个dll文件 
       | 14  | IMAGE_FILE_UP_SYSTEM_ONLY          |文件不能在多处理器计算机上运行
       | 15  | IMAGE_FILE_REMOVABLE_RUN_FROM_SWAP |不能从网络运行 
       | 16  | IMAGE_FILE_BYTES_REVERSED_HI       |大尾方式 
       ``` c++ 
       // 扩展pe头
      type struct _IMAGE_OPTIONAL_HEADER {
           WORD Magic;                 // pe32:10B pe32+:20B
           BYTE MajorLinkerVersion     // 连接器主版本号
           BYTE MinorLinkerVersion     // 连接器次版本号
           DWORD SizeOfCode            // 所有代码节的总和， 文件对齐后大小
           DWORD SizeOfInitializedData;// 包含所有已经初始化数据的节的总大小，文件对齐后的大小，编译器填写，没用
           DWORD SizeOfUninitializedData; 包含所有为初始化数据的节的总大小，文件对齐后的大小，编译器填写，没用
           DWORD AddressOfEntryPoint;  // 程序入口
           DWORD BaseOfCode;           // 代码开始的基址，编译器填写，没用
           DWORD BaseOfData;           // 数据开始的基址，编译器填写，用用
           DWORD ImageBase;            // 内存镜像基址
           DWORD SectionAlignment;     // 内存对齐
           DWORD FileAglignment;        // 文件对齐
           WORD MajorOperationSystemVersion;  // 操作系统主版本号
           WORD MinorOperationSystemVersion;  // 操作系统次版本号
           WORD MajorImageVersion;     // pe主版本号
           WORD MinorImageVersion;     // pe次版本号
           WORD MajorSubsystemVersion; // 运行所需的主子系统版本号  
           WORD MinorSubsystemVersion; // 运行所需的次子系统版本号
           DWORD Win32VersionValue;    //子系统版本值，必须为0
           DWORD SizeOfImage;          // 内存中整个pe文件的映射尺寸，可比实际值大，必须是SectionAlignment的倍数
           DWORD SizeOfHeaders;        // 所有头-节表按照文件对齐后的大小，否则加载会出错 
           DWORD CheckSum;             // 校验和，一些系统文件有要求，用来判断文件是否被修改
           
           // CheckSum如何产生的？ 文件中每2个字节相加的结果 + 文件的长度 = 校验和 ，没有任何意义，修改程序后可以一起修改这个值 
           WORD Subsystem              // 子系统 驱动程序1，图形界面2，控制台、dll3
           WORD DllCharacteristics;    // 文件特性，不是针对dll文件的，不要被名字迷惑
           DWORD SizeOfStackReserve;   // 初始化时保留的栈大小
           DWORD SizeOfStackCommit;    // 初始化时实际提交大小
           DWORD SizeOfHeapReserve;    // 初始化时保留的堆大小
           DWORD SizeOfHeapCommit;     // 初始化时实际提交大小
           DWORD LoaderFlags;          // 调试相关
           DWORD NumberOfRvaAndSizes;  // 目录项目数
           IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]; // 16 * 8 = 128
      }
   
      ```     
      重要属性：
       * <font color=red>AddressOfEntryPoint 程序入口地址</font>
       * <font color=red>ImageBase pe文件在内存中展开的起始位置</font>
       * <font color=red>SectionAlignment 内存对齐</font>
       * <font color=red>FileAglignment 文件对齐</font>
      
   3. pe文件的分节结构 
      * 节的数量：标准pe头中NumberOfSections属性
      实验：找到pe节表的位置。
      扩展pe头是可以改的，要确认扩展pe头有没有被修改，可以查看标准pe头中的SizeOfOptionalHeader，
      节表是结构体数组，结构体定义如下：
      ```c++
      typedef struct _IMAGE_SECTION_HEADER {
          BYTE Name[IMAGE_SIEOF_SHORT_NAME] // 8个字节，ASCII字符串，可自定义，只截取8个
          union {                           // Misc双字，是该节在没有对齐前的真实尺寸，该值可以不准确
            DWORD PhysicalAddress;
            DWORD VirtualSize;              // 在内存中实际大小
          } Misc                            // 4个字节
          DWORD VirtualAddress;             // 在内存中的偏移地址，加上ImageBase才是在内存中的真正地址， 内存中的地址
          DWORD SizeOfRawData;              // 节在文件中对齐后的尺寸， 文件中的大小
          DWORD PointerToRawData;           // 节区在文件中的偏移， 文件中的地址
          DWORD PointerToRelocations;       // 调试相关
          DWORD PointerToLinenumbers;
          WORD NumberOfRelocations;
          WORD NumberOfLinenumbers;
          DWORD Characteristics;            // 节属性
      } IMAGE_SECTION_HEADER *PIMAGE_SECTION_HEADER
      ```   
    Characteristics 32位 每位的定义如下：

     | 数据位 | 常量符号                                       | 为1时的含义                   |                                     
     |--------------------------------------------|--------------------------|---|                                            
     | 5   | IMAGE_SCN_CNT_CODE或00000020h               | 节中包含代码                   | 
     | 6   | IMAGE_SCN_CNT_INITIALIZED_DATA或00000040h   | 节中包含已初始化数据               | 
     | 7   | IMAGE_SCN_CNT_UNINITIALIZED_DATA或00000080h | 节中包含未初始化数据               | 
     | 8   | IMAGE_SCN_LNK_OTHER或00000100h              | 保留供未来使用                  | 
     | 25   | IMAGE_SCN_MEM_DISCARDABLE或02000000h        | 节中的数据在进程开始以后将被丢弃，如.reloc | 
     | 26   | IMAGE_SCN_MEM_NOT_CACHED或04000000h         | 节中的数据不会经过缓存              | 
     | 27   | IMAGE_SCN_MEM_NOT_PAGED或08000000h          | 节中包含代码                   | 
     | 28   | IMAGE_SCN_CNT_CODE或00000020h               | 节中包含代码                   | 
     | 29   | IMAGE_SCN_CNT_CODE或00000020h               | 节中包含代码                   | 
     | 30   | IMAGE_SCN_CNT_CODE或00000020h               | 节中包含代码                   | 
     | 31   | IMAGE_SCN_CNT_CODE或00000020h               | 节中包含代码                   | 
      实验：找一个程序的真实入口点。
      * 先找到dos头 
      * 找到pe头，找到pe标准头 
      * 找到pe扩展头，找到AddressOfEntryPoint和ImageBase，两者相加得到真实程序入口点
      * 使用od工具验证
 
4. RVA与FOA转换
   * 全局变量，如果有初始值则存储在pe中，如果没有初始值只有当文件加载到内存中的时候才会分配空间。
   * RVA: Relative Virtual Address 相对虚拟地址
   * FOA: File Offset Address 文件偏移地址
   * RVA与FOA转换：
     1. 得到RVA的值：内存地址 - ImageBase
     2. 判断RVA是否位于pe头中，如果是：foa == rva
     3. 判断rva位于哪个节：
        * Rva >= 节.VirtualAddress
        * RVA <= 节.VirtualAddress + 当前节内存对齐后的大小
        差值 = rva - 节.virtualAddress；
     4. FOA = 节.PointerToRawData + 差值     
   实验：
     * 修改一个可执行程序的全局变量的值。
       1. 使用vc++6.0写一个带有全局变量的程序，编译可执行程序。
       2. 使用ultraEdit修改可执行程序中全部变量的值。
     * 将对话框注入到可执行程序中。
       1. 构造要注入的代码
          1. 使用vc++6.0写一个对话框程序 
          ```c++
          MessageBox(0, 0, 0, 0);
          ```
          2. 打断点，f5进入debug，alt+8显示反汇编窗口，alt+6显示内存地址。可以查看到程序硬编码。
          
          6A(PUSH) 00 6A 00 6A 00 6A 00(4个参数 0) E8(CALL) 跳转地址 E9(JMP) 跳转地址
       
          <font color=red>跳转地址 = 要跳转的地址-e8指令当前地址（ImageBase + e8地址）-5</font>
          
       2. 在pe空白区构造一段代码
          1. pe头和段之间有大量空白区域
          2. 找到messagebox函数的地址，使用od，按顶部e按钮，找到user32.dll双击，ctrl+n进入函数列表，在这里可以找到MessageBoxA函数，这里可以看到地址
          3. 找到AddressOfEntryPoint和ImageBase
       3. 修改入口地址为新增代码
         1. 修改AddressOfEntryPoint地址为我们的代码地址
       4. 新增代码执行后，跳回入口地址
         1. E9即为跳转
5. 扩大节     
   1. 分配一块新的空间，大小为s
   2. 将最后一个节的SizeOfRawData和VirtualSize改成N，这两个值看谁大选谁
   
      N = (SizeOfRawData（文件大小）或者VirtualSize（内存大小）内存对齐后的值) + s
   3. 修改SizeOfImage（内存中）大小，扩展pe头中数56个字节，第57个字节， 
   SizeOfImage = SizeOfImage + 内存中扩大的值
   4. 注意此节有没有可执行的属性，如果没有则要修改节表结构的Characteristics字段，加上可执行属性
   
   实验：在最后一个节增加500字节
   1. 用ue打开程序，查看最后一个节的解表
   2. 500字节，转换为10进制 = 1280
   3. 右键-》16进制插入/删除，插入1280
   4. 修改SizeOfRawData和VirtualSize的值，并计算出差值。
   5. 修改SizeOfImage，SizeOfImage += 差值
   6. 看程序是否能正常打开。
   
6. 新增节
   1. 判断是否有足够的空间，可以添加一个节表
   2. 在解表中新增一个成员
   3. 修改pe头中节的数量
   4. 修改sizeOfImage的大小
   5. 再原有数据的最后，新增一个节的数据（内存对齐的整数倍）
   6. 修正新增节表的属性
7. 合并节
8. 导出表
   扩展pe头里的最后一个成员有16个结构体
   ```c++
   struct _IMAGE_DATA_DIRECTORY {
      0X00 DWORD VirtualAddress; // 地址， 这个值是rav，需要转换成foa
      0x04 DWORD Size;           // 大小
   }
   ```
   1. 导出表结构体
   ```c++
   typedef struct _IMAGE_EXPORT_DIRECTORY {
       DWORD Characteristics;        // 未使用
       DWORD TimeDateStamp;          // 时间戳
       WORD MajorVersion;            // 未使用
       WORD MinorVersion;            // 未使用
       DWORD Name;                   // 指向该导出表文件名字字符串
       DWORD Base;                   // 导出函数起始序号
       DWORD NumberOfFunctions;      // 所有导出函数的个数
       DWORD NumberOfName;           // 以函数名字导出的函数个数
       DWORD AddressOfFunctions;     // 导出函数地址表rva
       DWORD AddressOfNames;         // 导出函数名称表rva
       DWORD AddressOfNameOrdinals;  // 导出函数序号表rva
   }
   ```
   2. 
9. 导入表 确定依赖模块
   1. 导入表结构
   ```c++
   typedef struct _IMAGE_IMPORT_DESCRIPTOR {
       union {
          DWORD Characteristics;
          DWORD OriginalFirstThunk;  // RVA指向IMAGE_THUNK_DATA结构数组
       };
       DWORD TimeDateStamp;          // 时间戳
       DWORD ForwarderChain;
       DWORD Name;                   // RVA指向dll名字，该名字以20个0结尾
       DWORD FirstThunk;             // RVA指向IMAGE_THUNK_DATA结构数组
   }
   
   typedef struct _IMAGE_THUNK_DATA32 {
     union {
        PBYTE ForwarderString;
        PDWORD Function;
        DWORD Ordinal;                       // 序号
        PIMAGE_IMPORT_BY_NAME AddressOfData; // 指向IMAGE_IMPORT_BY_NAME
     }
   }
   
   typedef struct _IMAGE_IMPORT_BY_NAME {
     WORD Hint;     // 可能为空，编辑器决定如果不为空是函数在导出表中的索引
     BYTE Name[1];  // 函数名称，以0结尾
   } IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
   ```
10. 导入表 确定依赖函数
11. 导入表 确定函数地址
12. 重定位表
    重定位表结构体
    ```c++
    typedef struct _IMAGE_BASE_RELOCATION {
      DWORD VirtualAddress;
      DWORD SizeOfBlock;
    } IMAGE_BASE_RELOCATION;
    ```
13. shellcode
    1. 什么是shellcode
    不依赖环境，放到任何地方都可以运行的机器码
    2. 编写规则
       * 不能有全局变量
       * 不能使用常量字符串
       
         要这样写： char szBuffer[] = {'H', 'e', 'l', 'l', 'o', 0};
       * 不能使用系统调用
       * 不能嵌套调用其他函数
    3. 啊
14. 是
