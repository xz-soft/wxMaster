# wxMaster
wxMaster是电脑版微信客户端的自动化框架，你可以用任何编程语言通过HTTP协议调用wxMaster框架的API接口，实现微信自动收发消息。

# 核心功能
**个人微信自动化 + 任意语言HTTP方式调用API接口 + 安全不封号**

# 实现功能
	1. 采用安全合规的 UIAutomation 技术对PC版微信客户端实现自动化。
	2. 以下所有功能提供HTTP协议API接口。
	3. 获取、发送私聊消息。
	4. 获取、发送群聊消息。
	5. 获取、发送图片消息。
	6. 获取语音转文字消息。
	7. 获取指定联系人指定条数历史消息。
	8. 获取通讯录全部联系人。
	9. 获取群成员列表。
	10. 微信转账自动收款。

# 注意事项
- 本框架会控制鼠标对微信进行操作，适合全自动挂机运行。
- 调用API接口时，由于某些接口会消耗较长时间，所以请设置合适的超时时长。

# API接口

## 通用

- **说明**  
	所有接口都按照此通用格式返回，不同接口的 `resultContent` 值不同。

- **http地址**  
`http://127.0.0.1:34567`
	>其中 `34567` 是默认端口，允许用户自定义。

- **请求方式**  
`postMsgText` 为 `POST` 方式，其余均为 `GET` 方式。

- **返回类型**  
`json`

- **返回**

	| 键 | 类型 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: |
	| resultCode | 整数 | 1 | 代码。1表示成功，其它表示失败 |
	| resultMsg | 字符串 | success | 成功时为success，失败时为失败原因 |
	| resultContent | json对象 / json数组 | \{...\} / [] | 接口返回值。不同接口返回值不同 |

- **返回示例**  
	```
	{
		"resultCode": 1,
		"resultMsg": "success",
		"resultContent": {...}
	}
	```

## findWxWindow

- **api**  
	`http://127.0.0.1:34567/findWxWindow`

- **功能**  
	查找所有微信窗口。  
	后续的所有操作都需要提供此接口返回的**窗口句柄hwnd**。

- **参数**  
	无

- **resultContent**   
	返回类型：数组。  
	数组内每一个元素对应一个微信窗体（微信账号）。
	
	| 键 | 类型 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: |
	| accountNick | 字符串 | "昵称1" | 昵称 |
	| hwnd | 整数 | 111111 | 微信窗口句柄 |
	
- **返回示例：**  
	```json
	{
		"resultCode": 1,
		"resultMsg": "success",
		"resultContent": [
			{
				"accountNick": "昵称1",
				"hwnd": 111111
			},
			{
				"accountNick": "昵称2",
				"hwnd": 222222
			}
		]
	}
	```

## getNewMsgAll

- **API**  
	`http://127.0.0.1:34567/getNewMsgAll?hwnd=111111&getImage=0&getVoice=0`

- **功能**  
	获取全部新消息，包括私聊消息和群聊消息。  
	轮询此接口可获取新消息。为尽量保证账号安全，轮询间隔不要低于5秒。  
	**免打扰**消息不会被获取。  
	如果getImage=1，则图片会被保存到 ` \微信图片\ ` 目录下，同时 `content` 内容为图片路径。
	>自己发送的消息不会被获取。  
	>被排除的发送人：订阅号、公众号、微信支付、微信团队、服务通知、文件传输助手、小程序客服消息

- **参数**  

	| 参数 | 必须 | 默认 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: | :---: |
	| hwnd | 是 | 无 | 111111 | 被获取消息的微信窗口句柄 |
	| getImage | 否 | 0 | 1 | 是否获取并保存图片。<br>1：是<br>0：否 |
	| getVoice | 否 | 0 | 0 | 是否获取语音转文本消息。<br>1：是<br>0：否 |
	
- **resultContent**  
	>如果没有新消息，返回的resultContent会是一个空数组：[]

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| from | 张三 / 群聊名称 | 消息来源。<br>好友昵称，或群聊名称 |
	| list | [...] | 消息数组。<br>按时间先后排序，最近的消息在最前面 |
	| type | text<br>image<br>voice<br>wxhb<br>wxzz<br>video<br>file<br>link<br>mp<br>map<br>emoji<br>other | 消息类型。<br>text：文本消息<br>image：图片<br>voice：语音<br>wxhb：微信红包<br>wxzz：微信转账<br>video：视频<br>file：文件<br>link：链接<br>mp：小程序<br>map：地图位置<br>emoji：动画表情<br>other：无法识别的消息 |
	| sender | 张三 | 发送人昵称 |
	| content | 我是消息内容 | 消息内容。<br>text：文本消息内容<br>image：图片路径<br>voice：语音转文字 |

