---
title: Linux环境下Drupal 8页面Not Found的解决办法
date: 2018-11-02 00:18:54
tags:
- Linux
- Drupal
- LAMP
---

最近在学习前端开发，在LAMP环境下安装了drupal 8.62。完成安装看到主页时本以为已经万事大吉，没想到随便点进一个页面都是一个大大的404 Not Found. 综合网上的一些文章[^1]，尽管没有搞清楚具体原理，但是这个问题大致跟以下几个方面有关：

> 1. .htaccess文件没有正确配置
> 2. .conf文件的中的重写权限没有开启
> 3. 没有启用rewrite模块

根据以上几点，总结出以下解决方法，按照步骤操作应该可以修复。

1. **检查`.htaccess`文件是否已经拷贝到drupal主目录下。**

   由于`.htaccess`文件默认隐藏，在安装drupal时很容易忽视。如果`/var/www/html/drupal`目录下没有`.htaccess`文件，则需要从压缩包中拷贝。

2. **修改.conf文件中的重写权限**

   需要修改`/etc/apache2/sites-available/000-default.conf`和`/etc/apache2/apache2.conf` 文件中的内容：

   在`000-default.conf`中插入：

   ```
       <Directory /var/www/html/000-default>
               Options Indexes FollowSymLinks MultiViews
               AllowOverride All
               Order allow,deny
               allow from all
       </Directory
   ```

   在`apache2.conf`中找到以下代码：

   ```
   <Directory />
   	Options FollowSymLinks
   	AllowOverride None
   	Require all denied
   </Directory>
   
   <Directory /usr/share>
   	AllowOverride None
   	Require all granted
   </Directory>
   
   <Directory /var/www/>
   	Options Indexes FollowSymLinks
   	AllowOverride None
   	Require all granted
   </Directory>
   
   <Directory /var/www/html/drupal>
           Options Indexes FollowSymLinks MultiViews
           AllowOverride None
           Order allow,deny
           allow from all
   </Directory>
   ```

   将其中的`AllowOverride None`全部改为`AllowOverride All`，允许所有用户组进行重写。

3. **启用rewrite模块**

   使用以下命令开启：

   ```shell
   $ sudo a2enmod rewrite
   ```

   重启apache2：

   ```shell
   $ sudo service apache2 restart
   ```



现在drupal应该可以正常使用了！:raised_hands:

[^1]: https://www.drupal.org/node/2134281