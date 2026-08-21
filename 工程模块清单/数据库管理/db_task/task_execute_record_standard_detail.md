# task_execute_record_standard_detail (db_task)

- 用途：任务模板上位机执行时的一些配置
- 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | int | 主键 |
| task_execute_record_id | int | 关联的任务模id |
| cover_install | int | 执行前卸载安装。卸载完后进行安装。对应上位机需要的字段coverInstall。1为开启卸载安装，0为不开启 |
| overwrite_install | int | 执行前的覆盖安装，1为覆盖安装，0为不覆盖 |
| clean_data | int | 执行后清理数据。1为清理，0为不清理。-1为使用上位机自己的配置 |
| uninstall | int | 执行后不卸载app。1为不卸载，0为卸载。 |
| install | int | 安装应用； 不下发该字段时，默认值为-1；0代表false； 1 代表true |
| start_up | int | 启动应用，1为启动，0为不启动 |
| keep_app | int | 执行后关闭应用。1为不关闭，0为关闭 |
| video | int | 是否录制视频，0为不录制，1为录制，2为仅失败录制 |
| resign | int | ios重签配置，1为重签，0为不重签 |
| task_execute_mode | int | 上位机执行的形式。1为简单形式，表示对应的配置都来源于后台配置，有该值时下面的配置每次执行需要进行额外查询。2为前端配置模式，对应的配置使用传参信息。用于重测的反展示 |
| termination_on_error | int | 当前脚本执行错误 是否终止下边的脚本继续执行 0：继续执行 1、终止后续脚本 |
| step_global_timeout | int | 步骤全局的超时时间，单位毫秒 |
| custom_file_path | varchar | 采集app输出的日志的位置 |
| log_collection | int | 是否记录日志 0关1开 |
| performance_data_collection | int | 是否记录性能数据 0关1开 |
| traversal_time | bigint | 遍历时长。具体含义目前未知 |
| monkey_time | bigint | monkey时长。具体含义目前未知 |
| retry_num | int | 脚本失败后重测次数 |
| create_time | datetime | 创建时间 |
| update_time | datetime | 更新时间 |
| app_step_global_time_out | int | 用例模板 app脚本步骤全局的超时时间 |
| web_step_global_time_out | int | 用例模板 web脚本步骤全局的超时时间 |
| pc_step_global_time_out | int | 用例模板 pc脚本步骤全局的超时时间 |
| android_global_control_accelerated | int | 全局智能加速 0 全局 1关闭 2开启 3自动 |
| fail_step_texts | int | 是否返回 脚本失败后截图信息文本 |
| harmony_global_control_accelerated | int | 鸿蒙智能加速 |
| i_o_s_global_control_accelerated | int | iOS智能加速 |

- 关联接口：
  - [TaskExecuteRecordStandardDetailController](../../任务管理服务/07-开放接口文档/任务报告/TaskExecuteRecordStandardDetailController.md)
  - [TaskExecuteRecordController](../../任务管理服务/07-开放接口文档/执行记录/TaskExecuteRecordController.md)

- 关联表：
  - 主表 [task_execute_record](task_execute_record.md)
