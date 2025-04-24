# FFmpeg Plugin 使用指南

FFmpeg插件是一个强大的Dify扩展，利用FFmpeg功能来处理视频和音频文件。本插件包含多个工具，可以获取视频信息、转换格式、剪切视频、压缩视频和提取音频。

## 前提条件

使用此插件需要满足以下条件：

- FFmpeg必须安装在运行Dify的服务器上
- 确保FFmpeg命令可以在命令行中访问（在PATH环境变量中）

## 工具概述

### 1. 视频信息 (Video Info)

获取视频文件的详细技术信息。

**输入参数：**
- 视频文件（必填）：要分析的视频文件

**输出：**
- 视频格式、编解码器、分辨率、帧率、时长等详细信息

### 2. 视频格式转换 (Convert Video)

将视频从一种格式转换为另一种格式。

**输入参数：**
- 视频文件（必填）：要转换的视频文件
- 目标格式（必填）：要转换的目标格式，例如mp4、avi、mov、mkv等

**输出：**
- 指定格式的转换后视频文件

### 3. 视频剪切 (Trim Video)

剪切视频以提取特定的片段。

**输入参数：**
- 视频文件（必填）：要剪切的视频文件
- 开始时间（必填）：剪切的开始时间，格式为HH:MM:SS或秒数
- 结束时间（必填）：剪切的结束时间，格式为HH:MM:SS或秒数

**输出：**
- 剪切后的视频片段文件

### 4. 视频压缩 (Compress Video)

压缩视频以减小文件大小。

**输入参数：**
- 视频文件（必填）：要压缩的视频文件
- 压缩级别（可选）：可以是"low"、"medium"或"high"，默认为"medium"

**输出：**
- 压缩后的视频文件

### 5. 提取音频 (Extract Audio)

从视频文件中提取音频轨道。

**输入参数：**
- 视频文件（必填）：要提取音频的视频文件
- 音频格式（可选）：输出音频的格式，可以是mp3、aac、wav、ogg、flac，默认为mp3

**输出：**
- 提取出的音频文件

## 使用示例

下面是一些使用示例：

### 视频信息

```
获取视频信息
视频文件：[上传视频文件]
```

### 转换视频格式

```
转换视频格式为WebM
视频文件：[上传视频文件]
目标格式：webm
```

### 剪切视频

```
剪切视频从1分30秒到2分45秒
视频文件：[上传视频文件]
开始时间：00:01:30
结束时间：00:02:45
```

### 压缩视频

```
高度压缩视频
视频文件：[上传视频文件]
压缩级别：high
```

### 提取音频

```
从视频中提取高质量音频
视频文件：[上传视频文件]
音频格式：flac
```

## 疑难解答

如果遇到问题，请检查：

1. FFmpeg是否正确安装，可通过在命令行运行`ffmpeg -version`验证
2. 上传的视频文件是否损坏或格式不受支持
3. 对于格式转换，确保指定了有效的目标格式
4. 对于视频剪切，确保开始时间小于结束时间

## 支持的格式

- 视频格式：mp4、avi、mov、mkv、webm、flv、wmv等
- 音频格式：mp3、aac、wav、ogg、flac

## 性能注意事项

- 大型视频文件的处理可能需要较长时间
- 高清视频的压缩和转换可能会消耗大量系统资源

## User Guide of how to develop a Dify Plugin

Hi there, looks like you have already created a Plugin, now let's get you started with the development!

### Choose a Plugin type you want to develop

Before start, you need some basic knowledge about the Plugin types, Plugin supports to extend the following abilities in Dify:
- **Tool**: Tool Providers like Google Search, Stable Diffusion, etc. it can be used to perform a specific task.
- **Model**: Model Providers like OpenAI, Anthropic, etc. you can use their models to enhance the AI capabilities.
- **Endpoint**: Like Service API in Dify and Ingress in Kubernetes, you can extend a http service as an endpoint and control its logics using your own code.

Based on the ability you want to extend, we have divided the Plugin into three types: **Tool**, **Model**, and **Extension**.

- **Tool**: It's a tool provider, but not only limited to tools, you can implement an endpoint there, for example, you need both `Sending Message` and `Receiving Message` if you are building a Discord Bot, **Tool** and **Endpoint** are both required.
- **Model**: Just a model provider, extending others is not allowed.
- **Extension**: Other times, you may only need a simple http service to extend the functionalities, **Extension** is the right choice for you.

I believe you have chosen the right type for your Plugin while creating it, if not, you can change it later by modifying the `manifest.yaml` file.

### Manifest

Now you can edit the `manifest.yaml` file to describe your Plugin, here is the basic structure of it:

