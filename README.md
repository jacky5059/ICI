# 一、介绍    

## ICI是什么？

    ICI是一个基于CI扩展出来的框架;      

## 做这个框架的愿景是什么？     

    对可预想到的共性问题提供优雅的解决方案，为框架增加实用的工具，减少项目前期框架性工作。

## 为什么要基于CI开发？

    基于CI开发，并不是因为CI有多好（相反CI其实挺烂）。
	基于CI开发，是因为我对CI的源码很熟，而且其它框架也好不到哪里去，它们各有各的烂。     
    所谓“君子性非已也，善假于物也”，框架再烂，至少很稳定，而且做了大量基础工作，我没必要重复造轮子。   
    我要做的是把已有的工具类包装成更好用的工具类，实在丑陋的代码，就重写覆盖之。  

# 二、ICI使用手册      

## ICI完全支持CI的操作。  

    CI的介绍和使用，请参见官网
    http://codeigniter.org.cn   

## ICI相对于CI做了哪些升级？    

### 1 日志      
1）进一步细化了日志级别，有8个日志级别，如下	

	'FATAL' => 1,
	'ERROR' => 1,
	'WARNING' => 2,
	'NOTICE' => 4,
	'TRACE' => 8,
	'DEBUG' => 16,
	'INFO' => 16,
	'ALL' => 32

可以在config.php里，配置最小记录级别，例如

	$config['log_threshold'] = 8;

2）写磁盘buffer，有效减少刷磁盘次数

	达到4096（页大小）才刷一次磁盘，最后log_finish再做一次强刷。

3）自动添加log_request, log_finish

	log_request是利用钩子（hook），在请求一开始记录requet数据；
	log_finish是通过扩展regist_shutdown_hanler，在php脚本执行退出前，记录的本次请求的各项数据统计。

4）规范日志格式
一条典型的log_finish日志，如下：

	[NOTICE][2017-04-20 01:32:29:906105][log_id=1492623149897549001414][line=/ICI/application/core/MY_common.php +7 function=::log_finish][uri=/module1/welcome][mark=request_out][proc_time=0.009400][time_total=0.009400][time_load_base=0.004800][time_ac_exe=0.004300 (s)][memory_use=1.750000][memory_peak=1.750000 (MB)]

5）封装了log_helper

	可以方便的记录各级别日志，如write_notice、write_warning、write_fatal


### 2 参数检查	

框架添加了参数检查的钩子，将参数验证可配置化。把琐碎的令人讨厌的“参数检查”从业务代码里剥离出来。
如果请求的参数不符合配置的规则，框架会自动返回一个“参数错误”。对应的核心代码为，hooks/Param.php

	使用时，用户只需要在config/param目录下，添加每个controller的参数检查配置。
	比如，对于uri是/module1/welcome/index的请求，可以添加config/param/module1/welcome.php，代码如下：

	<?php  if ( ! defined('BASEPATH')) exit('No direct script access allowed');
	/**
	* welcome接口规则
	* @author Mr.Nobody
	*
	*/
	$config['param']['index']['method'] = HTTP_REQUEST_METHOD_GET;
	$config['param']['index']['rules'] = array(
		array(
			'field' => 'hello',
			'rules' => 'trim|required|max_length[32]'
		),
		array(
			'field' => 'arg1',
			'rules' => 'trim|integer|max_length[32]'
		),
	);

### 3 标准返回
封装了标准返回类 libraries/StdReturn.php  
1）response的数据格式为json格式（Content-Type:application/json）  
2）接口返回三元组(status, msg, data)，如果不符合要求，请自行修改。  
3）支持jsonp，前端需要在GET里传入jsonpCallback参数。  
该类已经做了autoload，可以直接使用，如下：    

	$this->stdreturn->ok($data);
	$this->stdreturn->failed('4001', $error);

至于错误码&错误提示，需要在语言包中配置。如：language/zh_cn/myerror_lang.php     

	<?php

	$lang['myerror'] = array(
		'500' => '服务器内部错误',
		'404' => '所访问内容不存在',
		
		// 5开头，服务端错误
		'5000' => '服务器内部错误',//未知错误
		'5001' => '服务器内部错误',//db 异常
		'5002' => '服务器内部错误',//redis 异常
		'5003' => '服务器内部错误',//php 异常
		
		// 4开头，客户端错误
		'4001' => '参数错误',
	);


	/* End of file myerror_lang.php */
	/* Location: ./wc_content/language/zh_cn/myerror_lang.php */

### 4 在third_party中使用自动加载    
1）引入自动加载方法，使得可以在third_party内引入带命名空间的类库。  
2）因为CI本身没有使用命名空间和类自动加载。  
3）在third_party内引入带命名空间的类库之前，必须先过滤CI类、CI覆盖类。  

### 5 常驻进程类 
封装了cli模式启动的常驻进程类。该类具有如下几个特点  
1）可以配置最大执行时间  
2）支持平滑启动  

### 6 Curl类 
支持 get、post、mutiGet；

### 7 ApiProxy类 
该类是处理api调用的一个标准类，可以通过继承该类，实现快速接入外部api。    
1）api配置文件，config/development/api.php

	/*
	| -------------------------------------------------------------------
	| API REQUEST SETTINGS
	| -------------------------------------------------------------------
	*/
	// host
	$config['xxx_api']['host'] = 'http://{HOST}/{URI}';
	// uri
	$config['xxx_api']['uri'] = array(
		'xxx_ooo' => 'xxx/ooo/v1',
	);
	// 需要做log的api
	$config['xxx_api']['request_log'] = array(
		'xxx_ooo',
	);

2)继承ApiProxy类后，子类一版需要覆盖签名方法、添加通用参数的方法。




