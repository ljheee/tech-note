Browser MCP 实现有：
@modelcontextprotocol/server-puppeteer	官方 Puppeteer MCP
browser-tools-mcp	社区版，功能更丰富
mcp-server-playwright	基于 Playwright


Playwright Codegen 是一个神器：
你在浏览器里手动操作，它自动生成对应代码，完全不用手写！

```
# 基础录制
npx playwright codegen https://example.com

# 指定输出文件（自动保存代码）
npx playwright codegen -o script.js https://example.com

# 指定浏览器
npx playwright codegen --browser firefox https://example.com

# 模拟手机设备
npx playwright codegen --device "iPhone 14" https://example.com

# 设置语言（避免网站跳转到英文版）
npx playwright codegen --lang zh-CN https://example.com

# 带 Cookie 录制（已登录状态）
npx playwright codegen --load-storage auth.json https://example.com
```
