# Windows VC++电源节能模式监听示例
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
					// 节点模式关闭
					// 1. 打开WIFI开关
					//ShellExecute(NULL, L"runas", L"cmd", L"/c netsh wlan connect \"TATER\"", NULL, SW_SHOWNORMAL);
					ShellExecute(NULL, L"runas", L"cmd", L"/c netsh wlan connect \"TATER\"", NULL, SW_HIDE);
					// 2. 打开蓝牙开关
				}
				else if (dw == 1) {
					std::cout << "节电模式打开" << std::endl;
					// 节点模式关闭
					// 1. 关闭WIFI开关
					//ShellExecute(NULL, L"runas", L"cmd", L"/c netsh wlan disconnect", NULL, SW_SHOWNORMAL);
					ShellExecute(NULL, L"runas", L"cmd", L"/c netsh wlan disconnect", NULL, SW_HIDE);
					// 2. 关闭蓝牙开关
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
