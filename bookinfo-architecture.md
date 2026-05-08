# Bookinfo Architecture on EKS without Istio

Open this file in VS Code and use `Markdown: Open Preview` or `Cmd+Shift+V` to render the diagram.

```mermaid
graph LR
  Internet["Internet"] -->|HTTP/HTTPS| LB["LoadBalancer Service"]
  subgraph EKS_Cluster["AWS EKS Cluster"]
    LB --> SP["productpage Service (ClusterIP)"]
    SP --> DP["productpage Deployment"]
    DP --> PP["productpage Pod(s)"]

    PP -->|calls| SD["details Service"]
    SD --> DD["details Deployment"]
    DD --> PD["details Pod(s)"]

    PP -->|calls| SR["ratings Service"]
    SR --> DR["ratings Deployment"]
    DR --> PR["ratings Pod(s)"]

    PP -->|calls| SRev["reviews Service"]
    SRev --> DRev["reviews Deployment"]
    DRev --> PRev["reviews Pod(s)"]

    PRev -->|calls| SM["mongodb Service"]
    SM --> DM["mongodb Deployment"]
    DM --> PM["mongodb Pod(s)"]
  end
```

## If Mermaid does not render

- Install `Markdown Preview Mermaid Support` in VS Code.
- Open preview with `Cmd+Shift+V` or `Markdown: Open Preview`.
- Or use the plain text diagram below:

```
Internet
   |
   v
LoadBalancer Service
   |
   v
productpage Service (ClusterIP)
   |
   v
productpage Deployment
   |
   v
productpage Pod(s)
   |
   +--> details Service --> details Deployment --> details Pod(s)
   |
   +--> ratings Service --> ratings Deployment --> ratings Pod(s)
   |
   +--> reviews Service --> reviews Deployment --> reviews Pod(s)
                         |
                         v
                      mongodb Service
                         |
                         v
                     mongodb Deployment
                         |
                         v
                      mongodb Pod(s)
```
