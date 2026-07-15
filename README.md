# 无人机线路巡检本体系统演示项目

## 1. 项目定位

本项目建设一个可一键运行、可逐步讲解的本体系统演示，以“无人机电力线路热缺陷巡检”为场景，展示本体如何完成任务理解、知识抽取、数据校验、知识推理、图谱查询和结论解释。

核心原则：**AI负责模拟输入和业务角色，开源组件负责真实处理，推理结果不得由AI伪造。**

## 2. 要演示的业务故事

1. AI模拟指挥员下发“巡检L-203线路，需要热成像能力”的任务。
2. AI模拟装备台账、载荷说明、天气信息和实时感知事件。
3. 系统抽取无人机、载荷、任务、能力、线路、杆塔和异常事件。
4. AI模拟专家审核候选知识，但必须通过审核接口操作。
5. SHACL真实检查缺失时间、错误类型和不合法关系。
6. 推理引擎真实推出候选无人机、装备能力和异常等级。
7. AI模拟业务用户提问，系统返回结论、依据和来源。

## 3. 第一版范围

第一版必须实现：

- 一个经过验证的巡检领域本体；
- 一个AI全流程演示场景；
- AI生成任务、台账、事件、审核意见和查询问题；
- 文档解析、候选知识抽取和RDF映射；
- 候选知识审核、SHACL校验和知识入库；
- 基础OWL/RDFS推理和业务规则推理；
- 图谱查询、推理解释和证据追踪；
- 演示控制台的自动、单步、暂停、重置能力；
- Docker Compose一键启动。

第一版暂不实现：

- 真实无人机和飞控接入；
- 完整航线规划、飞行控制和全局优化编配；
- 生产级多租户、复杂权限和高可用部署；
- 大规模图谱和复杂多Agent协作；
- 无人工边界的本体自动发布。

## 4. 总体架构

```text
AI场景编排器
  ├─ 模拟指挥员、数据源、专家和业务用户
  └─ 按场景脚本调用统一输入API
                     ↓
Tika/OCR解析 → AI抽取 → RDF映射 → 候选知识审核
                     ↓
             pySHACL真实校验
                     ↓
         Fuseki/TDB2正式知识图谱
                     ↓
        Jena/HermiT真实推理与规则计算
                     ↓
       FastAPI知识服务 → React演示控制台
```

## 5. 技术选型

| 能力 | 第一版选型 | 使用方式 |
| --- | --- | --- |
| 本体建模 | Protégé Desktop | 人工查看和维护OWL本体 |
| 文档解析 | Apache Tika | 解析PDF、Word、Excel和元数据 |
| OCR | PaddleOCR，可选 | 仅处理扫描件和图片文字 |
| RDF处理 | RDFLib | JSON/RDF转换、IRI和三元组生成 |
| 数据校验 | pySHACL | 执行SHACL约束并输出报告 |
| 图谱存储 | Apache Jena Fuseki/TDB2 | 存储命名图并提供SPARQL |
| 基础推理 | Jena推理；HermiT辅助 | 运行期推理与建模期一致性检查 |
| 后端服务 | FastAPI | 封装业务API，隔离原始SPARQL |
| 图谱展示 | React + Cytoscape.js | 展示实体关系和推理路径 |
| 环境编排 | Docker Compose | 一键启动各组件 |

## 6. 目录规划

