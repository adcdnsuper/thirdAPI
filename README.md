
## PHP代码类：


```
<?php
/**
 * @desc Utils
 * @Author lyt 395078304@qq.com
 * @Date 2020/08/28
 * @CopyRight YuXia(XM) Information Technology Co., Ltd.
 * @Version ADCDN V3.0
 * @todo 通用基础工具类
 */
class Utils
{
    //jwt头部
    private static $header=array(
        'alg'=>;'HS256', //生成signature的算法
        'typ'=>;'JWT'    //类型
    );


    public static function getJwtToken($payload)
    {
        if(is_array($payload))
        {
            $base64header=self::base64UrlEncode(json_encode(self::$header,JSON_UNESCAPED_UNICODE));
            $base64payload=self::base64UrlEncode(json_encode($payload,JSON_UNESCAPED_UNICODE));
            $token=$base64header.'.'.$base64payload.'.'.self::signature($base64header.'.'.$base64payload,self::$key,self::$header['alg']);
            return $token;
        }else{
            return false;
        }
    }

    public static function verifyJwtToken($Token)
    {
        $tokens = explode('.', $Token);
        if (count($tokens) != 3)
            return false;

        list($base64header, $base64payload, $sign) = $tokens;

        //获取jwt算法
        $base64decodeheader = json_decode(self::base64UrlDecode($base64header), JSON_OBJECT_AS_ARRAY);
        if (empty($base64decodeheader['alg']))
            return false;

        //签名验证
        if (self::signature($base64header . '.' . $base64payload, self::$key, $base64decodeheader['alg']) !== $sign)
            return false;

        $payload = json_decode(self::base64UrlDecode($base64payload), JSON_OBJECT_AS_ARRAY);

        return $payload;
    }

    private static function base64UrlEncode($input)
    {
        return str_replace('=', '', strtr(base64_encode($input), '+/', '-_'));
    }

    private static function base64UrlDecode($input)
    {
        $remainder = strlen($input) % 4;
        if ($remainder) {
            $addlen = 4 - $remainder;
            $input .= str_repeat('=', $addlen);
        }
        return base64_decode(strtr($input, '-_', '+/'));
    }

    private static function signature($input, $key, $alg = 'HS256')
    {
        $alg_config=array(
            'HS256'=>'sha256'
        );
        return self::base64UrlEncode(hash_hmac($alg_config[$alg], $input, $key,true));
    }

}
```
## GOLANG代码类：

```
package jwtService

import (
	"github.com/dgrijalva/jwt-go"
	jsoniter "github.com/json-iterator/go"
)

const THIRDPARYTOKEN = "THIRD_PARY_TOKEN"
func CreateToken(postData jwt.MapClaims,salt string) (string,error){
	at := jwt.NewWithClaims(jwt.SigningMethodHS256, postData)
	token, err := at.SignedString([]byte(THIRDPARYTOKEN+salt))
	if err != nil {
		return "", err
	}
	return token, nil
}

func ParseToken(token,salt string)(string,error){
	claim, err := jwt.Parse(token, func(token *jwt.Token) (interface{}, error) {
		return []byte(THIRDPARYTOKEN+salt), nil
	})
	if err != nil {
		return "", err
	}
	re, err := jsoniter.Marshal(claim.Claims.(jwt.MapClaims))
	if err != nil {
		return "", err
	}
	return string(re), nil
}
```


## 加密说明信息
    
1. 所有接口都为post请求

2. salt为分配给开发者的盐值参数不参与加密

3. 使用标准JWT加密处理：(参考上面代码)

4. 必带参数salt 分配给开发者的盐值

5. 请求url：https://api.game.box.apiadcdn.com 

6. 当前云币与RMB的兑换比例PHP代码示例：

```
require 'Utils.php';
function curl($url,$data){
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);

    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);

    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);

    curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);

    // POST数据

    curl_setopt($ch, CURLOPT_POST, 1);

    // 把post的变量加上

    curl_setopt($ch, CURLOPT_POSTFIELDS, $data);

    $output = curl_exec($ch);

    curl_close($ch);
    var_dump($output);
}

$url = 'https://api.game.box.apiadcdn.com/gv1/thirdParty/ExchangeRatio';
$params = array(
    'appId'=>600002;,
    'timeStamp'=>time(),
);
$token = Utils::getJwtToken($params);
$data = array(
    'salt'=>'bCL5IB',
    'token'=>$token
);
curl($url,$data);
```






##  获取指定月份用户获得云币数的排行  /gv1/thirdParty/CheckWithMouth

### 加密参数

1. appId 开发者应用ID

2. month 月份 格式（2020-08）

3. timeStamp 时间戳

### 不参与加密参数
1. page 分页 不传默认1 

2. limit  每页条数 不传默认10 

 ###返回示例

```
{
  "APIDATA": [
    {
      "gold": "206770", //云币金额
      "userId": "57257" //用户id
    }
  ],
  "APIDEC": "ok",
  "APISTATUS": "2000"
}
```

## 指定日期指定用户获得的云币数 /gv1/thirdParty/CheckWithPlayerAndDate
### 加密参数

1. appId 开发者应用ID

2. date 日期 格式（2020-08-29）

3. userId sdk传递的用户id

4. timeStamp 时间戳
 
### 不参与加密参数（无）


 ###返回示例

```
{
  "APIDATA": 206770,// 金币余额
  "APIDEC": "ok",
  "APISTATUS": "2000"
}
```

##  当前云币与RMB的兑换比例  /gv1/thirdParty/ExchangeRatio

### 加密参数

1. appId 开发者应用ID


3. timeStamp 时间戳

### 不参与加密参数（无）

 ###返回示例

```
{
  "APIDATA": {
    "CNY": 1,//人命币
    "YGOLD": 100000 //云币
  },
  "APIDEC": "ok",
  "APISTATUS": "2000"
}
```





