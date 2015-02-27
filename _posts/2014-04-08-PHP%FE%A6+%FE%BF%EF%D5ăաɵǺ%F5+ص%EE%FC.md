---
layout: post
title: 140408 PHP线程原子性保持
---

## 1. Php线程如何保持原子性进行执行：

http://liuw.net/PHP-zhong-cao-zuo-xin-hao.html

实际文章已经很清楚的描述了通过While方式做后台程序监控队列，读取输出。但是希望在某次请求的时候结束当次的请求。说起来很拗口.说白了就是有一个用户发了一篇文章是垃圾文，要放弃掉当前循环，如果通过exit则整个后台程序都会被断掉。

作者源码实际描述的时候关键部分被隐去了：

	function sig_handler($signo)
	{
	     global $stop_flag;
	     switch ($signo) {
	         case SIGKILL:
	         case SIGTERM:
	             // handle shutdown tasks
	             $stop_flag = true;
	             break;
	         case SIGHUP:
	             // handle restart tasks
	             break;
	         case SIGUSR1:
	             echo "Caught SIGUSR1...\n";
	             break;
	         default:
	             // handle all other signals
	     }
	 
	}
	 
	declare(ticks = 1);
	$stop_flag = false;
	// setup signal handlers
	pcntl_signal(SIGTERM, "sig_handler");
	pcntl_signal(SIGHUP,  "sig_handler");
	pcntl_signal(SIGUSR1, "sig_handler");
	 
	while(!$stop_flag) {
	    $data = read_from_queue();
	    process_with($data);
	}

PHP手册关于pcntl_signal使用用Demo中有关于如何通知当前线程去结束当次操作。

	posix_kill(posix_getpid(), SIGUSR1);

把这个放在函数read_from_queue中就可完整实现类似作者原子性操作。


## 2. 关于PHP Ticks机制

在PHP代码开始的时候添加*declare(ticks=n)*后表示：在当前scope内，每执行N句internal statements（opcodes），就会中断当前的业务语句，去执行通过register_tick_function注册的函数（如果存在的话），然后再继续之前的代码。需要注意的是这里的N是指的PHP的一些OPCODE，而OPCODE与我们见到的PHP语句却不是一一对应的。

**常见statement定义：**

	(1) 简单语句：空语句（就一个；号），return,break,continue,throw, goto,global,static,unset,echo, 内置的HTML文本，分号结束的表达式等均算一个语句。

	(2) 复合语句：完整的if/elseif,while,do...while,for,foreach,switch,try...catch等算一个语句。

	(3) 语句块：{} 括出来的语句块。

	(4) 最后特别的：declare块本身也算一个语句(按道理declare块也算是复合语句，但此处特意将其独立出来)。

## 3. PCNTL使用Ticks作为信号处理机制

PCNTL使用ticks机制来作为信号处理机制（signal handle callback mechanism），可以最小程度地降低处理异步事件时的负载。这里的关键在于PCNTL扩展的模块初始化函数（PHP_MINIT_FUNCTION(pcntl)）。在此模块做模块初始化时，它会调用： php_add_tick_function(pcntl_signal_dispatch);将pcntl的分发执行函数添加到ticks机制的调用函数中去，从而当ticks触发时就会调用PCNTL扩展函数中指定的所有方法。

## 4. 参考: 
	
1. http://www.phppan.com/2013/02/php-ticks/
2. http://www.php.net/manual/zh/function.pcntl-signal.php
3. http://my.oschina.net/Jacker/blog/32936