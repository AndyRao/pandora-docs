##推荐系统API
### API接收地址

```
https://pipeline.qiniu.com
```

### API返回内容

**响应报文** 

* 如果请求成功,返回HTTP状态码`200`:

```
HTTP/1.1 200 OK
```

* 如果请求失败,返回包含如下内容的JSON字符串（已格式化,便于阅读）:

```
{
    "error":   "<errMsg    string>"
}
```

* 如果请求包含数据获取,则返回相应数据的JSON字符串；



### 数据推送

**接口描述**

用户可以通过这个接口将产品信息数据、用户信息数据、用户行为数据上传到Pandora的推荐系统中。

**请求语法**

```
POST /v2/repos/<RepoName>/data
Content-Type: application/text
Authorization: Pandora <auth>
<RepoContent>
```

**请求参数说明**

|参数|类型|必填|说明|
|:---|:---|:---|:---|
|RepoName |string|  是  |消息队列名称|
|RepoContent |string|是|消息内容,格式如下<br/> keyName=valName keyName=valName </br> keyName=valName keyName=valName </br> ...</br> keyName为字段名称,keyName的取值根据不同的RepoName是预定义好的,具体在下面会有定义。</br>valName是对应字段名称的数据内容<br/> 注意：如果valName是`string`类型</br>那么 `\t`、`\r`、`\n` `\` 需要用`\`转义</br>空格`' '` 可以不转义|




> 对于`array`类型：
> 
> 打点格式为`[e1,e2,...,en]`，数组元素采用逗号分割，且所有元素使用`[]`包括，当元素类型为`string`时，需要加上双引号;
> 
> 对于`map`类型：
> 
> 打点格式为json字符串，比如`{"f1":123,"f2":"abc"}`，注意所有元素使用`{}`包括;另外，多个`keyName`和`valName`之间应使用单个 `<TAB>` 分隔，单次分隔的长度不超过100KB。

**请求参数详细说明**

- RepoName

对于推荐系统来说，消息队列RepoName只能是以下表格中给出的值。请对照不同的RepoName的含义，上传数据到正确的Repo。

各个Repo和字段的详细说明，请参见后文“Repo表详细说明”。

RepoName|含义
|:---|:---|
|item|物品信息表
|user|用户信息表
|user_action|用户行为信息表
|recshow|推荐结果表



- RepoContent

RepoContent字段的值对应的是上报的数据内容，必须是UTF8编码。
支持批量传输，即一次请求传输多条数据，RepoContent每一行必须包含“cmd”和“fields”两个字段，也就是说keyName必须只能是cmd和fields两个字段。

（1）cmd可能的取值和定义如下：

cmd|含义
|:--|:---|
|add|	新增一条记录，如果主键对应的记录已经存在，则对该记录做覆盖操作
|update|更新一条记录，如果主键对应的记录不存在，则不处理，user_action和recshow表不支持update操作
delete|删除一条记录，如果主键对应的记录不存在，则认为删除成功，user_action和recshow表不支持delete操作
refresh_all|批量下线item数据，仅对item表有效

（2）fields 字段的值为一个词典（dict），是具体上传的数据内容，请将您所要上传的字段及相应的值构建成词典，比如 { “itemid”: “28394556”, “cateid”: “9_2_1”, “score”: 459, “title”: “天穿修炼记最新版”, “item_tags”: “修仙”}。
注：根据上传的RepoName的不同，**fields** 应包含不同的字段，详见后文“**上报数据表详细说明**”。

**Http 返回结果说明**

参数|	类型|	描述
|:--|:--|:--|
|status|	string	|执行结果，OK为成功，FAIL为失败，WARN为有部分非重要字段异常，请根据返回错误信息进行排查。
|errors|	string	|错误信息
|request_id|	string|	该条上报记录的序号，仅用于排查问题使用


###Repo表详细说明
####概述

预定义了4个数据表格，以下给出每个表格的字段定义，以及取值的格式，请严格按照以下规定进行数据上报。
如果上报的数据字段和以下数据表的字段含义不一致，请务必按照本文档规定的字段名进行上报，否则可能会影响推荐等功能的效果。


####item（物品信息表)

说明: 用于上报主要的产品数据，比如商品信息、视频信息、主播信息等等。

参数|	类型|	必需|	描述
|:--|:--|:--|:--|
itemid|	string|	是|	item的唯一id
cateid|	string|	否|	cateid，item所属分类，多级分类用“_”进行分隔。例如“1_2”,表示一级分类为1，二级分类为2。如果同时属于多个分类，用英文分号“;”分隔。比如“1_2;3_1”。
score|	int|	否|	item的热门程度，0~1000的整数
title|	string|	否|	item的标题
content|	string|	否|	正文，比如帖子或新闻的正文
price|	int|	否|	item的价格，单位为分
item_tags|	string|	否|	item的标签等信息，多个标签以英文分号“;”分割。使用推荐功能的客户可上传该字段以提高推荐准确率；如果需要根据标签进行搜索，此项也需要填上。
item_modify_time|	int|	否|	item的最新修改时间，unix时间戳，精确到秒。使用推荐功能的客户可上传该字段以提高推荐时效性；如果需要根据该字段进行搜索筛选，则此项需要填上。
其他|	string|	否|	其他字段也可以进行上报，如果有时间字段，需要采用unix时间戳（精确到秒）
注意事项

1. item_modify_time必须是秒级的时间戳，不是毫秒
2. cateid不要错写成cate
3. cateid和item_tags必须使用英文分号分隔，不能用逗号（中英文）或者其它


**CURL调用示例**

```
curl -X POST https://pipeline.qiniu.com/v2/repos/item/data \
-H 'Content-Type: application/text' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
	cmd=add		fields={ "itemid": "28394557", "cateid": "9_2_1", "score": 434, "title": "比利林恩的中场战事", "item_tags": "战争"}	
	cmd=add		fields={ "itemid": "28394556", "cateid": "9_2_1", "score": 459, "title": "天穿修炼记最新版", "item_tags": "修仙"}
	}'
