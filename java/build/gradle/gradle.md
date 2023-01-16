[toc]

## introduce

1. content

   - groovy syntax
   - build script block
   - gradle api

2. feature

   - 灵活性
   - 粒度细
   - 扩张性好: plugin
   - 兼容性: ant / maven

3. 目录结构

   ```txt
   ├─ build.gradle                          // build script
   ├─ gradlew                               // gradlew
   ├─ gradlew.bat
   ├─ settings.gradle                       // init
   ├─.gradle
   │  ├─5.2.1
   │  ├─buildOutputCleanup
   │  └─vcs-1
   ├─build                                  // build output
   │  ├─libs
   │  └─tmp
   ├─gradle                                 // for gradlew
   │  └─wrapper
   │     ├─ gradle-wrapper.jar
   │     ├─ gradle-wrapper.properties
   ```

4. lifecycle: `gradle wrapper` 初始化 gradle 环境

   ![avatar](/static/image/common/gradle/gralde.png)

   - 初始化阶段
   - 配置阶段
   - 执行阶段

   ![avatar](/static/image/common/gradle/gralde-build.png)

   ```groovy

   /*===================== lifecycle ========================*/
   /**
   * 配置阶段之前的监听回调
   */
   this.beforeEvaluate {}

   /**
   * * 配置阶段之后的监听回调
   */
   this.afterEvaluate {}

   /**
   * gradle 执行完毕之后的监听
   */
   this.gradle.buildFinished {
       getAllProjects()
   }

   def getAllProjects() {
       this.getSubprojects().eachWithIndex { Project project, int i ->
           println "get-subprojects: ${project.name}"
       }
   }

   /**
   * beforeEvaluate
   */
   this.gradle.beforeProject {
   }

   /**
   * afterEvaluate
   */
   this.gradle.afterProject {}

   // add listener for monitor
   ```

5. 监听阶段

   ```groovy
   /**
    * 配置阶段之前的监听回调
    */
    this.beforeEvaluate {}

    /**
    * * 配置阶段之后的监听回调
    */
    this.afterEvaluate {}

    /**
    * gradle 执行完毕之后的监听
    */
    this.gradle.buildFinished {}
   ```

## dependency

1. implementation

   - 对该项目有依赖的项目将无法访问到使用该命令编译的依赖中的任何程序[也就是将该依赖隐藏在内部]
   - A 依赖 B, B 依赖 C, 如果 B 依赖 C 是使用的 implementation 依赖, 那么在 A 中是访问不到 C 中的方法的[如果需要访问, 请使用 api(compile)依赖]

2. compile/api

   - 编译和打包

3. providedCompile

   - 仅在编译的时候需要, 但是在运行时不需要依赖

4. debugCompile (debugImplementation)

   - debugCompile 只在 debug 模式的编译和最终的 debug 打包时有效

5. releaseCompile (releaseImplementation)

   - releaseCompile 仅仅针对 Release 模式的编译和最终的 Release 打包

6. testCompile (testImplementation)

   - testCompile 只在单元测试代码的编译以及最终打包测试 apk 时有效。