- version(version, required)：Plugin's version
- type(type, required)：Plugin's type, currently only supports `plugin`, future support `bundle`
- author(string, required)：Author, it's the organization name in Marketplace and should also equals to the owner of the repository
- label(label, required)：Multi-language name
- created_at(RFC3339, required)：Creation time, Marketplace requires that the creation time must be less than the current time
- icon(asset, required)：Icon path
- resource (object)：Resources to be applied
  - memory (int64)：Maximum memory usage, mainly related to resource application on SaaS for serverless, unit bytes
  - permission(object)：Permission application
    - tool(object)：Reverse call tool permission
      - enabled (bool)
    - model(object)：Reverse call model permission
      - enabled(bool)
      - llm(bool)
      - text_embedding(bool)
      - rerank(bool)
      - tts(bool)
      - speech2text(bool)
      - moderation(bool)
    - node(object)：Reverse call node permission
      - enabled(bool) 
    - endpoint(object)：Allow to register endpoint permission
      - enabled(bool)
    - app(object)：Reverse call app permission
      - enabled(bool)
    - storage(object)：Apply for persistent storage permission
      - enabled(bool)
      - size(int64)：Maximum allowed persistent memory, unit bytes
- plugins(object, required)：Plugin extension specific ability yaml file list, absolute path in the plugin package, if you need to extend the model, you need to define a file like openai.yaml, and fill in the path here, and the file on the path must exist, otherwise the packaging will fail.
  - Format
    - tools(list[string]): Extended tool suppliers, as for the detailed format, please refer to [Tool Guide](https://docs.dify.ai/plugins/schema-definition/tool)
    - models(list[string])：Extended model suppliers, as for the detailed format, please refer to [Model Guide](https://docs.dify.ai/plugins/schema-definition/model)
    - endpoints(list[string])：Extended Endpoints suppliers, as for the detailed format, please refer to [Endpoint Guide](https://docs.dify.ai/plugins/schema-definition/endpoint)
  - Restrictions
    - Not allowed to extend both tools and models
    - Not allowed to have no extension
    - Not allowed to extend both models and endpoints
    - Currently only supports up to one supplier of each type of extension
- meta(object)
  - version(version, required)：manifest format version, initial version 0.0.1
  - arch(list[string], required)：Supported architectures, currently only supports amd64 arm64
  - runner(object, required)：Runtime configuration
    - language(string)：Currently only supports python
    - version(string)：Language version, currently only supports 3.12
    - entrypoint(string)：Program entry, in python it should be main

### Install Dependencies

- First of all, you need a Python 3.11+ environment, as our SDK requires that.
- Then, install the dependencies:
    ```bash
    pip install -r requirements.txt
    ```
- If you want to add more dependencies, you can add them to the `requirements.txt` file, once you have set the runner to python in the `manifest.yaml` file, `requirements.txt` will be automatically generated and used for packaging and deployment.

### Implement the Plugin

Now you can start to implement your Plugin, by following these examples, you can quickly understand how to implement your own Plugin:

- [OpenAI](https://github.com/langgenius/dify-plugin-sdks/tree/main/python/examples/openai): best practice for model provider
- [Google Search](https://github.com/langgenius/dify-plugin-sdks/tree/main/python/examples/google): a simple example for tool provider
- [Neko](https://github.com/langgenius/dify-plugin-sdks/tree/main/python/examples/neko): a funny example for endpoint group

### Test and Debug the Plugin

You may already noticed that a `.env.example` file in the root directory of your Plugin, just copy it to `.env` and fill in the corresponding values, there are some environment variables you need to set if you want to debug your Plugin locally.

- `INSTALL_METHOD`: Set this to `remote`, your plugin will connect to a Dify instance through the network.
- `REMOTE_INSTALL_HOST`: The host of your Dify instance, you can use our SaaS instance `https://debug.dify.ai`, or self-hosted Dify instance.
- `REMOTE_INSTALL_PORT`: The port of your Dify instance, default is 5003
- `REMOTE_INSTALL_KEY`: You should get your debugging key from the Dify instance you used, at the right top of the plugin management page, you can see a button with a `debug` icon, click it and you will get the key.

Run the following command to start your Plugin:

```bash
python -m main
```

Refresh the page of your Dify instance, you should be able to see your Plugin in the list now, but it will be marked as `debugging`, you can use it normally, but not recommended for production.

### Package the Plugin

After all, just package your Plugin by running the following command:

```bash
dify-plugin plugin package ./ROOT_DIRECTORY_OF_YOUR_PLUGIN
```

you will get a `plugin.difypkg` file, that's all, you can submit it to the Marketplace now, look forward to your Plugin being listed!


## User Privacy Policy

Please fill in the privacy policy of the plugin if you want to make it published on the Marketplace, refer to [PRIVACY.md](PRIVACY.md) for more details.