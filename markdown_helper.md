## NGINX
services/markdown
```nginx
server {
    listen       8080;
	server_name  172.16.1.220;

    error_log  /home/linuxbrew/.linuxbrew/var/log/nginx/md-error.log;
    access_log /home/linuxbrew/.linuxbrew/var/log/nginx/md-access.log;


	location ~ '(.*)\.md' {
		root          /mnt/hgfs/local/md/output;
        include       fastcgi_params;
        fastcgi_pass  127.0.0.1:9001; 
        fastcgi_param SCRIPT_FILENAME   /home/xu/mdhtml;
        fastcgi_param GIT_PROJECT_ROOT  $document_root;
        fastcgi_param MD_FILE_NAME		$1;
	}
	
	location ~ '.*\.css$' {
		root   /mnt/hgfs/local/md/output;
	}
}
```

## fcgi SHELL
mdhtml
```bash
#!/bin/sh
# -*- coding: utf-8 -*-

input=/mnt/hgfs/local/md/input$MD_FILE_NAME.md
output=/mnt/hgfs/local/md/input$MD_FILE_NAME.html
test -f $output || generate-md --layout github --input /mnt/hgfs/local/md/input --output /mnt/hgfs/local/md/output >> /dev/null 2>&1
test $input -nt $output && generate-md --layout github --input /mnt/hgfs/local/md/input --output /mnt/hgfs/local/md/output >> /dev/null 2>&1
echo -e "X-Tool: generate-md\r\n"
cat /mnt/hgfs/local/md/output$MD_FILE_NAME.html

```

## Sublime Plugin
mdviewer.py
```python
import sublime
import sublime_plugin
import os
import webbrowser


class BuildMarkdownCommand(sublime_plugin.WindowCommand):
	def run(self, lint=False, integration=False, kill=False):
		vars = self.window.extract_variables()
		file_name = vars['file_name']
		webbrowser.open('http://172.16.1.220:8080/'+file_name)
```

## Sublime Build
Markdown.dublime-build
```jsonc
{
    "selector": "source.md",
    "file_patterns": ["*.md"],
    "target": "build_markdown"
}
```
