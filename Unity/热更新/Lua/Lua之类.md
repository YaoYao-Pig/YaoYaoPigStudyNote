

## Lua多态

```lua
Object={}
Object.i=1
function Object:new()
    local object={}
    self.__index=self
    setmetatable(object,self)
    return object
end
Object.x=0
Object.y=0
function Object:Move()
    self.x=self.x+1
    self.y=self.y+1
    print(self.x.." "..self.y)
end

function Object:subClass(subClassName)
    _G[subClassName]={}
    local sub=_G[subClassName]
    self.__index=self
    setmetatable(sub,self)
end



-- local o1=Object:new()
-- print(o1.i)
-- o1.i=2
-- print(o1.i)
Object:subClass("GameObject")
local go=GameObject:new()
local o1=Object:new()
go:Move()

```

> 我的问题是这样的，当我调用go:Move（）的时候，    self.x=self.x+1发生了什么？或者说，子类是如何利用到基类的变量的
>
> 我的理解是：因为go当中没有等号右边的self.x,所以从元表里__index指向的查找，找到了基类Object里的x=0，然后加一，因此等号右侧等于1。然后赋值的时候，因为元表的__newIndex没有赋值，因此会在 **`go` 实例中新增字段**中（而不是GameObject类，仔细体会self，它是调用函数时“传入”的那个表）创建一个新的x，然后把1赋值给GameObject里的x



多态注意：

```lua
-- function GameObject:Move()
--     self.base:Move()
-- end

function GameObject:Move()
    self.base.Move(self)
end
```

第一种写法，如果有多个子类对象，会重复使用基类的变量