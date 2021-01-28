# Minecraft Log 分析技术
 这是由ksqeib撰写的分析log的技术性文档
 如有错误 欢迎补充！

---
## 概念:
### 停顿
在本文中 我将停顿划分为两种类型 *主动型* 和 *被动型*
#### 主动型停顿
插件/模组/服务端主动发起的停顿
#### 被动性停顿
插件/模组/服务端由于处理数据而造成的停顿

### Java应用
#### 调用栈
可以简要理解为程序执行的路径/过程
### 常用线程状态
#### BLOCKED
阻塞
#### WAITING
等待
#### TIMED_WAITING
等待一段时间

---
## 案例分析
### 案例一
来源: https://www.mcbbs.net/thread-1159037-1-1.html

    [13:00:45 ERROR]: --- DO NOT REPORT THIS TO PAPER - THIS IS NOT A BUG OR A CRASH  - git-Paper-444 (MC: 1.16.5) ---
    [13:00:45 ERROR]: The server has not responded for 10 seconds! Creating thread dump
    [13:00:45 ERROR]: ------------------------------
    [13:00:45 ERROR]: Server thread dump (Look for plugins here before reporting to Paper!):
    [13:00:45 ERROR]: ------------------------------
    [13:00:45 ERROR]: Current Thread: Server thread
    [13:00:45 ERROR]:       PID: 21 | Suspended: false | Native: false | State: TIMED_WAITING
    [13:00:45 ERROR]:       Stack:
    [13:00:45 ERROR]:               java.base@11.0.10/java.lang.Thread.sleep(Native Method)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.util.language.game.MojangGameLanguageImpl.<init>(MojangGameLanguageImpl.java:68)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.util.MsgUtil.loadGameLanguage(MsgUtil.java:237)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.util.MsgUtil.loadCfgMessages(MsgUtil.java:250)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.util.MsgUtil.updateMessages(MsgUtil.java:1024)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.util.MsgUtil.loadCfgMessages(MsgUtil.java:316)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.QuickShop.onEnable(QuickShop.java:693)
    [13:00:45 ERROR]:               app//org.bukkit.plugin.java.JavaPlugin.setEnabled(JavaPlugin.java:263)
    [13:00:45 ERROR]:               app//org.bukkit.plugin.java.JavaPluginLoader.enablePlugin(JavaPluginLoader.java:380)
    [13:00:45 ERROR]:               app//org.bukkit.plugin.SimplePluginManager.enablePlugin(SimplePluginManager.java:483)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.command.subcommand.SubCommand_Reload.onCommand(SubCommand_Reload.java:39)
    [13:00:45 ERROR]:               org.maxgamer.quickshop.command.CommandManager.onCommand(CommandManager.java:413)
    [13:00:45 ERROR]:               app//org.bukkit.command.PluginCommand.execute(PluginCommand.java:45)
    [13:00:45 ERROR]:               app//org.bukkit.command.SimpleCommandMap.dispatch(SimpleCommandMap.java:159)
    [13:00:45 ERROR]:               app//org.bukkit.craftbukkit.v1_16_R3.CraftServer.dispatchCommand(CraftServer.java:807)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.PlayerConnection.handleCommand(PlayerConnection.java:2021)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.PlayerConnection.c(PlayerConnection.java:1832)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.PlayerConnection.a(PlayerConnection.java:1785)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.PacketPlayInChat.a(PacketPlayInChat.java:47)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.PacketPlayInChat.a(PacketPlayInChat.java:5)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.PlayerConnectionUtils.lambda$ensureMainThread$1(PlayerConnectionUtils.java:23)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.PlayerConnectionUtils$$Lambda$6939/0x0000000801c21840.run(Unknown Source)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.TickTask.run(SourceFile:18)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.IAsyncTaskHandler.executeTask(IAsyncTaskHandler.java:136)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.IAsyncTaskHandlerReentrant.executeTask(SourceFile:23)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.IAsyncTaskHandler.executeNext(IAsyncTaskHandler.java:109)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.MinecraftServer.bb(MinecraftServer.java:1132)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.MinecraftServer.executeNext(MinecraftServer.java:1125)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.IAsyncTaskHandler.awaitTasks(IAsyncTaskHandler.java:119)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.MinecraftServer.sleepForTick(MinecraftServer.java:1086)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.MinecraftServer.w(MinecraftServer.java:1000)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.MinecraftServer.lambda$a$0(MinecraftServer.java:173)
    [13:00:45 ERROR]:               app//net.minecraft.server.v1_16_R3.MinecraftServer$$Lambda$3125/0x0000000800905c40.run(Unknown Source)
    [13:00:45 ERROR]:               java.base@11.0.10/java.lang.Thread.run(Thread.java:834)
    [13:00:45 ERROR]: ------------------------------
    [13:00:45 ERROR]: --- DO NOT REPORT THIS TO PAPER - THIS IS NOT A BUG OR A CRASH ---
    [13:00:45 ERROR]: ------------------------------