- **wxzz**  

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| amount | ￥100.00 | 转账金额 |
	| remark | 转账说明1 | 转账说明 |
	
- **返回示例**
	```json
	{
		"resultCode": 1,
		"resultMsg": "success",
		"resultContent": [
			{
				"from":"张三",
				"list":[
					{
						"type": "text",
						"sender": "张三",
						"content": "这是消息内容"
					},
					{
						"type": "image",
						"sender": "张三",
						"content": "\\微信图片\\2025-01-02_03-04-05_123.jpg"
					},
					{
						"type": "voice",
						"sender": "张三",
						"content": "这是语音转文字消息内容"
					}
				]
			},
			{
				"from":"群聊1",
				"list":[
					{
						"type": "text",
						"sender": "张三",
						"content": "这是消息内容"
					},
					{
						"type": "image",
						"sender": "李四",
						"content": "\\微信图片\\2025-01-02_03-04-05_123.jpg"
					},
					{
						"type": "voice",
						"sender": "王二麻子",
						"content": "这是语音转文字消息内容"
					}
				]
			}
		]
	}
	```
	
## getMsgNick

- **API**  
	`http://127.0.0.1:34567/getNewMsgNick?hwnd=111111&nick=昵称1&count=20&getImage=1&getVoice=0`

- **功能**  
	获取指定联系人指定条数的历史消息。  

- **参数**  

	| 参数 | 必须 | 默认 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: | :---: |
	| hwnd | 是 | 无 | 111111 | 被获取消息的微信窗口句柄 |
	| nick | 是 | 无 | 昵称1 | 被获取消息的好友昵称或群聊名称 |
	| count | 否 | 0 | 20 | 获取历史消息的条数。<br>不指定则获取1页所有消息，大约40个。 |
	| getImage | 否 | 0 | 1 | 是否获取并保存图片。<br>1：是<br>0：否 |
	| getVoice | 否 | 0 | 0 | 是否获取语音转文本消息。<br>1：是<br>0：否 |
	
- **resultContent**  
	>resultContent是一个数组类型：[]

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| type | text<br>image<br>voice<br>wxhb<br>wxzz<br>video<br>file<br>link<br>mp<br>map<br>emoji<br>other | 消息类型。<br>text：文本消息<br>image：图片<br>voice：语音<br>wxhb：微信红包<br>wxzz：微信转账<br>video：视频<br>file：文件<br>link：链接<br>mp：小程序<br>map：地图位置<br>emoji：动画表情<br>other：无法识别的消息 |
	| sender | 张三 | 发送人昵称 |
	| content | 我是消息内容 | 消息内容。<br>text：文本消息内容<br>image：图片路径<br>voice：语音转文字 |
	
- **返回示例**
	```json
	{
		"resultCode": 1,
		"resultMsg": "success",
		"resultContent": [
				{
					"type": "text",
					"sender": "张三",
					"content": "我是消息内容"
				},
				{
					"type": "image",
					"sender": "张三",
					"content": "\\微信图片\\2025-01-02_03-04-05_123.jpg"
				},
				{
					"type": "voice",
					"sender": "张三",
					"content": "这是语音转文字消息内容"
				}
		]
	}
	```

## getContactsFriends

- **API**  
	`http://127.0.0.1:34567/getContactsFriends?hwnd=111111`

- **功能**  
	获取通讯录全部好友。不包含群聊。  

- **参数**  

	| 参数 | 必须 | 示例 | 说明 |
	| :---: | :---:| :---: | :---: |
	| hwnd | 是 | 111111 | 被获取的微信窗口句柄 |
	
- **resultContent**  
	>resultContent是一个数组类型：[]

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| nick | 张三 | 好友昵称 |
	| remark | 备注名 | 好友备注 |
	| label | 标签 | 好友标签 |
	
- **返回示例**
	```
	{
		"resultCode":1,
		"resultMsg":"success",
		"resultContent":[
			{
				"nick":"昵称1",
				"remark":"备注1",
				"label":"标签1"
			},
			{
				"nick":"昵称2",
				"remark":"备注2",
				"label":"标签2"
			}
		]
	}
	```

## getContactsGroups

- **API**  
	`http://127.0.0.1:34567/getContactsGroups?hwnd=111111`

- **功能**  
	获取通讯录全部群聊。  

- **参数**  

	| 参数 | 必须 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: |
	| hwnd | 是 | 111111 | 被获取的微信窗口句柄 |
	
- **resultContent**  
	>resultContent是一个数组类型：[]

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| name | 群名1 | 群聊的名称 |
	| count | 50 | 群成员总数 |
	