7. apk(runtimeOnly）

   - 只在生成 apk 的时候参与打包, 编译时不会参与, 很少用

8. runtime

   - 仅在运行的时候需要, 但是在编译时不需要依赖

9. look up dependencies

   ```groovy
   gradle :{module}:dependencies
   gradle :{module}:dependencies --configuration compile
   gradle :{module}:dependencies --configuration compileOnly
   gradle :{module}:dependencies --configuration runtime
   gradle :{module}:dependencies --configuration testCompile
   gradle :{module}:dependencies --configuration testCompileOnly
   gradle :{module}:dependencies --configuration testRuntime
   ```

## project

1. project api

   - gradle api
   - project api

     1. this.getAllprojects()
     2. this.getSubprojects()
     3. this.getParent()

     ```groovy
     /**
     * 对指定项目进行配置
     */
     project('groovy') { Project project ->
         println "groovy: ${project.name}"
     }

     /**
     * 所有项目公共的配置
     */
     allprojects {
         group 'cn.ntu.edu'
         version '1.0-SNAPSHOT'
     }

     println "groovy-group: ${project('groovy').group}"

     /**
     * 所有子项目通用的配置, 脚本执行目录是子目录
     */
     subprojects { Project project ->
         println "subprojects: ${project.name}"
         if (!project.plugins.hasPlugin('com.android.libary')) {
             // 引入外部 publishToMaven 文件
             apply from: '../publishToMaven.gradle'
         }
     }
     ```

   - task api
   - property api

     1. DEFAULT_BUILD_FILE = "build.gradle"
     2. PATH_SEPARATOR = ":"
     3. DEFAULT_BUILD_DIR_NAME = "build"
     4. GRADLE_PROPERTIES = "gradle.properties"
     5. 定义扩展属性[闭包]

   - file api
   - other api

2. build-script

   ```groovy
   buildscript {
   /*===================== repositories ========================*/
       repositories {
           maven { url "https://maven.aliyun.com/repository/central" }
       }
   /*===================== dependency ========================*/
       dependencies {
           classpath 'net.sourceforge.jtds:jtds:1.2.4'
       }
   }
   ```

3. gradle projects

   - root project: 管理子项目
   - sub project:

   ```java
   Root project 'gralde'
   \--- Project ':groovy'
   ```

   - 在父模块内编写公用的逻辑: `allprojects{}`

4. property

   ```groovy
    /*===================== property ========================*/
    // 扩展属性: ext
    ext {
        project_version = 1.5
        project_author = 'zack'
    }
    println "ext property: ${this.project_version}"

    subprojects {
        ext {
            project_version = 1.5
        }
    }
    println "groovy-project-version: ${project('groovy').project_version}"

    // 引入扩展属性, 在所有的项目中都有效
    apply from: this.file('common.gradle')
    println "common.gradle property: ${this.java_version}"

    if (hasProperty('author') ? author.toUpperCase() : 'zack.zhang') {
        println "gradle property: ${this.author}"
    }
   ```

5. file

   ```groovy
   /*===================== file ========================*/

   // 1. path
   println "file-- root dir: ${this.getRootDir()}"
   println "file-- build dir: ${this.getBuildDir()}"
   println "file-- project dir: ${this.getProjectDir().absolutePath}"

   // 2. content
   println getContent('common.gradle')

   /**
   * @param path 相对路径
   * @return
   */
   def getContent(String path) {
       try {
           def fi = file(path)
           return fi.text
       } catch (GradleException e) {
           println "${path} file not found.."
       }
   }

   // 3. copy
   copy {
       from file('groovy/')
       into(getBuildDir().absolutePath + '/groovy-source-code')
       // exclude {}
       // rename {}
   }

   // 4. fileTree
   fileTree('groovy/') { FileTree fileTree ->
       fileTree.visit { FileTreeElement element ->
           // println "fileTree file name: ${element.name}"
       }
   }

   fileTree('groovy/') {
       visit { FileTreeElement element ->
           // println "fileTree file name: ${element.name}"
       }
   }

   // 5. execute outer command
   task copyJar(group: 'gradlew', description: 'gradlew tasks') {
       doLast {

           def sourcePath = this.buildDir.path
           def distPath = 'D:/tmp/libs/'
           def command = "mv -f ${sourcePath} ${distPath}"
           exec {
               try {
                   executable 'bash'
                   args '-c', command
                   println 'command execute finished'
               } catch (GradleException e) {
                   println 'command execute failed'
               }
           }
       }
   }



   ```

## task

1. 创建: task 会被 TaskContainer 统一管理

   - task 内除了 **`doLast` 和 `doFirst`**[会在执行周期执行] 的逻辑之外都会在 `初始化阶段执行`

   ```groovy
   /*===================== task ========================*/
   task helloTask(group: 'gradlew', description: 'gradlew tasks') {
       println "helloTask ${this.author}"
       doFirst {
           println "inner helloTask doFirst ${this.author}"
       }
   }

   helloTask.doFirst {
       println "outer helloTask doFirst ${this.author}"
   }

   this.tasks.create(name: 'hello') {
       setGroup('gradlew')
       setDescription('gradlew tasks')
       println "hello ${this.author}"
   }
   ```

2. task sequence

   - `dependsOn`
   - 通过指定输入输出

     ![avatar](/static/image/common/gradle/gralde-task-squence.png)

   - 通过 API 指定顺序

3. task type

   - https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Copy.html

## others

1. third-party module
2. init class: `setting
3. `SourceSet`
4. gradlew

   ```groovy
   /*===================== gradlew ========================*/
   wrapper {
       gradleVersion = '5.2.1' // version required
   }
   ```
