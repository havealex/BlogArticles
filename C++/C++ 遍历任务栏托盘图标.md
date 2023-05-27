# [C/C++ 遍历任务栏托盘图标](https://www.cnblogs.com/LyShark/p/15019562.html)
托盘图标，就是底部任务栏托盘，可以直接遍历，如果需要输出任务栏中的图片内容可使用如下方式。
![image](https://github.com/havealex/BlogArticles/assets/74997847/1eb93fd7-e40c-405a-b716-dcc67bf2c613)
实现代码:
```cpp
#include <iostream>
#include <windows.h>
#include <string>
#include <commctrl.h>
#include <atlstr.h>

using namespace std;

// 判断 x64 系统
BOOL Is64bitSystem()
{
	SYSTEM_INFO si;
	GetNativeSystemInfo(&si);
	if (si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_AMD64 ||
		si.wProcessorArchitecture == PROCESSOR_ARCHITECTURE_IA64)
		return TRUE;
	else
		return FALSE;
}

// 获取托盘窗口句柄
HWND FindTrayWnd()
{
	HWND hWnd = NULL;

	hWnd = FindWindow(_T("Shell_TrayWnd"), NULL);
	hWnd = FindWindowEx(hWnd, NULL, _T("TrayNotifyWnd"), NULL);
	hWnd = FindWindowEx(hWnd, NULL, _T("SysPager"), NULL);
	hWnd = FindWindowEx(hWnd, NULL, _T("ToolbarWindow32"), NULL);

	return hWnd;
}

// 获取折叠托盘窗口句柄
HWND FindNotifyIconOverflowWindow()
{
	HWND hWnd = NULL;

	hWnd = FindWindow(_T("NotifyIconOverflowWindow"), NULL);
	hWnd = FindWindowEx(hWnd, NULL, _T("ToolbarWindow32"), NULL);

	return hWnd;
}

// 遍历窗口
BOOL EnumNotifyWindow(HWND hWnd)
{
	// 获取托盘进程ID
	DWORD dwProcessId = 0;
	GetWindowThreadProcessId(hWnd, &dwProcessId);
	if (dwProcessId == 0) {
		cout << "GetWindowThreadProcessId failed:" << GetLastError() << endl;
		return FALSE;
	}

	// 获取托盘进程句柄
	HANDLE hProcess = OpenProcess(
		PROCESS_VM_OPERATION |	// 需要在进程的地址空间上执行操作
			PROCESS_VM_READ |	// 需要使用 ReadProcessMemory 读取进程中的内存
			PROCESS_VM_WRITE,	// 需要在使用 WriteProcessMemory 的进程中写入内存
		FALSE,					// 子进程不继承句柄
		dwProcessId				// 目标进程 PID
	);
	if (hProcess == NULL) {
		cout << "OpenProcess failed:" << GetLastError() << endl;
		return FALSE;
	}

	// 在进程虚拟空间中分配内存，用来接收 TBBUTTON 结构体指针
	LPVOID p_tbbutton = VirtualAllocEx(
		hProcess,					// 目标进程句柄
		0,							// 内存起始地址（默认）
		4096,						// 内存大小
		MEM_COMMIT,					// 内存类型（提交）
		PAGE_EXECUTE_READWRITE		// 内存保护属性（可读可写可执行）
	);
	if (p_tbbutton == NULL) {
		cout << "VirtualAllocEx failed:" << GetLastError() << endl;
		return FALSE;
	}
	/*
		typedef struct _TBBUTTON {
		    int iBitmap;				// 索引
		    int idCommand;				// 与按钮关联的命令标识符
		    BYTE fsState;				// 按钮状态
		    BYTE fsStyle;				// 按钮风格
		#ifdef _WIN64
		    BYTE bReserved[6];          // 对齐
		#elif defined(_WIN32)
		    BYTE bReserved[2];          // 对齐
		#endif
		    DWORD_PTR dwData;			// 存放程序自定义数据
		    INT_PTR iString;			// 存放信息字符
		} TBBUTTON, NEAR* PTBBUTTON, *LPTBBUTTON;
	*/

	// 初始化
	DWORD dw_addr_dwData = 0;
	BYTE buff[1024] = { 0 };
	wstring ws_filePath = L"";
	wstring ws_tile = L"";
	HWND h_mainWnd = NULL;
	int i_data_offset = 12;		
	int i_str_offset = 18;		
	
	// 判断 x64
	if (Is64bitSystem()) {
		i_data_offset += 4;	
		i_str_offset += 6;
	}

	// 获取托盘图标个数
	int i_buttons = 0;
	i_buttons = SendMessage(hWnd, TB_BUTTONCOUNT, 0, 0);
	if (i_buttons == 0) {
		cout << "TB_BUTTONCOUNT message failed:" << GetLastError() << endl;
		return FALSE;
	}

	// 遍历托盘
	for (int i = 0; i < i_buttons; i++) {
		// 获取 TBBUTTON 结构体指针
		if (!SendMessage(hWnd, TB_GETBUTTON, i, (LPARAM)p_tbbutton)) {
			cout << "TB_GETBUTTON message failed:" << GetLastError() << endl;
			return FALSE;
		}

		// 读 TBBUTTON.dwData（附加信息）
		if (!ReadProcessMemory(hProcess, (LPVOID)((DWORD)p_tbbutton + i_data_offset), &dw_addr_dwData, 4, 0)) {
			cout << "ReadProcessMemory failed:" << GetLastError() << endl;
			return FALSE;
		}

		// 读文本
		if (dw_addr_dwData) {
			if (!ReadProcessMemory(hProcess, (LPCVOID)dw_addr_dwData, buff, 1024, 0)) {
				cout << "ReadProcessMemory failed:" << GetLastError() << endl;
				return FALSE;
			}
			h_mainWnd = (HWND)(*((DWORD*)buff));
			ws_filePath = (WCHAR*)buff + i_str_offset;
			ws_tile = (WCHAR*)buff + i_str_offset + MAX_PATH;
			cout << "hMainWnd = " << hex << h_mainWnd << endl;
			wcout << "strFilePath = " << ws_filePath << endl;
			wcout << "strTile = " << ws_tile << endl;
		}

		// 清理
		dw_addr_dwData = 0;
		h_mainWnd = NULL;
		ws_filePath = L"";
		ws_tile = L"";
		
		cout << endl;
	}
	if (VirtualFreeEx(hProcess, p_tbbutton, 0, MEM_RELEASE) == 0) {
		cout << "VirtualFreeEx failed:" << GetLastError() << endl;
		return FALSE;
	}
	if (CloseHandle(hProcess) == 0) {
		cout << "CloseHandle failed:" << GetLastError() << endl;
		return FALSE;
	}

	return TRUE;
}

int main()
{
	// 解决控制台中文 '？'
	setlocale(LC_ALL, "chs");

	// 获取托盘句柄
	HWND h_tray = FindTrayWnd();
	HWND h_tray_fold = FindNotifyIconOverflowWindow();

	// 遍历托盘窗口
	if (EnumNotifyWindow(h_tray) == FALSE || EnumNotifyWindow(h_tray_fold) == FALSE) {
		cout << "EnumNotifyWindow false." << endl;
	}

	cin.get();
	return 0;
}
```

其中，有几个需要注意的地方

这一句的目的是定位 TBBUTTON.dwData ，因为 64位系统和32位系统 bReserved 参数大小差了 4字节大小，所以要加 4
```cpp
int i_data_offset = 12;	

if (Is64bitSystem()) {
		i_data_offset += 4;		// <--
		i_str_offset += 6;
}
```
2.这个是为了找到数据之前最后一个 “?” ：
```cpp
int i_str_offset = 18;
if (Is64bitSystem()) {
		i_data_offset += 4;	
		i_str_offset += 6;		// <--
}
```

这一句代码的目的是跳过 data 的覆盖范围：
```cpp
ws_tile = (WCHAR*)buff + i_str_offset + MAX_PATH;
```
