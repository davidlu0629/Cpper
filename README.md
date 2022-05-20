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
