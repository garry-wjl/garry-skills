# UI Project Structure and Setup

## UI Engineering Storage and Naming

- **Storage location**: Design output **must** be placed in **`doc/ui` folder** (relative to project root). Create `doc/ui` if it doesn't exist.
- **Naming rule (1-to-1 with PRD)**:  
  - PRD path: `doc/产品方案/YYYY-MM-DD_<product-or-requirement-name>-PRD.md`  
  - Corresponding UI project: `doc/ui/YYYY-MM-DD_<product-or-requirement-name>/`  
  - Maintain consistency: UI directory name (excluding `-PRD.md`) matches PRD file name prefix (includes date + product/requirement name).
  - Example: PRD `doc/产品方案/2025-03-16_家庭收支记录系统-PRD.md` → UI project `doc/ui/2025-03-16_家庭收支记录系统/`

- **1-to-1 correspondence**: Each PRD corresponds to one UI project directory. Multiple PRDs on same date require separate subdirectories. Project README **MUST document corresponding PRD path** (e.g., `doc/产品方案/2025-03-16_家庭收支记录系统-PRD.md`).

---

## UI Project Structure Recommendation

Design creates a standalone frontend project separated from backend, placed under `doc/ui/YYYY-MM-DD_<product-or-requirement>/`. **PC and Mobile MUST be separate independent projects, not mixed with routing in same repo.**

**Structure convention (PC + Mobile):**

```
doc/ui/YYYY-MM-DD_<product-or-requirement>/
├── web/                  # PC project: Ant Design Pro (Umi or Vite+React+antd+@ant-design/pro-components)
│   ├── package.json      # Dependencies: antd, @ant-design/pro-components, react, react-router-dom (per Umi scaffold for Umi approach)
│   ├── vite.config.ts
│   ├── index.html
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── theme/
│   │   ├── routes/
│   │   ├── pages/
│   │   └── components/
│   └── README.md
├── mobile/               # Mobile project: Vite + React + antd-mobile
│   ├── package.json      # Dependencies: react, antd-mobile, react-router-dom
│   ├── vite.config.ts
│   ├── index.html
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── theme/
│   │   ├── routes/
│   │   ├── pages/
│   │   └── components/
│   └── README.md
└── README.md             # Overall doc: PRD path reference + web/mobile entry points & launch instructions
```

- **PC only**: Create only `web/` project
- **Mobile only**: Create only `mobile/` project
- **PC `web/`**: **MUST use Ant Design Pro** — recommend **Umi + Pro official scaffold** or **Vite + React + TypeScript** with `antd` and `@ant-design/pro-components`. **`mobile/`** installs `antd-mobile`; both use independent build pipelines.
- Agent conversations: If present, add `@ant-design/x-sdk` to relevant endpoint (usually `web/`), use Ant Design X Hooks & components on conversation pages.

---

## Ant Design Pro (PC `web/`)

Enterprise **PC `web/` project MUST use Ant Design Pro** tech stack: not only main layout, but also list pages, form pages, and page containers should prioritize ProComponents (consistent with [Ant Design Pro](https://pro.ant.design) documentation).

**Recommended components**:
- **ProLayout**: Main layout (sidebar, top bar, breadcrumb, tabs, menu & routing).
- **PageContainer**: Page-level title, breadcrumb, action area.
- **ProTable**, **ProForm**, etc.: Standard admin list & form.

**Setup methods (choose one)**:
1. **Official scaffold**: Use Ant Design Pro **Umi / create-umi** flow per documentation; place output into `doc/ui/YYYY-MM-DD_<name>/web/`, maintain PRD and naming conventions.
2. **Manual integration**: In `web/`, install `antd` and `@ant-design/pro-components`; wrap routes with **ProLayout**; link menu & routing; use ProTable/ProForm for admin pages instead of bare Layout + Table.

**Theme**: Configure `antd` via ConfigProvider (or Umi theme settings) consistently with "Ant Design Color Scheme"; Pro components follow antd token changes.

---

## Ant Design Mobile (Mobile `mobile/`)

When requirement includes **mobile UI** (H5, WeChat public account pages, mini-programs), **MUST use Ant Design Mobile in separate independent `mobile/` project**, separate from PC `web/` (Ant Design Pro).

- **Documentation**: [Ant Design Mobile](https://ant-design-mobile.com)
- **Install**: In `mobile/` project run `npm install antd-mobile` or `pnpm add antd-mobile`
- **Usage**: All mobile pages in `doc/ui/YYYY-MM-DD_<name>/mobile/` using only antd-mobile components; PC uses only `web/` (antd + @ant-design/pro-components).
- **Common components**: TabBar, NavBar, Capsule, List, Form, Input, Button, Dialog, Toast, Picker, DatePicker, SwipeAction, PullToRefresh, InfiniteScroll, etc.; layout/navigation follow Ant Design Mobile spec.
- **Theme**: antd-mobile via `ConfigProvider` supports theme & primary color configuration; align with PC on brand primary color; document `web/` and `mobile/` library usage and theme conventions in root README.

---

## Options to Confirm Before Design

- **Mobile need?** (H5/WeChat, etc.) → Yes: use **Ant Design Mobile** in separate `mobile/` project (independent from `web/` PC); only PC: use `web/` only; mobile-only: use `mobile/` only.
- **Agent conversation/chat?** → Yes: integrate Ant Design X, create conversation/chat page.
- **Color/theme**: Select 1-2 schemes from "Ant Design Color Scheme" below (e.g., default blue, compact, dark, or 12-color palette primary), confirm, then set theme & ConfigProvider.
