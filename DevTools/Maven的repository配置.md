中央仓库

    //默认的中央仓库定义在${M2_HOME}/lib/maven-model-builder-X.X.X.jar中
    <repositories>
      <repository>
        <id>central</id>
        <name>Central Repository</name>
        <url>https://repo.maven.apache.org/maven2</url>
        <layout>default</layout>
        //这个不会在中央仓库下载SNAPSHOT构件
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
      </repository>
    </repositories>

POM中配置仓库

    <repositories>
      <repository>
        <id>ali-maven</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <releases><enabled>true</enabled></releases>
        <snapshots><enabled>false</enabled></snapshots> 
      </repository>
      //可以配置多个<repository></repository>
    </repositories>

settings.xml中配置仓库

    <profiles>  
      <profile>  
        <id>ali</id>  
        <repositories>
          <repository>
            <id>ali-maven</id>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <releases><enabled>true</enabled></releases>
            <snapshots><enabled>false</enabled></snapshots>
          </repository>
        </repositories>
      </profile>
    </profiles>
    
    <activeProfiles>
      <activeProfile>ali</activeProfile>  
    </activeProfiles>
    
settings.xml中配置镜像

    //镜像本质代理
    <mirrors>
      <mirror>
        <id>ali</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        //代理中央仓库
        <mirrorOf>central</mirrorOf>
      </mirror>
    </mirrors>

---
