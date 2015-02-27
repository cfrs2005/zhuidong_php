---
layout: post
title: 140403 微信服务号查询用户信息demo
---

用来查询微信用户信息的，获取用户openid后后去更多相关信息。

	define ( 'APP_ID', 'wxccccf7a6gggg4e2' );
	define ( 'APP_SECRET', 'bc0ff2e74ebadddd0be31cad2c658a2' );

	$token = get_token ();
	$user_array = array (
			'o_asdhAHoIkccccccc_dgFokI',
			'asdasdasd-NeLQibjnbQ',
			'deqweqweqweQlrRfxl87k6YKA',
			'occccdee-elL8KPiVEeZwelNhmAY' 
	);
	$users = array ();
	foreach ( $user_array as $user ) {
		$users [] = get_user_info ( $user, $token );
	}
	function get_token() {
		$url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" . APP_ID . "&secret=" . APP_SECRET;
		$data = json_decode ( file_get_contents ( $url ), true );
		if ($data ['access_token']) {
			return $data ['access_token'];
		} else {
			return "获取access_token错误";
		}
	}
	function get_user_info($openid, $token) {
		$url = "https://api.weixin.qq.com/cgi-bin/user/info?access_token=" . $token . "&openid=" . $openid . "&lang=zh_CN";
		$data = json_decode ( file_get_contents ( $url ), true );
		if ($data) {
			return $data;
		} else {
			echo "获取信息错误错误";
			exit();
			return null;
		}
	}

### 关于访问结果错误

	{"errcode":42001,"errmsg":"access_token expired"}

是因为你的好不是服务好无权限获取用户信息。


### 参考：

1. http://mp.weixin.qq.com/wiki/index.php?title=%E9%A6%96%E9%A1%B5