- **返回示例**
	```
	{
		"resultCode":1,
		"resultMsg":"success",
		"resultContent":[
			{
				"name":"群名1",
				"count":"50"
			},
			{
				"name":"群名2",
				"count":"60"
			}
		]
	}
	```

## getGroupMembers

- **API**  
	`http://127.0.0.1:34567/getGroupMembers?hwnd=111111&groupName=群聊名1`

- **功能**  
	获取群成员。  

- **参数**  

	| 参数 | 必须 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: |
	| hwnd | 是 | 111111 | 被获取的微信窗口句柄 |
	| groupName | 是 | 群聊名1 | 被获取的群名称 |
	
- **resultContent**  
	>resultContent是一个数组类型：[]

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| nick | 张三 | 群成员的微信昵称 |
	| groupNick | 张三2 | 群成员的群昵称 |
	
- **返回示例**
	```
	{
		"resultCode":1,
		"resultMsg":"success",
		"resultContent":[
			{
				"nick":"张三",
				"groupNick":"张三2"
			},
			{
				"nick":"李四",
				"groupNick":"李四2"
			}
		]
	}
	```

## postMsgText

- **API**  
	`http://127.0.0.1:34567/postMsgText?hwnd=111111&nick=昵称或群名称`

- **功能**  
	给好友或群发送文字消息。  

- **请求方式**  
	`POST`

- **参数**  

	| 参数 | 必须 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: |
	| hwnd | 是 | 111111 | 微信窗口句柄 |
	| nick | 是 | 昵称或群名称 | 好友昵称或群聊名称 |
	| [postData] | 是 | 这是文字消息内容 | post内的所有数据将被发送 |
	>如果postData的数据里包含回车换行，需要将微信客户端的发送消息快捷键修改为 `Ctrl + Enter`  ，否则会被回车拆分为多条消息。  
	>修改方法：微信客户端左下角三横线 -> 设置 -> 快捷键 -> 发送消息 -> `Ctrl + Enter`
	
- **resultContent**  

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| resultCode | 1 | 1表示发送成功，其它代表失败 |
	| resultMsg | success | 成功时为success，失败时为失败原因 |
	
- **返回示例**
	```
	{
		"resultCode":1,
		"resultMsg":"success",
		"resultContent":{}
	}
	```

## sendMsgImage

- **API**  
	`http://127.0.0.1:34567/sendMsgImage?hwnd=111111&nick=昵称或群名称&path=D:\img\1.jpg`

- **功能**  
	给好友或群发送图片消息。  

- **参数**  

	| 参数 | 必须 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: |
	| hwnd | 是 | 111111 | 微信窗口句柄 |
	| nick | 是 | 昵称或群名称 | 好友昵称或群聊名称 |
	| path | 是 | D:\img\1.jpg | 要发送的图片的完整路径<br>支持格式：.png .jpg .bmp .gif |
	
- **resultContent**  

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| resultCode | 1 | 1表示发送成功，其它代表失败 |
	| resultMsg | success | 成功时为success，失败时为失败原因 |
	
- **返回示例**
	```
	{
		"resultCode":1,
		"resultMsg":"success",
		"resultContent":{}
	}
	```

## transferReceive

- **API**  
	`http://127.0.0.1:34567/transferReceive?hwnd=111111&nick=好友昵称`

- **功能**  
	微信转账收款。查询是否有转账未被接收，如果有则自动收款。  
	>调用逻辑：如果用 `getNewMsgAll` 接口获取到有新的微信转账，再调用此接口自动收款。

- **参数**  

	| 参数 | 必须 | 示例 | 说明 |
	| :---: | :---: | :---: | :---: |
	| hwnd | 是 | 111111 | 微信窗口句柄 |
	| nick | 是 | 好友昵称 | 好友的昵称 |
	
- **result**  

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| resultCode | 1 | 1表示成功自动收款，其它代表失败 |
	| resultMsg | success | 成功时为success，失败时为失败原因 |
	| resultContent | [...] | 数组类型<br>如果收款成功，数组内每个元素对应一条收款信息 |

- **resultContent**  

	| 键 | 示例 | 说明 |
	| :---: | :---: | :---: |
	| amount | ￥100.00 | 收款金额 |
	| remark | 转账说明1 | 转账说明 |
	
- **返回示例**
	```
	{
		"resultCode":1,
		"resultMsg":"success",
		"resultContent":[
			{
				"amount":"￥100.00",
				"remark":"转账说明1"
			},
			{
				"amount":"￥888.88",
				"remark":"转账说明2"
			}
		]
	}
	```
