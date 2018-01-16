---
layout: post
---

最近想给Github Page加个回复，Google之，最后决定自己搭个[Discourse][Discourse]

* [官方默认安装过程][Discourse_Beginner_Docker_install_guide]，其中SMTP相关的参数如下：
```
DISCOURSE_SMTP_ADDRESS: smtp.qq.com
DISCOURSE_SMTP_PORT: 587
DISCOURSE_SMTP_USER_NAME: 你的QQ邮箱
DISCOURSE_SMTP_PASSWORD: '16位SMTP授权码，不是你的邮件密码'
DISCOURSE_SMTP_ENABLE_START_TLS: true
DISCOURSE_SMTP_AUTHENTICATION: login
```

* `$ vim /var/discourse/containers/app.yml`，修改其中一行：
```
- exec: rails r "SiteSetting.notification_email='你的QQ邮箱'"
```
`$ /var/discourse/launcher rebuild app`

[Discourse]: https://www.discourse.org
[Discourse_Beginner_Docker_install_guide]: https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md
[Discourse_Install_On_CentOs]: https://blog.yumc.pw/posts/Discourse-Install-On-CentOs/