```

**refresh_all调用示例**：
注意：
1. refresh_all命令需要在fields中填all_itemid，all_itemid是所有要保持在线的itemid构成的list，refresh_all会把不在all_itemid中的所有item都下线，请谨慎使用！
2. all_itemid必须传多于100个itemid

```
curl -X POST https://pipeline.qiniu.com/v2/repos/item/data \
-H 'Content-Type: application/text' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
	cmd=refresh_all		fields={"all_itemid": ["28394556","23423423"]}
	}'
```

**成功返回示例**:

```
{
"status":"OK",
"request_id":"148642065805100373587"
}
```
 
**错误返回示例**:

```
{
"status":"FAIL",
"errors":{
"code":1012,
"message":"table does not exist"
},
"request_id":"142234873908422234072"
}
```

 
**警告返回示例**:

```
{
"status":"WARN",
"errors":{
"code":8012,
"message":"sex value is incorrect"
},
"request_id":"142234873908422234072"
}
```

####user（用户信息表）
说明: 用于上报用户的基本信息数据。

参数|	类型|	必需|	描述
|:--|:--|:--|:--|
userid|	string|	是|	用户的唯一id
user_tags|	string|	否|	用户的标签列表，多个标签以英文分号“;”分隔，例如“经典;美观;科学”
tel|string|	否|	用户的手机号
nick_name|	string|	否|	用户昵称
email|	string|	否|	用户邮箱
birthday|	string|	否|	用户的生日日期，格式建议采用“19901123”（生日为1990年11月23日）
sex|	int|	否|	用户的性别，值为1表示是男性，值为2表示是女性，值为0表示未知
city|	string|	否|	用户所在城市，例如“杭州”
province|	string|	否|	用户所在省份，例如“浙江”
其他|	string|	否|	其他字段也可以进行上报，如果有时间字段，需要采用unix时间戳（精确到秒）

**CURL调用示例**

```
curl -X POST https://pipeline.qiniu.com/v2/repos/item/data \
-H 'Content-Type: application/text' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
	cmd=add		fields={"imei":"2333", "userid": "1111", "birthday":"19990101", "tel":"12345"}

	}'
