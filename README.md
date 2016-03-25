# Ant-Multiple-Package
Ant多渠道打包说明文档

一 文档说明

   本项目是针对android在eclipse开发环境下利用android官方自带的ant打包工具进行多渠道打包的一个工具，具有打包快速配置简单的特点。
   
二 环境准备

   需要正确配置java环境，ANDROID_HOME开发环境，ANT_HOME开发环境。
   
三 使用说明

1 使用前期事项

	将ant,auto, build,custom_rules，log4j文件放到项目文件目录下。
	 文件说明：ant文件中
	 key.store=加密文件
        key.alias=alias名称
        key.store.password=加密密码
        key.alias.password=alias密码
        umeng_channels=baidu,google,umeng  //渠道包，用逗号分隔
        output.apk.dir=F:\\output_apk   //打包文件输出路径，这个可设可不设。
        Log4j是日志输出文件
        build是ant基本配置文件
        custom_rules是项目主要工作文件
	
2 在使用前先在项目目录中执行官方命令打包

 	android update project -p . -s -t "android-21"
 	然后执行
	ant debug
	如果能够正常运行则可以用自动化打包。
	在正式运行前建议执行命令
	ant auto-copy-workspace
	将当前目录复制到临时工作目录"项目名称" + Temp 下面。
	然后转换工作目录到临时工作目录下面。
	ant auto-debug
	打测试包可以在bin目录下看到生成的测试包。
	执行命令
	ant auto-release
	则批量打包umeng_channels字段下的渠道包.
        执行命令
	ant auto-release –DUMENG_CHANNEL=umeng –Dpackage=com.package.test –Dversion=time 
	更换渠道包和包名以及版本号。
	最好在bin文件夹下可以找到打包后的文件：
	{ant.project.name}-${channelname}-signed.apk
	
四 多渠道打包中的注意点

       1 必须将ant-contrib-1.0b3.jar放到${env.ANT_HOME}/lib/目录下。
       2 <property name="sdk-tool-dir" value="${sdk.dir}/build-tools/19.1.0"/>
         需要根据当前的android版本去修改sdk-tool-dir的路径，我的是19.1.0.
       3 如果在编译的时候使用了第三方库，但不需要一起打包出去的时候可以这么可以修改-pre-comlile.在比如这里自定义baidulib.lib,引用了三   个jar库，然后再将baidulib.lib引入到userlibrary.lib。而userlibrary.lib本身是不存在的。因此需要修改文件${sdk.dir}/tools/ant/build.xml。
        添加<path id="userlibrary.lib"/>
    并在<target name="-compile" depends="-pre-build, -build-setup, -code-gen, -pre-compile"> 
        <do-only-if-manifest-hasCode elseText="hasCode = false. Skipping...">
            <!-- merge the project's own classpath and the tested project's classpath -->
            <path id="project.javac.classpath">
                <path refid="project.all.jars.path" />
                <path refid="tested.project.classpath" />
                <path refid="userlibrary.lib" />
                <path path="${java.compiler.classpath}" />
            </path>
      做修改。
    定义第三方编译依赖库，但不导出。
		<target name="-pre-compile">
				<echo message="JARPATH=${toString:project.all.jars.path}"></echo>
				      <property name="baidulib.dir" value="D:/library_project"> </property>
						 <path id="baidulib.lib">
				        <pathelement location="${baidulib.dir}/zxing.jar"></pathelement>
				        <pathelement location="${baidulib.dir}/fastjson-1.2.3.jar"></pathelement>
				        <pathelement location="${baidulib.dir}/pushservice-4.5.1.8.jar"></pathelement>
				    </path>
				    <path id="userlibrary.lib">
				    	<path refid="baidulib.lib"/>
				    </path>
				    <echo message="JARPATH=${toString:baidulib.lib}"></echo>
		    </target>
    对于这个处理办法有更好的可以联系我。Android自带编译工具是能够导入userlibrary库的，但我用ant不知道如何实现。
    4 如果项目引入了library,运行前需要将library bin/res/crunch文件删除。
