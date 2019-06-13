+++
title = "{{ replace .Name "-" " " | title }}"  # 文章标题
date = {{ .Date }}  # 自动添加日期信息
draft = true  # 设为false可被编译为HTML，true供本地修改
tags = [""]  # 文章标签，可设置多个，用逗号隔开。Hugo会自动生成标签的子URL
comments = true  # 是否开启Disqus评论功能
share = true  # 是否开启分享
+++
--------------------- 
作者：xiepeng 
版权声明：本文为博主原创文章，转载请附上博文链接！