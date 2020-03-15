

nginx实际把http请求处理流程划分为了11个阶段，这样划分的原因是将请求的执行逻辑细分，以模块为单位进行处理，各个阶段可以包含任意多个http模块并以流水线的方式处理请求。这样做的好处是使处理过程更加灵活、降低耦合度。这11个http阶段如下所示：

1）ngx_http_post_read_phase：

接收到完整的http头部后处理的阶段，它位于uri重写之前，实际上很少有模块会注册在该阶段，默认的情况下，该阶段被跳过。

2）ngx_http_server_rewrite_phase：

uri与location匹配前，修改uri的阶段，用于重定向，也就是该阶段执行处于server块内，location块外的重写指令，在读取请求头的过程中nginx会根据host及端口找到对应的虚拟主机配置。

3）ngx_http_find_config_phase：

根据uri寻找匹配的location块配置项阶段，该阶段使用重写之后的uri来查找对应的location，值得注意的是该阶段可能会被执行多次，因为也可能有location级别的重写指令。

4）ngx_http_rewrite_phase：

上一阶段找到location块后再修改uri，location级别的uri重写阶段，该阶段执行location基本的重写指令，也可能会被执行多次。

5）ngx_http_post_rewrite_phase：

防止重写url后导致的死循环，location级别重写的后一阶段，用来检查上阶段是否有uri重写，并根据结果跳转到合适的阶段。

6）ngx_http_preaccess_phase：

下一阶段之前的准备，访问权限控制的前一阶段，该阶段在权限控制阶段之前，一般也用于访问控制，比如限制访问频率，链接数等。

7）ngx_http_access_phase：

让http模块判断是否允许这个请求进入nginx服务器，访问权限控制阶段，比如基于ip黑白名单的权限控制，基于用户名密码的权限控制等。

8）ngx_http_post_access_phase：

访问权限控制的后一阶段，该阶段根据权限控制阶段的执行结果进行相应处理，向用户发送拒绝服务的错误码，用来响应上一阶段的拒绝。

9）ngx_http_try_files_phase：

为访问静态文件资源而设置，try_files指令的处理阶段，如果没有配置try_files指令，则该阶段被跳过。

10）ngx_http_content_phase：

处理http请求内容的阶段，大部分http模块介入这个阶段，内容生成阶段，该阶段产生响应，并发送到客户端。

11）ngx_http_log_phase：

处理完请求后的日志记录阶段，该阶段记录访问日志。

以上11个阶段中，http无法介入的阶段有4个：

3）ngx_http_find_config_phase

5）ngx_http_post_rewrite_phase

8）ngx_http_post_access_phase

9）ngx_http_try_files_phase

剩余的7个阶段，http模块均能介入，每个阶段可介入模块的个数也是没有限制的，多个http模块可同时介入同一阶段并作用于同一请求。



# nginx lua插载点
+ init_by_lua：系统启动时调用
+ init_worker_by_lua：worker进程启动时调用
+ set_by_lua：nginx变量用复杂lua return
+ rewrite_by_lua：重写url规则
+ access_by_lua：权限验证阶段
+ content_by_lua：内容输出节点




使用最多的是context_by_lua内容输出节点