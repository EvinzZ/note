# JAVA动态加载外部JAR，并调用方法以及卸载关闭打开的外部JAR

```Java
public class AddJar {

    public static void main(String[] args){
        //外部jar所在位置
        String path = "file:D:\\Program File\\IDEA\\WorkSpase\\Test20191015\\out\\artifacts\\test191015\\test191015.jar";
        URLClassLoader urlClassLoader =null;
        Class<?> MyTest = null;
        try {
            //通过URLClassLoader加载外部jar
            urlClassLoader = new URLClassLoader(new URL[]{new URL(path)});
            //获取外部jar里面的具体类对象
            MyTest = urlClassLoader.loadClass("cn.hjljy.MyTest");
            //创建对象实例
            Object instance = MyTest.newInstance();
            //获取实例当中的方法名为show，参数只有一个且类型为string的public方法
            Method method = MyTest.getMethod("show", String.class);
            //传入实例以及方法参数信息执行这个方法
            Object ada = method.invoke(instance, "ada");
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //卸载关闭外部jar
            try {
                urlClassLoader.close();
            } catch (IOException e) {
                System.out.println("关闭外部jar失败："+e.getMessage());
            }
        }
    }
}
```

方案二：

```Java
public static void loadJar(String jarPath) {
    File jarFile = new File(jarPath);
    // 从URLClassLoader类中获取类所在文件夹的方法，jar也可以认为是一个文件夹
    Method method = null;
    try {
        method = URLClassLoader.class.getDeclaredMethod("addURL", URL.class);
    } catch (NoSuchMethodException | SecurityException e1) {
        e1.printStackTrace();
    }
    //获取方法的访问权限以便写回
    boolean accessible = method.isAccessible();
    try {
        method.setAccessible(true);
        // 获取系统类加载器
        URLClassLoader classLoader = (URLClassLoader) ClassLoader.getSystemClassLoader();
        URL url = jarFile.toURI().toURL();
        method.invoke(classLoader, url);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        method.setAccessible(accessible);
    }
```

## 工具类

```Java
package com.datacollection.unit.util;

import cn.hutool.core.collection.CollectionUtil;
import cn.hutool.core.io.FileUtil;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.serializer.SerializerFeature;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import java.io.*;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 动态jar加载
 */
@Slf4j
public class DynamicJarManager {

    public static Map<String, Map<String, Class<?>>> instanceMap = new ConcurrentHashMap<>();
    public static final String BASE_PATH = "./dynamicjar/";
    public static final String dynamic_jar_info_file_name = "./dynamic_jar_info_file_name.txt";

    public static List<String> compare(List<String> paramList) {
        return compare(paramList, new ArrayList<>(instanceMap.keySet()));
    }

    public static void load(byte[] bytes, List<String> classList, String jarName) {
        if (CollectionUtil.isEmpty(classList) || jarName.equals("")) {
            return;
        }
        File jarFile = new File(BASE_PATH + jarName);
        FileUtil.writeBytes(bytes, jarFile); // 存储文件
        for (String className : classList) {
            load(jarFile, className);
        }
        overrideInfoFile();
    }

    /**
     * 初始化，服务器启动时加载所有的jar
     */
    public static void init() {
        try {
            File baseDir = new File(BASE_PATH);
            if (!baseDir.exists()) {
                baseDir.mkdir();
            }
            File dynamic_jar_info_file = new File(dynamic_jar_info_file_name);
            if (!dynamic_jar_info_file.exists()) {
                dynamic_jar_info_file.createNewFile();
            } else {
                String jar_info = FileUtil.readUtf8String(dynamic_jar_info_file);
                Map<String, Map<String, Object>> map = JSONObject.parseObject(jar_info, Map.class);
                if (map != null) {
                    for (Map.Entry<String, Map<String, Object>> mapEntry : map.entrySet()) {
                        Map<String, Object> value = mapEntry.getValue();
                        for (Map.Entry<String, Object> entry : value.entrySet()) {
                            File f0 = new File(BASE_PATH + mapEntry.getKey());
                            if (f0.exists()) {
                                load(f0, entry.getKey());
                            }
                        }
                    }
                    overrideInfoFile();
                }
            }
        } catch (Exception var0) {
            log.error(var0.getMessage(), var0);
        }
    }

    /**
     * 加载jar
     *
     * @param file
     * @param fullClassName
     */
    public static void load(File file, String fullClassName) {
        if (file == null || !file.exists()) {
            return;
        }
        URLClassLoader urlClassLoader = null;
        try {
            urlClassLoader = new URLClassLoader(new URL[]{file.toURI().toURL()});
            Class<?> classStudentServiceImpl = urlClassLoader.loadClass(fullClassName);
            String key = FileUtil.getName(file);
            Map<String, Class<?>> classMap = instanceMap.get(key);
            if (classMap == null) {
                instanceMap.put(key, new HashMap<String, Class<?>>() {
                    {
                        put(fullClassName, classStudentServiceImpl);
                    }
                });
            } else {
                classMap.put(fullClassName, classStudentServiceImpl);
            }
            log.info("[动态jar包]class加载成功:{}", fullClassName);
        } catch (Exception e) {
            log.error("[动态jar包]class加载失败:{}", fullClassName, e);
        } finally {
            try {
                if (urlClassLoader != null) {
                    urlClassLoader.close();
                }
            } catch (IOException e) {
                log.error(e.getMessage(), e);
            }
        }
    }
    /**
     * 卸载jar
     */
    public static void unLoad(String key) {
        instanceMap.remove(key);
        overrideInfoFile();
    }

    public static List<String> compare(List<String> dynamicJarList, List<String> localJarList) {
        List<String> reqList = new ArrayList<>();
        if (dynamicJarList == null && CollectionUtil.isEmpty(localJarList)) {
            return reqList;
        }
        if (dynamicJarList == null) {
            File baseDir = new File(DynamicJarManager.BASE_PATH);
            File[] files = baseDir.listFiles();
            for (File file : files) {
                FileUtil.del(file);
            }
            instanceMap.clear();
        } else {
            for (String jar : dynamicJarList) {
                if (!localJarList.contains(jar) || !(new File(BASE_PATH + jar).exists())) {
                    // 下载jar
                    reqList.add(jar);
                }
            }
            for (String jar : localJarList) {
                if (!dynamicJarList.contains(jar)) {
                    FileUtil.del(new File(BASE_PATH + jar));
                    instanceMap.remove(jar);
                    log.info("[动态jar包]jar卸载成功:{}", jar);
                }
            }
        }
        overrideInfoFile();
        return reqList;
    }

    @SneakyThrows
    public static void overrideInfoFile() {
        File dynamic_jar_info_file = new File(dynamic_jar_info_file_name);
        if (!dynamic_jar_info_file.exists()) {
            dynamic_jar_info_file.createNewFile();
        }
        FileWriter writer = new FileWriter(dynamic_jar_info_file);
        BufferedWriter out = new BufferedWriter(writer);
        out.write(JSONObject.toJSONString(instanceMap, SerializerFeature.PrettyFormat));
        out.flush();
        out.close();
    }

    @SneakyThrows
    public static Object getInstance(String fullName) {
        Collection<Map<String, Class<?>>> values = DynamicJarManager.instanceMap.values();
        for (Map<String, Class<?>> value : values) {
            if (value.get(fullName) != null) {
                return value.get(fullName).newInstance();
            }
        }
        return null;
    }
}
```