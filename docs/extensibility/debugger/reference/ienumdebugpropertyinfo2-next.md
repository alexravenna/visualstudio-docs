---
description: "Returns the next set of DEBUG_PROPERTY_INFO elements from the enumeration."
title: IEnumDebugPropertyInfo2::Next
ms.date: 11/04/2016
ms.topic: reference
f1_keywords:
- IEnumDebugPropertyInfo2::Next
helpviewer_keywords:
- IEnumDebugPropertyInfo2::Next
author: tinaschrepfer
ms.author: tinali
manager: mijacobs
ms.subservice: debug-diagnostics
dev_langs:
- CPP
- CSharp
---
# IEnumDebugPropertyInfo2::Next

Returns the next set of elements from the enumeration.

## Syntax

### [C#](#tab/csharp)
```csharp
int Next(
   uint                  celt,
   DEBUG_PROPERTY_INFO[] rgelt,
   ref uint              pceltFetched
);
```
### [C++](#tab/cpp)
```cpp
HRESULT Next(
   ULONG                 celt,
   DEBUG_PROPERTY_INFO** rgelt,
   ULONG*                pceltFetched
);
```
---

## Parameters
`celt`\
[in] The number of elements to retrieve. Also specifies the maximum size of the `rgelt` array.

`rgelt`\
[in, out] Array of [DEBUG_PROPERTY_INFO](../../../extensibility/debugger/reference/debug-property-info.md) elements to be filled in.

`pceltFetched`\
[out] Returns the number of elements actually returned in `rgelt`.

## Return Value
 If successful, returns `S_OK`. Returns `S_FALSE` if fewer than the requested number of elements could be returned; otherwise, returns an error code.

## See also
- [IEnumDebugPropertyInfo2](../../../extensibility/debugger/reference/ienumdebugpropertyinfo2.md)
- [DEBUG_PROPERTY_INFO](../../../extensibility/debugger/reference/debug-property-info.md)
