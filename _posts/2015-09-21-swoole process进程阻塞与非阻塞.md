---
layout: post
title: "php swoole process进程阻塞与非阻塞"
categories: php
description: 
keywords: php,swoole,多进程
---


> 有一个定时脚本的需求，要求是在同一时刻异步的执行多个任务。采用Swoole扩展满足需求，但在swoole_process使用中遇到阻塞与非阻塞的疑惑。

### 在timer创建process进程

>按照官方的demo,在swoole_timer_add种创建多个process进程。

```php
swoole_timer_add(2000, function() {
    $tasks = array(
        array('uniqid'=>'A1'),
        array('uniqid'=>'B2'),
    );
    foreach ($tasks as $task) {
        $process = new swoole_process('callback_function', true);
        $process->write(json_encode($task));
        $process->start();
        $process->wait();
    } 
});

function callback_function(swoole_process $worker){
    $task   = json_decode($worker->read(), true);
    file_put_contents('/tmp/crontab.log', '['.date('H:i:s').'] =>'.$task['uniqid']."\r\n", FILE_APPEND);
	sleep(rand(1,5));
}
```

>结果:两次process写入的时间不一致,可以看出是阻塞运行的。

```bash
[root@ebf82a3d9afa process]# tail -f /tmp/crontab.log 
[19:12:22] =>A1
[19:12:27] =>B2
[19:12:40] =>A1
[19:12:43] =>B2
[19:12:58] =>A1
[19:13:00] =>B2
```


### 在timer创建process进程,用exec执行

>也是在swoole_timer_add种创建多个process进程,在process通过exec方式执行

```php
swoole_timer_add(2000, function() {
    $tasks = array(
        array('uniqid'=>'A1'),
        array('uniqid'=>'B2'),
    );
    foreach ($tasks as $task) {
        $process = new swoole_process('callback_function', true);
        $process->write(json_encode($task));
        $process->start();
        $process->wait();
    } 
});

function callback_function(swoole_process $worker){
    $task = json_decode($worker->read(), true);
    $worker->exec(
        '/usr/local/php-5.4.40/bin/php', 
        array('/data/www/process/demo.php', $task['uniqid'])
    );
}

demo.php
$time = time();
file_put_contents('/tmp/crontab.log', '['.date('H:i:s', $time).'] =>'.$argv[1]."\r\n", FILE_APPEND);
sleep(rand(1,5));
```

>结果:同样是阻塞的，exec不是在孙进程中运行，与父进程仍然是父子关系。

```bash
[root@ebf82a3d9afa process]# tail -f /tmp/crontab.log 
[18:30:25] =>A1
[18:30:29] =>B2
[18:30:42] =>A1
[18:30:44] =>B2
[18:30:52] =>A1
[18:30:57] =>B2
[18:31:10] =>A1
[18:31:15] =>B2
[18:31:29] =>A1
[18:31:34] =>B2
[18:31:46] =>A1
[18:31:50] =>B2
```

### 在timer创建process进程，设置wait(false)

>按照官方的demo,在swoole_timer_add种创建多个process进程,但在wait时设为false

```php
swoole_timer_add(2000, function() {
    $tasks = array(
        array('uniqid'=>'A1'),
        array('uniqid'=>'B2'),
    );
    foreach ($tasks as $task) {
        $process = new swoole_process('callback_function', true);
        $process->write(json_encode($task));
        $process->start();
        $process->wait(false);
    } 
});

function callback_function(swoole_process $worker){
    $task   = json_decode($worker->read(), true);
    file_put_contents('/tmp/crontab.log', '['.date('H:i:s').'] =>'.$task['uniqid']."\r\n", FILE_APPEND);
    sleep(rand(1,5));
    $worker->exit(0);
}
```

>结果:非阻塞执行, 缺点是若执行等待时间较长会占用进程资源。如果没有及时exit会产生僵尸进程。

```bash
[root@ebf82a3d9afa process]# tail -f /tmp/crontab.log 
[19:07:00] =>A1
[19:07:00] =>B2
[19:07:02] =>A1
[19:07:02] =>B2
[19:07:04] =>B2
[19:07:04] =>A1
[19:07:06] =>B2
[19:07:06] =>A1
[19:07:08] =>A1
[19:07:08] =>B2
[19:07:10] =>A1
[19:07:10] =>B2
[19:07:12] =>A1
[19:07:12] =>B2
[19:07:14] =>B2
[19:07:14] =>A1
^C
[root@ebf82a3d9afa process]# ps -A -o stat,ppid,pid,cmd | grep php
S        1 27768 /usr/local/php-5.4.40/bin/php test.php
Z    27768 27884 [php] <defunct>
Z    27768 27885 [php] <defunct>
Z    27768 27886 [php] <defunct>
S    27768 27887 /usr/local/php-5.4.40/bin/php test.php
S    27768 27888 /usr/local/php-5.4.40/bin/php test.php
S    27768 27889 /usr/local/php-5.4.40/bin/php test.php
S+       1 27891 grep php
```
