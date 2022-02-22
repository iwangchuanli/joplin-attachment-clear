# joplin-attachment-clear

删除在 Joplin 中未引用的附件（资源）。

**警告：始终备份和使用风险自负!**

使用 Joplin 2.1.9 (prod, win32)  + Python 3.8。

## 条件注意

1. 仅在笔记历史中引用的资源被标记为孤立的。
2. 如果同步不完整，或者Joplin在所有设备上的状态不一样，这个脚本可能会导致当前版本使用的资源被误删除。

## 要求

Python 3.6+

没有第三方依赖。

## 怎么使用

将注释导出到 JEX 文件时，将忽略未引用的附件。 JEX 文件只是一个 TAR 文件，通过读取其 `resources` 文件夹下的附件 ID，可以创建引用附件的列表。

在 Joplin 中，可以通过 `Tools` - `Note Attachments...` 找到所有附件的完整列表。 Joplin Data API 还允许以编程方式获取列表。

因此，可以使用所有附件集和所引用附件集之间的差异来计算未引用（孤立）的附件。

### 相关链接

- 孤立资源问题：[orphaned resources don’t get deleted - Issue #932 - Joplin](https://github.com/laurent22/joplin/issues/932)
- 基于直接 DB 访问的清理解决方案：[patrick-atgithub/joplintool](https://github.com/patrick-atgithub/joplintool)
- Joplin Batch Web：用于执行 Joplin 清洁作业的 Web 界面 [Joplin Batch (rxliuli.com)](https://joplin-utils.rxliuli.com/joplin-batch-web/#/unusedResource)。

## 用法

```
用法：vacuum.py [-h] [--port PORT] [--token TOKEN] [--limit LIMIT] [--confirm] [--test-del-1] jex_path

位置参数：
  jex_path JEX 文件的路径

可选参数：
  -h, --help 显示此帮助信息并退出
  --port PORT 用于连接 Joplin 的端口，留空为自动查询
  --token TOKEN 覆盖 API 令牌
  --limit LIMIT 从 Joplin 查询附件的分页限制
  --confirm 确认删除
  --test-del-1（用于测试目的）删除一个未提及的附件。需要与确认一起使用。
```

## 推荐步骤

0. 在继续之前，请确保您所有设备上的 Joplin 同步到相同的状态。

1. 启动 Joplin，将所有笔记导出到 JEX 文件。 （`文件 - 全部导出 - JEX`）

2. 备份 Joplin 数据库，关闭 Joplin 并复制 `joplin-desktop` 文件夹。 （可以使用“设置 - 常规”面板找到完整路径）

3. 暂时禁用自动同步，以防止在真空过程中触发同步。 （`Settings - Synchronization - Synchronization interval - Disabled`）另外，不要在其他客户端上执行任何更新以避免版本控制问题。

4. 以试运行模式运行脚本。对于第一次运行，您需要从 Joplin GUI 授予访问权限。检查找到的孤立文件是否实际上是孤立的。

   ```重击
   python Vacuum.py [JEX_PATH]
   ```

5. 使用 `--test-del-1` 和 `--confirm` 标志运行脚本。这将删除 1 个孤立的附件，同时保持其他附件完好无损。此步骤检查是否一切正常，包括删除操作。尝试在此步骤之后启动同步。如果 Joplin 显示 `Removing remote object: 1` 并完成同步，那么就可以开始了。

   ```重击
   python Vacuum.py [JEX_PATH] --confirm --test-del-1
   ```

6. 如果工作正常，只使用 `--confirm` 标志来删除所有孤立的附件。

   ```重击
   python Vacuum.py [JEX_PATH] --确认
   ```

7. 执行手动同步（通过按`Ctrl+S` 或单击`Synchronize` 按钮），并检查`Removing remote object` 的计数是否符合预期。如果手动同步成功完成，那么您可以启用自动同步，以及在其他设备上手动同步。

##  附加信息

1.可以在没有人为干预的情况下执行vacuum过程（例如使用`cron`来调度）。但不建议这样做。首先，您将需要其他可以以编程方式创建 JEX 导出的工具。然后，在不禁用所有写入的情况下执行真空可能会导致状态不一致。

## 示例输出

```
$ python vacuum.py .\2021-07-21.jex --confirm
trying port 41184 200 OK port find!
loading token from file
requesting page 1... 200 OK got 26, total 26, has_more False
referred: 17, all 26
orphaned count: 9

id - filename
--------------------------------------
60414bdd2c3a493a8c9a06be82130858 - C7intaNotes_8aaaae945ac54d0588df35428e123db5.png
2f67c439207a4cb6bcb29068bf21d8fa - cintaanltesa-14100-3_c971215474ae42c2b30446b7f0217220.jpg
defd6c6ff6914ca98b960d2170b721ab - 3691d864f3730b2dcbc59c116338cc27.png
989b920202a241cbb4dde6c381d6884a - df83c2a0c7da98c08b00cfd6ea08d636.png
2556556e5a0c4b969b0b188154aa4b6d - 6ad4a4e235c9f6384b9da11c323f0fcd.png
31206111f5a54d97ba312f2257b62c46 - 640_wx_fmt_gif_tp_webp_wxfrom_5__f9b95901a1e04478a1b6b84682acfd26.gif
1765a4bc9db54414968c2d6265683b54 - cintaanltesa-14100-5_92fcfd8df37b48cc9cc251ed3a05e277.jpg
0a9b69625f1d4592ad3727f95676074f - cintaanltesa-14100-4_8a657a4c6a644daca790f6ccf5505094.jpg
5c7ec35321dc4e9797ba479d4d1d9fd2 - 5af424e5adc4970b6e68868204eb3850.png
--------------------------------------

deleting 1 of 9, id=60414bdd2c3a493a8c9a06be82130858 200 OK deleted
deleting 2 of 9, id=2f67c439207a4cb6bcb29068bf21d8fa 200 OK deleted
deleting 3 of 9, id=defd6c6ff6914ca98b960d2170b721ab 200 OK deleted
deleting 4 of 9, id=989b920202a241cbb4dde6c381d6884a 200 OK deleted
deleting 5 of 9, id=2556556e5a0c4b969b0b188154aa4b6d 200 OK deleted
deleting 6 of 9, id=31206111f5a54d97ba312f2257b62c46 200 OK deleted
deleting 7 of 9, id=1765a4bc9db54414968c2d6265683b54 200 OK deleted
deleting 8 of 9, id=0a9b69625f1d4592ad3727f95676074f 200 OK deleted
deleting 9 of 9, id=5c7ec35321dc4e9797ba479d4d1d9fd2 200 OK deleted
Done.