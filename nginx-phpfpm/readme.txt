tail -n 100 /opt/rh/php54/root/var/log/php-fpm/error.log

Error log:

2016/05/09 14:53:03 [error] 26837#0: *1564600 connect() to unix:/var/run/php5-fpm.sock failed (11: Resource temporarily unavailable) while connecting to upstream, client: 99.236.212.113, server: www.example.com, request: "GET / HTTP/1.1", upstream: "fastcgi://unix:/var/run/php5-fpm.sock:", host: "www.example.com"

php-fpm optimization guide
==========================
max_children

ps -ylC php-fpm --sort:rss

    -y = Do not show flags; show rss in place of addr.  This option can
              only be used with -l.
    -l = long format
    -C = command name supplied

if the average php-fpm memory consumption is 50MB then divide your RAM(spare some some for os and mysql) by the average memory consumed by php-fpm

eg. if you have 4G ram then subtract 1GB(for OS) 1G(for mysql) , 4-1-1=2G(for phpfpm)

On this basis

2048/50 = 40.96 ( set 40 or 41 as pm.max_children )

max_spare_server should be 2x or 4x times number of cores 
eg.  if 8 core CPU then max_spare_server can start with 8X2= 16 and go up to 8X4=32 

start_servers value should be around half the size of max_spare_servers
consider dropping max_requests to around 500

pm value can also be set to static or ondemand.

Also, in addition to dynamic, the pm value can also be set to static or ondemand. Static will always have a fixed number of servers running at any given time. This is good if you have a consistent amount of users or you want to guarantee you don't breach the max memory. Ondemand will only start processes when there is a need for them. The downside is obviously having to constantly start/kill processes which will usually translate into a very slight delay in request handling. The the upside, you only use resources when you need them. "Dynamic" always starts X amount of servers specified in the start_servers option and creates additional processes on an as-need basis.

If you are still experiencing issues with memory then consider changing pm to ondemand.


https://stackoverflow.com/questions/25097179/warning-pool-www-seems-busy-you-may-need-to-increase-pm-start-servers-or-pm/25098060#25098060

#nginx error connect to php5-fpm.sock failed (13: Permission denied)

Open /etc/php5/fpm/pool.d/www.conf
Uncomment all permission lines, like:

    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0660
    
Restart fpm - 
    sudo service php5-fpm restart
Note: if your webserver runs as user other than www-data, you will need to update the www.conf file accordingly

# Increase file upload size limit in PHP-Nginx

vim /etc/php5/fpm/php.ini

    upload_max_filesize = 100M
    post_max_size = 100M
    #max_input_time; it is unlimited by default you can adjust if not
post_max_size should always be larger than upload_max_filesize but for large numbers like 100M you can safely make them equal.

Add following line to http{..} block in nginx config:

    http   {
	#...
        client_max_body_size 100m;
	#...
    }
    Note: For very large files, you may need to change value of client_body_timeout parameter. Default is 60s.
    
    service php5-fpm reload
    service nginx reload
    
    
