# Cpper
C++相關知識

## 目錄
* 繼承
* 虛擬(virtual)
* 執行緒
* 開檔共用規則
* MFC
* IOCP
* 資料型態

## 繼承
* 概念:沿用程式碼來提升開發效率
* 分類:
  1. 公共繼承
  2. 保護繼承
  3. 私有繼承
* ex. 父類別: Point2D，子類別: Point3D
  ```cpp=
  class Point2D { 
  public: 
      Point2D() {
          _x = 0;
          _y = 0;
      }
      Point2D(int x, int y) : _x(x), _y(y) {
      }
      int x() {return _x;} 
      int y() {return _y;} 
      void x(int x) {
          _x = x;
      }
      void y(int y) {
          _y = y;
      }
  private:
      int _x;
      int _y;
  }; 
  ```
  ```cpp=
  #include "Point2D.h"

  class Point3D : public Point2D { 
  public: 
      Point3D() { 
          _z = 0; 
      } 
      Point3D(int x, int y, int z) : Point2D(x, y), _z(z) { 
      } 
      int z() {return _z;}
      void z(int z) {_z = z;}

  private:
      int _z;
  }; 
  ```

## 執行緒(Multithread)
* Reference: https://docs.microsoft.com/zh-tw/windows/win32/procthread/creating-threads
* tip: 
  1. 代入執行緒的參數可以是物件指標，之後使用物件的內容是使用指標的方式進行
  2. 指標外的變數需要是static才能使用

    ```cpp=
    // parameter on the heap to avoid possible threading bugs
    int* id = new int(1);
    CreateThread(NULL, NULL, mHandler, id, NULL, NULL);


    DWORD WINAPI mHandler(LPVOID sId) {
        // make a copy of the parameter for convenience
        int id = *static_cast<int*>(sId);
        delete sId;

        // now do something with id
    }
    ```
    other example
    ```cpp=
    class MyClass
    {
        static DWORD WINAPI StaticThreadStart(void* Param)
        {
            MyClass* This = (MyClass*) Param;
            return This->ThreadStart();
        }

        DWORD ThreadStart(void)
        {
            // Do stuff
        }

        void startMyThread()
        {
           DWORD ThreadID;
           CreateThread(NULL, 0, StaticThreadStart, (void*) this, 0, &ThreadID);
        }
    };
    ```

## 開檔共用規則
* Reference: https://docs.microsoft.com/zh-tw/windows/win32/fileio/creating-and-opening-files

## MFC
### 關掉執行緒  
```cpp=
EndDialog(1);
```
### CString to CTime
```cpp=
COleDateTime::ParseDateTime(CString)
```
### CString functions / CString cheatsheet
* Reference: https://docs.microsoft.com/zh-tw/cpp/atl-mfc-shared/reference/cstringt-class?view=msvc-170

### 視窗大小、位置問題
#### 共同點:
* 邏輯座標:是為了屏蔽掉不同設備屬性差別而設置的抽象坐標系，是相對於設備座標的統一接口，在邏輯座標繪圖(CDC)，之後會由設備驅動去負責虛擬座標到實際設備座標之間的轉換，通常左上角為原點，x軸向右為正，y軸向下為正
#### GetWindowRect
#### GetClientRect
#### SetWindowPos設定物件位置
```cpp=
CRect rect;
GetDesktopWindow()->GetWindowRect(&rect); // 抓運行機器螢幕大小by pixel

CWnd* pwndCLabel;
pwndCLabel = GetDlgItem(IDC_ALARMINFO); // 利用id抓物件(得到指標)，MFC中所有元件都算dialog?

pwndCLabel->SetWindowPos(NULL, 0, 0, rect.BottomRight().x/4, rect.BottomRight().y/4, SWP_NOZORDER); // 設定元件位置
//內容分別為(0, 0): 元件左上角相對畫面左上角距離by pixel
//(rect.BottomRight().x/4, rect.BottomRight().y/4): 元件的寬與高
//dialog使用SetWindowPos時上述4個參數會變為以螢幕左上角起點為標準的畫面左上角、右下角點
```

### 建立視窗(中國用語:對話框?)(dialog)
* MFC新建視窗需繼承CDialog中的OnInitDialog()，但不能用寫程式的方式完成新建視窗的動作，會無法通過編譯，需要手動到UI(.rc)介面新建類並在類中繼承虛擬函數中的OnInitDialog()

### 一個視窗常用需手動建立連結的function
* 需要手動到UI(.rc)介面新建類並在類中繼承虛擬函數
* 功能:
  1. Create
  2. OnInitDialog

## IOCP
### 評論
* 實現高容量網路伺服器的最佳方法:高負載、延伸性

