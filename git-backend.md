
### Nginx Config
```nginx
server {
    listen       8080;
    server_name  '172.16.1.220';

    #error_log  /home/linuxbrew/.linuxbrew/var/log/nginx/git/nginx-error.log;
    #access_log /home/linuxbrew/.linuxbrew/var/log/nginx/git/nginx-access.log;


    # static repo files for cloning over https
    location ~ ^.*\.git/objects/([0-9a-f]+/[0-9a-f]+|pack/pack-[0-9a-f]+.(pack|idx))$ {
        root /home/xu/.gitrepo/;
    }

    # requests that need to go to git-http-backend
    location ~ ^.*\.git/(HEAD|info/refs|objects/info/.*|git-(upload|receive)-pack)$ {
        root /home/xu/.gitrepo/;

        # Remove auth_* if you don't want HTTP Basic Auth
        auth_basic "example Git";
        auth_basic_user_file /home/linuxbrew/.linuxbrew/etc/nginx/.htpasswd;

        #fastcgi_pass  unix:/var/run/fcgiwrap.socket;
        fastcgi_pass  localhost:9001; 
        fastcgi_param SCRIPT_FILENAME   /home/linuxbrew/.linuxbrew/Cellar/git/2.34.0/libexec/git-core/git-http-backend;
        fastcgi_param PATH_INFO         $uri;
        fastcgi_param GIT_PROJECT_ROOT  $document_root;
        fastcgi_param GIT_HTTP_EXPORT_ALL "";
        fastcgi_param REMOTE_USER $remote_user;
        include fastcgi_params;
    }
}
```

### Create the Password File Using the OpenSSL Utilities
```bash
sudo sh -c "echo -n 'sammy:' >> /etc/nginx/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```
