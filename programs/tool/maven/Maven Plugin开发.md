[TOC]

# 创建plugin项目
```
mvn archetype:generate 
    -DgroupId=com.fq.plugins -DartifactId=lc-maven-plugin -Dversion=0.0.1-SNAPSHOT
    -DarchetypeArtifactId=maven-archetype-plugin 
    -DinteractiveMode=false 
    -DarchetypeCatalog=internal
```
使用maven-archetype-plugin Archetype可以快速创建一个Maven插件项目。
** pom.xml **
插件本身也是Maven项目, 特殊之处在于packaging方式为maven-plugin:
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.fq.plugins</groupId>
    <artifactId>lc-maven-plugins</artifactId>
    <packaging>maven-plugin</packaging>
    <version>0.0.1-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>19.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.maven</groupId>
            <artifactId>maven-plugin-api</artifactId>
            <version>3.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugin-tools</groupId>
            <artifactId>maven-plugin-annotations</artifactId>
            <version>3.3</version>
        </dependency>
    </dependencies>

</project>
```
maven-plugin 打包方式能控制Maven为其在生命周期阶段绑定插件处理的相关目标.

# 编写目标Mojo
```
@Mojo(name = "lc", defaultPhase = LifecyclePhase.VERIFY)
public class LCMavenMojo extends AbstractMojo {

    private static final List<String> DEFAULT_FILES = Arrays.asList("java", "xml", "properties");

    @Parameter(defaultValue = "${project.basedir}", readonly = true)
    private File baseDir;

    @Parameter(defaultValue = "${project.build.sourceDirectory}", readonly = true)
    private File srcDir;

    @Parameter(defaultValue = "${project.build.testSourceDirectory}", readonly = true)
    private File testSrcDir;

    @Parameter(defaultValue = "${project.build.resources}", readonly = true)
    private List<Resource> resources;

    @Parameter(defaultValue = "${project.build.testResources}", readonly = true)
    private List<Resource> testResources;

    @Parameter(property = "lc.file.includes")
    private Set<String> includes = new HashSet<>();

    private Log logger = getLog();

    @Override
    public void execute() throws MojoExecutionException, MojoFailureException {
        if (includes.isEmpty()) {
            logger.debug("includes/lc.file.includes is empty!");
            includes.addAll(DEFAULT_FILES);
        }
        logger.info("includes: " + includes);

        try {
            long lines = 0;
            lines += countDir(srcDir);
            lines += countDir(testSrcDir);

            for (Resource resource : resources) {
                lines += countDir(new File(resource.getDirectory()));
            }
            for (Resource resource : testResources) {
                lines += countDir(new File(resource.getDirectory()));
            }

            logger.info("total lines: " + lines);
        } catch (IOException e) {
            logger.error("error: ", e);
            throw new MojoFailureException("execute failure: ", e);
        }
    }

    private LineProcessor<Long> lp = new LineProcessor<Long>() {

        private long line = 0;

        @Override
        public boolean processLine(String fileLine) throws IOException {
            if (!Strings.isNullOrEmpty(fileLine)) {
                ++this.line;
            }
            return true;
        }

        @Override
        public Long getResult() {
            long result = line;
            this.line = 0;
            return result;
        }
    };


    private long countDir(File directory) throws IOException {
        long lines = 0;
        if (directory.exists()) {
            Set<File> files = new HashSet<>();
            collectFiles(files, directory);

            for (File file : files) {
                lines += CharStreams.readLines(new FileReader(file), lp);
            }

            String path = directory.getAbsolutePath().substring(baseDir.getAbsolutePath().length());
            logger.info("path: " + path + ", file count: " + files.size() + ", total line: " + lines);
            logger.info("\t-> files: " + files.toString());
        }

        return lines;
    }

    private void collectFiles(Set<File> files, File file) {
        if (file.isFile()) {
            String fileName = file.getName();
            int index = fileName.lastIndexOf(".");
            if (index != -1 && includes.contains(fileName.substring(index + 1))) {
                files.add(file);
            }
        } else {
            File[] subFiles = file.listFiles();
            for (int i = 0; subFiles != null && i < subFiles.length; ++i) {
                collectFiles(files, subFiles[i]);
            }
        }
    }
}

```

@Parameter: 配置点, 提供Mojo的可配置参数. 大部分Maven插件及其目标都是可配置的, 通过配置点, 用户可以自定义插件行为

```xml
<plugin>
    <groupId>com.fq.plugins</groupId>
    <artifactId>lc-maven-plugins</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <executions>
        <execution>
            <id>lc</id>
            <phase>verify</phase>
            <goals>
                <goal>lc</goal>
            </goals>
            <configuration>
                <includes>
                    <include>java</include>
                    <include>lua</include>
                    <include>json</include>
                    <include>xml</include>
                    <include>properties</include>
                </includes>
            </configuration>
        </execution>
    </executions>
</plugin>
```
execute(): 实际插件功能;
异常: execute()方法可以抛出以下两种异常:
MojoExecutionException: Maven执行目标遇到该异常会显示 BUILD FAILURE 错误信息, 表示在运行期间发生了预期的错误;
MojoFailureException: 表示运行期间遇到了未预期的错误, 显示 BUILD ERROR 信息

# 测试&执行
通过mvn clean install将插件安装到仓库后, 就可将其配置到实际Maven项目中, 用于统计项目代码了:
```
$ mvn com.fq.plugins:lc-maven-plugins:0.0.1-SNAPSHOT:lc
```
你可能注意到为了调用该插件的goal，我们需要给出该插件的所有坐标信息，包裹groupId, artifactId，version号，你可能之前已经执行过"mvn eclipase:eclipase"或"mvn idea:idea"这样简洁的命令，让我们也来将自己的插件调用变简单一点。要通过简单别名的方式调用Maven插件，我们需要做到以下两点：

+ 插件的artifactId应该遵循-maven-plugin或maven--plugin命名规则，对于本文中的插件，我们已经遵循了。
+ 需要将插件的groupId放在Maven默认的插件搜寻范围之内，默认情况下Maven只会在org.apache.maven.plugins和org.codehaus.mojo两个groupId下搜索插件，要让Maven同时搜索我们自己的groupId，我们需要在~/.m2/settings.xml中加入：
```xml
<pluginGroups>
       <pluginGroup>com.fq.plugins</pluginGroup>
</pluginGroups>
```

在达到以上两点之后，我们便可以通过以下命令来调用自己的插件了：
mvn lc:lc

要在别的项目中应用插件也是简单的，我们只需要在该项目的pom.xml文件中使用上面<plugin>标签声明该插件即可。
