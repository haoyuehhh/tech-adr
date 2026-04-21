# ADR-001: 从REST工具调用迁移至MCP协议

- **状态**: 已接受
- **日期**: 2026-03-15
- **作者**: 季澔辰
- **相关项目**: GraphRAG-Agent

## 背景
GraphRAG初期使用传统REST API封装检索工具，LLM通过HTTP调用外部服务。随着工具数量增加（检索/计算/外部API），出现三个问题：
1. 工具描述与调用逻辑耦合在Prompt里，维护困难
2. 每新增一个工具需修改后端路由+前端配置，扩展成本高
3. Claude/Cursor等客户端无法直接复用这些工具能力

## 决策
将工具层全面迁移至MCP（Model Context Protocol）协议，独立开发GraphRAG MCP Server，使工具能力成为标准化服务。

## 备选方案

| 方案 | 优点 | 缺点 | 结论 |
|---|---|---|---|
| 继续REST+手动Prompt | 实现快，团队熟悉 | 工具一多就崩，LLM理解成本高 | 否 |
| LangChain Tools封装 | 生态成熟 | 与Claude/Cursor客户端不互通， vendor lock-in | 否 |
| MCP协议 | 标准化、客户端无关、双向生态 | 需额外开发Server，文档较少 | **采纳** |

## 影响
- **正面**: Claude/Cursor可直接调用知识图谱检索；新增工具只需实现MCP标准接口，无需改LLM侧代码
- **负面**: 需维护MCP Server独立进程，本地调试链路变长
- **技术债务**: 当前仅实现stdio传输，未来需补充SSE传输支持高并发场景

## 验证
- 代码: [graphrag-agent/mcp-server/](https://github.com/haoyuehhh/graphrag-agent)
- 效果: MCP Server接入后，Claude Desktop可直接执行图谱检索，工具调用成功率从REST时代的72%提升至98%
- 时间投入: 3天开发，1天联调

## 相关链接
- MCP协议规范: https://modelcontextprotocol.io/
- 实现代码: https://github.com/haoyuehhh/graphrag-agent/tree/main/mcp
