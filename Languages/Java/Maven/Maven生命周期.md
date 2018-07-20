# Maven生命周期

Maven有三套独立的生命周期:

- Clean Lifecycle：在进行真正的构建之前进行一些清理工作
- Default Lifecycle：构建的核心部分，编译、测试、打包、部署
- Site Lifecycle：生成项目报告、生成站点、发布站点

maven的三套生命周期是相互独立的,每套生命周期都由一组阶段组成,在一个生命周期中运行某个阶段的时候,它之前的所有阶段都会被运行

## Clean Lifecycle

- pre-clean 执行一些需要在clean之前完成的工作 
- clean 移除所有上一次构建生成的文件 
- post-clean 执行一些需要在clean之后立刻完成的工作 

## Default Lifecycle

- validate 
- generate-sources 
- process-sources 
- generate-resources 
- process-resources 复制并处理资源文件，至目标目录，准备打包。
- compile 编译项目的源代码。 
- process-classes 
- generate-test-sources 
- process-test-sources 
- generate-test-resources 
- process-test-resources 复制并处理资源文件，至目标测试目录。
- test-compile 编译测试源代码。 
- process-test-classes 
- test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。 
- prepare-package 
- package 接受编译好的代码，打包成可发布的格式，如 JAR 。 
- pre-integration-test 
- integration-test 
- post-integration-test 
- verify 
- install 将包安装至本地仓库，以让其它项目依赖。 
- deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享。

## Site Lifecycle

- pre-site 执行一些需要在生成站点文档之前完成的工作
- site 生成项目的站点文档 
- post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
- site-deploy 将生成的站点文档部署到特定的服务器上

