flowchart LR
  subgraph Dev["CI/CD"]
    GH[GitHub Actions\n(plan/apply)]
  end

  subgraph AWS["AWS (IaC)"]
    TF[(Terraform)]
    S3[(S3 Bucket)]
    SQS[[SQS Queue\n(batch & debounce)]]
    LBD[[Lambda\nPython + boto3]]
    SNS[(SNS Topic)]
    EMAIL[(Email Subscription)]
    CW[(CloudWatch Logs)]
    IAM[(IAM Roles & Policies)]
    TState[(Terraform State\n(S3+Locking))]
  end

  GH -->|terraform plan/apply| TF
  TF -->|provisions| S3
  TF --> SQS
  TF --> LBD
  TF --> SNS
  TF --> IAM
  TF --> TState

  S3 -- "ObjectCreated events" --> SQS
  SQS -- "messages (batched)" --> LBD
  LBD -- "publish summary\n(one email per batch)" --> SNS
  SNS --> EMAIL
  LBD -- "logs/metrics" --> CW

  classDef aws fill:#0b1,stroke:#0b1,color:#fff,stroke-width:0;
  classDef comp fill:#111,stroke:#333,color:#eee,stroke-width:1,rx:6,ry:6;
  class AWS,Dev comp
  class TF,S3,SQS,LBD,SNS,EMAIL,CW,IAM,TState aws
