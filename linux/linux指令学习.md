##### 移动文件夹到另外的地方

~~~shell
mv /home/packageA /home/packageB/
~~~

##### tar解压

~~~shell
tar -zxvf a.tar.gz
~~~

##### 复制文件夹到另外的地方

~~~shell
cp -r /home/packageA/* /home/cp/packageB/
~~~

##### 为文件赋权限

语法：

~~~shell
chmod abc file
~~~

abc各为一个数字，分别表示user、group、other的权限。

r=4，w=2，x=1

- 若要rwx则为7
- 若要rw-则为6
- 若要r-x则为5

示例：

~~~shell
chmod 777 file
~~~