从 `State: TIMED_WAITING` 可以看出，停顿状态是`等待一段时间`。

根据报错堆栈第一行
`[13:00:45 ERROR]:               java.base@11.0.10/java.lang.Thread.sleep`

我们可以发现调用了`Thread.sleep()`方法，该方法的作用是让线程暂时停止执行，所以是插件故意暂停执行导致了阻塞

根据

    [13:00:45 ERROR]:               org.maxgamer.quickshop.util.MsgUtil.loadGameLanguage(MsgUtil.java:237)

可以看出是QuickShop造成的。

得出结论：QuickShop插件故意暂停执行导致了阻塞

### 案例二
来源:https://www.mcbbs.net/thread-1158938-1-1.html
（重复报错已删除）

    [12:39:52 INFO]: Done (48.832s)! For help, type "help"
    [12:39:52 INFO]: Timings Reset
    [12:39:53 INFO]: [CS-CoreLib2] Loaded Protection Module "Residence v4.9.2.3"
    [12:39:53 INFO]: [Residence] Enabled compatability with SlimeFun plugin
    [12:39:53 INFO]: Could not perform update. Is the Github down?
    [12:39:53 WARN]: java.io.IOException: Server returned HTTP response code: 403 for URL: https://api.github.com/repos/ClayCoffee/ClayTech/releases
    [12:39:53 WARN]:        at java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1924)
    [12:39:53 WARN]:        at java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1520)
    [12:39:53 WARN]:        at java.base/sun.net.www.protocol.https.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:250)
    [12:39:53 WARN]:        at java.base/java.net.URL.openStream(URL.java:1140)
    [12:39:53 WARN]:        at cn.claycoffee.ClayTech.ClayTechUpdater$1.run(ClayTechUpdater.java:33)
    [12:39:53 WARN]:        at org.bukkit.craftbukkit.v1_16_R3.scheduler.CraftTask.run(CraftTask.java:99)
    [12:39:53 WARN]:        at org.bukkit.craftbukkit.v1_16_R3.scheduler.CraftAsyncTask.run(CraftAsyncTask.java:54)
    [12:39:53 WARN]:        at com.destroystokyo.paper.ServerSchedulerReportingWrapper.run(ServerSchedulerReportingWrapper.java:22)
    [12:39:53 WARN]:        at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
    [12:39:53 WARN]:        at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
    [12:39:53 WARN]:        at java.base/java.lang.Thread.run(Thread.java:834)
    [12:40:03 WARN]: java.lang.reflect.InvocationTargetException
    [12:40:03 WARN]:        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    [12:40:03 WARN]:        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    [12:40:03 WARN]:        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    [12:40:03 WARN]:        at java.base/java.lang.reflect.Method.invoke(Method.java:566)
    [12:40:03 WARN]:        at io.izzel.taboolib.module.inject.TScheduleLoader$2.run(TScheduleLoader.java:65)
    [12:40:03 WARN]:        at org.bukkit.craftbukkit.v1_16_R3.scheduler.CraftTask.run(CraftTask.java:99)
    [12:40:03 WARN]:        at org.bukkit.craftbukkit.v1_16_R3.scheduler.CraftScheduler.mainThreadHeartbeat(CraftScheduler.java:468)
    [12:40:03 WARN]:        at net.minecraft.server.v1_16_R3.MinecraftServer.b(MinecraftServer.java:1293)
    [12:40:03 WARN]:        at net.minecraft.server.v1_16_R3.DedicatedServer.b(DedicatedServer.java:377)
    [12:40:03 WARN]:        at net.minecraft.server.v1_16_R3.MinecraftServer.a(MinecraftServer.java:1208)
    [12:40:03 WARN]:        at net.minecraft.server.v1_16_R3.MinecraftServer.w(MinecraftServer.java:996)
    [12:40:03 WARN]:        at net.minecraft.server.v1_16_R3.MinecraftServer.lambda$a$0(MinecraftServer.java:173)
    [12:40:03 WARN]:        at java.base/java.lang.Thread.run(Thread.java:834)
    [12:40:03 WARN]: Caused by: java.lang.NoSuchMethodError: me.clip.placeholderapi.PlaceholderAPIPlugin.getExpansionCloud()Lme/clip/placeholderapi/expansion/cloud/ExpansionCloudManager;
    [12:40:03 WARN]:        at me.arasple.mc.trchat.utils.Vars.downloadExpansions(Vars.java:33)
    [12:40:03 WARN]:        at me.arasple.mc.trchat.utils.Vars.downloadExpansions(Vars.java:24)
    [12:40:03 WARN]:        ... 13 more
