# Hermes Agent × Gateway Pro 推荐许可架构图

## 结论

- 推荐采用三层许可架构：上游 MIT、适配层、专有企业版。
- 最稳的商业路径不是把整套产品统一做成 MIT，而是把上游组件、适配层和企业治理层拆开许可。
- 只要保留 Hermes 上游 MIT 声明与第三方 notice，企业治理平面和商业交付层可以继续保持专有。

## 推荐许可架构图

```mermaid
flowchart TB
    subgraph L1[第一层 上游 MIT 组件层]
        U1[Hermes Agent 上游原始代码]
        U2[基于 Hermes 的修改部分<br/>继续保留 MIT notice]
        U3[第三方依赖声明与 NOTICE]
    end

    subgraph L2[第二层 集成适配层]
        A1[Connector / Sidecar / API Adapter]
        A2[Policy Bridge / Event Mapper / Rollout Bridge]
        A3[许可建议<br/>MIT 或 Apache-2.0 或双许可证<br/>也可保持专有]
    end

    subgraph L3[第三层 专有企业版]
        E1[Gateway Pro Governance Plane]
        E2[企业控制面<br/>审批 审计 Trace Rollout SSO 策略包]
        E3[专有商业许可 / EULA]
    end

    U1 --> A1
    U2 --> A1
    U3 --> A1
    A1 --> E1
    A2 --> E1
    A3 --> E1
    E1 --> E2
    E2 --> E3

    N1[必须做的事<br/>保留 MIT 版权声明<br/>保留许可证文本<br/>输出第三方许可证清单]:::must
    N2[不必做的事<br/>不必整仓 MIT<br/>不必整仓开源<br/>不必把专有治理层改成社区版]:::note

    U1 -.分发义务.-> N1
    U2 -.分发义务.-> N1
    E3 -.商业边界.-> N2

    classDef must fill:#e8f5e9,stroke:#2e7d32,color:#1b5e20;
    classDef note fill:#fff8e1,stroke:#f9a825,color:#6d4c41;
```

## 三层怎么理解

### 第一层：上游 MIT 组件层

- 这一层包括 Hermes Agent 原始上游代码，以及你直接修改过并继续分发的 Hermes 部分。
- 这一层最关键的义务不是开源全部产品，而是保留 MIT 对应的版权和许可声明。
- 如果集成版里打包了 Hermes 源码、二进制或其修改版，都要把上游 attribution 留住。

### 第二层：集成适配层

- 这一层是你自己写的连接器、API 适配器、事件桥、审批桥、策略桥、rollout 桥。
- 这层最灵活，可以专有，也可以 MIT，也可以 Apache-2.0，也可以做双许可证。
- 如果你想做生态合作版，把这一层开放最合适；如果你想把核心控制权抓在自己手里，也可以闭源。

### 第三层：专有企业版

- 这一层是 Gateway Pro 本身的治理控制平面，以及企业专属的审批、审计、SSO、通知、策略包、交付资产。
- 这一层最适合维持专有商业许可。
- 它是你的商业护城河，不需要为了兼容 MIT 上游而被动开源。

## 推荐的对外许可策略

| 层 | 推荐策略 | 原因 |
| --- | --- | --- |
| 上游 MIT 组件层 | 保留 MIT 与 notice | 满足上游分发义务 |
| 集成适配层 | Apache-2.0 或 MIT 或双许可证 | 便于生态扩展，边界清晰 |
| 专有企业版 | 商业许可 / EULA | 保留商业控制权与交付壁垒 |

## 最稳的商业分发方式

### 方案 A：闭源企业版主线

- Hermes 作为第三方 MIT 组件被引用或打包。
- 适配层最小化开放，治理层全部专有。
- 对外附带第三方许可证清单与 NOTICE 文件。

这是最适合你当前 Gateway Pro 路线的主方案。

### 方案 B：社区连接器 + 企业治理版

- 把 connector、SDK、bridge 这一层开放出来。
- 用 MIT 或 Apache-2.0 吸引生态接入。
- Gateway Pro 治理平面和企业交付层继续闭源。

这是更适合做生态扩张的方案。

## 你至少要补的三个法务交付件

1. Gateway Pro 自身的明确许可文本或 EULA。
2. Hermes 与其他第三方依赖的许可证清单。
3. 集成版分发包中的 NOTICE / attribution 文件。

## 最终建议

- 许可架构上，不建议把整套产品简单做成单一 MIT。
- 更建议采用“三层分治”：上游 MIT、适配层灵活、企业版专有。
- 这样既能合法纳入 Hermes，又不会把你的企业治理资产白白让渡成纯社区公共品。
