# quickstart-ai-real-time-digital-human-call

阿里云计算巢「AI实时数字人通话」服务源，包含服务定义（`.computenest/`）与部署文档（`docs/`）。

服务将 GPU 驱动、本地大模型（llama.cpp + Qwen3）、口型同步（LiveTalking + wav2lip256）、语音链路与 Studio 制作平台在部署物镜像构建期烘焙，部署期补齐 Studio、HTTPS 与按 GPU 算力重编的组件，一键交付实时数字人对话与制作平台。

## 目录结构

- `.computenest/` — 计算巢服务定义（`config.yaml`、ROS 模板、服务测试用例、图标与技术说明 README）
- `docs/` — 面向客户的部署文档（MkDocs 源，发布至 GitHub Pages）

## 部署文档

查看服务实例部署在线文档，请访问 [服务实例部署文档](https://aliyun-computenest.github.io/quickstart-ai-real-time-digital-human-call)。

本文档通过 [MkDocs](https://github.com/mkdocs/mkdocs) 生成，参考[使用文档](https://www.mkdocs.org/getting-started/#installation)：

1）安装和使用：

```shell
$ pip install mkdocs # or use pip3 安装文档工具
$ pip install --upgrade mkdocs-aliyun-computenest # or use pip3 安装计算巢主题
$ mkdocs serve # in root folder
```

2）本地预览：本地在浏览器打开 [http://localhost:8000/](http://localhost:8000/) 。

3）本地新建分支后，提交 `Pull request` 到 `main` 分支。

4）合并至 `main` 分支后，查看 pages 部署结果。
