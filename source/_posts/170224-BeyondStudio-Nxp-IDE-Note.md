---
title: BeyondStudio Nxp IDE Note
toc: true
date: 2017-02-24 21:31:37
categories: Nxp
tags: BeyondStudio
---


# 简述
最近研究JN5168，需要用到NXP官方提供的开发环境，使用中遇到的小东小西就纪录在这里。

# 问题
## Type 'uint8' could not be resolved
参考官方文档JN-UG-3098，page 25：  
Step 4 Configure the workspace preferences as follows:  
* Open the Preferences dialogue box by following the menu path Window>Preferences.
* In the left tree of the Preferences dialogue box, open the C/C++ entry and click on the Indexer sub-entry. The right side of the dialogue box is now populated with the Indexer options.
* In the section Build configuration for the indexer, select the radio-button Use active build configuration.
* Click Apply and then OK.

问题解决。每次新建workspace都需要配置。
<!---more--->
### 有时候执行完上面的操作后仍然有问题
[LittleBoy's Blog](http://blog.163.com/rainsmell_/blog/static/212827113201431605936633/)找到如下解决办法：  
``` 
也不知道这算不算是一个bug，即便是添加了所有object所依赖的head files，include path也完整，依然会出现这个问题。refresh工程，重启Eclipse也无济于事。
Google了一番,终于在stackoverflow里找到了解决办法：  
http://stackoverflow.com/questions/10041453/eclipse-c-type-could-not-be-resolved-error-even-though-build-is-successful  
记录一下，Project -> C/C++ index -> Freshen All Files，问题解决。
```

## workspace
自定义workspace，不能编译问题。  
原因:  
`SDK_BASE_DIR`变量没有正确获取SDK路径。abspath函数返回的是当前路径的绝对路径，也就是workspace的绝对路径。
解决：  
重新安装sdk，安装路径设置为workspace所在路径。例如：  
1. workspace路径为：d:/workspace/nxp-workspace
2. 安装路径应设置为：d:/workspace

使用中，先执行clean project，然后再编译

## import工程后文件夹为空
问题：  
导入`JN-AN-1229.zip`工程时，导入后项目文件夹下只有两个文件`.cproject``.project`。当然也无法编译了，怎么办呢？  
解决：  
* 将`JN-AN-1229.zip`解压到`当前`文件夹;
* 从文件夹中导入工程时，把options->Copy projects into workspace 选项取消；
* 导入后工程的目录里面显示的链接标志，不过可以编译了。

原因：  
暂时不清楚。

后来遇到上述办法不能解决的项目`JN-AN-1217.zip`。经过一番研究最后发现`JN-AN-1217-Zigbee-3-0-Base-Device-v1005\JN-AN-1217-Zigbee-3-0-Base-Device\JN516x`这个目录下有个`.project`文件,文件最后有代码：
```
	<linkedResources>
		<link>
			<name>Common</name>
			<type>2</type>
			<location>C:/NXP/bstudio_nxp/workspace/JN-AN-1217-Zigbee-3-0-Base-Device/Common</location>
		</link>
		<link>
			<name>Coordinator</name>
			<type>2</type>
			<location>C:/NXP/bstudio_nxp/workspace/JN-AN-1217-Zigbee-3-0-Base-Device/Coordinator</location>
		</link>
		<link>
			<name>EndDevice</name>
			<type>2</type>
			<location>C:/NXP/bstudio_nxp/workspace/JN-AN-1217-Zigbee-3-0-Base-Device/EndDevice</location>
		</link>
		<link>
			<name>Router</name>
			<type>2</type>
			<location>C:/NXP/bstudio_nxp/workspace/JN-AN-1217-Zigbee-3-0-Base-Device/Router</location>
		</link>
	</linkedResources>
```
意思很直白了。那么其他工程是怎么配置的呢
```
	<linkedResources>
		<link>
			<name>Common</name>
			<type>2</type>
			<locationURI>PARENT-1-PROJECT_LOC/Common</locationURI>
		</link>
		<link>
			<name>Coordinator</name>
			<type>2</type>
			<locationURI>PARENT-1-PROJECT_LOC/Coordinator</locationURI>
		</link>
		<link>
			<name>EndDevice</name>
			<type>2</type>
			<locationURI>PARENT-1-PROJECT_LOC/EndDevice</locationURI>
		</link>
		<link>
			<name>Router</name>
			<type>2</type>
			<locationURI>PARENT-1-PROJECT_LOC/Router</locationURI>
		</link>
	</linkedResources>
```
明白了吧，放到C盘才行，😄。

## include文件不够
问题：  
缺少`C:\NXP\bstudio_nxp\sdk\JN-SW-4170\Components\ZigbeeCommon\Include`

解决： 
在`项目右键->properties -> general -> paths and symbols -> include` 点击add，添加如上路径即可。


## app.zpscfg
app.zpscfg如何打开呢？其实`JN-UG-3098 Beyond Studio for NXP`文件中已经进行了详细描述。文件的`1.2.3Installing the ZigBee Plug-ins`章节有着详细的描述。需要在eclipse环境下安装2个插件分别是`ZPS Configuration Editor`和`JenOS Configuration Editor`,具体请参考文档[JN-UG-3098](https://www.nxp.com/docs/en/user-guide/JN-UG-3098.pdf)


## 不喜欢IDE，如何用命令行进行编译？
其实demo文档中是有介绍的，以JN-AN-1217-Zigbee-3-0-Base-Device.pdf为例。文件的5.7.2.1介绍的就是用makefile编译的方法。摘抄如下：
1. Ensure that the project directory is located in
	<IDE installation root>\workspace
2. Start an MSYS shell by following the Windows Start menu path: All Programs > NXP > MSYS Shell
3. Navigate to the Build directory for the application to be built and at the command prompt enter an appropriate make command for your chip type, as illustrated below.
For example, for JN5169:
make JENNIC_CHIP_FAMILY=JN516x JENNIC_CHIP=JN5169 clean all
The binary file will be created in the Build directory, the resulting filename indicating the chip type (e.g. 5169) for which the application was built.
4. Load the resulting binary file into the board. You can do this from the command line using the JN51xx Production Flash Programmer, as described in Section 4.1.






