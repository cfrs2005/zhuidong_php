---
layout: post
title: 140403 discuz分类信息还原
---

上次说到很脑残的需求竟然需要转移discuz正文帖子内容到其他系统上去，要去还原原帖内容包括附件：[discuz帖子内容](http://www.80aj.com/2766.html)。

这不没过多久又来个更更脑残的还原，**disucz还原信息分类选择结果**,分化不多说直接上代码，大部分已经注释。


	$tid = 3891;
	//获取帖子信息
	$thread_info = DB::fetch_first ( "select a.`tid`, a.`fid`,a.`authorid`, a.`author`,a.`dateline`, a.`subject`, b.`message` from " . DB::table ( 'forum_thread' ) . " as a, " . DB::table ( 'forum_post' ) . " as b where a.tid=$tid and a.tid=b.tid and b.first=1 order by pid desc limit 1" );
	
	if ($thread_info) {
		//获取板块扩展信息
		$get_fid_info = DB::fetch_first ( "select * from " . DB::table ( 'forum_forumfield' ) . " where fid=" . $thread_info ['fid'] );
	}
	//反序列化分类信息
	$threadsorts = unserialize ( $get_fid_info ['threadsorts'] );
	$sortoptionarray = array ();
	
	
	if (! empty ( $threadsorts ['types'] )) {
		//加载分类序列化库
		require_once libfile ( 'function/threadsort' );
		foreach ( $threadsorts ['types'] as $stid => $sortname ) {
			loadcache ( array (
					'threadsort_option_' . $stid,
					'threadsort_template_' . $stid 
			) );
			sortthreadsortselectoption ( $stid );
			$sortoptionarray [$stid] = $_G ['cache'] ['threadsort_option_' . $stid];
		}
	}
	$sort_info_list = array ();
	//获取帖子分类信息存储部分
	$sort_info_rs = DB::query ( "select * from " . DB::table ( 'forum_typeoptionvar' ) . " where tid=" . $thread_info ['tid'] . " order by sortid desc" );
	
	if (DB::num_rows ( $sort_info_rs ) > 0) {
		while ( $fetch = DB::fetch ( $sort_info_rs ) ) {
			$sort_info_list [] = $fetch;
		}
	}
	foreach ( $sort_info_list as $sort_info ) {
		echo $sortoptionarray [$sort_info ['sortid']] [$sort_info ['optionid']] ['title'];
		echo $sortoptionarray [$sort_info ['sortid']] [$sort_info ['optionid']] ['choices'] [$sort_info ['value']] ['content'];
	}
	
希望喜欢，不一定很有用，只是省去了你的部分时间。