# UnRecallQQMsg
prevent mac qq form revoking message

## **IMPORTANT**: all of the code are from here:  
[https://c.ecl.me/archives/mac-qq-unrecall.html](https://c.ecl.me/archives/mac-qq-unrecall.html)  
[https://github.com/doveccl/QQU/](https://github.com/doveccl/QQU/)

---

Here is just a backup.  

create file `QQUnrecall.m`:  

```
#import <Foundation/Foundation.h>
#import <objc/runtime.h>
void handleRecallNotifyIsOnline(id _i, SEL _s, void * _p, BOOL _b) {
    // NSLog(@"已经阻止 QQ 撤回一条消息");
}
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
static void __attribute__((constructor)) initialize(void) {
    method_setImplementation(
        class_getInstanceMethod(
            NSClassFromString(@"QQMessageRevokeEngine"),
            @selector(handleRecallNotify:isOnline:)
        ),
        (IMP)&handleRecallNotifyIsOnline
    );
}
#pragma clang diagnostic pop
```

generate dylib:  


```
clang -dynamiclib -framework Foundation QQUnrecall.m -o libQQUnrecall.dylib
```

Usage:  

* execute command:  

```
export DYLD_INSERT_LIBRARIES=libQQUnrecall.dylib && /Applications/QQ.app/Contents/MacOS/QQ
```

* **OR** create a mac Xcode project (CocoaApp), you can delete the irrelevant files such as `AppDelegate.m`). Remember to add `libQQUnrecall.dylib` to the project and make sure it is added to `Build Phases - Copy Bundle Resources`, and edit `main.m`:   


```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define QPATH "/Applications/QQ.app/Contents/MacOS/"
#define LIBENV "DYLD_INSERT_LIBRARIES"
#define LIBNAME "libQQUnrecall.dylib"
#define HOOK "export " LIBENV "=%s" LIBNAME
#define RUN QPATH "QQ"
#define AND " && "
#define CMD HOOK AND RUN
#define M 1024
char path[M], buff[M];
unsigned long len, i;
int main(int argc, char **argv) {
    strcpy(path, argv[0]);
    len = strlen(path);
    for (i = len - 1; i; i--) {
        if (path[i] == '/')
            break ;
        path[i] = 0;
    }
    sprintf(buff, CMD, path);
    system(buff);
    return 0;
}
```
