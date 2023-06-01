# VC++监听USB插拔事件
- [C++之Windows监听USB热插拔事件](https://blog.csdn.net/qccz123456/article/details/80006820)

- [C++实现usb插入操作监控](https://blog.csdn.net/Felix_Dreammaker/article/details/79150137)
- [windows系统睡眠与合上盖子事件捕获](https://blog.csdn.net/gb173652770/article/details/124611984)
- [Determining the Monitor's On/Off (sleep) Status](https://www.codeproject.com/Articles/1193099/Determining-the-Monitors-On-Off-sleep-Status)
- [Power Management Functions](https://learn.microsoft.com/en-us/windows/win32/power/power-management-functions)
- [GetSystemPowerStatus function](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-getsystempowerstatus)

# 电源设置 GUID
GUID_POWER_SAVING_STATUS

E00958C0-C213-4ACE-AC77-FECCED2EEEA5

节电器已关闭或关闭，以响应不断变化的电源条件。 此通知对于参与节能的组件非常有用。 当节电器打开时，这些应用程序应注册此通知并节省电源。

数据成员是指示节电状态的 DWORD。

0x0 - 节电器已关闭。

0x1 - 节电器已打开。 尽可能节省能源。

有关节电器的一般信息，请参阅 硬件组件指南中的节电器 。

- [RegisterSuspendResumeNotification 函数](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-registersuspendresumenotification)
注册以在系统暂停或恢复时接收通知。 类似于 PowerRegisterSuspendResumeNotification，但在用户模式下运行，可以采用窗口句柄。
- [电源设置 GUID](https://learn.microsoft.com/zh-cn/windows/win32/power/power-setting-guids)
- [节电模式](https://learn.microsoft.com/zh-cn/windows-hardware/design/component-guidelines/battery-saver)

```cpp
#include <Windows.h>
#include <tchar.h>
#include <atlstr.h> // CString
#include <powerbase.h>
#include <tchar.h>
#include <Dbt.h>
#include <setupapi.h>
#include <iostream>
#include <winnt.h>
#include <sstream>
#include <powrprof.h>
using namespace std;

#pragma comment (lib, "Kernel32.lib")
#pragma comment (lib, "User32.lib")
#pragma comment(lib,"Powrprof.lib")

#define THRD_MESSAGE_EXIT WM_USER + 1
const _TCHAR CLASS_NAME[] = _T("Sample Window Class");

HWND hWnd = NULL;

static const GUID GUID_DEVINTERFACE_LIST[] =
{
	// GUID_DEVINTERFACE_USB_DEVICE
	{ 0xA5DCBF10, 0x6530, 0x11D2, { 0x90, 0x1F, 0x00, 0xC0, 0x4F, 0xB9, 0x51, 0xED } },
	// GUID_DEVINTERFACE_DISK
	{ 0x53f56307, 0xb6bf, 0x11d0, { 0x94, 0xf2, 0x00, 0xa0, 0xc9, 0x1e, 0xfb, 0x8b } },
	// GUID_DEVINTERFACE_HID, 
	{ 0x4D1E55B2, 0xF16F, 0x11CF, { 0x88, 0xCB, 0x00, 0x11, 0x11, 0x00, 0x00, 0x30 } },
	// GUID_NDIS_LAN_CLASS
	{ 0xad498944, 0x762f, 0x11d0, { 0x8d, 0xcb, 0x00, 0xc0, 0x4f, 0xc3, 0x35, 0x8c } }
	// GUID_DEVINTERFACE_COMPORT
	//{ 0x86e0d1e0, 0x8089, 0x11d0, { 0x9c, 0xe4, 0x08, 0x00, 0x3e, 0x30, 0x1f, 0x73 } },
	//  GUID_DEVINTERFACE_SERENUM_BUS_ENUMERATOR
	//{ 0x4D36E978, 0xE325, 0x11CE, { 0xBF, 0xC1, 0x08, 0x00, 0x2B, 0xE1, 0x03, 0x18 } },
	// GUID_DEVINTERFACE_PARALLEL
	//{ 0x97F76EF0, 0xF883, 0x11D0, { 0xAF, 0x1F, 0x00, 0x00, 0xF8, 0x00, 0x84, 0x5C } },
	//  GUID_DEVINTERFACE_PARCLASS
	//{ 0x811FC6A5, 0xF728, 0x11D0, { 0xA5, 0x37, 0x00, 0x00, 0xF8, 0x75, 0x3E, 0xD1 } }
};

void UpdateDevice(PDEV_BROADCAST_DEVICEINTERFACE pDevInf, WPARAM wParam)
{
	CString szDevId = pDevInf->dbcc_name + 4;
	int idx = szDevId.ReverseFind(_T('#'));
	szDevId.Truncate(idx);
	szDevId.Replace(_T('#'), _T('\\'));
	szDevId.MakeUpper();

	CString szClass;
	idx = szDevId.Find(_T('\\'));
	szClass = szDevId.Left(idx);

	CString szTmp;
	
	if (DBT_DEVICEARRIVAL == wParam) // USB插入
		szTmp.Format(_T("Adding %s\r\n"), szDevId.GetBuffer());
	else
		// USB移除
		szTmp.Format(_T("Removing %s\r\n"), szDevId.GetBuffer());

	_tprintf(szTmp);
}


LRESULT DeviceChange(UINT message, WPARAM wParam, LPARAM lParam)
{
	// USB插入和USB移除（USB插拔事件发生）
	if (DBT_DEVICEARRIVAL == wParam || DBT_DEVICEREMOVECOMPLETE == wParam)
	{
		PDEV_BROADCAST_HDR pHdr = (PDEV_BROADCAST_HDR)lParam;
		PDEV_BROADCAST_DEVICEINTERFACE pDevInf;
		PDEV_BROADCAST_HANDLE pDevHnd;
		PDEV_BROADCAST_OEM pDevOem;
		PDEV_BROADCAST_PORT pDevPort;
		PDEV_BROADCAST_VOLUME pDevVolume;
		switch (pHdr->dbch_devicetype)
		{
		case DBT_DEVTYP_DEVICEINTERFACE:
			pDevInf = (PDEV_BROADCAST_DEVICEINTERFACE)pHdr;
			UpdateDevice(pDevInf, wParam);
			break;

		case DBT_DEVTYP_HANDLE:
			pDevHnd = (PDEV_BROADCAST_HANDLE)pHdr;
			break;

		case DBT_DEVTYP_OEM:
			pDevOem = (PDEV_BROADCAST_OEM)pHdr;
			break;

		case DBT_DEVTYP_PORT:
			pDevPort = (PDEV_BROADCAST_PORT)pHdr;
			break;

		case DBT_DEVTYP_VOLUME:
			pDevVolume = (PDEV_BROADCAST_VOLUME)pHdr;
			break;
		}
	}
	return 0;
}

// 窗口过程函数，用于处理电源事件
LRESULT PowerSettingChange(UINT message, WPARAM wParam, LPARAM lParam)
{
	if (message == WM_POWERBROADCAST)
	{
		std::cout << "WM_POWERBROADCAST" << std::endl;

		DWORD dwWParam = (DWORD)wParam;
		if (dwWParam == PBT_POWERSETTINGCHANGE) {
			//std::cout << "PBT_POWERSETTINGCHANGE" << std::endl;
			PPOWERBROADCAST_SETTING dwLParam = (PPOWERBROADCAST_SETTING)lParam;
			if (dwLParam->PowerSetting == GUID_POWER_SAVING_STATUS) {
				std::cout << "GUID_POWER_SAVING_STATUS." << std::endl;

				DWORD dw = dwLParam->Data[0];
				if (dw == 0) {
					std::cout << "节电模式关闭" << std::endl;
					// TODO: 节点模式打开
					// 1. 关闭WIFI开关
					// 2. 关闭蓝牙开关
				}
				else if (dw == 1) {
					std::cout << "节电模式打开" << std::endl;
					// TODO: 节点模式打开
					// 1. 打开WIFI开关
					// 2. 打开蓝牙开关
				}
				else {
					std::cout << "power saving no open neither close" << std::endl;
				}
			}
		}
		return TRUE;
	}
	else
		return DefWindowProc(hWnd, message, wParam, lParam);
}


LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
	switch (message)
	{
	case WM_PAINT:
		break;
	case WM_SIZE:
		break;
	case WM_DEVICECHANGE:
		return DeviceChange(message, wParam, lParam);
	case WM_POWERBROADCAST:
		return PowerSettingChange(message, wParam, lParam);
	}

	return DefWindowProc(hWnd, message, wParam, lParam);
}

ATOM MyRegisterClass()
{
	WNDCLASS wc = { 0 };
	wc.lpfnWndProc = WndProc;
	wc.hInstance = GetModuleHandle(NULL);
	wc.lpszClassName = CLASS_NAME;
	return RegisterClass(&wc);
}

bool CreateMessageOnlyWindow()
{
	hWnd = CreateWindowEx(0, CLASS_NAME, _T(""), WS_OVERLAPPEDWINDOW,
		CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT,
		NULL,       // Parent window    
		NULL,       // Menu
		GetModuleHandle(NULL),  // Instance handle
		NULL        // Additional application data
	);

	return hWnd != NULL;
}

void RegisterDeviceNotify()
{
	HDEVNOTIFY hDevNotify;
	for (int i = 0; i < sizeof(GUID_DEVINTERFACE_LIST) / sizeof(GUID); i++)
	{
		DEV_BROADCAST_DEVICEINTERFACE NotificationFilter;
		ZeroMemory(&NotificationFilter, sizeof(NotificationFilter));
		NotificationFilter.dbcc_size = sizeof(DEV_BROADCAST_DEVICEINTERFACE);
		NotificationFilter.dbcc_devicetype = DBT_DEVTYP_DEVICEINTERFACE;
		NotificationFilter.dbcc_classguid = GUID_DEVINTERFACE_LIST[i];
		hDevNotify = RegisterDeviceNotification(hWnd, &NotificationFilter, DEVICE_NOTIFY_WINDOW_HANDLE);
	}
}

DWORD WINAPI ThrdFunc(LPVOID lpParam)
{
	if (0 == MyRegisterClass())
		return -1;

	if (!CreateMessageOnlyWindow())
		return -1;

	// 注册USB事件通知函数
	RegisterDeviceNotify();

	// 注册应用程序以接收特定电源设置事件的电源设置通知。 
	// GUID_POWER_SAVING_STATUS 代表省电模式
	RegisterPowerSettingNotification(hWnd, &GUID_POWER_SAVING_STATUS, DEVICE_NOTIFY_WINDOW_HANDLE);

	MSG msg;
	while (GetMessage(&msg, NULL, 0, 0))
	{
		if (msg.message == THRD_MESSAGE_EXIT)
		{
			cout << "worker receive the exiting Message..." << endl;
			return 0;
		}

		TranslateMessage(&msg);
		DispatchMessage(&msg);
	}

	return 0;
}

void GetPowerInfo()
{
	SYSTEM_POWER_STATUS sysPowerStatus;
	GetSystemPowerStatus(&sysPowerStatus);

	int powerStatus = static_cast<int>(sysPowerStatus.ACLineStatus);
	if (powerStatus == 1) {
		std::cout << "电池状态: AC" << std::endl;
	}
	else {
		std::cout << "电池状态: DC" << std::endl;
	}
	cout << "电源状态: " << ((int)sysPowerStatus.ACLineStatus == 1 ? "AC" : "DC") << endl;
	cout << "电池状态: " << (int)sysPowerStatus.BatteryFlag << endl;
	cout << "电量百分比: " << (int)sysPowerStatus.BatteryLifePercent << " %" << endl;
	cout << "剩余能量: " << sysPowerStatus.BatteryLifeTime << " 秒" << endl;
	cout << "总能量: " << sysPowerStatus.BatteryFullLifeTime << " 秒" << endl;
	cout << "节电模式: " << (int)sysPowerStatus.SystemStatusFlag << std::endl;
}

int main(int argc, char** argv)
{
	DWORD iThread;
	HANDLE hThread = CreateThread(NULL, 0, ThrdFunc, NULL, 0, &iThread);
	if (hThread == NULL) {
		cout << "error" << endl;
		return -1;
	}

	char chQtNum;
	do
	{
		cout << "enter Q/q for quit: " << endl;
		cin >> chQtNum;

	} while (chQtNum != 'Q' && chQtNum != 'q');

	PostThreadMessage(iThread, THRD_MESSAGE_EXIT, 0, 0);
	WaitForSingleObject(hThread, INFINITE);
	CloseHandle(hThread);

	return 0;
}
```
