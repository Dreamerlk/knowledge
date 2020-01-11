```
server {
	##商铺##
	listen ip:80;
        server_name *.cn.XXX.com;
	include allow.conf;
	if ($host ~* (.*).cn.XXX.com) {
                        set $domain $1;                                                                                                
        }
	set $mobileRewrite "0";
	if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|dolphin|dolphin-browser|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows (ce|phone)|xda|xiino") {
		set $mobileRewrite "1";
	}
	if ($http_cookie ~* "showpc"){
	    set $mobileRewrite "0";
	}
	if ($mobileRewrite = "1") {
		rewrite ^/(.*) https://m.XXX.com/shop/$domain/$1 redirect;
	}
	#强制https
	rewrite ^(.*)$ https://$domain.cn.XXX.com$1 permanent;

	location / {
		proxy_cache my-cache;
		proxy_cache_valid 200 206 304 301 302 10d;
		proxy_cache_key $host$uri$is_args$args;
		proxy_set_header Host $host:$server_port;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		add_header Nginx-Cache $upstream_cache_status;
		proxy_pass http://cn.XXX.com;
        }
    #access_log  /home/nginx/logs/access.log logstash_json;
}
```
