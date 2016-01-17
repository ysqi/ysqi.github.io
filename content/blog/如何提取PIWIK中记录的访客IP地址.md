---
date: 2015-09-08
title: 如何提取Piwik中记录的访客IP地址
slug: piwik-get-visit_ip
tags : 
- piwik
- ip
- 教程
Topics: 
- 大数据
---

 最近在使用Piwik做网站数据统计，基于PIWIK做二次开发，各种文档，而昨天死活不知道PIWIK中记录访客IP的格式是什么，拉个粑粑后茅塞顿开，下面记记录下如何在PIWIK中获取访客的IP地址信息。
 
 
 
 ### 访客IP地址Piwik存储在哪里？
 Piwk将访客信息记录在表`log_visit`中，当然访客IP也是跟随访客继续记录的，也在这个表中，字段是`location_ip` ,该表的表结构如下：

+ `idsite`:the ID of the the website it was tracked for
+ `idvisitor`:a visitor ID (an 8 byte binary string)
+ `visitor_localtime`:the visit datetime in the visitor's time of day
+ `visitor_returning`:whether the visit is the first visit for this visitor or not
+ `visitor_count_visits`:the number of visits the visitor has made up to this one
+ `visitor_days_since_last`:the number of days since this visitor's last visit (if any)
+ `visitor_days_since_order`:the number of days since this visitor's last order (if any)
+ `visitor_days_since_first`:the number of days since this visitors' first visit
+ `visit_first_action_time`:the datetime of the visit's first action
+ `visit_last_action_time`:the datetime of the visit's last action
+ `visit_exit_idaction_url`:the ID of the URL action type of the visit's last action
+ `visit_exit_idaction_name`:the ID of the page title action type of the visit's last action
+ `visit_entry_idaction_url`:the ID of the URL action type of the visit's first action
+ `visit_entry_idaction_name`:the ID of the page title action type of this visit's first action
+ `visit_total_actions`:the count of actions performed during this visit
+ `visit_total_searches`:the count of site searches performed during this visit
+ `visit_total_events`:the count of custom events performed during this visit
+ `visit_total_time`:the total elapsed time of the visit
+ `visit_goal_converted`:whether this visit converted a goal or not
+ `visit_goal_buyer`:whether the visitor ordered something during this visit or not
+ `referer_type`:the type of this visitor's referrer. Can be one of the following values:
+ `Common::REFERRER_TYPE_DIRECT_ENTRY = 1`:If set to this value, other referer_... fields have no meaning.
+ `Common::REFERRER_TYPE_SEARCH_ENGINE = 2`:If set to this value, referer_url is the url of the search engine and referer_keyword is the keyword used (if we can find it).
+ `Common::REFERRER_TYPE_WEBSITE = 3`:If set to this value, referer_url is the url of the website.
+ `Common::REFERRER_TYPE_CAMPAIGN = 6`:If set to this value, referer_name is the name of the campaign.
+ `referer_name`:referrer name; its meaning depends on the specific referrer type
+ `referer_url`:the referrer URL; its meaning depends on the specific referrer type
+ `referer_keyword`:the keyword used if a search engine was the referrer
+ `config_id`:a hash of all the visit's configuration options, including the OS, browser name, browser version, browser language, IP address and all browser plugin information
+ `config_os`:a short string identifiying the operating system used to make this visit. See UserAgentParser for more info
+ `config_browser_name`:a short string identifying the browser used to make this visit. See UserAgentParser for more info
+ `config_browser_version`:a string identifying the version of the browser used to make this visit
+ `config_resolution`:a string identifying the screen resolution the visitor used to make this visit (eg, '1024x768')
+ `config_pdf`:whether the visitor's browser can view PDF files or not
+ `config_flash`:whether the visitor's browser can view flash files or not
+ `config_java`:whether the visitor's browser can run Java or not
+ `config_director:
+ `config_quicktime`:whether the visitor's browser uses quicktime to play media files or not
+ `config_realplayer`:whether the visitor's browser can play realplayer media files or not
+ `config_windowsmedia`:whether the visitor's browser uses windows media player to play media files
+ `config_gears:
+ `config_silverlight`:whether the visitor's browser can run silverlight programs or not
+ `config_cookie`:whether the visitor's browser has cookies enabled or not
+ `location_ip`:the IP address of the computer that the visit was made from. Can be anonymized
+ `location_browser_lang`:a string describing the language used in the visitor's browser
+ `location_country`:a two character string describing the country the visitor was located in while visiting the site. Set by the UserCountry plugin.
+ `location_region`:a two character string describing the region of the country the visitor was in. Set by the UserCountry plugin.
+ `location_city`:a string naming the city the visitor was in while visiting the site. Set by the UserCountry plugin.
+ `location_latitude`:the latitude of the visitor while he/she visited the site. Set by the UserCountry plugin.
+ `location_longitude`:the longitude of the visitor while he/she visited the site. Set by the UserCountry plugin.
+ `custom_var_k1`:the custom variable name of the visit in the first slot for visit custom variables.
+ `custom_var_v1`:the custom variable value of the visit in the first slot for visit custom variables.
+ `custom_var_k2`:the custom variable name of the visit in the second slot for visit custom variables.
+ `custom_var_v2`:the custom variable value of the visit in the second slot for visit custom variables.
+ `custom_var_k3`:the custom variable name of the visit in the third slot for visit custom variables.
+ `custom_var_v3`:the custom variable value of the visit in the third slot for visit custom variables.
+ `custom_var_k4`:the custom variable name of the visit in the fourth slot for visit custom variables.
+ `custom_var_v4`:the custom variable value of the visit in the fourth slot for visit custom variables.
+ `custom_var_k5`:the custom variable name of the visit in the fifth slot for visit custom variables.
+ `custom_var_v5`:the custom variable value of the visit in the fifth slot for visit custom variables.


### 为什么Piwik中记录的访客IP为空？

这是因为Piwik可以设置隐私模式，不跟踪记录用户的IP信息，菜单位置：网站管理->隐私设置-> IP跟踪设置

### Piwik表中log_visit记录的location_Ip是乱码

这个真不是乱码？ 而是这个location_IP格式是BOLB存储，需要`Hex()`转换下查看。

### 从Piwik中获取访客IP地址信息

从piwik.log_visit表中字段获取：hex(location_ip),其结果是将IP地址按16位进制存储的。如下：

```
select hex(location_ip) from piwik.log_visit
```
假设访问的IP是：`192.168.1.101`  => 十六进制：`C0.A8.1.65`

则存储的十六进制数据为`C0A80165`


这样反向便可以提取Piwik中记录的十六进制访客IP地址信息。





