### admim

```sql
CREATE TABLE `apg_health_calendar_admin` (
  `id` varchar(50) NOT NULL,
  `username` varchar(50) NOT NULL,
  `password` varchar(256) NOT NULL,
  `remark` varchar(128) DEFAULT NULL,
  `is_deleted` tinyint DEFAULT '0' COMMENT '是否删除 1-是 0-否',
  `create_time` timestamp NULL DEFAULT NULL,
  `update_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='金管家健康日历 - admin';
```

### config

```sql
CREATE TABLE `apg_health_calendar_config` (
  `id` varchar(50) NOT NULL,
  `system_time` timestamp NOT NULL COMMENT '日历时间',
  `calendar_pic` varchar(1024) NOT NULL COMMENT '日历底图',
  `question` varchar(45) NOT NULL COMMENT '问题',
  `answer` varchar(90) NOT NULL COMMENT '答案',
  `important_information` varchar(45) NOT NULL COMMENT '重要信息',
  `title` varchar(20) NOT NULL COMMENT '业务标题',
  `address` varchar(45) NOT NULL COMMENT '业务地址',
  `button_content` varchar(5) NOT NULL COMMENT '按钮文案',
  `button_url` varchar(512) NOT NULL COMMENT '按钮链接',
  `befit_content` varchar(3) NOT NULL COMMENT '宜 文案',
  `tabu_content` varchar(3) NOT NULL COMMENT '忌 文案',
  `remark` varchar(128) DEFAULT NULL,
  `is_deleted` tinyint DEFAULT '0' COMMENT '是否删除 1-是 0-否',
  `create_time` timestamp NULL DEFAULT NULL,
  `update_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_system_time` (`system_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='金管家健康日历 - 配置表';
```

