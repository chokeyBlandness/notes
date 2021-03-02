# maven打包遇到的问题

### 一：maven打包时将依赖jar一起打包

通过jar-with-dependencies，使用maven-assembly-plugin对package命令进行功能绑定。

~~~xml
<!-- Maven Assembly Plugin -->
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-assembly-plugin</artifactId>
	<version>3.1.1</version>
	<configuration>
		<descriptorRefs>
			<!-- add all project dependencies to jar -->
			<descriptorRef>jar-with-dependencies</descriptorRef>
		</descriptorRefs>
		<!-- MainClass in mainfest make a executable jar -->
		<archive>
			<manifest>
				<!-- your main class -->
				<mainClass>com.XXX.XXX.MainClass</mainClass>
			</manifest>
		</archive>
	</configuration>
	<executions>
		<execution>
			<id>make-assembly</id>
			<!-- bind to the packaging phase -->
			<phase>package</phase>
			<goals>
				<goal>single</goal>
			</goals>
		</execution>
	</executions>
</plugin>
~~~

### 二：maven打包可运行的jar包

转载自 https://blog.csdn.net/wqc19920906/article/details/79257402

maven可以使用`maven package`指令对项目进行打包操作。如果未进行任何打包配置，使用`java -jar XXX.jar`指令运行jar包时，可能出现 “no main manifest attribute, in xxx.jar” 等问题。

#### 方法一： **使用maven-jar-plugin和maven-dependency-plugin插件打包**

在pom.xml中配置

~~~xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-jar-plugin</artifactId>  
            <version>2.6</version>  
            <configuration>  
                <archive>  
                    <manifest>  
                        <addClasspath>true</addClasspath>  
                        <classpathPrefix>lib/</classpathPrefix>  
                        <mainClass>com.wqc.main.SpringStart</mainClass>  
                    </manifest>  
                </archive>  
            </configuration>  
        </plugin>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-dependency-plugin</artifactId>  
            <version>2.10</version>  
            <executions>  
                <execution>  
                    <id>copy-dependencies</id>  
                    <phase>package</phase>  
                    <goals>  
                        <goal>copy-dependencies</goal>  
                    </goals>  
                    <configuration>  
                        <outputDirectory>${project.build.directory}/lib</outputDirectory> 
                    </configuration>  
                </execution>  
            </executions>  
        </plugin>  
    </plugins>  
</build>
~~~

#### 方法二：使用maven-assembly-plugin插件打包

在pom.xml中配置

~~~xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-assembly-plugin</artifactId>  
            <version>2.5.5</version>  
            <configuration>  
                <archive>  
                    <manifest>  
                        <mainClass>com.wqc.main.SpringStart</mainClass>  
                    </manifest>  
                </archive>  
                <descriptorRefs>  
                    <descriptorRef>jar-with-dependencies</descriptorRef>  
                </descriptorRefs>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>
~~~

