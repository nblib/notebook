# 概述
* 共享内存在多个进程中共享.
# 使用方法
1. 在`nginx.conf`中定义:
    ``` 
    lua_shared_dict blacklist_list 5m;
    ```
2. 代码中使用:
    ``` 
    local sharedDict = ngx.shared.blacklist_list
    if not sharedDict then
        ngx.log(ngx.ERR,"[blacklist-plugin]not set shared dic [blacklist_list]")
        return false
    end
    -- 根据isNew子段是否存在判断是否需要更新,存在不需要
    local isNew = sharedDict:get("isNew")
    ```
    * 注意调用方法使用`:`