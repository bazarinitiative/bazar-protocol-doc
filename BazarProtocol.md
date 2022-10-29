# BazarProtocol

A p2p social networking protocol based upon public key algorithm.

## Version

0.2

- Version is a decimal number, or a decimal number lead by a letter `v`
- Bazar protocol use command time to control the protocol version upgrade. Any commmand with specific version that exceed specific time will be treated as invalid. Any command cannot interactive to a Post with later time. Any UGC with huge gap of command time and receive time should be marked as low trustworthy and low priority to display.
- Affective v0.2 time start from: 2022/10/29 UTC
- Atleaset v0.2 time start from: 2022/11/30 UTC

### Changes of this version

- Introduce version upgrade mechanism: depend on `commandTime` and `version`
- Add `version` to `Command`
- Introduce `Command Content Basic Fields`
- Add `commandType` to `Command Content Basic Fields`

## User Account

- An unique ECDSA key pair means an account.
- UserID is just a shortcut to a ECDSA publicKey. Which is generated from publicKey with a [specific algorithm](#calculate-userid).
- One biological or logical user may have several userID, if he lost his privateKey. There won't be any way to findback the lost privatKey.
- Account-Inherit service will announce that a new UserID is replacing an old UserID. Clients can choose which service to trust or not.

## Bazar Node

- Bazar node is an instance that runs [BazarServer](https://github.com/bazarinitiative/BazarServer)
- User can post his command to any bazar node
- Bazar node will sync user command with peers

## Security

- We use [ECDSA algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) to ensure command comes from user, See also [code example](#signature-and-verification)
- We use rate limit to prevent spam content. If spam content bypass the rules, we can hide them from display when necessary.

    | Rate Limit | Rule |
    |-|-|
    |Commands per IP|50 commands per 5 seconds |
    |Commands per IP|100 commands per 100 seconds |
    |Queries per IP|50 queries per 5 seconds |
    |||

- We use reputation to prevent spam peer nodes. Only top N peer nodes will be listed in get-peer-nodes API

    | Situation | Reputation |
    |-|-|
    |Fail to get command|Reduce|
    |No new command|Reduce|
    |Duplicate command|Reduce|
    |Fail to process command|Reduce|
    |Succeed process command|Increase|
    |||

## Command

- Any eligible command will be accepted by BazarServer.
- Command is a json string with specific fields

    | Field          | FieldType     | Required  | Comment                  |
    | -------------- |:------------- | :-------- | :----------------------- |
    | CommandID      | char(30)      | Y | uniqueID of this command |
    | commandTime    | int64         | Y | milliseconds since EPOCH (seconds for some old data, should auto adapt) |
    | userID         | char(30)      | Y | who initiative this command |
    | commandType    | varchar(20)   | Y | Post, Following, Repost, Like, Delete, etc... |
    | version        | varchar(20)   | Y | v0.1, etc |
    | [commandContent](#command-content) | varchar(51*1024) | Y | a json string with detail content of this command |
    | signature      | varchar(200)  | Y | signature of commandContent |

## Command Content Basic Fields

- Every Command Cotent should contains the following fields

    | Field          | FieldType     | Required  | Comment                  |
    | -------------- |:------------- | :-------- | :----------------------- |
    | CommandID      | char(30)      | Y | uniqueID of this command |
    | commandType    | varchar(20)   | Y | Post, Following, Repost, Like, Delete, etc... |
    | commandTime    | int64         | Y | milliseconds since EPOCH (seconds for some old data, should auto adapt) |
    | userID         | char(30)      | Y | who initiative this command |

## Command Content

- UserInfo

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |publicKey|varchar(300)|Y|publicKey that everyone can see|
    |userName|varchar(50)|Y|user can set a name to display|
    |bot|bool|Y|if this user is a bot|
    |biography|varchar(300)|Y|biography of this user|
    |location|varchar(100)|Y|location to display|
    |website|varchar(100)|Y|website to display|

- UserPic

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |pic|varchar(50*1024)|Y|base64 encoded user picture. string length 50KB at most.|

- Post

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |postID|char(30)|Y|unique ID of a post. reply/repost are also post. |
    |threadID|char(30)|Y|postID of the original post|
    |replyTo|char(30)|Y|postID of which we reply to. we can reply to an original post or a reply|
    |isRepost|bool|Y||
    |content|char(300)|Y|content of a post. RTF 1.7 standard. Word between '#' and first non-letter-nor-digit char will be treat as a hashtag declaration.|
    |contentLang|varchar(20)|Y|content language like 'en-US', 'de', 'ja', 'zh-CN'. as a reference value to help language auto detect.|

- Like

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |postID|char(30)|Y||

- Bookmark

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |postID|char(30)|Y||

- Following

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |targetType|varchar(20)|Y|"User" or "Channel"|
    |targetID|char(30)|Y|userID or channelID|

- Channel

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |channelID|char(30)|Y|uniqueID of this Channel. every user can have 50 channels at most|
    |channelName|varchar(100)|Y|displayName of this Channel|
    |description|varchar(300)|Y||

- ChannelMember

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |cmID|char(30)|Y|uniqueID of this channel-member relationship. every channel can have 200 members at most|
    |channelID|char(30)|Y||
    |memberID|char(30)|Y|add this user to channel. |

- BlockUser
    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |targetID|char(30)|Y|user to block. can not see your post or profile|

- MuteUser
    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |targetID|char(30)|Y|user to mute. will not display his post/reply|

- Delete

    | Field | FieldType | Required | Comment |
    |-|-|-|-|
    |commandID|char(30)|Y||
    |commandType| varchar(20)   | Y | |
    |commandTime|int64|Y||
    |userID|char(30)|Y||
    |deleteType|varchar(20)|Y|resourceType to delete. Post, Repost, Like, Following, etc|
    |targetID|char(30)|Y|resourceID to delete|

## Peer API

- API for peer node

    |Url|Description|
    |-|-|
    |[/Peer/RegisterPeer](#peerregisterpeer)|register a peer node|
    |[/Peer/GetPeerList](#peergetpeerlist)|get peer node list|
    |[/Peer/RetrieveCommandBatch](#peerretrievecommandbatch)|get command in batch|
    |[/Peer/GetCommand](#peergetcommand)|get one command|

### /Peer/RegisterPeer

```bash
curl -X 'POST' \
  'https://api.bazar.social/Peer/RegisterPeer' \
  -H 'accept: text/plain' \
  -H 'Content-Type: multipart/form-data' \
  -F 'baseUrl=https://api.aaa.com'
```

```bash
{
  "success": false,
  "msg": "check health fail: Name or service not known (api.aaa.com:443)",
  "data": null
}
```

### /Peer/GetPeerList

```bash
curl -X 'GET' \
  'https://api.bazar.social/Peer/GetPeerList?topN=100'
```

```bash
{
  "success": true,
  "msg": "",
  "data": [
    {
      "baseUrl": "https://bzea.azurewebsites.net/",
      "reputation": 2.591420891093879e-30,
      "receiveCount": 658,
      "receiveDupCount": 7,
      "receiveErrorCount": 0,
      "receiveOkCount": 651
    },
    {
      "baseUrl": "https://bzwu.azurewebsites.net/",
      "reputation": 2.4862893351537513e-30,
      "receiveCount": 658,
      "receiveDupCount": 658,
      "receiveErrorCount": 0,
      "receiveOkCount": 0
    }
  ]
}
```

### /Peer/RetrieveCommandBatch

```bash
curl -X 'GET' \
  'https://api.bazar.social/Peer/RetrieveCommandBatch?lastOffset=0&forwardCount=1' \
  -H 'accept: text/plain'
```

```bash
{
  "success": true,
  "msg": "",
  "data": [
    {
      "commandID": "89oGjHkfjhnnln2osqR5LG8bGpafGy",
      "commandTime": 1636611848,
      "userID": "SeQLiznuyoKPkaSMZ0r7DZJv35AhUn",
      "commandType": "UserInfo",
      "commandContent": "{\"userID\":\"SeQLiznuyoKPkaSMZ0r7DZJv35AhUn\",\"publicKey\":\"MIGbMBAGByqGSM49AgEGBSuBBAAjA4GGAAQBU2Lejd0WYe4tmj58oAFIm97lmvitXS8pYm9//OYtJA5dWC7rYArPPXFjS5+8qb/kbWFKDGLy5AYoGJePH6v1+y4BiylYV85tl/AjXk0uaM/XKTtcDDurKCZi74NV3SMEEaqkCmt+6lTLGiS1B04hl2sFhPZEcXRTssNo5TpZB0IZmyU=\\t\",\"userName\":\"user127931102\",\"website\":\"\",\"location\":\"\",\"bio\":\"\",\"commandID\":\"89oGjHkfjhnnln2osqR5LG8bGpafGy\",\"commandTime\":1636611848}",
      "signature": "AFV8t244GtjOt9qiriJqiCJDCedY9ux+2r4rjaJjmFTOrBq+66SdSs7y6UVENrSdFVJY3uAXOugF1jAn58qLa4EPAXD7/urzjERInfsGe8etMMkhL90kQChC3JIhPbVAnFkMgFaQY1tmdA+ywS5bO31GoJQnuyRR6MqWp8IX8RgSUYa8",
      "receiveTime": 1643103015952,
      "receiveOffset": 1
    }
  ]
}
```

### /Peer/GetCommand

```bash
curl -X 'GET' \
  'https://api.bazar.social/UserQuery/GetCommand?commandID=89oGjHkfjhnnln2osqR5LG8bGpafGy' \
  -H 'accept: text/plain'
```

```bash
{
  "success": true,
  "msg": "",
  "data": {
    "commandID": "89oGjHkfjhnnln2osqR5LG8bGpafGy",
    "commandTime": 1636611848,
    "userID": "SeQLiznuyoKPkaSMZ0r7DZJv35AhUn",
    "commandType": "UserInfo",
    "commandContent": "{\"userID\":\"SeQLiznuyoKPkaSMZ0r7DZJv35AhUn\",\"publicKey\":\"MIGbMBAGByqGSM49AgEGBSuBBAAjA4GGAAQBU2Lejd0WYe4tmj58oAFIm97lmvitXS8pYm9//OYtJA5dWC7rYArPPXFjS5+8qb/kbWFKDGLy5AYoGJePH6v1+y4BiylYV85tl/AjXk0uaM/XKTtcDDurKCZi74NV3SMEEaqkCmt+6lTLGiS1B04hl2sFhPZEcXRTssNo5TpZB0IZmyU=\\t\",\"userName\":\"user127931102\",\"website\":\"\",\"location\":\"\",\"bio\":\"\",\"commandID\":\"89oGjHkfjhnnln2osqR5LG8bGpafGy\",\"commandTime\":1636611848}",
    "signature": "AFV8t244GtjOt9qiriJqiCJDCedY9ux+2r4rjaJjmFTOrBq+66SdSs7y6UVENrSdFVJY3uAXOugF1jAn58qLa4EPAXD7/urzjERInfsGe8etMMkhL90kQChC3JIhPbVAnFkMgFaQY1tmdA+ywS5bO31GoJQnuyRR6MqWp8IX8RgSUYa8",
    "receiveTime": 1643103015952,
    "receiveOffset": 1
  }
}
```

***

## Code snippets

### Create key-pair

```C#
public static (string privateKey, string publicKey) GeneratePair()
{
    //keySize 521 by default
    using ECDsa sa = ECDsa.Create();
    var buf = sa.ExportPkcs8PrivateKey();
    string privateKey = Convert.ToBase64String(buf);
    string publicKey = Convert.ToBase64String(sa.ExportSubjectPublicKeyInfo());

    return (privateKey, publicKey);
}
```

### Calculate userID

```C#
public static string CalculateUserID(string publicKey)
{
    char[] ay = new char[30];

    for (int i = 0; i < publicKey.Length; i++)
    {
        int pos = i % ay.Length;
        ay[pos] += publicKey[i];
    }

    var chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
    for (int i = 0; i < ay.Length; i++)
    {
        int idx = (int)ay[i] % chars.Length;
        ay[i] = chars[idx];
    }

    return new string(ay);
}
```

### Signature and verification

```C#

public static string Signing(string privateKey, string str, string encoding = "utf-8")
{
    byte[] bt = Encoding.GetEncoding(encoding).GetBytes(str);

    ECDsa dsa = ECDsa.Create();
    dsa.ImportPkcs8PrivateKey(Convert.FromBase64String(privateKey), out _);
    var buf = dsa.SignData(bt, HashAlgorithmName.SHA256);
    return Convert.ToBase64String(buf);
}

public static bool CheckSignature(string strContent, string signature, string publicKey, string encoding = "utf-8")
{
    try
    {
        byte[] bt = Encoding.GetEncoding(encoding).GetBytes(strContent);

        ECDsa dsa = ECDsa.Create();
        dsa.ImportSubjectPublicKeyInfo(Convert.FromBase64String(publicKey), out _);
        byte[] rgbSignature = Convert.FromBase64String(signature);
        bool ret = dsa.VerifyData(bt, rgbSignature, HashAlgorithmName.SHA256);
        return ret;
    }
    catch
    {
        return false;
    }
}
```