```

####user_action（用户行为信息表）

说明: 用户的行为数据，尤其是对物品的比较重要的行为数据，比如购买。
* 接入推荐功能的话，用户对推荐结果的点击行为也通过该表进行上报。

参数|	类型|	必需|	描述
|:--|:--|:--|:--|
userid|	string|	否|	用户唯一id，对应user表的userid
imei|	string|	否|	用户的手机IMEI号
cid|	string|	否|	用户的cookieid，（userid、imei、cid至少存在一个）
itemid|	string|	是|	对应item表的itemid
action_type|	string|	是|	用户对item执行的操作，必须是有意义的字符串
action_num|	int|	否|	动作数量。对于购买，可以是购买价格（单位为分）
action_detail|	string|	否|	该行为的一些描述信息，该字段可以自行定义，比如登录的话，可以是qq登录或者微信登录，发送短信可以是短信发送成功或者失败等等。在大数据平台会对这些数据进行汇总，以;进行分隔
timestamp|	timestamp|	否|	动作发生的时间，linux时间戳，精确到秒，必须是最近一周以内的时间，没有传则置为收到请求的时间
其他|	string|	否|	其他字段也可以进行上报，如果有时间字段，需要采用unix时间戳（精确到秒）
常用的的action_type如下表所示：

action_type|	含义
|:--|:--|
view|	浏览
search_click|	点击搜索结果
rec_click|	点击推荐结果
collect|	收藏
subscribe|	订阅
comment|	评论
gift|	送礼物
share|	分享
like|	点赞
dislike|	点衰
cart|	加入购物车（或加入书架）
buy|	购买

注意事项

1. 上报用户浏览、点击行为时，上报的itemid不能为空
2. userid imei cid三个用户至少有一个不能为空
3. timestamp必须是秒级的时间戳，不是毫秒
4. action_type必须是有意义的字符串，如果用户的操作是上面表格所列的动作，请按照表格规定进行上报


**CURL调用示例**

```
curl -X POST https://pipeline.qiniu.com/v2/repos/item/data \
-H 'Content-Type: application/text' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
	cmd=add		fields={"timestamp":1489040079, "action_type":"view", "userid": "1234", "itemid": "abc12"}

	}'
```
 
###recshow（推荐结果表）

说明: 记录给用户推荐了哪些产品。
* 接入推荐功能的话，需要将推荐页面给每个用户推荐了哪些产品的数据，上报到该表，用于计算推荐的点击率，做达观推荐上线前后的效果对比。

参数|	类型|	必需|	描述
|:--|:--|:--|:--|
pvtime	|timestamp|	是|	推荐发生的时间，linux时间戳，精确到秒
rec_type|	string|	是|	推荐类型，不支持自定义取值，个性化推荐请填：personal，相关推荐填：relate，热门推荐填：hot
userid|	string|	否|	被推荐用户的唯一id，对应user表的userid
imei|	string|	否|	被推荐用户的手机IMEI号
cid	|string|	否|	被推荐用户的cookieid，（userid、imei、cid至少存在一个）
scene_type|	string|	否|	推荐场景类型，常用的包括pc_home（PC首页个性推荐）,android_detail（安卓详情页个性化推荐）等。不在上述中的可以自行添加。
result|	list|	是|	推荐结果，列表类型，即给用户推荐的产品id列表，例如 [“1”, “2”, “3”]
src_itemid|	string|	否|	源itemid，仅对相关推荐有效，表示给哪个itemid进行相关推荐

注意事项

1. pvtime必须是秒级时间戳，而而不是毫秒，或者pvtime没有上报
2. rec_type的值必须是 personal，relate，hot三者之一
3. 上传的result必须是list类型
4. userid imei cid三个用户id至少有一个不为空

**CURL调用示例**

```
curl -X POST https://pipeline.qiniu.com/v2/repos/item/data \
-H 'Content-Type: application/text' \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
-d '{
	cmd=add		fields={"result":["1", "2"], "rec_type":"personal", "userid":"1212", "pvtime": 1489028471}
	}'
