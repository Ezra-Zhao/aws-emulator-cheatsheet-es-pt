# aws-emulator 速查表 | aws-emulator Cheatsheet | Guía rápida de aws-emulator

> 📚 **Colección / Coleção**: esta hoja forma parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 hojas de referencia en un solo repositorio. / Esta folha faz parte de [cheatsheets](https://github.com/Ezra-Zhao/cheatsheets) — 100 folhas de referência em um só repositório. ⭐

**测 AWS 不用先开账号、先刷卡——但保真度逐个服务不同，先看第一屏的分工卡，再查分数再信结果**

> 一页看懂 AWS 官方出的本地模拟器 aws-emulator：起一个本地 HTTPS 端点，把 AWS CLI 或 SDK 指过去，就能在没有 AWS 账号、不产生云费用的情况下测自己的软件。它卖的不是「又一个模拟器」，是官方保真度分数和从 LocalStack 搬家的时点。这张卡把搬家选路、保真度核查、三条红线和遥测关闭收在一页。

- 上游项目：[aws/aws-emulator](https://github.com/aws/aws-emulator)（2026-11-13 实测 6,668 ★ / 253 forks；18 个 release，31,104 个 commit，根目录只有英文 README；许可证 CC0-1.0，`vendor/` 目录为 Apache-2.0）
- 平台：本地端点 `https://127.0.0.1:9297`（自签证书）；服务器模式（Linux 原生包 / Windows 与 macOS 走 Docker）、嵌入模式（Rust 测试进程内）
- 本页结构：中文主文 → English → Español → Português
- 同系列：社区轻量模拟器看 [floci 速查表](https://github.com/Ezra-Zhao/floci-cheatsheet-es-pt)；这张是 AWS 官方出品，两张组成新旧双联

---

Apoyo y más: [GitHub Sponsors](https://github.com/sponsors/Ezra-Zhao) · [Gumroad — plantillas y cuadernos que uso](https://ezraflow28.gumroad.com) · [Ko-fi](https://ko-fi.com/ezra777zhao)

## 第一屏：与 floci 的分工卡

| 问 | floci（社区） | aws-emulator（官方） |
|---|---|---|
| 出身 | 社区做的轻量本地 AWS 模拟器 | AWS 官方出品，服务模型直接来自公开的 AWS API 模型 |
| 卖点 | 轻、快、上手简单 | 官方保真度：每次 release 附各服务的保真度分数，像不像真 AWS 有数可查 |
| 什么时候选它 | 只测少数常用服务、要轻量 | 要测的服务多、想按官方口径核对行为差异，或正从 LocalStack 搬家 |

**从 LocalStack 搬家的时点**：LocalStack 自 2026 年 3 月起要求账号与 token 才能用（西/葡语发布文口径，证据 B），免费裸跑的时代结束——这正是官方模拟器在 2026-11-11 发布后两天冲到 6.7K 星的背景。搬家前先用下面保真度卡核对你依赖的服务，再切 CI。

覆盖口径：据发布报道，它覆盖 200 多个服务、由 438 个公开 API 模型建成（报道口径，证据 B）；具体每个服务像到什么程度，以每次 release 附的保真度分数为准，别拿总数当承诺。

## 三条起步路，先选哪条

| 路 | 装法 | 适合谁 |
|---|---|---|
| 让 agent 装 | 装好 AWS CLI 2.35.0 以上，跑 `aws agent-toolkit add-skill --skill-name aws-emulator --region us-east-1` 装上它的 skill，再让你的编程 agent 把模拟器配好 | 已经在用编程 agent 干活的人，最省手 |
| 服务器模式 | Linux 装 RPM 或 Debian 包后跑 `aws-emulator`；Windows 与 macOS 按官方指南走 Docker。然后设环境变量把 AWS CLI / SDK 指到本地端点 | 团队共用一台开发机或 CI 里长期跑 |
| 嵌入模式 | 在 Rust 测试里把它加为 dev-dependency，进程内跑，每个测试一个隔离实例；没有端点地址、没有证书、没有环境变量，测试把模拟器的 HTTP 客户端直接接进 SDK | 写 Rust 的人，要测试间互不干扰 |

## 保真度怎么查（这张的独门价值）

- **看分数**：每个服务与真 AWS 有多像，看每次 release 附的保真度分数（在原仓 releases 页），你依赖的服务分数低，就别把本地通过当成上线通过。
- **未实现不编答案**：还没实现的操作返回 501，不假装成功——测出 501 是信息，不是故障，先分清是你的代码错还是模拟器没实现。
- **查缺口登记**：已知差距登记在原仓 `issues/` 目录里，分两处：`things-that-do-not-work`（还不行）和 `things-that-now-work`（已经修好）；你的项目能不能跑，先去这两处对一遍，再看官方的项目兼容性检查指南。

## 三条红线卡

| 红线 | 说明（原仓自述） |
|---|---|
| 只作构建与测试 | 不跑生产负载，也不用它测安全；它是开发与 CI 的工具 |
| 不评估 IAM 策略 | 它不像真 AWS 那样评估 IAM 策略：本地放行的请求，真上线可能被拒。权限相关的结论一律回真环境复核 |
| 遥测默认开着 | 它默认向 AWS 回传使用数据（原仓自述：做过数据最小化，不含请求与响应内容、代码、凭据与个人数据，不与第三方分享）。不想传就关，关法在下一卡、四选一 |

## 遥测关闭四法（选一个就行）

```bash
# 1. 启动参数（只管这一次进程；容器在镜像名后加同样的参数）
aws-emulator --telemetry off

# 2. 环境变量（管这个 shell 里启动的所有实例）
export EMULATOR_TELEMETRY=off
```

```toml
# 3. 配置文件（管这台机器的每一次运行）
# Linux/macOS: ~/.aws/emulator/config.toml ; Windows: %USERPROFILE%\.aws\emulator\config.toml
[telemetry]
enabled = false
```

```rust
// 4. 嵌入模式 API（在创建模拟器时传选项）
let emulator = AwsEmulatorEmbedded::new(
    EmbeddedOptions::default().telemetry(Telemetry::Off),
);
```

## 成本卡

- **模拟器本身没有服务费**：请求都打在本地，不产生 AWS 费用；只有真部署到 AWS 时才按资源付费，账号都不是必需的。
- **别自己编源码**：从源码构建要在 `target/` 放至少 100 GB 磁盘，还要 Rust 工具链与 C 编译器，且源码构建只支持 Linux；普通用户走预编译包。嵌入模式会把模拟器编进你的测试二进制，第一次构建的成本与源码构建同量级，后面增量构建才快——CI 里记得缓存 `target/` 并把版本锁到某个 release 的 commit。

## 诚实标注区

- **仓不收 PR**：原仓明说不接收 PR，只能报 issue；报问题要写清服务名、操作、SDK 与版本、你发的请求、以及响应和真 AWS 差在哪，写全了才有人处理。
- **许可证分两层**：主体 CC0-1.0，近乎公共领域；`vendor/` 目录是 Apache-2.0，引用时分清（2026-11-13 原仓实测）。
- **报道口径标证据 B**：覆盖 200 多个服务、438 个 API 模型、LocalStack 2026-03 起要求账号 token，这三条来自发布报道与西/葡语发布文，不是原仓 README 的自述；原仓自述只有「看 release 的保真度分数」这一条核查法。
- **势头数据**：6,668 ★ / 253 forks、18 个 release（2026-11-13 原仓页实测，证据 A）；今日第三方榜位第 1 为第三方统计口径（证据 B）。2026-11-11 才发布，两天站上 6.7K，分数体系随 release 迭代，引用分数时注明版本。

---

## English (summary)

aws-emulator (6,668 stars / 253 forks as of 2026-11-13, 18 releases, 31,104 commits; CC0-1.0 with `vendor/` under Apache-2.0) is AWS's official local emulator of the AWS APIs: it serves a local HTTPS endpoint at `https://127.0.0.1:9297` (self-signed certificate), you point the AWS CLI or an SDK at it through environment variables, and you test your software with no AWS account and no cloud charges — the emulator itself has no service fee; you pay only for real AWS resources when you deploy. Versus the community floci emulator covered in the companion cheatsheet, the official one's selling points are its published fidelity scores and the migration moment: LocalStack has required an account and token since March 2026 (press/community reports, evidence grade B), and launch coverage credits the emulator with 200+ services built from 438 public API models (also evidence B — check the per-service fidelity scores shipped with each release instead of trusting the headline number). Three ways in: let an agent install it (AWS CLI 2.35.0+, then `aws agent-toolkit add-skill --skill-name aws-emulator --region us-east-1`), server mode (RPM or Debian package on Linux, Docker on Windows and macOS), or embedded mode (a Rust dev-dependency running in-process, one isolated instance per test, no endpoint, certificate or environment variables). Fidelity checking is the point of this page: read the fidelity scores attached to each release, treat a 501 as information (unimplemented operations return 501 rather than faking success), and check the gap registries in the repo's `issues/` directory — `things-that-do-not-work` and `things-that-now-work`. Three red lines from upstream: build-and-test only, never production or security testing; it does not evaluate IAM policies the way real AWS does, so a request allowed locally can still be denied after deployment; telemetry is on by default (upstream describes data minimization: no request/response content, code, credentials or personal data, and no sharing with third parties) and can be turned off four ways — the `--telemetry off` flag, the `EMULATOR_TELEMETRY=off` environment variable, `[telemetry] enabled = false` in `~/.aws/emulator/config.toml`, or the embedded options API. Cost warning: building from source needs at least 100 GB of disk for `target/` and is Linux-only — use the prebuilt packages. The repository does not accept PRs; fidelity gaps are reported as issues with the service, operation, SDK version, the request you sent, and how the response differed from real AWS.

---

## Español (guía rápida — traducción de estudio, pendiente de revisión por un hablante nativo)

**Para probar AWS ya no hace falta abrir cuenta ni pasar la tarjeta. Pero la fidelidad varía por servicio: mira primero el reparto con floci y comprueba la puntuación antes de fiarte del resultado.**

aws-emulator (6.668 estrellas / 253 forks a 13 de noviembre de 2026, 18 releases, 31.104 commits; licencia CC0-1.0 y el directorio `vendor/` bajo Apache-2.0) es el emulador local oficial de las API de AWS: levanta un endpoint HTTPS local en `https://127.0.0.1:9297` (certificado autofirmado), apuntas la AWS CLI o un SDK hacia él con variables de entorno y pruebas tu software sin cuenta de AWS y sin cargos en la nube; el emulador en sí no tiene tarifa de servicio, solo pagas recursos reales cuando despliegas en AWS. Frente a floci, el emulador comunitario de la hoja compañera de esta serie, lo oficial vende dos cosas: las puntuaciones de fidelidad publicadas con cada release y el momento de migración — LocalStack exige cuenta y token desde marzo de 2026 (prensa y artículos de lanzamiento, evidencia B), y la cobertura del lanzamiento le atribuye más de 200 servicios construidos desde 438 modelos públicos de API (también evidencia B: no te fíes del número grande, comprueba la puntuación de cada servicio que uses).

Tres rutas de entrada: dejar que lo instale un agente (AWS CLI 2.35.0 o superior y `aws agent-toolkit add-skill --skill-name aws-emulator --region us-east-1`), el modo servidor (paquete RPM o Debian en Linux; Docker en Windows y macOS) o el modo embebido (dependencia de desarrollo en tests de Rust, en el mismo proceso, una instancia aislada por test, sin endpoint, sin certificado y sin variables de entorno). La fidelidad se comprueba así: mira las puntuaciones que acompañan a cada release, trata un 501 como información (las operaciones no implementadas devuelven 501 en lugar de fingir éxito) y revisa los registros de carencias en el directorio `issues/` del repositorio: `things-that-do-not-work` y `things-that-now-work`. Tres líneas rojas del propio proyecto: solo para construir y probar, nada de producción ni de pruebas de seguridad; no evalúa políticas IAM como el AWS real, así que una petición permitida en local puede ser denegada al desplegar; y la telemetría viene activada por defecto (el proyecto describe minimización de datos: sin contenido de peticiones ni respuestas, sin código, sin credenciales ni datos personales, y sin compartir con terceros), con cuatro formas de apagarla: el parámetro `--telemetry off`, la variable `EMULATOR_TELEMETRY=off`, `[telemetry] enabled = false` en `~/.aws/emulator/config.toml`, o la API de opciones del modo embebido. Costos: compilar desde el código fuente exige al menos 100 GB de disco para `target/` y solo está soportado en Linux — usa los paquetes precompilados. El repositorio no acepta PR: las carencias de fidelidad se reportan como issues indicando servicio, operación, SDK y versión, la petición enviada y en qué difirió la respuesta del AWS real. Esta página es una síntesis con mis propias palabras a partir de la documentación pública del proyecto, no una traducción de su README.

---

## Português (guia rápido — tradução de estudo, aguardando revisão de um falante nativo)

**Para testar a AWS não precisa mais abrir conta nem passar cartão. Mas a fidelidade varia por serviço: olhe primeiro a divisão com o floci e confira a pontuação antes de confiar no resultado.**

O aws-emulator (6.668 estrelas / 253 forks em 13 de novembro de 2026, 18 releases, 31.104 commits; licença CC0-1.0 e o diretório `vendor/` sob Apache-2.0) é o emulador local oficial das APIs da AWS: ele sobe um endpoint HTTPS local em `https://127.0.0.1:9297` (certificado autoassinado), você aponta a AWS CLI ou um SDK para ele com variáveis de ambiente e testa o seu software sem conta da AWS e sem cobranças na nuvem; o emulador em si não tem tarifa de serviço, você só paga recursos reais quando implanta na AWS. Diante do floci, o emulador comunitário da folha companheira desta série, o oficial vende duas coisas: as pontuações de fidelidade publicadas a cada release e o momento da migração — o LocalStack exige conta e token desde março de 2026 (imprensa e artigos de lançamento, evidência B), e a cobertura do lançamento atribui ao emulador mais de 200 serviços construídos a partir de 438 modelos públicos de API (também evidência B: não confie no número grande, confira a pontuação de cada serviço que você usa).

Três rotas de entrada: deixar um agente instalar (AWS CLI 2.35.0 ou superior e `aws agent-toolkit add-skill --skill-name aws-emulator --region us-east-1`), o modo servidor (pacote RPM ou Debian no Linux; Docker no Windows e no macOS) ou o modo embutido (dependência de desenvolvimento em testes Rust, no mesmo processo, uma instância isolada por teste, sem endpoint, sem certificado e sem variáveis de ambiente). A fidelidade se confere assim: olhe as pontuações que acompanham cada release, trate um 501 como informação (operações não implementadas devolvem 501 em vez de fingir sucesso) e revise os registros de lacunas no diretório `issues/` do repositório: `things-that-do-not-work` e `things-that-now-work`. Três linhas vermelhas do próprio projeto: só para construir e testar, nada de produção nem de testes de segurança; ele não avalia políticas IAM como a AWS real, então um pedido permitido localmente pode ser negado na implantação; e a telemetria vem ligada por padrão (o projeto descreve minimização de dados: sem conteúdo de pedidos nem respostas, sem código, sem credenciais nem dados pessoais, e sem compartilhar com terceiros), com quatro jeitos de desligar: o parâmetro `--telemetry off`, a variável `EMULATOR_TELEMETRY=off`, `[telemetry] enabled = false` em `~/.aws/emulator/config.toml`, ou a API de opções do modo embutido. Custos: compilar do código-fonte exige pelo menos 100 GB de disco para `target/` e só é suportado no Linux — use os pacotes pré-compilados. O repositório não aceita PR: as lacunas de fidelidade são reportadas como issues indicando serviço, operação, SDK e versão, o pedido enviado e em que a resposta diferiu da AWS real. Esta página é uma síntese com as minhas próprias palavras a partir da documentação pública do projeto, não uma tradução do README dele.

---

## 来源与声明

- 灵感来源 / Source / Fuente / Fonte: [aws/aws-emulator](https://github.com/aws/aws-emulator)（2026-11-13 实测 6,668 ★ / 253 forks，18 个 release，31,104 个 commit；许可证 CC0-1.0，`vendor/` 目录为 Apache-2.0；官方用户指南 docs.aws.amazon.com/emulator）
- 本仓是个人学习整理：内容用我自己的话总结自上游公开文档，不搬运原文，不冒充官方；装法、红线与遥测关法以 2026-11-13 的上游 README 为准，星数以同日 GitHub 实测为准；覆盖服务数与 LocalStack 口径为发布报道口径（证据 B），上游迭代后请以官方文档为最终依据。
- 西语、葡语部分为学习译稿，待母语校对。Las secciones en español y portugués son traducciones de estudio pendientes de revisión nativa.
- License: MIT © 2026 Guangyi Zhao（本仓自有内容；上游项目主体为 CC0-1.0、`vendor/` 为 Apache-2.0）
