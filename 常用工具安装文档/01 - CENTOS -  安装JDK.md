# 1. 下载jdk安装包
	jdk-8u92-linux-x64.tar.gz
	
# 2. 解压JDK

	tar zxvf jdk-8u92-linux-x64.tar.gz
	
# 3. 复制解压包到目录位置
	cp -r jdk1.8.0_92/ /usr/lib/java/

	
# 4. 添加环境变量

	在/etc/profile文件中添加以下内容:

	JAVA_HOME=/usr/lib/java/jdk1.8.0_92
	PATH=$JAVA_HOME/bin:$PATH

	export PATH JAVA_HOME
	
	
# 5. 刷新环境变量
	source /etc/profile

