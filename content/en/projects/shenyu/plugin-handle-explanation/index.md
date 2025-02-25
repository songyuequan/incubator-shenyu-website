---
title: Plugin Handle Explanation
keywords: plugin
description: plugin handle explanation
---

## Explanation

* In our Shenyu-Admin background, each plugin uses the Handle field to represent a different processing, and plugin processing is used to manage and edit custom processing fields in JSON.
* This feature is mainly used to support the plugin handling template configuration

## Table Design

* sql
```sql
CREATE TABLE IF NOT EXISTS `plugin_handle` (
  `id` varchar(128) NOT NULL,
  `plugin_id` varchar(128) NOT NULL COMMENT 'plugin id',
  `field` varchar(100) NOT NULL COMMENT 'field',
  `label` varchar(100) DEFAULT NULL COMMENT 'label',
  `data_type` smallint(6) NOT NULL DEFAULT '1' COMMENT 'data type 1 number 2 string 3 select box',
  `type` smallint(6) NULL COMMENT 'type, 1 means selector, 2 means rule',
  `sort` int(4)  NULL COMMENT 'sort',
  `ext_obj` varchar(1024) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT 'extra configuration (json format data)',
  `date_created` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'create time',
  `date_updated` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'update time',
  PRIMARY KEY (`id`),
  UNIQUE KEY `plugin_id_field_type` (`plugin_id`,`field`,`type`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE = utf8mb4_unicode_ci ROW_FORMAT = Dynamic;
```

## Tutorial

eg. When we developed the `springCloud` plugin, the rule table needed to store some configuration into the handle field,
Configure the corresponding entity class as follows:

```java
    public class SpringCloudRuleHandle implements RuleHandle {
    
        /**
         * this remote uri path.
         */
        private String path;
    
        /**
         * timeout is required.
         */
        private long timeout = Constants.TIME_OUT;    
    }
```

**step1.** We can go directly to `PluginHandle` page `http://localhost:9095/#/config/pluginhandle` add/edit handle fields

![](/img/shenyu/basicConfig/pluginHandle/01.png)

**step2.** For example, add a string type field path and a numeric type timeout to springCloud

![](/img/shenyu/basicConfig/pluginHandle/02.png)

**step3.** Finally, you can enter path, timeout and commit to the handle field when you add a rule in the plugin rule configuration page

![](/img/shenyu/basicConfig/pluginHandle/03.png)

*Note: If data_type is configured to be `3` `selection box`, the input field drop-down selection on the new rule page is displayed by going to the [shenyu_dict](../dictionary-management) table to find all the options available.*

* The Sentinel plug-in, for example, is shown below:

![](https://yu199195.github.io/images/soul/sentinel-rule-handle.png)
