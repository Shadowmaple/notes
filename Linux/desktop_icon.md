# 建立桌面图标

桌面图标的文件是 `*.desktop`，在 `/usr/share/applications/`目录中

```shell
cd /usr/share/applications/
vi charles.desktop
```

编写 `.desktop`文件，以 `charles` 为例

```shell
[Desktop Entry]
Encoding=UTF-8
Name=Charles
Comment=Charles
# 文件启动位置
Exec=/home/lawler/software/charles-proxy-4.5.6_amd64/charles/bin/charles 
# 图标位置
Icon=/home/lawler/software/charles-proxy-4.5.6_amd64/charles/icon/128x128/apps/charles-proxy.png
Terminal=false
StartupNotify=false
Type=Application
Categories=Application;Development;
```

这样就 OK 了！