### 什麼是IOCP?
* socket連接完成端口，當一個事件發生後，完成端口將被加入一個隊列中，之後應用程式可以對核心進行查詢以得到完成端口
* [疑問] select()? -> non-blocking，function被call後立即回傳，不卡住(等待)
### 觀念
#### 非同步 通信

我們知道外部設備I/O（比如磁碟讀寫，網路通信等）速度和CPU速度比起來是很慢的，如果我們進行外部I/O操作時線上程中等待I/O操作完成的話，此線程就會被阻塞住，相當於強迫CPU適應I/O設備的速度，這樣會造成極大的CPU資源浪費。我們沒必要線上程中等待I/O操作完成再執行後續的代碼，而是將I/O操作請求交給設備驅動去處理，我們線程可以繼續做其他事情，然後等待I/O操作完成的通知

#### 同步/異步
* 同步: 做完一件事後再去做另一件事
* 異步: 同時一起做兩件或兩件以上事

#### 同步/堵塞/異步/非堵塞
* 堵塞(accept)(blocking):等待function返回，否則一直等待

#### 完成"端口"
完成端口中所謂的"端口"並不是我們在TCP/IP中所提到的端口，可以說是完全没有關係。我到现在也没想通一個I/O設備(I/O Device)和端口(IOCP中的Port)有什么關係。IOCP只不過是用來進行讀寫操作，和文件I/O倒是有些類似。既然是一個讀寫設備，我們所能要求它的只是在處理讀與寫上的高效

#### 線程/進程
* 線程:thread
* 進程:運行中的一個程序，可能由多個線程組成
#### 1. 基礎網路程式: 接到的指令都轉成多線程 -> cpu耗在轉換線程，效率低
```cpp
int main()
{
    WSAStartup(MAKEWORD(2, 2), &wsaData); //WinSock第一個必須被執行的函式，用來註冊動態連結函式庫
    ListeningSocket = socket(AF_INET, SOCK_STREAM, 0); 
    bind(ListeningSocket, (SOCKADDR*)&ServerAddr, sizeof(ServerAddr));
    listen(ListeningSocket, 5);
    int nlistenAddrLen = sizeof(ClientAddr);
    while(TRUE)
    {
        NewConnection = accept(ListeningSocket, (SOCKADDR*)&ClientAddr, &nlistenAddrLen);
        HANDLE hThread = CreateThread(NULL, 0, ThreadFunc, (void*) NewConnection, 0, &dwTreadId);
        CloseHandle(hThread);
    }
    return 0;
}
```
#### 2. 進化，不想無限增加的多線程，提供一個消息隊列，多線程會讀隊列的內容，讀到消息後就加以處理 -> I/O完成端口
* 與端口的關係:應用程序和操作系統溝通的一個接口

### 實現方法
1. CreateIoCompletionPort (將客戶的socket作為HANDLE傳進隊列)
  ```cpp
  HANDLE CreateIoCompletionPort (
      HANDLE FileHandle,              // handle to file
      HANDLE ExistingCompletionPort,  // handle to I/O completion port
      ULONG_PTR CompletionKey,        // completion key
      DWORD NumberOfConcurrentThreads // number of threads to execute concurrently完成端口上同時允許運行的線程最大數(經驗法則:線程數=CPU數*2+2)
  );
  ```
  * 目的(端口其實就是隊列?)
    1. 用於創建一個完成端口對象
    2. 將一個HANDLE和完成端口關連在一起

  * 有的功能
    1. 每當你向端口關聯一個設備時，系統向該完成端口的設備列表中加入一條信息記錄
2. GetQueuedCompletionStatus
  ```cpp
  BOOL GetQueuedCompletionStatus(
      HANDLE CompletionPort,        // handle to completion port
      LPDWORD lpNumberOfBytes,      // bytes transferred
      PULONG_PTR lpCompletionKey,   // file completion key
      LPOVERLAPPED *lpOverlapped,   // buffer
      DWORD dwMilliseconds       　 // optional timeout value
  );
  ```
  * 解說
    1. 第一個參數指出了線程要監視哪一個完成端口
    2. 很多服務應用程式只是使用一個I/O完成端口，所有的I/O請求完成以後的通知都將發給該端口

  * 功能(情境)
    1. dwMilliseconds是可以等待一個completion packet出現的時間，時間內沒出現會直接return並return FALSE，*lpOverlapped是NULL
      * dwMilliseconds is INFINITE: never time out
      * dwMilliseconds is 0: if there is no I/O operation to dequeue, the function will timeout immediately
    2. 回傳值: Returns nonzero (TRUE) if successful or zero (FALSE) otherwise.

## 資料型態
### string 也可以存寬字符
```cpp
wstring a = L"string content";
```
