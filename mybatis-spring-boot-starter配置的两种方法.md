# mybatis-spring-boot-starter配置的两种方法

1. 通过mybatis.config-location指定mybatis-config.xml，里面设置别名和mapper包
   
   ```
   #application.properities
   mybatis.config-location=mybatis-config.xml
   
   <?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <package name="sample.mybatis.domain"/>
    </typeAliases>
    <mappers>
        <mapper resource="sample/mybatis/mapper/CityMapper.xml"/>
        <mapper resource="sample/mybatis/mapper/HotelMapper.xml"/>
    </mappers>
</configuration>
   ```
   
   
2. 通过mybatis.type-aliases-package和@Mapper来配置

    ```
        #application.properties
        mybatis.type-aliases-package=com.expert.pojo

        package com.expert.dao;
        import org.apache.ibatis.annotations.Mapper;
        import org.apache.ibatis.annotations.Select;
        import com.expert.pojo.User;
        @Mapper
        public interface UserMapper   {
        //    @Select("SELECT * FROM user WHERE id = #{ id }")
            User getById(String id);
            
            @Select("SELECT * FROM user WHERE id = #{ id }")
            User getById2(String id);
}
    ```

