POM中定义deploy的地址

    <distributionManagement>
      <repository>
        <id>nexus-releases</id>
        <name>Nexus Release Repository</name>
        <url>http://127.0.0.1:8080/nexus/content/repositories/releases/</url>
      </repository>
      <snapshotRepository>
        <id>nexus-snapshots</id>
        <name>Nexus Snapshot Repository</name>
        <url>http://127.0.0.1:8080/nexus/content/repositories/snapshots/</url>
      </snapshotRepository>
    </distributionManagement>

settings.xml中定义认证信息

    <servers>
      <server>
        //id跟上面要一致
        <id>nexus-releases</id>
        <username>user</username>
        <password>xxx</password>
      </server>
      <server>
        <id>nexus-snapshots</id>
        <username>user</username>
        <password>xxx</password>
      </server>
    </servers>

---
