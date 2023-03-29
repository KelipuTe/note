## beego框架

- 2021-11-17
- go-1.17
- bee-v1.12.3
- beego-v1.12.3

### 安装bee工具

bee工具是一个为了协助快速开发beego项目而创建的项目。

通过bee，可以很容易的进行beego项目的创建、热编译、开发、测试、和部署。

```
> go get github.com/beego/bee
go: downloading github.com/beego/bee v1.12.3
go: downloading gopkg.in/yaml.v2 v2.3.0
go: downloading github.com/fsnotify/fsnotify v1.4.9
go: downloading github.com/gadelkareem/delve v1.4.2-0.20200619175259-dcd01330766f
go: downloading github.com/gorilla/websocket v1.4.2
go: downloading github.com/lib/pq v1.7.0
go: downloading github.com/davecgh/go-spew v1.1.1
go: downloading github.com/flosch/pongo2 v0.0.0-20200529170236-5abacdfa4915
go: downloading github.com/pelletier/go-toml v1.7.0
go: downloading github.com/smartwalle/pongo2render v1.0.1
go: downloading github.com/spf13/viper v1.7.0
go: downloading github.com/astaxie/beego v1.12.1
go: downloading golang.org/x/sys v0.0.0-20191005200804-aed5e4c7ecf9
go: downloading github.com/hashicorp/hcl v1.0.0
go: downloading github.com/magiconair/properties v1.8.1
go: downloading github.com/mitchellh/mapstructure v1.1.2
go: downloading github.com/spf13/afero v1.1.2
go: downloading github.com/spf13/cast v1.3.0
go: downloading github.com/spf13/jwalterweatherman v1.0.0
go: downloading github.com/spf13/pflag v1.0.3
go: downloading github.com/subosito/gotenv v1.2.0
go: downloading gopkg.in/ini.v1 v1.51.0
go: downloading go.starlark.net v0.0.0-20190702223751-32f345186213
go get: installing executables with 'go get' in module mode is deprecated.
        To adjust and download dependencies of the current module, use 'go get -d'.
        To install using requirements of the current module, use 'go install'.
        To install ignoring the current module, use 'go install' with a version,
        like 'go install example.com/cmd@latest'.
        For more information, see https://golang.org/doc/go-get-install-deprecation
        or run 'go help get' or 'go help install'.
go get: added github.com/beego/bee v1.12.3
```

### 安装beego

```
> go get github.com/astaxie/beego
go: downloading github.com/astaxie/beego v1.12.3
go: downloading golang.org/x/net v0.0.0-20190620200207-3b0461eec859
go: downloading github.com/prometheus/procfs v0.1.3
go: downloading golang.org/x/sys v0.0.0-20200615200032-f1bc736245b1
go: downloading github.com/cespare/xxhash v1.1.0
go: downloading github.com/peterh/liner v1.0.1-0.20171122030339-3681c2a91233
go get: upgraded github.com/astaxie/beego v1.12.1 => v1.12.3
```