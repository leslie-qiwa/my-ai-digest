# Show HN: Make PDFs look scanned (CLI or in the browser via WASM)

一款命令行工具，能将 PDF 文件处理成物理打印件扫描件的效果——包括倾斜、灰度、暖色纸张色调、扫描噪点、虚焦、边缘阴影和 JPEG 压缩伪影。同时支持通过 WASM 在浏览器中运行。

每一页先被光栅化为图像，经过效果处理流水线，再重新组装成一个纯图像的新 PDF（原始可选中文本将被去除——忠实模拟基础扫描仪的效果）。

需要 Go 和 C 工具链（go-fitz 通过 cgo 链接 MuPDF，因此生成的二进制文件是自包含的——运行时无需额外安装）。

```
go build -o make-look-scanned .
make-look-scanned [flags] input.pdf
```

参数可以出现在输入文件名之前或之后。

```
make-look-scanned in.pdf                # -> in.scanned.pdf
make-look-scanned in.pdf -o out.pdf
make-look-scanned in.pdf --noise 0.4 --skew 2.5 --jpeg-quality 30
```

| 参数 | 默认值 | 含义 |
|---|---|---|
| -o | `<input>.scanned.pdf` | 输出路径 |
| --preset | — | 来自 config.toml 的命名预设 |
| --seed | 内容哈希 | 随机种子（覆盖以获得不同效果） |
| --force | false | 覆盖已有的输出文件 |
| --dpi | 150 | 渲染分辨率 |
| --skew | 0.6 | 最大旋转角度（0 表示禁用） |
| --grayscale | true | 去色（--grayscale=false 保留彩色） |
| --paper-tone | 0.6 | 暖色纸张色调强度 0..1 |
| --noise | 0.08 | 扫描噪点 0..1 |
| --blur | 0.4 | 虚焦高斯 sigma |
| --edge-shadow | 0.15 | 边缘暗角 0..1 |
| --jpeg-quality | 70 | JPEG 质量 1..100 |

每个数值参数设为 0 即可禁用对应效果。

默认情况下输出是确定性的：种子值由输入 PDF 的内容派生，因此相同文件始终产生相同的扫描效果。传入 `--seed N` 可获得不同（但可复现的）外观。相同输入加相同种子会生成字节级一致的 PDF。

可在 `$XDG_CONFIG_HOME/make-look-scanned/config.toml`（未设置 `XDG_CONFIG_HOME` 时回退至 `~/.make-look-scanned/config.toml`）中定义可复用的预设组合。键名与参数名相同，使用下划线：

```toml
[presets.medium]
skew = 1.5
paper_tone = 0.6
noise = 0.2
blur = 0.6
edge_shadow = 0.3
jpeg_quality = 45
```

```
make-look-scanned --preset medium in.pdf
```

优先级：内置默认值 → 选定预设 → 显式命令行参数（命令行参数始终优先）。

效果处理流水线也可在浏览器中运行。go-fitz/MuPDF 无法编译为 WASM，因此浏览器版本使用 PDF.js 来光栅化页面，然后将像素数据交给编译为 WASM 的相同 Go 效果处理和组装代码。

开发模式（需要网络以加载 PDF.js CDN）：

```
./web/build.sh                          # 构建 web/main.wasm + wasm_exec.js
(cd web && python3 -m http.server 8080) # 然后打开 http://localhost:8080
```

单文件自包含版本（离线可用，无需服务器）：

```
task build:web    # 生成 dist/make-look-scanned.html（约 8 MB）
```

`dist/make-look-scanned.html` 将 WASM、Go 运行时胶水代码和 PDF.js（库 + worker）以 base64 形式内联——直接在浏览器中打开即可。输出与 CLI 在视觉上等效，但不是字节级一致的，因为 PDF.js 和 MuPDF 的光栅化方式不同。

AGPL-3.0 许可证。CLI 静态链接了 MuPDF（通过 go-fitz），MuPDF 采用 AGPL-3.0 许可，因此组合后的二进制文件也是 AGPL-3.0——分发它需要提供对应的源代码。浏览器版本不包含 MuPDF（使用 PDF.js，Apache-2.0 许可）。
