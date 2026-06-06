# Adapter Package Structure Convention

## Package Structure Rules

- **config subdirectory (必须)**: Contains default built-in BaseController, GlobalExceptionHandler, TokenValidationFilter (framework code + TODO, compilable).
- **Domain-based subdirectories**: Rest organized by business domain, matching application/domain structure. Each domain subdirectory contains **controller/**; if domain listens to events (MQ, Spring events, domain events), add **listener/** subdirectory.
- **Application startup class** (`@SpringBootApplication`): Placed in **adapter root**, parallel to config and domain subdirectories.

**Example directory tree:**
```
adapter/
├── <ApplicationName>Application.java ← Startup class in adapter root
├── config/                           ← Cross-domain/global
│   ├── BaseController.java
│   ├── GlobalExceptionHandler.java
│   ├── TokenValidationFilter.java
│   └── ...
├── conversation/                     ← By domain
│   └── controller/
│       ├── ConversationCommandController.java
│       └── ConversationQueryController.java
├── message/                          ← Add listener/ when domain listens to events
│   ├── controller/
│   │   ├── MessageCommandController.java
│   │   └── MessageQueryController.java
│   └── listener/
│       └── MessageEventListener.java
├── auth/
│   └── controller/
│       └── AuthController.java        ← Single entry controller
├── agent/
│   └── controller/
│       └── AgentController.java
└── resource/
    └── controller/
        └── EmailCodeController.java
```
