
proxy_cache_path /usr/local/openresty/nginx/emp_cache levels=1:2 keys_zone=tmp_cache:100m inactive=7d max_size=10g;