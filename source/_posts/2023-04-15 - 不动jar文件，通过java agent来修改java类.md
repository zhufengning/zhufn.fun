---
title: 不动jar文件，通过java agent来修改java类
urlname: 不动jar文件，通过java agent来修改java类

date: April 15, 2023 12:01 AM
tags: [java, web]
---
辽宁省赛线下awd，出了那么一道java题：

```java
@PostMapping({"/cmd"})
@ResponseBody
public String cmd(@RequestParam String command) throws Exception {
    Base64.getDecoder();
    StringBuffer sb = new StringBuffer("");
    String[] c = {"/bin/bash", "-c", command};
    Process p = Runtime.getRuntime().exec(c);
    CopyInputStream(p.getInputStream(), sb);
    CopyInputStream(p.getErrorStream(), sb);
    return sb.toString();
}
```

遗憾的是：只有3个队伍会修。而我们正因为不会修，屈居第四（还好kali上有个jadx-gui，但是为什么jadx-gui只能看不能改啊！！）。

研究了一下，发现了java agent这个东西，可以弄出类似hook的效果（感觉不如。。xposed），但是感觉有很多坑，环境方面，所以记录一下。

首先最不重要的代码部分是从这里参考的，加了点针对函数重载的精确搜索（关于java agent的介绍也可以看这里）：[https://www.cnblogs.com/rickiyang/p/11368932.html](https://www.cnblogs.com/rickiyang/p/11368932.html)

```java
package agent2;

import java.io.IOException;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.lang.instrument.Instrumentation;
import java.security.ProtectionDomain;
import javassist.*;

public class PreMainAgent {

    public static void premain(String agentArgs, Instrumentation inst) {
        System.out.println("agentArgs : " + agentArgs);
        inst.addTransformer(new DefineTransformer(), true);
    }

    static class DefineTransformer implements ClassFileTransformer {

        @Override
        public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
            if (!"com/example/ezjava/controller/EvilController".equals(className)) {
                return null;
            }
            System.out.println("premain load Class:" + className);
            try {
                System.out.println("trying!");
                final ClassPool classPool = ClassPool.getDefault();
                final CtClass clazz = classPool.get("com.example.ezjava.controller.EvilController");
                final CtClass str_class = classPool.get("java.lang.String");
                System.out.println(str_class);
                CtMethod cmd = clazz.getDeclaredMethod("cmd", new CtClass[]{str_class});
                //这里对 java.util.Date.convertToAbbr() 方法进行了改写，在 return之前增加了一个 打印操作
                String methodBody = "{return \"stupid!\";}";
                System.out.println("Setting body!");
                cmd.setBody(methodBody);

                // 返回字节码，并且detachCtClass对象
                byte[] byteCode = clazz.toBytecode();
                //detach的意思是将内存中曾经被javassist加载过的Date对象移除，如果下次有需要在内存中找不到会重新走javassist加载
                clazz.detach();
                System.out.println("returning!");
                return byteCode;
            } catch (IOException | CannotCompileException | NotFoundException e) {
                System.out.println("Error!");
                System.out.println(e);
                return null;
            }
        }
    }
}
```

坑部分：

下载`javassist.jar` 。

然后他用的是maven来改manifest，但是我一直没有找到把依赖打进包里的办法，就改用ant了。

IDE我用的netbeans。可以直接在Libraries上右键添加jar。

修改manifest参考这个链接：[https://www.javaxt.com/wiki/Tutorials/Netbeans/How_to_Add_Version_Information_to_a_Jar_File_with_Netbeans](https://www.javaxt.com/wiki/Tutorials/Netbeans/How_to_Add_Version_Information_to_a_Jar_File_with_Netbeans)

但是，其中的这部分不太对

> First, you must update your Netbeans "project.properties" file found in the "nbproject" directory. Add the following line to the file:
> 
> 
> manifest.file=manifest.mf
> 

应该是把`project.properties`里的`manifest.file=manifest.mf`改成`manifest.file=MANIFEST.MF`

manifest里除了`Premain-Class`和`Agent-Class`以外还要再加两行（记得把他写的乱七八糟的属性都删掉）：

```xml
<attribute name="Can-Redefine-Classes" value="true"/>
<attribute name="Can-Retransform-Classes" value="true"/>
```

虽然ant也没把依赖打进包里，但他创建了个lib目录并在manifest里写入了classpath的属性，所以可以跑起来。

然后启动命令是`java -javaagent:'/home/zfn/NetBeansProjects/agent2/dist/agent2.jar' -jar awd.jar`

![Untitled](../images/不动jar文件，通过java%20agent来修改java类%20ea74ef5a453e46dea12ce194cb149653/Untitled.png)

好
