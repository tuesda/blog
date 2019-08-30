## Gradle:publishing overview

> https://docs.gradle.org/current/userguide/publishing_overview.html#publishing_overview

publish process

- define `what`
- define `where`
- execute

repo types:

- Maven -> `Maven Publish Plugin`
- Ivy -> `Ivy Publish Plugin`



**What**

- artifacts
- metadata



**Where**

- repository



**How**

- tasks



### example

``` groovy
plugins {
    id 'java-library'
    id 'maven-publish'
    
    group = 'org.example'
    version = '1.0'
    
    publishing {
        publications {
             // 👇 定义要发布的软件包
            myLibrary(MavenPublication) {
                from components.java // components.java 由 'java-library' 提供，不同的语言可能不同
            }
        }
        
        repositories {
            maven {   // 👇 定义 maven 仓库
                name = 'myRepo'
                url = 'file://${buildDir}/repo' // 文件协议 maven 仓库，使用时一般使用基于 https 的
            }
        }
    }
}
```

根据上面配置会生成一个 task 用来完成发布工作，例如上面的配置会生成 task：publish*MyLibrary*To*MyRepo*Repository，这里的 MyLibrary 和 MyRepo 对应上面配置里的名字。

发布的时候执行 `publish` task 就一键执行所有需要执行的 task。



### 添加组件到软件包

添加 `source` 和 `javadoc` Jar 包：

``` groovy
task sourcesJar(type: Jar) {
    archiveClassifier = 'sources'
    from sourceSets.main.allJava
}

task javadocJar(type: Jar) {
    achiveClassifier = 'javadoc'
    from javadoc.destinationDir
}

publishing {
    publications {
        mavenJava(MavenPublication) {
        	from components.java
        
	        artifact sourceJar // artifact() 接受 archive task 或者 Project.file() 的参数类型
    	    artifact javadocJar
    	}
    }
}
```

### 发布一个自定义软件包

``` groovy
def rpmFile = file("$buildDir/rpms/my-package.rpm")
// rpmArtifact 类型是 PublishArtifact
def rpmArtifact = artifacts.add('archives', rpmFile) {
    type 'rpm'
    builtBy 'rpm'
}

publishling {
    publications {
        maven(MavenPublication) {
            artifact rpmArtifact
        }
    }
}
```

这样就可以发布这个 rpm 包了，初次之外还可以通过下面这种方式依赖 rpm 包

``` groovy
project(path: ':my-project', configuration: 'archives')
```

### 软件包签名

``` groovy
plugins {
    id 'signing'
}

signing {
    sign publishing.publications.mavenJava
}
```

### 限制发布路径

`maven-publish` 会自动根据 publications 和 repository 两两配对生成对应的发布 task 可以通过如下方式加以限制：

``` groovy
tasks.withType(PublishToMavenRepository) {
    onlyIf {
        (repository == publishing.repositories.external &&
            publication == publishing.publications.binary) ||
        (repository == publishing.repositories.internal &&
            publication == publishing.publications.binaryAndSources)
    }
}
tasks.withType(PublishToMavenLocal) {
    onlyIf {
        publication == publishing.publications.binaryAndSources
    }
}
```

还可以通过一个 task 完成多个发布任务：

``` groovy
task publishToExternalRepository {
    group = 'publishing'
    description = 'Publishes all Maven publications to the external Maven repository.'
    dependsOn tasks.withType(PublishToMavenRepository).matching {
        it.repository == publishing.repositories.external
    }
}
```

### Task 配置

由于发布 task 都是动态生成的，所以要配置它们时需要使用延迟配置：

``` groovy
tasks.withType(GenerateMavenPom).all {
    def matcher = name =~ /generatePomFileFor(\w+)Publication/
    def publicationName = matcher[0][1]
    destination = "$buildDir/poms/${publicationName}-pom.xml"
}
```

### Others

Configuration，一组 dependencies 或者 artifacts。它们的区别是：project 消费 dependencies，生产 artifacts。



