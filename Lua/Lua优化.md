# 优化

## 1. 不要多重拼接字符串

由于字符串的不可变性，所以在多重拼接的时候，会创建多个中间的字符串变量存在字符串池里，造成内存的压力

## 2. 多用local

尤其是在循环当中，减少查表，穿梭的次数，用local缓存。同时，也可以提高可读性

## 3. 设计local的生命周期(慎用table)

在循环高频的代码里面，local变量的生命周期要注意，尤其是引用的表，创建一个生命周期过短的表，会引发gc。尤其这种行为发生在循环里

```lua
-- 错误
for k, v in configList do
    local config = {
        name = v.name,
        age = v.age
    }
    func（confg）
 end
-- 正确1

for k, v in configList do
   
    func（v.name,v.age）
 end
-- 正确2
local config  = {}
for k, v in configList do
    config.name = v.name
    config,age = v.age
     func（config）
 end
```

