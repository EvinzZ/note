# Java强制删除java程序占用的文件

```Java
public boolean forceDelete(File file) {
        boolean result = file.delete();
        int tryCount = 0;
        while (!result && tryCount++ < 10) {
            System.gc();    //回收资源
            result = file.delete();
        }
        return result;
    }
```

虽然System.gc()可以这样强制解除对文件的占用，但是也不是万能的，因为： ① 及时关闭输入流、输出流是很有必要的 ② System.gc()也不是一定能够回收垃圾的，能否成功回收取决于JVM的回收机制，外部人员是无法掌控的。就像我们去吃饭等时间长了催单，我们只是向餐馆提交了我们的申请，能不能快点上菜依然取决于厨师 ③ 即使不用System.gc()进行强制回收，JVM的回收机制也会在一段时间后对资源进行回收 ④ 这种方法也只对被java程序占用的文件有用，对于被其他进行占用的文件就无能为力了