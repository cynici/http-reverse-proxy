# HTTP reverse proxy

This provides HTTP(S) reverse proxy service using Apache that is capable of HTML response body rewriting.

HAProxy is not designed to rewrite HTML contents while NGINX [http_sub_module](http://nginx.org/en/docs/http/ngx_http_sub_module.html) is not built-in and requires compilation from source.

## 2016-07-29 Update

Added *jessie* branch because required packages not available in *Debian testing*.

## Usage

### Apache configuration

Create a Apache configuration file e.g. reverse_proxy.conf:

```
ServerName foobar.example.com

# http://apache.webthing.com/svn/apache/filters/proxy_html/proxy_html.conf
ProxyHTMLLinks	a		href
ProxyHTMLLinks	area		href
ProxyHTMLLinks	link		href
ProxyHTMLLinks	img		src longdesc usemap
ProxyHTMLLinks	object		classid codebase data usemap
ProxyHTMLLinks	q		cite
ProxyHTMLLinks	blockquote	cite
ProxyHTMLLinks	ins		cite
ProxyHTMLLinks	del		cite
ProxyHTMLLinks	form		action
ProxyHTMLLinks	input		src usemap
ProxyHTMLLinks	head		profile
ProxyHTMLLinks	base		href
ProxyHTMLLinks	script		src for

# To support scripting events (with ProxyHTMLExtended On),
# you'll need to declare them too.

ProxyHTMLEvents	onclick ondblclick onmousedown onmouseup \
		onmouseover onmousemove onmouseout onkeypress \
		onkeydown onkeyup onfocus onblur onload \
		onunload onsubmit onreset onselect onchange

<VirtualHost *:*>
    AddDefaultCharset utf-8
    ProxyPreserveHost On

    ProxyRequests Off
    ProxyPass /webapp1/ http://{internal_ip}:{internal_port}/
    ProxyPassReverse /webapp1/ http://{internal_ip}:{internal_port}/
    <Location /webapp1/>
        ProxyPassReverse /

        # The following two directives cause error 'AH01431: Got charset utf-8 from HTTP headers'
	#ProxyHTMLEnable On
        #ProxyHTMLExtended On
        SetOutputFilter INFLATE;proxy-html;DEFLATE 

        ProxyHTMLURLMap http://{internal_ip}:{internal_port}/ /webapp1/
	ProxyHTMLURLMap / /webapp1/
        LogLevel Info
    </Location>
</VirtualHost>
```

### docker-compose.yml

```
version: '2'
services:
  apache2:
    build: .
    ports:
    - "80:80"
    volumes:
    - ./reverse_proxy.conf:/etc/apache2/sites-enabled/reverse_proxy.conf:ro
    command:
    - /usr/sbin/apachectl
    - -DFOREGROUND
```

### Build image

```
docker-compose build
```

### Run image as a daemon

```
docker-compose up -d
```
