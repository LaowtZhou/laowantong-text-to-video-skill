# 老顽童文生视频 Skill

这是一个 Codex skill，用于把老顽童周老师的口播稿、SRT 字幕、公众号长文或视频脚本，转成可在 Remotion 中逐段预览、返修和渲染的视频工程。

核心原则：

- SRT 管时间轴。
- MD/原稿只用于校准转写错别字，不改句子、不增删句子。
- AI 图片只负责主视觉隐喻，不把中文核心文字画进图片。
- Remotion 只叠加标题、关键词、箭头标签和镜头运动，不烧录逐句字幕。
- 长视频图片生成后必须立即无损保存到项目目录，并维护 manifest，避免对话压缩、中断或 Token 用尽导致资产丢失。

## 适用场景

- 老顽童周老师口播稿转文生视频。
- SRT 精确时间轴分镜。
- Remotion 分段 composition 预览。
- AI 批量生成解释类视频分镜图。
- 长视频逐段审片、补图、调整切点、最终合成。

## 安装

把仓库里的 `laowantong-text-to-video` 文件夹复制到 Codex skills 目录。

常见位置：

```text
C:\Users\<你的用户名>\.codex\skills\laowantong-text-to-video
```

如果你在某个项目内使用项目级 skill，也可以放到：

```text
<项目根目录>\.agents\skills\laowantong-text-to-video
```

当前这个仓库是标准 skill 包结构，真正需要被 Codex 读取的是：

```text
laowantong-text-to-video/
  SKILL.md
  agents/
    openai.yaml
  scripts/
    extract_imagegen_results.py
```

## 推荐 Remotion 项目结构

这个 skill 默认把 `remotion-animation` 当成多个视频共用的 Remotion 工作区，而不是每个视频都新建一套 `node_modules`。

```text
remotion-animation/
  package.json
  package-lock.json
  node_modules/
  remotion.config.ts
  tsconfig.json
  src/
    shared/
      components/
      styles/
      utils/
  videos/
    <video-slug>/
      public/
        generated-frames/
        reference-images/
      src/
        StoryboardComposition.tsx
        storyboard.ts
      out/
      README.md
```

规则：

- 共用依赖放在 `remotion-animation` 根目录。
- 每条视频放在 `remotion-animation/videos/<video-slug>/`。
- 每条视频自己的图片、SRT 分镜、preview stills、导出视频都放在对应 video 文件夹。
- 共用组件放在根目录 `src/shared/`。

## 典型工作流

1. 确认输入：SRT 是时间轴基准，MD/原稿只校准同音错字和专有名词。
2. 拆分分镜：按语义转折分组，不机械每 5 秒一张图。
3. 生成图片：一张成功就立刻保存到 `public/generated-frames/`，并更新 manifest。
4. 接入 Remotion：注册分段 composition，先逐段审片，再做整片合成。
5. 返修：按用户指出的时间点补图、切图、调 Part 边界。
6. 验证：跑 lint、渲染 still、检查帧间隙、确认图片尺寸和路径。

## 图片生成标准

- 横屏优先 `1920x1080` 或更高。
- 如果没有 API/CLI 高分辨率生成路径，可以接受 Codex 内置 `image_gen` 的原生横屏输出，例如已验证的 `1672x941`，但必须在 manifest 记录实际尺寸和生成方式。
- 生成图里不要包含可读文字、水印、logo、字幕、真实人脸、黑板粉笔风背景。
- 中文标题、关键词、箭头标签统一由 Remotion 后期叠加。

## 长任务安全规则

长视频生成几十张甚至上百张图片，不能依赖聊天窗口或 session 记忆。

- 每生成成功 1 张图，立即保存到视频项目目录。
- 保存后检查文件大小、PNG 头和图片尺寸。
- 更新 `public/generated-frames/manifest.json`。
- 中断后先读 manifest，只补缺失图片。
- 补图不要覆盖旧图，使用 `-v2` 或语义化新文件名。

## 辅助脚本

`scripts/extract_imagegen_results.py` 用于从 Codex session JSONL 中提取内置 `image_gen` 生成结果。

示例：

```powershell
python .\laowantong-text-to-video\scripts\extract_imagegen_results.py `
  --session C:\Users\Alex\.codex\sessions\YYYY\MM\DD\rollout-xxx.jsonl `
  --out-dir C:\AI_workspace\AI内容创作\remotion-animation\videos\salary-average\public\generated-frames `
  --count 1 `
  --prefix scene `
  --start-index 17 `
  --min-width 1672 `
  --min-height 941
```

## 维护说明

- `SKILL.md` 是给 Codex 读取的核心流程文档，保持精简和可执行。
- `README.md` 是给 GitHub 访问者看的项目说明，不放进 skill 文件夹。
- 修改 skill 后建议运行 Codex skill 校验脚本：

```powershell
python C:\Users\Alex\.codex\skills\.system\skill-creator\scripts\quick_validate.py C:\path\to\laowantong-text-to-video
```

## 许可

暂未声明开源许可证。未获得明确授权前，不默认添加 MIT、Apache-2.0 或其他 License。
