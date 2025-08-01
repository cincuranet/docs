---
description: "Learn more about: IMetaDataImport::EnumMemberRefs Method"
title: "IMetaDataImport::EnumMemberRefs Method"
ms.date: "03/30/2017"
api_name:
  - "IMetaDataImport.EnumMemberRefs"
api_location:
  - "mscoree.dll"
api_type:
  - "COM"
f1_keywords:
  - "IMetaDataImport::EnumMemberRefs"
  - "IMetaDataImport::EnumMemberRefs method [.NET metadata]"
topic_type:
  - "apiref"
---
# IMetaDataImport::EnumMemberRefs Method

Enumerates MemberRef tokens representing members of the specified type.

## Syntax

```cpp
HRESULT EnumMemberRefs (
   [in, out] HCORENUM    *phEnum,
   [in]   mdToken        tkParent,
   [out]  mdMemberRef    rMemberRefs[],
   [in]   ULONG          cMax,
   [out]  ULONG          *pcTokens
);
```

## Parameters

 `phEnum`
 [in, out] A pointer to the enumerator.

 `tkParent`
 [in] A TypeDef, TypeRef, MethodDef, or ModuleRef token for the type whose members are to be enumerated.

 `rMemberRefs`
 [out] The array used to store MemberRef tokens.

 `cMax`
 [in] The maximum size of the `rMemberRefs` array.

 `pcTokens`
 [out] The actual number of MemberRef tokens returned in `rMemberRefs`.

## Return Value

| HRESULT | Description |
|-------------|-----------------|
| `S_OK` | `EnumMemberRefs` returned successfully. |
| `S_FALSE` | There are no MemberRef tokens to enumerate. In that case, `pcTokens` is to zero. |

## Requirements

 **Platforms:** See [.NET supported operating systems](https://github.com/dotnet/core/blob/main/os-lifecycle-policy.md).

 **Header:** Cor.h

 **Library:** CorGuids.lib

## See also

- [IMetaDataImport Interface](imetadataimport-interface.md)
- [IMetaDataImport2 Interface](imetadataimport2-interface.md)
