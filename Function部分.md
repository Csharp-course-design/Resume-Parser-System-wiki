> ### 即基础服务支持组件
该部分提供各层所需要的基础服务

其中,    已经包含的基础服务如下

## 包含的基础服务

### `Base64Helper`

- `Base64`编码支持

  - 将文件以`Base64`编码

  - 将文件以`Base64`解码

> ```csharp
> // 将简历文件编码为 Base64
> string fileContent = Convert.ToBase64String(File.ReadAllBytes(filePath));
> string fileName = Path.GetFileName(filePath);
> 
> using System;
> using System.IO;
> 
> class ResumeFileHandler
> {
>     /// <summary>
>     /// 将简历文件编码为 Base64 字符串并获取文件名
>     /// </summary>
>     /// <param name="filePath">简历文件路径</param>
>     /// <param name="fileName">输出文件名</param>
>     /// <returns>Base64 编码字符串</returns>
>     public static string EncodeFileToBase64(string filePath, out string fileName)
>     {
>         if (string.IsNullOrWhiteSpace(filePath))
>         {
>             throw new ArgumentException("文件路径不能为空", nameof(filePath));
>         }
> 
>         if (!File.Exists(filePath))
>         {
>             throw new FileNotFoundException("文件未找到", filePath);
>         }
> 
>         try
>         {
>             // 获取文件名
>             fileName = Path.GetFileName(filePath);
> 
>             // 读取文件内容并编码为 Base64
>             byte[] fileBytes = File.ReadAllBytes(filePath);
>             return Convert.ToBase64String(fileBytes);
>         }
>         catch (Exception ex)
>         {
>             Console.WriteLine($"发生错误: {ex.Message}");
>             throw;
>         }
>     }
>     
>     
>     /// <summary>
>     /// 将 Base64 编码内容解码并写回指定目录
>     /// </summary>
>     /// <param name="base64Content">Base64 编码内容</param>
>     /// <param name="outputDirectory">目标文件目录</param>
>     /// <param name="fileName">原始文件名</param>
>     public static void DecodeBase64ToFile(string base64Content, string outputDirectory, string fileName)
>     {
>         if (string.IsNullOrWhiteSpace(base64Content))
>         {
>             throw new ArgumentException("Base64 内容不能为空", nameof(base64Content));
>         }
> 
>         if (string.IsNullOrWhiteSpace(outputDirectory))
>         {
>             throw new ArgumentException("目标文件目录不能为空", nameof(outputDirectory));
>         }
> 
>         if (string.IsNullOrWhiteSpace(fileName))
>         {
>             throw new ArgumentException("文件名不能为空", nameof(fileName));
>         }
> 
>         try
>         {
>             // 确保输出目录存在
>             if (!Directory.Exists(outputDirectory))
>             {
>                 Directory.CreateDirectory(outputDirectory);
>             }
> 
>             // 解码 Base64 内容
>             byte[] fileBytes = Convert.FromBase64String(base64Content);
> 
>             // 构建完整的文件路径
>             string outputFilePath = Path.Combine(outputDirectory, fileName);
> 
>             // 将解码后的内容写入文件
>             File.WriteAllBytes(outputFilePath, fileBytes);
> 
>             Console.WriteLine($"文件已成功写入: {outputFilePath}");
>         }
>         catch (Exception ex)
>         {
>             Console.WriteLine($"发生错误: {ex.Message}");
>             throw;
>         }
>     }
> }
> 
> // 示例使用
> class Program
> {
>     static void Main(string[] args)
>     {
>         string base64Content = "your_base64_encoded_string_here"; // 替换为实际 Base64 字符串
>         string outputDirectory = @"C:\DecodedResumes"; // 替换为目标目录
>         string fileName = "resume.pdf"; // 替换为实际文件名
> 
>         ResumeFileHandler.DecodeBase64ToFile(base64Content, outputDirectory, fileName);
>     }
> }
> 
> ```

### `Factory`

- 对象工厂
  - `ResumeFileFactory`简历文件对象工厂,支持将指定路径下的文件生成为简历文件对象
  - `ResumeImfoFactory`简历信息对象工厂,支持将指定格式`Json`格式字符转换为简历信息对象.

### 基于 NPL 的文字分词与关键字提取

通过使用 `OpenNLP` 对传入的文本进行分词

- `Split(string ) -> string[]`
  - 对传入的字符串进行分词
  - 返回分好的所有词语。