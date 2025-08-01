---
description: "This method informs the process that a session is no longer debugging the process."
title: IDebugProcessEx2::Detach
ms.date: 11/04/2016
ms.topic: reference
f1_keywords:
- IDebugProcessEx2::Detach
helpviewer_keywords:
- IDebugProcessEx2::Detach method
author: tinaschrepfer
ms.author: tinali
manager: mijacobs
ms.subservice: debug-diagnostics
dev_langs:
- CPP
- CSharp
---
# IDebugProcessEx2::Detach

This method informs the process that a session is no longer debugging the process.

## Syntax

### [C#](#tab/csharp)
```csharp
int Detach(
   IDebugSession2 pSession
);
```
### [C++](#tab/cpp)
```cpp
HRESULT Detach( 
   IDebugSession2* pSession
);
```
---

## Parameters
`pSession`\
[in] A value that uniquely identifies the session to detach this process from.

## Return Value
 If successful, returns `S_OK`; otherwise, returns an error code.

## Remarks
 The interface passed in `pSession` is to be treated only as a cookie, a value that uniquely identifies the session debug manager that originally attached to this process; none of the methods on the supplied interface are functional.

## See also
- [IDebugProcessEx2](../../../extensibility/debugger/reference/idebugprocessex2.md)