```
 


###热门推荐服务

接口描述

根据item数据和用户行为数据进行深入的挖掘和分析，分别针对全部item和各个分类下的item按照热门程度进行排序，得到全局和各个分类的热门item列表，并且按照热门程度从高到低排序。不带cateid参数，返回的是全局热门推荐结果。带有cateid参数，返回的是此分类下的热门推荐结果。

Http| 请求参数说明

参数|	类型|	是否必需|	描述
|:--|:--|:--|:--|
appid|	string|	否|	应用id
userid|	string|	否|	用户id，没有的话请用空字符串表示
imei|	string|	否|	用户手机IMEI号，没有的话请用空字符串表示
cid|	string|	否|	用户网站cookieid，没有的话请用空字符串表示（注意：userid、imei、cid至少包含一项，可都填写）
scene_type|	string|	否|	场景类型，用于标识不同的场景，常用的包括pc_home（PC首页个性推荐）,android_detail（安卓详情页个性化推荐）等。不在上述中的可以自行添加。
cateid|	string|	否|	如果该项为空，则为全局排行，如果不为空，则为相应分类排行
start|	int|	否|	开始项，用于翻页，默认为0并且关闭状态，如需打开，请跟达观联系
cnt|	int|	是|	单次推荐请求返回的结果数量，最大为64

注意事项

1. userid/imei/cid 必须至少有一个，用于唯一标识一个用户，有利于优化推荐效果。如果某一个没有值，请不要传NULL、null、-1等异常值，可以不传此参数或者传空字符串。
2. scene_type 主要用于标示不同推荐位置的，便于跟进各个位置的推荐效果。建议只有一个热门推荐位置可以不传此参数，有多个的话最好带上。
3. start 默认不使用。如果您有明确的翻页需求，请跟达观联系以启用此参数。
4. cateid 用于控制本次推荐请求只返回此cateid下的itemid列表，前向匹配，且只支持传入一个。比如itemid_1的 cateid为1_2_3, itemid_2的cateid为1_2_5, 传入cateid为1或者1_2，结果可返回 itemid_1和itemid_2；传入cateid为1_2_3的话，只返回itemid_1；传入cateid为2或者2_3，itemid_1和itemid_2都不会返回。


Http 返回结果说明

字段|	类型|	描述
|:--|:--|:--|
status|	string|	执行结果，OK为成功，FAIL为失败，，WARN为有部分非重要字段异常，请根据返回错误信息进行排查
recdata|	string|	推荐结果，为一个LIST,每一项为一个dict，包括推荐itemid
request_id|	string|	该条查询的记录id，主要用于排查问题使用
errors|	string|	如果FAIL或WARN时返回错误码和错误信息


**CURL调用示例**

```
curl  https://pipeline.qiniu.com/v2/repos/item/data?cate=1_2&start=0&cnt=2&imei=abcdefg&userid=1234345&scene_type=pc_home \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \
```

成功返回示例:

```
{
	"status":"OK",
	"request_id":"1422348642065805100373587",
	"recdata":[
		{
			"itemid":"34"
		},
		{
			"itemid":"15"
		}
	]
}
```

 
错误返回示例:

```
{
	"status":"FAIL",
	"request_id":"1422348739084222300234072"
	"errors":{
		"code":2015,
		"message":"userid, imei, cid all null"
	}
}
```

 
警告返回示例:

```
{
	"status":"WARN",
	"request_id":"1422348739084222300234072"
	"errors":{
		"code":2012,
		"message":"cnt value is incorrect"
	}
}
```
###相关推荐服务

接口描述

根据传入的itemid，结合item数据和用户行为数据进行深入的挖掘和分析，生成与当前item最相关的item列表，并且按照“相关性”从高到低排序。不带cateid参数，返回的是全局相关推荐结果。带有cateid参数，返回的是此分类下的相关推荐结果。

Http 请求参数说明

参数|	类型|	是否必需|	描述
|:--|:--|:--|:--|
appid|	string|	否|	应用id
itemid|	string|	是|	当前物品id，请求和该itemid相关的推荐物品
userid|	string|	否|	用户id，没有的话请用空字符串表示
imei|	string|	否|	用户手机IMEI号，没有的话请用空字符串表示
cid	|string|	否|	用户网站cookieid，没有的话请用空字符串表示（注意：userid、imei、cid至少包含一项，可都填写）
scene_type|	string|	否|	场景类型，用于标识不同的场景，常用的包括pc_home（PC首页个性推荐）,android_detail（安卓详情页个性化推荐）等。不在上述中的可以自行添加。
start|	int|	否|	开始项，用于翻页，默认为0并且关闭状态，如需打开，请跟达观联系
cnt|	int|	是|	单次推荐请求返回的结果数量，最大为64
exclude|	string|	否|	需要过滤的itemid列表，这些itemid推荐结果不会返回。如果有多个itemid请以,分隔
cateid|	string|	否|	是否需要按照类别过滤itemid列表，有则推荐的item必须是这个类别的

注意事项

1. userid/imei/cid 必须至少有一个，用于唯一标识一个用户，有利于优化推荐效果。如果某一个没有值，请不要传NULL、null、-1等异常值，可以不传此参数或者传空字符串。
2. scene_type 主要用于标示不同推荐位置的，便于跟进各个位置的推荐效果。建议只有一个相关推荐位置可以不传此参数，有多个的话最好带上。
3. start 默认不使用。如果您有明确的翻页需求，请跟达观联系以启用此参数。
4. exclude 本次推荐请求需要过滤掉的itemid列表，主要是为了方便客户灵活应用达观推荐结果和自身业务规则相融合。比如客户有一部分强推的itemid列表，然后和达观推荐结果向结合，为了避免重复，可以把强推的itemid列表放在exclude参数里。当然客户有一些不希望被推荐出来的itemid列表，也可以使用这个参数进行处理。
5. cateid 用于控制本次推荐请求只返回此cateid下的itemid列表，前向匹配，且只支持传入一个。比如itemid_1的 cateid为1_2_3, itemid_2的cateid为1_2_5, 传入cateid为1或者1_2，结果可返回 itemid_1和itemid_2；传入cateid为1_2_3的话，只返回itemid_1；传入cateid为2或者2_3，itemid_1和itemid_2都不会返回。


Http 返回结果说明

字段|	类型|	描述
|:--|:--|:--|
status|	string|	执行结果，OK为成功，FAIL为失败，，WARN为有部分非重要字段异常，请根据返回错误信息进行排查
recdata|	string|	推荐结果，为一个LIST,每一项为一个dict，包括推荐itemid
request_id|	string|	该条查询的记录id，主要用于排查问题使用
errors|	string|	如果FAIL或WARN时返回错误码和错误信息

**CURL调用示例**

```
curl  https://pipeline.qiniu.com/v2/repos/item/data?cate=1_2&start=0&cnt=2&imei=abcdefg&userid=1234345&scene_type=pc_home \
-H 'Authorization: Pandora 2J1e7iG13J66GA8vWBzZdF-UR_d1MF-kacOdUUS4:NTi3wH_WlGxYOnXsvgUrO4XMD6Y=' \

成功返回示例:

```
{
	"status":"OK",
	"request_id":"1422348642065805100373587",
	"recdata":[
		{
			"itemid":"34"
		},
		{
			"itemid":"15"
		}
	]
}
```
 
错误返回示例：

```
{
	"status":"FAIL",
	"request_id":"1422348739084222300234072"
	"errors":{
		"code":2012,
		"message":"itemid is required"
	}
}
```

 
警告返回示例：

```
{
	"status":"WARN",
	"request_id":"1422348739084222300234072"
	"errors":{
		"code":2013,
		"message":"cnt value is incorrect"
	}
}
```








