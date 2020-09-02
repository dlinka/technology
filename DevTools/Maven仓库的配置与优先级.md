中央仓库

    //默认的中央仓库定义在${M2_HOME}/lib/maven-model-builder-X.X.X.jar中
    <repositories>
      <repository>
        <id>central</id>
        <name>Central Repository</name>
        <url>https://repo.maven.apache.org/maven2</url>
        <layout>default</layout>
        //不会在中央仓库下载SNAPSHOT构件
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
      </repository>
    </repositories>

POM中配置仓库

    <repositories>
      <repository>
        <id>maven-net-cn</id>
        <name>Maven China Mirror</name>
        <url>http://maven.net.cn/content/groups/public/</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>false</enabled>
        </snapshots> 
      </repository>
      //可以配置多个
      ...
    </repositories>

settings.xml中配置仓库

    <profiles>  
      <profile>  
        <id>develop</id>  
        <repositories>
          <repository>
            <id>maven-net-cn</id>
            <name>Maven China Mirror</name>
            <url>http://maven.net.cn/content/groups/public/</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>false</enabled></snapshots>
          </repository>
        </repositories>
      </profile>
    </profiles>
    
    <activeProfiles>
      <activeProfile>develop</activeProfile>  
    </activeProfiles>
    
settings.xml中配置镜像

    //镜像就是代理
    <mirrors>
      <mirror>
        <id>ali</id>
        <name>Ali Repo</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
      </mirror>
    </mirrors>

---
