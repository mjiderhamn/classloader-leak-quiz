# ClassLoader leak quiz

Use the [ClassLoader Leak Test Framework](https://github.com/mjiderhamn/classloader-leak-prevention/tree/master/classloader-leak-test-framework) and the process detailed in the first parts of [this blog series](http://java.jiderhamn.se/2011/12/11/classloader-leaks-i-how-to-find-classloader-leaks-with-eclipse-memory-analyser-mat/) to solve the following quiz.

*Note*, if you choose to run your tests with Maven, make sure to fork for each test case, or you'll aggregate leaked
ClassLoaders for each test run.
```xml
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.19</version>
        <configuration>
          <forkCount>1</forkCount>
          <reuseForks>false</reuseForks>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

## Goals

For each question in the quiz, you will need to find a specific letter. All letters, in the order of the questions,
will form a word. Your goal is to find that word. Note that _classname_ always refers to the simple/unqualified name of a class, i.e. ignore the package name.

In order to make sure you have not simply guessed missing letters, you need to note, for each question, the _type of 
leak_ caused.
* System class reference
* Unstopped thread
* Uncleared `ThreadLocal`

Let's begin!

---

## 1. Axis 
Find the first letter of the class causing the leak

```xml
<dependency>
  <groupId>org.apache.axis</groupId>
  <artifactId>axis</artifactId>
  <version>1.4</version>
  <scope>test</scope>
</dependency>
```

```java
org.apache.axis.utils.XMLUtils.getDocumentBuilder();
```

## 2. Batik
Find the third letter of the Batik class causing the leak

```xml
<dependency>
  <groupId>org.apache.xmlgraphics</groupId>
  <artifactId>batik-svg-dom</artifactId>
  <version>1.7</version>
</dependency>
```

```java
org.apache.batik.dom.GenericDocument document = (org.apache.batik.dom.GenericDocument) 
    org.apache.batik.dom.GenericDOMImplementation.getDOMImplementation()
      .createDocument("foo:bar", "foobar", 
        new org.apache.batik.dom.GenericDocumentType("foo", "bar", "baz"));
org.apache.batik.dom.GenericAttr attr = 
    (org.apache.batik.dom.GenericAttr) document.createAttribute("foo");
attr.setOwnerElement((org.apache.batik.dom.GenericElement) document.createElement("foo"));
attr.setIsId(true);
attr.setValue("bar");
```

## 3. Java Advanced Imaging (JAI)
Find the first letter of the class causing the leak. (As for the type, your dump may contain two references to this class, but only one of them has a single path to GC root.)

```xml
<repositories>
  <!-- Repo for JAI -->
  <repository>
    <id>jboss-thirdparty-releases</id>
    <url>https://repository.jboss.org/nexus/content/repositories/thirdparty-releases</url>
  </repository>
</repositories>
```
```xml
<dependency>
  <groupId>com.sun.media</groupId>
  <artifactId>jai-codec</artifactId>
  <version>1.1.3</version>
</dependency>
```
 
```java
com.sun.media.jai.codec.SeekableStream.wrapInputStream(
  new java.io.ByteArrayInputStream(new byte[0]), true);
```

## 4. GeoTools
Find the second letter of the class causing the leak. (Ignore any references passing through JAI.)

```xml
<repositories>
  <!-- Repo for GeoTools -->
  <repository>
    <id>osgeo</id>
    <name>Open Source Geospatial Foundation Repository</name>
    <url>http://download.osgeo.org/webdav/geotools/</url>
  </repository>
</repositories>
```
```xml
<dependency>
  <groupId>org.geotools</groupId>
  <artifactId>gt-shapefile</artifactId>
  <version>13.2</version>
</dependency>
```
 
```java
org.geotools.util.SoftValueHashMap m = new org.geotools.util.SoftValueHashMap(1);
m.put("foo", "bar");
m.put("foo2", "bar2");
```

## 5. JEditorPane
Find the second uppercase letter of the name of the class causing the leak

```java
new javax.swing.JEditorPane("text/plain", "dummy");
```
 
## 6. JGroups
Find the second letter of the *outer* JGroups class causing one of the two leaks. You'll have to guess which of the two to get the letter from. Note the type of leaks for both!

```xml
<dependency>
  <groupId>org.jgroups</groupId>
  <artifactId>jgroups</artifactId>
  <version>3.0.0.Final</version>
</dependency>
```

```java
org.jgroups.stack.GossipRouter gossipRouter = new org.jgroups.stack.GossipRouter();
gossipRouter.start();
gossipRouter.stop();
```

## 7. MVEL
Find the the letter occurring 3 times in the name of the class causing the leak

```xml
<dependency>
  <groupId>org.mvel</groupId>
  <artifactId>mvel2</artifactId>
  <version>2.0</version>
</dependency>
```

```java
org.mvel2.MVEL.executeExpression(
    org.mvel2.MVEL.compileExpression("new Integer(1)"), Collections.emptyMap());
```

## 8. dom4j
Find the first letter of the _attribute_ that holds the reference to the dom4j class

```xml
<dependency>
  <groupId>dom4j</groupId>
  <artifactId>dom4j</artifactId>
  <version>1.5</version>
</dependency>
```

```java
org.dom4j.DocumentHelper.createDocument();
```

## 9. PIG
Find the second uppercase letter of *outer* class to the first non-PIG class in the GC path

```xml
<dependency>
  <groupId>org.apache.pig</groupId>
  <artifactId>pig</artifactId>
  <version>0.15.0</version>
</dependency>
```

```java
org.apache.pig.impl.util.SpillableMemoryManager.getInstance();
```
