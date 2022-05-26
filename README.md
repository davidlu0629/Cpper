# Cpper
C++相關知識

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
