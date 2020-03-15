[TOC]

# 概览
shared dict：共享内存字典，所有worker进程可见，lru淘汰


# 配置
nginx.conf
```
lua_shared_dict my_cache 128m;
```
shareddic.lua
```lua
// 从nginx的cache通过key获取value
function get_from_cache(key)
    // 这个local变量取自nginx的shared.my_cache
    local cache_ngx = ngx.shared.my_cache
    // lua引用使用:
    local value = cache_ngx:get(key)
    return value
end

function set_to_cache(key,value,exptime)
    if not exptime then
        exptime = 0
    end
    local cache_ngx = ngx.shared.my_cache
    // 元组
    local succ,err,forcible = cache_ngx:set(key,value,exptime)
    return succ
end

// 获取uri上的参数
local args = ngx.req.get_uri_args()
local id = args["id"]
local item_model = get_from_cache("item"..id)// ..是拼接
if item_model == nil then
    local resp = ngx.location.capture("/item/get?id="..id)
    item_model = resp.body
    // 缓存1分钟
    set_to_cache("item"..id,item_model,1*60)
end
ngx.say(item_model)
```
vim nginx.conf
```
location /luaitem/get{
    default_type "application/json";
    content_by_lua_file ../lua/sharedict.lua;
}
```
cat -f access.log

这种机制比proxyCache强

使用这种方式，可以免除一次tomcat网络rpc的调用，但是将负载的压力移到了nginx上面，并且对应的一个缓存容量大小，一个缓存的回援机制都是还需要有个短时间的脏读

优势：基于nginx内存的直接的缓存，离用户最近的节点
劣势：更新机制，容量问题