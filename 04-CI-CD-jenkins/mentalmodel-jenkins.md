                    GitHub
                       │
                    Webhook
                       ↓
              Jenkins Controller
                       │
                Choose Job Type
                 /             \
                /               \
        Freestyle              Pipeline
            │                      │
     Jenkins UI              Jenkinsfile
            │                      │
     Build Steps             Stages / Steps
            │                      │
            └──────────┬───────────┘
                       ↓
                 Select Agent
                       ↓
              Agent executes work
                       ↓
              Build / Test / Deploy