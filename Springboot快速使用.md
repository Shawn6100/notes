# SpringBoot快速上手

##  一、IDEA创建SpringBoot项目

### 2020版IDEA

1. 打开IDEA，创建新项目，选择Spring Initializr

![image-20220310173559876](https://gitee.com/yan-shipeng/images/raw/master/image-20220310173559876.png)

2. 填写信息，下一步

![image-20220310173852486](https://gitee.com/yan-shipeng/images/raw/master/image-20220310173852486.png)

3. 点击Web，勾选，点击SQL，勾选

![image-20220310174049241](https://gitee.com/yan-shipeng/images/raw/master/image-20220310174049241.png)

![image-20220310174031200](https://gitee.com/yan-shipeng/images/raw/master/image-20220310174031200.png)

4. 选择存储路径，下一步

![image-20220310174149755](https://gitee.com/yan-shipeng/images/raw/master/image-20220310174149755.png)

5. 等待Maven加载完毕



### 2021版IDEA

1. 打开IDEA，创建新项目，选择Spring Initializr，点Next

   ![image-20220310175002004](https://gitee.com/yan-shipeng/images/raw/master/image-20220310175002004.png)

2. 打开Web分类和SQL分类，勾选，点击Finish

![image-20220310175057538](https://gitee.com/yan-shipeng/images/raw/master/image-20220310175057538.png)

![image-20220310175138848](https://gitee.com/yan-shipeng/images/raw/master/image-20220310175138848.png)


​      

## 二、完善目录结构，添加项目配置

1. 查看初始目录结构

![image-20220310175455142](https://gitee.com/yan-shipeng/images/raw/master/image-20220310175455142.png)

2. 完善项目目录结构

   ![image-20220310180345247](https://gitee.com/yan-shipeng/images/raw/master/image-20220310180345247.png)

3. 编写基本配置

![image-20220310181558988](https://gitee.com/yan-shipeng/images/raw/master/image-20220310181558988.png)

   

## 三、测试运行

1. 测试打印Hello SpringBoot

   - 在Controller下编写Hello类

     ![image-20220310182039282](https://gitee.com/yan-shipeng/images/raw/master/image-20220310182039282.png)

   - 打开主程序（这里是QuickStartApplication），点击右边运行图标

     ![image-20220310182203263](https://gitee.com/yan-shipeng/images/raw/master/image-20220310182203263.png)

   - 正常启动效果，服务已经在8080浏览器启动

     ![image-20220310183029674](https://gitee.com/yan-shipeng/images/raw/master/image-20220310183029674.png)

   - 在浏览器中测试

     - localhost:8080

     - localhost:8080/test

     - localhost:8080/get

     - localhost:8080/post

       ![image-20220310183400308](https://gitee.com/yan-shipeng/images/raw/master/image-20220310183400308.png)

       ![image-20220310183424879](https://gitee.com/yan-shipeng/images/raw/master/image-20220310183424879.png)

       ![image-20220310183436031](https://gitee.com/yan-shipeng/images/raw/master/image-20220310183436031.png)

       ![image-20220310183454592](https://gitee.com/yan-shipeng/images/raw/master/image-20220310183454592.png)

     > 注意Hello类中**黄色的注解**，@RequestMapping、@GetMapping、@PostMapping
     >
     > 这三个注解用于将Http请求映射到Springboot中的控制器和方法上，类似于在Servlet的xml里配的路径
     >
     > 三者的区别是:
     >
     > - @RequestMapping可以接受GET和POST请求
     > - @GetMapping只能接收GET请求
     > - @PostMapping只能接收POST请求。
     >
     > 我们都知道，浏览器直接输入地址访问属于GET请求，所以 localhost:8080/post 这种访问方式会报错
     >
     > 不要盲目全用@RequestMapping注解，三者都有用处，具体再说~

   

2. 测试数据库是否能够正常连接

   - 在数据库中创建一张student表

     ![image-20220310211900808](https://gitee.com/yan-shipeng/images/raw/master/image-20220310211900808.png)

   - 填充数据（500条）

     ![image-20220310211926816](https://gitee.com/yan-shipeng/images/raw/master/image-20220310211926816.png)

   - 编写测试代码

     > 需求：给出查询学生接口，返回所有学生信息
     >
     > 实现：
     >
     > 1. Controller负责接收前端传入的参数（这里还没接），并请求Service提供服务
     > 2. Service负责处理Controller传入的数据，并调用Mapper对数据库进行存取操作
     > 3. Mapper负责对数据库进行存取操作

     - 我们先看项目结构，一共多了六个文件

     ![image-20220310215244505](https://gitee.com/yan-shipeng/images/raw/master/image-20220310215244505.png)

     - 再来看看代码

     > Student —— 实体类，对应数据库中的Student表

     ```java
     package com.quanta.quickstart.pojo;
     
     
     public class Student {
     
       private long id;
       private String name;
       private String number;
       private long age;
     
     
       public long getId() {
         return id;
       }
     
       public void setId(long id) {
         this.id = id;
       }
     
     
       public String getName() {
         return name;
       }
     
       public void setName(String name) {
         this.name = name;
       }
     
     
       public String getNumber() {
         return number;
       }
     
       public void setNumber(String number) {
         this.number = number;
       }
     
     
       public long getAge() {
         return age;
       }
     
       public void setAge(long age) {
         this.age = age;
       }
     
     }
     ```

     > StudentController —— 负责Student相关功能的控制器

     ```java
     package com.quanta.quickstart.controller;
     
     import com.alibaba.fastjson.JSONObject;
     import com.quanta.quickstart.pojo.Student;
     import com.quanta.quickstart.service.StudentService;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RequestMapping;
     import org.springframework.web.bind.annotation.RestController;
     
     import java.util.List;
     
     @RestController
     @RequestMapping("/student")
     public class StudentController {
     
         // 自动注入service
         @Autowired
         private StudentService studentService;
     
         /**
          * 查询所有学生信息
          */
         @GetMapping("/get")
         public JSONObject getStudentList() {
             JSONObject jsonObject = new JSONObject();
     
             // 调用 service 方法
             List<Student> studentList = studentService.getStudentList();
     
             // 判断
             if (studentList == null) {
                 jsonObject.put("code", 500);
                 jsonObject.put("msg", "查询失败");
                 return jsonObject;
             } else if (studentList.isEmpty()) {
                 jsonObject.put("code", 400);
                 jsonObject.put("msg", "数据为空");
                 return jsonObject;
             }
     
             // 构造返回
             String data = JSONObject.toJSONString(studentList);
     
             jsonObject.put("code", 200);
             jsonObject.put("msg", "查询成功");
             jsonObject.put("data", data);
     
             return jsonObject;
         }
     
     }
     ```

     > StudentService —— 负责Student功能的服务接口

     ```java
     package com.quanta.quickstart.service;
     
     import com.quanta.quickstart.pojo.Student;
     import org.springframework.stereotype.Service;
     
     import java.util.List;
     
     @Service
     public interface StudentService {
     
         // 查询所有学生信息
         List<Student> getStudentList();
     
     }
     ```

     > StudentServiceImpl —— StudentService接口的实现类

     ```java
     package com.quanta.quickstart.service.impl;
     
     import com.quanta.quickstart.mapper.StudentMapper;
     import com.quanta.quickstart.pojo.Student;
     import com.quanta.quickstart.service.StudentService;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.stereotype.Service;
     
     import java.util.List;
     
     @Service
     public class StudentServiceImpl implements StudentService {
     
         // 自动注入mapper
         @Autowired
         private StudentMapper studentMapper;
     
         // 查询所有学生信息
         @Override
         public List<Student> getStudentList() {
             return studentMapper.getStudentList();
         }
     
     }
     ```

     > StudentMapper —— 与Student功能相关的数据库操作接口

     ```java
     package com.quanta.quickstart.mapper;
     
     import com.quanta.quickstart.pojo.Student;
     import org.apache.ibatis.annotations.Mapper;
     import org.springframework.stereotype.Repository;
     
     import java.util.List;
     
     @Mapper
     @Repository
     public interface StudentMapper {
     
         // 查询所有学生信息
         List<Student> getStudentList();
     
     }
     ```

     > StudentMapper.xml

     ```
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
             PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.quanta.quickstart.mapper.StudentMapper">
     
         <select id="getStudentList" resultType="com.quanta.quickstart.pojo.Student">
             SELECT *
             FROM springboot_quick_start.student
         </select>
     
     </mapper>
     ```

   - 启动服务

     ![image-20220310215823848](https://gitee.com/yan-shipeng/images/raw/master/image-20220310215823848.png)

   - 接口测试

     ![image-20220310215134709](https://gitee.com/yan-shipeng/images/raw/master/image-20220310215134709.png)

   - 数据返回成功啦~




> 写在最后

​	这篇文档内容写的很浅，只写了一个无参查询接口，Controller和Mapper没有接收参数，也没有处理你们可能会遇到的跨域问题。

​	这篇文档的主要目的是给大家介绍Springboot接口一般怎么写，大家主要还是通过视频去学习Springboot的细节内容，源码原理部分可以先跳过，到时候学深了再回过头来看。

​	大家加油！