### Java执行Linux命令

今天开发一个简单程序，需要用Java程序调用Linux命令来执行datax任务，但是`.../datax.py ..../job.json`一直不能执行成功，但是也不抛出异常。后来改变一下写法即可。

存在问题
```
public static boolean executeLinuxCmd(String cmd){
        Runtime run = Runtime.getRuntime();
        boolean result = true;
        try {
            Process process = run.exec(cmd);
            //执行结果 0 表示正常退出
            int exeResult=process.waitFor();
            if(exeResult==0){
                log.info("Execute cmd success : {}",cmd);
            }else{
                result = false;
                log.error("Execute cmd failed : {}",cmd);
            }
        }
        catch (Exception e) {
            log.error("LinuxCmdUtils.executeLinuxCmd error message:{}",e.getMessage());
            return false;
        }
        return result;
    }
```

修改之后
```
public static boolean executeLinuxCmd(String cmd){
      Runtime run = Runtime.getRuntime();
      boolean result = true;
      try {
          String[] cmdArr = new String[]{"sh","-c",cmd};
          Process process = run.exec(cmdArr);
          //执行结果 0 表示正常退出
          int exeResult=process.waitFor();
          if(exeResult==0){
              log.info("Execute cmd success : {}",cmd);
          }else{
              result = false;
              log.error("Execute cmd failed : {}",cmd);
          }
      }
      catch (Exception e) {
          log.error("LinuxCmdUtils.executeLinuxCmd error message:{}",e.getMessage());
          return false;
      }
      return result;
  }
```
