通过调用[小析智能API]([API开发文档 - 小析智能](https://wiki.xiaoxizn.com/#parser)),   解析文件内容,    提供一个简历解析类以进行简历解析

## 类定义

### `LinkToAPI(string user, string token)`

- 构造函数
  - 输入:  小析智能API,   的用户名与Token

> ### 对pdf文件进行Base64编码
>
> [C#中pdf文件与base64字符串的相互转换_c# pdf转base64-CSDN博客](https://blog.csdn.net/qq_41760419/article/details/139118479)

### `GetResumeImfo(ResumeFile ) ->  ResumeImfo`

- 简历内容识别

  - 输入：
    - `ResumeFile`类型的对象
  - 输出：简历信息类的实例

### `GetSkillGrade(ResumeFile ) -> Dic`

- 技能评估

> [技能评估API](https://wiki.xiaoxizn.com/#portrait-result-skill)

  - 输入：
    - `ResumeFile`类型的对象

  - 输出：字典
    - 键为技能
    - 值为分数