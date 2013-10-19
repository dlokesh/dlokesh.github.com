<!-- {:title "Using leiningen for java projects" :date "2013-10-19" :tags "leiningen java"} -->

Since leiningen uses maven underneath for managing dependencies and leiningen being an awesome build tool, instead of dealing with cumbersome maven pom.xml files, why not use leiningen to build java projects?

Have a look at the [pom.xml][pom.xml] (256 lines) of a sample java project that produces a standalone jar. Specifying the dependencies is pretty straigforward with the `:dependencies` key in project.clj.

Build
=====

The default leiningen configuration in project.clj, starts compiling and building clj files found in `src`. In order for leiningen to start compiling java source files, we could use `:java-source-paths`

```
:java-source-paths ["src/main/java"]
```
Apart from this, we also specify resource folders in pom.xml - which would place files found there into the root of the jar.

```
<build>
    <finalName>${project.artifactId}-original</finalName>
    <resources>
        <resource>
            <directory>src/main/webapp</directory>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
        </resource>
    </resources>
...
```

The leiningen equivalent would be:

```
:resource-paths ["src/main/resources" "src/main/webapp"]
```

Please note that currently leiningen does not support resource [filtering][filtering].

Standalone JAR
==============

Next up is `maven-shade-plugin`, used to create standalone jars. Leiningen comes with `uberjar` built-in and it can be customized similar to the shade plugin. For eg. all spring dependencies have multiple same-named files, and by default shade plugin would overwrite the same file causing spring context load issues. To overcome this, shade plugin provides transformers:

```
<transformer
        implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
    <resource>META-INF/spring.handlers</resource>
</transformer>
<transformer
        implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
    <resource>META-INF/spring.schemas</resource>
</transformer>
```

The merge between the files is done using AppendingTransformer. With leiningen the possibilities are endless, as you could pass in any function for [merging][merging]:

```
:uberjar-merge-with 
	{"META-INF/spring.handlers" [slurp (fn [x y] (str x "\n" y "\n")) spit]
     "META-INF/spring.schemas" [slurp (fn [x y] (str x "\n" y "\n")) spit]
     "META-INF/spring.tooling" [slurp (fn [x y] (str x "\n" y "\n")) spit]
     "META-INF/cxf/bus-extensions.txt" [slurp (fn [x y] (str x "\n" y "\n")) spit]}
```

Have a look at the complete [project.clj][project.clj] (44 lines).

[project.clj]: https://gist.github.com/dlokesh/7057490#file-project-clj
[pom.xml]: https://gist.github.com/dlokesh/7057490#file-pom-xml
[merging]: https://github.com/technomancy/leiningen/blob/master/sample.project.clj#L335-L341
[filtering]: http://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html
