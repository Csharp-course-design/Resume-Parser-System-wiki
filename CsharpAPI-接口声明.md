> ### 以下是 `IServer` 接口的详细说明文档：

---

### **接口名称**：`IServer`

`IServer` 接口定义了与简历解析及技能评分相关的核心功能，用于解析简历文件并提供技能评级服务。

---

### **命名空间**
`CsharpAPI`

---

### **引用命名空间**
```csharp
using Models;
using Models.ResumeImfo;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
```

---

### **方法**

#### 1. **`ExtractResumeFile`**

**功能**：  
解析给定的简历文件对象并返回包含解析结果的简历信息类。

**方法签名**：
```csharp
ResumeImfo ExtractResumeFile(ResumeFile resumeFile);
```

**参数**：
- `resumeFile` (`ResumeFile`):  
  表示需要解析的简历文件对象。

**返回值**：
- `ResumeImfo`:  
  包含简历内容的对象，例如个人信息、工作经历和教育背景等。

**示例**：
```csharp
ResumeFile file = ResumeFileFactory.Get("path/to/resume.pdf");
ResumeImfo resume = server.ExtractResumeFile(file);
Console.WriteLine(resume.Name);
```

#### 2. **`GetSkillGrade`**

**功能**：  
根据给定的简历文件分析技能，并返回技能与评分的键值对。

**方法签名**：
```csharp
Dictionary<string, double> GetSkillGrade(ResumeFile resumeFile);
```

**参数**：
- `resumeFile` (`ResumeFile`):  
  表示需要评估技能的简历文件对象。

**返回值**：
- `Dictionary<string, double>`:  
  一个技能与评分的映射，其中：  
  - 键为技能的字符串表示，例如 `"C#"`、`"Project Management"`。
  - 值为对应技能的评分，范围通常为 0 到 10（具体评分标准根据实现定义）。

**备注**：  
技能评分可能基于简历中提到的关键词出现频率、上下文权重等因素计算。

**示例**：
```csharp
ResumeFile file = ResumeFileFactory.Get("path/to/resume.pdf");
Dictionary<string, double> skillGrades = server.GetSkillGrade(file);

foreach (var skill in skillGrades)
{
    Console.WriteLine($"Skill: {skill.Key}, Grade: {skill.Value}");
}
```

---

### **说明**

该接口适用于简历解析与技能评估场景，开发者可实现 `IServer` 接口以满足特定业务需求。  

具体实现需确保输入输出的一致性，且可能需要支持多种简历文件格式（如 PDF、Word）。

---