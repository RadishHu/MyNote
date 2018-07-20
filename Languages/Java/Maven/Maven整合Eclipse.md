# Maven整合Eclipse

- 确认eclipse知否安装maven的插件

  菜单栏Window ---> Preferences , 当Preferences窗口中存在Maven选项时, 说明eclipse中已经安装了maven插件

- maven 与 eclipse整合

  Preferences ---> Maven ---> Installations ---> Add ---> Directory 选择本地Maven所在的目录, 然后保存

- 选择配置文件

  Preferences ---> Maven ---> User Settings ---> GlobalSettings ---> Browse 选择conf\settings.xml文件, 然后保存

- 在eclipse中显示maven仓库

  Windows ---> Show View ---> Other  搜索Maven Repositories

  在MavenRepositories中右击Local Repositories,Rebuild Index