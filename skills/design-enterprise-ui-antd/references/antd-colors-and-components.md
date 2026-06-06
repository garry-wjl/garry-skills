# Ant Design Colors, Theme and Components

## Ant Design Color Schemes (官方体系)

All listed schemes are from official Ant Design system. Provide options during design; user chooses, then configure.

- **Brand primary color**
  - Default Blue: `#1677FF` (official Ant Design brand)
  - 12-color palette primary (choose one): Twilight (red), Volcano, Sunset, Sunflower, Sunrise, Lime, Cyan, Geek Blue, Jadeite Blue, Daybreak Blue, Sauce Purple, French Fuchsia
  - Reference: [Colors - Ant Design](https://ant.design/docs/spec/colors)

- **Theme variants**
  - `theme`: `default` | `compact` (compact) | `dark` (dark), switch via `<ConfigProvider theme={{ algorithm }}>` or `theme` package's `defaultAlgorithm`, `compactAlgorithm`, `darkAlgorithm`

- **Custom primary color**
  - Use `@ant-design/colors` generate derivative colors, inject via ConfigProvider's `theme.token.colorPrimary`
  - Install: `npm install @ant-design/colors`

Example: list 2-3 schemes for user choice, e.g., "Default Blue", "Daybreak Blue", "Dark + Cyan".

---

## Ant Design X (Agent Conversations)

When requirement includes **Agent conversation, AI chat, multi-turn sessions**, use **Ant Design X** within UI project.

- **Documentation**: [Ant Design X Introduction](https://x.ant.design/x-sdks/introduce-cn/)
- **Install**: `npm install @ant-design/x-sdk --save`
- **Core capabilities**
  - **useXChat**: Single session data mgmt (messages, state, operation API).
  - **useXConversations** / **useXAgent**: Multi-session list & mgmt, model dispatch.
  - **Chat Provider**: Connect different AI models (OpenAI compatible, DeepSeek, etc.).
  - **XRequest / XStream**: Request & streaming response.
- **UI components** (as needed): Bubble, Sender, Welcome, Conversations, Prompts, ThoughtChain, etc.
- Add a "conversation" or "chat" page in project using above SDK & components for dialogue interface; integrate with backend Agent API.
