# mdx2apple

将 MDict (.mdx) 词典转换为 macOS 词典格式。

## 为什么需要这个工具？

MDict 词典的 HTML 结构复杂，Apple Dictionary 无法正确渲染。本工具：

1. 将 MDX 转换为 AppleDict XML（通过 pyglossary）
2. **清理并简化 HTML** 以兼容 Dictionary.app
3. 生成优化的 CSS 和 plist
4. 编译并安装词典

## 系统要求

- macOS
- Python 3.8+
- Dictionary Development Kit（自动从 GitHub 镜像安装）

## 安装

```bash
# 克隆仓库
git clone https://github.com/nxxxsooo/mdx2apple.git
cd mdx2apple

# 安装依赖
pip install pyglossary lxml beautifulsoup4 html5lib

# 安装 DDK（如果尚未安装）
git clone https://github.com/ikey4u/macddk.git ~/Library/Developer/Dictionary-Development-Kit
```

## 使用方法

### 基本用法

```bash
python mdx2apple.py /path/to/dictionary.mdx
```

### 带参数

```bash
python mdx2apple.py /path/to/dictionary.mdx \
    --name "牛津词典" \
    --output ./my_dict \
    --install
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `--name` | 在 Dictionary.app 中显示的名称 |
| `--output`, `-o` | 输出目录（默认：`./output`） |
| `--identifier` | Bundle ID（如 `com.example.dict`） |
| `--install` | 转换后自动编译并安装 |
| `--keep-resources` | 保留音频/图片资源（文件较大） |

## 手动编译

转换后：

```bash
cd output
make          # 编译词典
make install  # 安装到 ~/Library/Dictionaries
```

添加音频/图片资源：

```bash
rsync -a OtherResources/ ~/Library/Dictionaries/词典名.dictionary/Contents/Resources/
```

## 限制

- **无音频播放**：Apple Dictionary 不支持 JavaScript 音频
- **样式简化**：MDict 的复杂 CSS 样式会被移除
- **大型词典**：处理可能需要几分钟

## 工作原理

1. **pyglossary** 将 MDX 转换为 AppleDict XML 格式
2. **BeautifulSoup** 解析并清理每个词条：
   - 提取：词头、发音、释义、例句
   - 移除：自定义属性、JavaScript、复杂嵌套
   - 转义：XML 特殊字符（`&`、`<`、`>`）
3. 生成简化的 **CSS**，包含关键的 `d|entry { display: block; }`
4. 创建精简的 **plist**（复杂 plist 可能导致问题）
5. **DDK** 编译为 `.dictionary` 包

## 常见问题

### 词典显示词条但内容空白

- 检查 CSS 是否包含 `d|entry { display: block; }`
- 使用简化版 plist
- 先尝试编译小型测试词典

### 编译失败提示 "Argument list too long"

- 编译前移走 `OtherResources/` 文件夹
- 安装后用 `rsync` 添加资源

### 词典不显示

```bash
rm -rf ~/Library/Caches/com.apple.Dictionary
killall Dictionary
killall DictionaryWorkerService
```

## 致谢

- [pyglossary](https://github.com/ilius/pyglossary) - 词典格式转换
- [macddk](https://github.com/ikey4u/macddk) - DDK GitHub 镜像

## 许可证

MIT
