---
description: "Resets the code contexts enumeration to the first element."
title: IEnumDebugCodeContexts2::Reset
ms.date: 11/04/2016
ms.topic: reference
f1_keywords:
- IEnumDebugCodeContexts2::Reset
helpviewer_keywords:
- IEnumDebugCodeContexts2::Reset
author: tinaschrepfer
ms.author: tinali
manager: mijacobs
ms.subservice: debug-diagnostics
dev_langs:
- CPP
- CSharp
---
# IEnumDebugCodeContexts2::Reset

Resets the enumeration to the first element.

## Syntax

### [C#](#tab/csharp)
```csharp
int Reset();
```
### [C++](#tab/cpp)
```cpp
HRESULT Reset(
   void
);
```
---

## Return Value
 If successful, returns `S_OK`; otherwise, returns an error code.

## Remarks
 After this method is called, the next call to the [Next](../../../extensibility/debugger/reference/ienumdebugcodecontexts2-next.md) method returns the first element of the enumeration.

## See also
- [IEnumDebugCodeContexts2](../../../extensibility/debugger/reference/ienumdebugcodecontexts2.md)