```text
uav-inspection-ontology-demo/
├─ README.md                         # 项目范围、架构和目录总览
├─ compose.yaml                      # frontend/backend/fuseki/tika编排
├─ .env.example                      # 模型、端口和数据集配置示例
├─ docs/
│  ├─ architecture.md                # 详细架构、数据流和接口边界
│  ├─ demo-script.md                 # 五分钟汇报演示脚本
│  └─ ontology-guide.md              # 本体概念、关系和建模约定
├─ scenarios/
│  ├─ inspection-001.yaml            # 默认AI演示场景及预期结果
│  └─ fixtures/                      # AI不可用时的确定性模拟输入
├─ ontology/
│  ├─ inspection.ttl                 # Protégé维护的正式本体
│  ├─ shapes.ttl                     # SHACL数据约束
│  ├─ rules/inspection.rules         # 业务推理规则
│  ├─ examples/                      # 正确与错误样例数据
│  └─ tests/expected-inferences.ttl  # 推理回归预期结果
├─ simulator/
│  ├─ orchestrator.py                # 自动/单步场景编排
│  ├─ scenario_loader.py             # 场景脚本加载与校验
│  └─ roles/                         # 指挥员、数据源、专家、用户模拟器
├─ backend/
│  ├─ app/main.py                    # FastAPI入口
│  ├─ app/api/                       # 演示、知识、审核和查询接口
│  ├─ app/services/                  # 解析、抽取、校验、推理、证据服务
│  ├─ app/adapters/                  # Fuseki、Tika和AI模型适配器
│  ├─ app/models/                    # 输入输出数据契约
│  └─ tests/                         # 单元测试和端到端测试
├─ frontend/
│  └─ src/                           # 演示控制台、审核页和图谱视图
├─ fuseki/
│  ├─ config.ttl                     # 数据集、端点和推理配置
│  └─ data/                          # TDB2持久化目录
├─ tika/                             # Tika服务配置
├─ data/                             # 上传文件和演示产物，默认不入库
└─ scripts/                          # 初始化、发布、重置和回归检查脚本
```

## 7. 核心数据分区

Fuseki使用命名图隔离不同状态的数据：

| 命名图 | 内容 |
| --- | --- |
| `urn:graph:ontology` | 已发布本体 |
| `urn:graph:shapes` | SHACL约束 |
| `urn:graph:candidate` | AI抽取但尚未审核的候选知识 |
| `urn:graph:asserted` | 审核通过的原始事实 |
| `urn:graph:inferred` | 推理产生的类型、关系和结论 |
| `urn:graph:provenance` | 来源、证据、置信度、审核和规则路径 |

## 8. AI模拟要求

- 所有模拟输入必须标记`sourceType=AI_SIMULATION`和`scenarioId`；
- 使用固定场景骨架和随机种子，保证演示结果可重放；
- AI只能提交候选知识，不能直接写入正式事实图和推理图；
- 专家模拟必须调用接受、修改、驳回和合并接口；
- AI可以生成自然语言变体，但核心实体、规则和预期结论固定；
- AI不可用时自动使用`scenarios/fixtures`中的离线模拟数据；
- 校验、入库、推理和查询必须由真实组件完成。

## 9. 第一版核心接口

```text
POST /api/demo/scenarios/{id}/start   启动AI演示
POST /api/demo/next                   执行下一步
POST /api/demo/reset                  重置图谱和演示状态
GET  /api/demo/events                 查询全过程事件
POST /api/documents/parse             解析模拟文档
POST /api/knowledge/extract           生成候选知识
GET  /api/candidates                  查询候选知识
POST /api/candidates/{id}/decision    提交模拟专家审核
POST /api/knowledge/validate          执行SHACL校验
POST /api/reasoning/run               执行真实推理
GET  /api/entities/{id}               查询实体及关系
GET  /api/reasoning/{id}/explanation 查询结论依据
```

## 10. 自研边界

重点自研领域本体、AI场景编排、实体关系抽取、本体映射、审核流程、业务规则、证据链、推理解释和知识服务API；编辑器、解析器、RDF处理、SHACL校验、基础推理、图谱存储和可视化框架优先复用开源组件。

## 11. 验收标准

- 环境启动后，无需真实业务数据即可一键完成全流程演示；
- 控制台能显示每一步“谁输入、输入什么、哪个组件处理、输出什么”；
- 至少演示一次错误数据被SHACL发现并修正；
- 至少推出“候选无人机”和“高风险热异常”两个新结论；
- 每个结论可查看原始事实、触发规则、本体版本和来源证据；
- 演示可重复执行，重置后结果一致；
- AI关闭时仍可用离线模拟数据完成相同闭环。
