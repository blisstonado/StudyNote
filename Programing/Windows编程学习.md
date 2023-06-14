1. 常见数据类型
   1. UINT   无符号32位整型
   2. DWORD  32位整数
   3. PDWORD 32位整数类型指针
   4. BOOL   布尔类型
   5. SHORT  带符号16位整数
   6. LRSULT 32位函数返回值
   7. WPARAM 32位的消息参数
   8. LPARAM 32位的消息参数
2. HANDLE 是句柄，表示操作系统中某个对象
   1. HANDLE 通用句柄
   2. HWND   窗口句柄
   3. HINSTANCE 实例句柄
3. 程序入口函数
   ```
     int WINAPI WinMain(
         HINSTANCE hInstance,     // 程序实例句柄
         HINSTANCE hPreInstance,  // 上一个程序实例句柄
         LPSTR lpCmdLine,         // 命令行参数
         int mCmdShow             // 显示方式
     ) 
   ```
4. 字符串处理
