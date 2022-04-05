# Jenkins+Github+Springboot实现自动部署配置

Springboot项目部署到服务器上之后，开发人员每次改动代码都需要在本地打jar包，然后把jar包放到服务器上，再启动服务。通常在前后端联调阶段，代码的改动是必不可少的，每次修改都要重复上述步骤，大大降低了开发效率。为了减少不必要的重复劳动，我们可以通过Jenkins实现自动部署，开发人员只需要在本地编写完代码，push到远程仓库即可，剩下的编译、打包、部署等工作均由Jenkins完成。Jenkins工作原理是**当远程仓库代码更新后，仓库会给Jenkins发送一个请求，Jenkins自动拉取远程仓库的代码，并通过开发人员预先设置好的指令执行编译打包等工作，然后将jar包发送到远程服务器，执行开发人员预先提供的shell脚本，完成服务的部署。**

我这里是Springboot项目，通过Github进行源码管理，Maven完成打包工作，部署在阿里云的轻量应用服务器上。

自动部署配置步骤如下：

- 配置Jenkins
- 配置Github
- 自动部署测试

## 配置Jenkins

### 1、安装插件

*不会安装插件的可以百度一下噢*

1. GitHub Plugin
2. Publish over SSH

### 2、配置GitHub Plugin

1. 在Jenkins系统配置中找到GitHub 

   ![](https://quantacenter.oss-cn-shenzhen.aliyuncs.com/font/image-20220129032711274.png)

2. 点击添加Github服务器（默认是空的，我这里添加过所以有一个）

3. 名称任意填，API URL填 **https://api.github.com**

4. 添加凭据（默认也为空）

   1. 点击添加

      ![image-20220129033110890](https://gitee.com/yan-shipeng/images/raw/master/image-20220129033110890.png)

   2. 选择Secret text

      ![image-20220129033217038](https://gitee.com/yan-shipeng/images/raw/master/image-20220129033217038.png)

   3. Secret需要在Github中生成，ID不用填，描述随意（我的叫做github-secret-text）

      ![image-20220129033630849](https://gitee.com/yan-shipeng/images/raw/master/image-20220129033630849.png)

   4. 生成Secret，进入Github的Settings

      ![image-20220129033942513](https://gitee.com/yan-shipeng/images/raw/master/image-20220129033942513.png)

   5. 点击Developer settings

      ![image-20220129034021808](https://gitee.com/yan-shipeng/images/raw/master/image-20220129034021808.png)

   6. 点击Personal access tokens

      ![image-20220129034105279](https://gitee.com/yan-shipeng/images/raw/master/image-20220129034105279.png)

   7. 点击Generate new token，然后验证一下密码

      ![image-20220129034440489](https://gitee.com/yan-shipeng/images/raw/master/image-20220129034440489.png)

   8. 生成token

      ![image-20220129034620936](https://gitee.com/yan-shipeng/images/raw/master/image-20220129034620936.png)

      ![image-20220129034645339](https://gitee.com/yan-shipeng/images/raw/master/image-20220129034645339.png)

   9. 复制或者拍照保存，token只显示这一次

      ![image-20220129034750410](https://gitee.com/yan-shipeng/images/raw/master/image-20220129034750410.png)

   10. 将token复制到jenkins中，点击添加

       ![image-20220129034919153](https://gitee.com/yan-shipeng/images/raw/master/image-20220129034919153.png)

   11. 选择刚刚添加的凭据，点击连接测试，如下代表成功（失败会报红，尝试重新添加凭据）

       ![image-20220129035100518](https://gitee.com/yan-shipeng/images/raw/master/image-20220129035100518.png)

### 3、配置Publish over SSH

1. 在Jenkins系统配置中找到Publish over SSH，在passphrase中填入服务器密码（我填的是root用户的密码），Path to key 和 Key 可以不填

   ![image-20220129143551728](https://gitee.com/yan-shipeng/images/raw/master/image-20220129143551728.png)

2. 点击新增，填写以下信息

   ![image-20220129144138948](https://gitee.com/yan-shipeng/images/raw/master/image-20220129144138948.png)

3. 点击应用，保存即可

## 配置Github

### 1、配置项目仓库

1. 进入你的项目仓库

   ![image-20220129171744991](https://gitee.com/yan-shipeng/images/raw/master/image-20220129171744991.png)

2. 点击Settings，然后点击侧边栏的Webhooks，点击添加，然后验证一下密码

   ![image-20220129172006036](https://gitee.com/yan-shipeng/images/raw/master/image-20220129172006036.png)

3. 配置Webhook（URL只需要改动ip和端口号即可，github-webhook不用改动，我在这被小坑了一手）

   ![image-20220129172406618](https://gitee.com/yan-shipeng/images/raw/master/image-20220129172406618.png)

## 创建测试项目

### 1、在Jenkins中创建测试项目

1. 新建一个Maven风格的项目

   ![image-20220129164602790](https://gitee.com/yan-shipeng/images/raw/master/image-20220129164602790.png)

2. 填写配置（填完后应用保存即可）

   ![image-20220129165000907](https://gitee.com/yan-shipeng/images/raw/master/image-20220129165000907.png)

   ![image-20220129165113427](https://gitee.com/yan-shipeng/images/raw/master/image-20220129165113427.png)

   ![image-20220129165209863](https://gitee.com/yan-shipeng/images/raw/master/image-20220129165209863.png)

   ![image-20220129165711977](https://gitee.com/yan-shipeng/images/raw/master/image-20220129165711977.png)

   ![image-20220129165840002](https://gitee.com/yan-shipeng/images/raw/master/image-20220129165840002.png)

   注：源码管理部分**凭证与仓库地址要对应**，账号密码凭证填入https地址，ssh凭证填入ssh地址

3. 填完后的效果

![image-20220129173124113](https://gitee.com/yan-shipeng/images/raw/master/image-20220129173124113.png)

![image-20220129173350224](https://gitee.com/yan-shipeng/images/raw/master/image-20220129173350224.png)

![image-20220129173559416](https://gitee.com/yan-shipeng/images/raw/master/image-20220129173559416.png)

![image-20220129173610504](https://gitee.com/yan-shipeng/images/raw/master/image-20220129173610504.png)

![image-20220129173623211](https://gitee.com/yan-shipeng/images/raw/master/image-20220129173623211.png)

### 2、在服务器中放入脚本

1. 连接上在Jenkins中配置的服务器，进入到对应的传输文件根目录（我的是aliyun这台服务器的/root目录）

   ![image-20220129144138948](https://gitee.com/yan-shipeng/images/raw/master/image-20220129144138948.png)

2. 创建对应的文件夹和写入shell脚本

   ![image-20220129165711977](https://gitee.com/yan-shipeng/images/raw/master/image-20220129165711977.png)

   这里代表我要将jar包传输到aliyun服务器上的/root/jenkins/路径中，然后执行/root/jenkins/test.sh这个脚本

   ![image-20220129182437179](https://gitee.com/yan-shipeng/images/raw/master/image-20220129182437179.png)

   ![image-20220129175821222](https://gitee.com/yan-shipeng/images/raw/master/image-20220129175821222.png)

   ![image-20220129185231606](https://gitee.com/yan-shipeng/images/raw/master/image-20220129185231606.png)

3. test.sh内容如下

   ```sh
   # 杀死port端口运行的进程
   port=8080
   pid=$(netstat -nlp |grep :$port | awk '{print $7}' | awk -F"/" '{print $1}');
   kill -9 $pid
   sleep 2
   
   # 将jar包移动到项目路径
   mv /root/Jenkins/test-0.0.1-SNAPSHOT.jar /root/test/
   cd /root/test/
   
   # nohup启动服务
   if test -e test-0.0.1-SNAPSHOT.jar
   then
   nohup java -jar test-0.0.1-SNAPSHOT.jar >> test.log 2>&1 &
   fi
   ```

### 3、测试是否成功

1. 在本地ide中提交代码到Github上，回到Jenkins查看状态（提交代码后Jenkins自动进行构建）

   ![image-20220129185924766](https://gitee.com/yan-shipeng/images/raw/master/image-20220129185924766.png)

2. 点进去，点侧边栏控制台输出，出现Success代表成功

   ![image-20220129190104384](https://gitee.com/yan-shipeng/images/raw/master/image-20220129190104384.png)

3. 最后测试你的服务是否能正常访问即可

   ![image-20220129190209539](https://gitee.com/yan-shipeng/images/raw/master/image-20220129190209539.png)