#go源代码根目录结构

|– AUTHORS — 文件，官方 Go语言作者列表

|– CONTRIBUTORS — 文件，第三方贡献者列表

|– LICENSE — 文件，Go语言发布授权协议

|– PATENTS — 文件，专利

|– README — 文件，README文件，大家懂的。提一下，经常有人说：Go官网打不开啊，怎么办？其实，在README中说到了这个。该文件还提到，如果通过二进制安装，需要设置GOROOT环境变量；如果你将Go放在了/usr/local/go中，则可以不设置该环境变量（Windows下是C:\go）。当然，建议不管什么时候都设置GOROOT。另外，确保$GOROOT/bin在PATH目录中。

|– VERSION — 文件，当前Go版本

|– api — 目录，包含所有API列表，方便IDE使用

|– doc — 目录，Go语言的各种文档，官网上有的，这里基本会有，这也就是为什么说可以本地搭建”官网”。这里面有不少其他资源，比如gopher图标之类的。

|– favicon.ico — 文件，官网logo

|– include — 目录，Go 基本工具依赖的库的头文件

|– lib — 目录，文档模板

|– misc — 目录，其他的一些工具，相当于大杂烩，大部分是各种编辑器的Go语言支持，还有cgo的例子等

|– robots.txt — 文件，搜索引擎robots文件

|– src — 目录，Go语言源码：基本工具（编译器等）、标准库

|– test — 目录，包含很多测试程序（并非_test.go方式的单元测试，而是包含main包的测试），包括一些fixbug测试。可以通过这个学到一些特性的使用。