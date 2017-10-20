---
layout: post
---

[TOC]

## Basic Type

* _Integer_: `i8 i16 i32 i64` `u8 u16 u32 u64`
* _Floating_: `f32 f64`
* _Boolean_: `b`
* _Bit_: `bit`
* _Byte_: `bye`
* _Character_: `c`
* _String_: `s`
* _Pointer_: type`*`
* _Reference_: type`&`

## Complex Type

* _Array_: `[`length?`]`vtype
* _Map_: `[`ktype`]`vtype

## Custom Type

* _Function/Method_
```
pkg.name (
    in param_name param_type
    out return_name return_type
)
```

* _Struct/Class/Interface_
```
pkg.Name {
    anonymous_custom_type
    field_name field_type
    method_name (in param_name param_type, out return_name return_type)
}
```

## Example
```
```