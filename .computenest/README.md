# AI实时数字人通话 ComputeNest 服务

在阿里云 **L20 GPU（ecs.gn8is）** ECS 上一键部署阿里云官方 AI 数字人方案。本服务将官方引导脚本
`bootstrap.sh` 的重资产（GPU 驱动、本地大模型、口型权重、形象包）**在部署物镜像构建期烘焙**，部署时仅补齐
Studio（写入百炼 Key）、HTTPS（绑定公网 IP）与未烘焙的 GPU 组件，实现端到端交付。

## 组件构成

| 组件 | 说明 |
| --- | --- |
| GPU 驱动 + CUDA | 通过阿里云官方 `auto_install` 安装，**构建期烘焙进镜像**，用户实例首启即加载（无需部署期重启） |
| llama.cpp + Qwen3 | 本地大模型 |
| LiveTalking + wav2lip256 | 口型同步 |
| 语音链路 + 前端 | 实时对话 |
| Studio 制作平台（BFF + 前端） | 结合**百炼 API-KEY** 提供 AI 生图/生视频 |
| nginx + HTTPS | 麦克风刚需；默认自签证书，配置域名后走 Let's Encrypt |

## 部署形态

- **规格**：`ecs.gn8is`（NVIDIA L20，单卡 48GB 显存）
- **系统盘**：默认 100 GiB（模型 + PyTorch 环境约 40G+）
- **网络**：直接暴露实例公网 IP；安全组放行 **TCP 22/80/443 + UDP 50000-50100**（WebRTC 媒体，缺则画面黑屏）
- **访问**：部署完成后输出 `https://<公网IP>/`

## 关键实现：镜像方式（构建期烘焙 + 部署期补齐）

采用 **EcsImage 部署物** 方案，把驱动与大体积下载物固化进镜像，规避部署期的驱动安装与强制重启：

1. **构建期**（`config.yaml` 的 `Artifact.EcsImage.ArtifactBuildProperty`，基于 **Alibaba Cloud Linux 4** + `EnableGpu`）：
   `bootstrap.sh --download-only` 下载并解压部署包/资产包，`deploy.sh driver` 安装 NVIDIA 驱动（DKMS 按本镜像内核编译）；
   若驱动免重启即加载，则顺带 `SKIP_STUDIO=1 SKIP_HTTPS=1 deploy.sh all` 烘焙 llm/livetalking/avatars/services。
2. **部署期**（模板 `RunCommand` → 幂等 systemd oneshot）：驱动随首启加载（兜底 `modprobe`/最多一次重启），
   随后 `deploy.sh all`（幂等）补齐 Studio（写入百炼 API-KEY）与 HTTPS（自签证书绑定公网 IP），
   完成后写入 `/root/ai-avatar.done` 并回调 `WaitCondition`（成功/失败）。

日志位于实例的 `/var/log/ai-avatar-install.log`。

## 参数说明

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| `EcsInstanceType` | 是 | L20 规格（`ecs.gn8is` 系列） |
| `SystemDiskSize` | 是 | 系统盘大小，默认 100，最小 100 |
| `InstancePassword` | 是 | 实例登录密码 |
| `BaiLianApiKey` | 否 | 百炼 API-KEY，用于 Studio 生图/生视频（不配核心功能仍可用） |
| `Domain` | 否 | 已解析到本实例公网 IP 的域名，配置后签发 Let's Encrypt 免告警证书 |
| `AvatarId` | 否 | 默认数字人形象 ID |

> 交付超时（`DeployTimeout` / WaitCondition `Timeout`）预留约 90 分钟：若构建期已烘焙 GPU 组件，部署期通常
> 仅需数分钟（Studio + HTTPS）；若构建期 GPU 未就绪，则部署期在线补齐 llm/livetalking 等（约 15–25 分钟）。

## 备注

- **基础镜像**：部署物镜像基于 **Alibaba Cloud Linux 4 LTS 64 位**（`SourceImageId: aliyun_4_x64_20G_alibase_20260801.vhd`）。
- 模板中 `InstanceGroup.Properties.ImageId: ecs_image` 为 ComputeNest 占位符，部署时替换为已构建的镜像 ID。
- `resources/icons/service_logo.png` 为占位图标，正式上架前请替换为服务专属图标。
