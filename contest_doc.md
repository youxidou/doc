## 查询用户信息接口
> GET http://game.yxd17.com/api/contest/user

## 参数:
| 参数名 | 必填  | 描述 |
| -----| :---| :---------|
| token | 是 | url中提供的token |

## 返回(JSON):
| 参数名 | 必填  | 描述 |
| -----| :---| :---------|
| name | 是 | 用户昵称 |
| avatar | 是 | 头像url |
| open_id | 是 | 用户和游戏关联的唯一ID |
| uid | 否 | 趣头条用户ID(如果不是趣头条用户则为null) |

## 示例:
> http://game.yxd17.com/api/contest/user?token=bb9289a46f05d14043ca013468e48ec79882b73598e9363f508c16bf58a58a10

## 流程:
> 用户进入趣头条游戏中心  
> ->进入赛事中心  
> ->进入赛事  
> ->**跳转到游戏方赛事页面, 并在url中加入token参数**  
> ->游戏方通过token获取用户信息
