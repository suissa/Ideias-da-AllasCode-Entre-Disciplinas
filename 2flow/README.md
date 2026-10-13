# 2flow — Notação de Fluxos Causais e Execução Semântica da AllasCode

A **2flow** é a notação declarativa da AllasCode para descrever a topologia causal de um processo e a fiação de execução dentro de cada nó. Ela busca manter o significado do fluxo independente do mecanismo físico que o executa: chamadas locais, Actors, filas e eventos distribuídos, RPC/QUIC, NATS ou módulos WebAssembly podem ser projeções diferentes de uma mesma definição semântica, desde que o backend ofereça as capacidades necessárias e preserve os contratos.

Este documento também explica a tese arquitetural de composição da AllasCode: **2flow + schema de propriedades semanticamente tipadas + configuração** podem descrever a maior parte da estrutura e do comportamento específico de um sistema. Essa descrição torna-se executável por meio de um compilador e de runtimes/adaptadores compatíveis. A fórmula é um modelo de definição, não a afirmação de que três artefatos, sem infraestrutura ou implementações, materializem literalmente qualquer tecnologia imaginável.

## Sumário

- [1. O que é 2flow](#1-o-que-é-2flow)
- [2. Dois níveis complementares](#2-dois-níveis-complementares)
- [3. Os oito operadores](#3-os-oito-operadores)
- [4. Semântica topológica](#4-semântica-topológica)
- [5. Semântica de execução](#5-semântica-de-execução)
- [6. Resultados tipados e fronteiras de agentes](#6-resultados-tipados-e-fronteiras-de-agentes)
- [7. Schema: propriedades semanticamente tipadas](#7-schema-propriedades-semanticamente-tipadas)
- [8. Configuração: política sem alterar a semântica](#8-configuração-política-sem-alterar-a-semântica)
- [9. A composição: fluxo + schema + configuração](#9-a-composição-fluxo--schema--configuração)
- [10. Da DSL à execução: AST, IR e compilação](#10-da-dsl-à-execução-ast-ir-e-compilação)
- [11. Como a mesma definição projeta arquiteturas diferentes](#11-como-a-mesma-definição-projeta-arquiteturas-diferentes)
- [12. Por que isso pode reduzir código, tokens e alucinações](#12-por-que-isso-pode-reduzir-código-tokens-e-alucinações)
- [13. Concorrência, falhas, Saga e intervenção humana](#13-concorrência-falhas-saga-e-intervenção-humana)
- [14. Segurança, 2FV e limites das garantias](#14-segurança-2fv-e-limites-das-garantias)
- [15. Exemplo integrado](#15-exemplo-integrado)
- [16. O que significa “criar qualquer sistema com qualquer arquitetura”](#16-o-que-significa-criar-qualquer-sistema-com-qualquer-arquitetura)
- [17. Invariantes canônicos](#17-invariantes-canônicos)
- [18. Validação e critérios de prova](#18-validação-e-critérios-de-prova)
- [Conclusão](#conclusão)

## 1. O que é 2flow

A 2flow é uma notação de fluxo causal e de execução. Em vez de esconder a intenção do processo dentro de callbacks, condicionais, filas, mutexes e tratamento de exceções espalhados pelo código, ela torna o grafo de negócio explícito e descreve, separadamente, como cada nó se conecta às suas entradas, comportamentos próprios e resultados.

Seus objetivos principais são:

- **Legibilidade semântica:** tornar visíveis ordem causal, paralelismo, falhas, compensações e pontos de autorização humana.
- **Contratos verificáveis:** conectar eventos, funções e resultados por tipos explícitos.
- **Separação de responsabilidades:** impedir que um agente invoque diretamente comportamentos que pertencem a outro agente.
- **Projeção para diferentes runtimes:** separar o que o sistema deve fazer de onde e como será executado.
- **Compilação antecipada:** detectar incompatibilidades antes da execução sempre que a informação necessária estiver disponível estaticamente.

A expressão “zero-cost abstraction” é uma meta de implementação para as projeções em que a compilação consegue reduzir a DSL a chamadas nativas. Não é uma propriedade automática de toda projeção: rede, filas distribuídas, serialização e persistência têm custos físicos inevitáveis.

## 2. Dois níveis complementares

### 2.1. 2flow Topológica

Descreve **o que acontece, em que ordem, quais etapas são independentes, como resultados habilitam os próximos nós e para onde segue uma falha**. É a visão de processo e de causalidade.

```2flow
flow RegisterPurchase

ProcurementAgent.RegisterPurchase
  :--: StockAgent.IncreaseStock
  :--: FinancialAgent.RegisterFinancialEntry
```

### 2.2. 2flow de Execução

Descreve **como cada nó recebe um evento, invoca um comportamento que possui e emite um resultado tipado**.

```2flow
execution StockAgent.IncreaseStock
  -> Ok<PurchaseRegistered>
  ->> IncreaseStock
  <<- IncreaseStock

  <- Ok<StockEntryCommitted>
  <- Error<StockEntryError>
```

Os dois níveis são complementares: o grafo topológico não deve precisar mudar apenas porque uma chamada local foi movida para um Actor remoto ou para um worker distribuído. A equivalência depende de preservar os contratos, a ordenação causal exigida e os efeitos observáveis.

## 3. Os oito operadores

| Camada | Operador | Significado |
|---|---|---|
| Topológica | `:--:` | Sequência causal condicionada ao sucesso |
| Topológica | `[A, B]` | Ramos paralelos e barreira de junção |
| Topológica | `!->` | Rota de falha, fallback ou compensação declarada |
| Topológica | `[? Gate]` | Gate de intervenção humana (*HITL*) |
| Execução | `->` | Evento ou resultado entra no escopo |
| Execução | `<-` | Resultado/evento sai do escopo |
| Execução | `->>` | Escopo proprietário invoca comportamento próprio |
| Execução | `<<-` | Comportamento recebe a invocação do proprietário |

Os operadores topológicos descrevem dependências do grafo; os operadores de execução descrevem fronteiras e invocações dentro de um nó. O significado exato de cada operador precisa ser implementado e testado pelo parser, validador e runtime, não apenas documentado em texto.

## 4. Semântica topológica

### 4.1. Sequência causal — `:--:`

`A :--: B` declara que a execução de `B` depende de um resultado de sucesso de `A` que satisfaça o contrato de entrada de `B`.

```2flow
ValidatePurchase
  :--: RegisterStock
  :--: RegisterFinancialEntry
```

Um `Error<E>` não atravessa a aresta normal como se fosse um sucesso. Deve ser propagado, tratado ou direcionado por uma política de falha explícita.

### 4.2. Fork-join — `[A, B]`

```2flow
ReceivePurchase
  :--: [RegisterFinancialEntry, RegisterAccountingEntry]
  :--: ContinueLifecycle
```

Os ramos podem ser executados concorrentemente se suas dependências e efeitos permitirem. O nó seguinte só é liberado quando a condição de junção declarada for satisfeita. O contrato deve definir o que acontece se um ramo falhar, expirar ou permanecer indeterminado; “todos concluídos” não significa necessariamente “todos tiveram sucesso”.

### 4.3. Falha e compensação — `!->`

```2flow
SalesAgent.ReserveStock !-> StockAgent.ReleaseReservation
```

`!->` é uma aresta do grafo, não um terceiro tipo de retorno. O nó pode produzir `Ok<T>` ou `Error<E>`; o grafo decide como tratar a falha. Em uma Saga, compensação não é rollback mágico nem transação ACID global: é uma ação de domínio explícita, que também pode falhar e precisa de idempotência, rastreamento e política de recuperação.

A semântica de sequência após uma rota de falha deve ser definida sem ambiguidade pelo formato canônico: por exemplo, se a compensação é executada somente na falha, se o fluxo normal continua após a compensação e qual resultado final é reportado.

### 4.4. Gate humano — `[? Gate]`

```2flow
PreparePayment
  :--: [? ComplianceDirectorApproval]
  :--: ExecutePayment
```

O gate suspende o progresso lógico até que uma evidência externa válida seja recebida. Um runtime pode persistir o estado e liberar recursos de CPU enquanto aguarda. A aprovação deve ser autenticada, autorizada, vinculada à operação e protegida contra replay; a mera presença de um texto “aprovado” não é prova suficiente.

## 5. Semântica de execução

### 5.1. Ingress — `->`

```2flow
execution StockAgent.IncreaseStock
  -> Ok<PurchaseRegistered>
```

O evento entra no escopo atual. A direção é relativa ao escopo descrito.

### 5.2. Egress — `<-`

```2flow
  <- Ok<StockEntryCommitted>
  <- Error<StockEntryError>
```

O escopo emite um resultado tipado para a fronteira de saída definida pelo contrato.

### 5.3. Invocar comportamento próprio — `->>`

```2flow
  ->> IncreaseStock
```

O agente ou escopo proprietário invoca uma função, Action ou AtomicBehavior pertencente ao próprio contexto.

### 5.4. Receber invocação — `<<-`

```2flow
  <<- IncreaseStock
```

Declara o comportamento que recebe a invocação do proprietário. O compilador deve verificar se a identidade e a assinatura correspondem ao comportamento declarado.

## 6. Resultados tipados e fronteiras de agentes

A lei de resultado proposta é:

[
\operatorname{execute}:Input\rightarrow Ok<Output>\mid Error<Failure>
]

Os resultados de domínio são explícitos. Valores sentinela, `null`, `undefined` ou exceções não tratadas não devem funcionar como caminhos implícitos de controle. Exceções internas de uma linguagem podem existir, mas a fronteira executável deve convertê-las em erros tipados segundo um contrato conhecido.

Um agente só invoca comportamentos que possui. A comunicação entre agentes ocorre por eventos ou contratos de fronteira, não por chamadas diretas a funções internas de outro agente.

```text
Agente A
  -> evento público tipado
       -> fronteira do Agente B
            -> comportamento próprio de B
```

A propriedade é de domínio e de arquitetura; um compilador só consegue garanti-la se a definição de ownership, os símbolos e as dependências estiverem disponíveis para validação.

## 7. Schema: propriedades semanticamente tipadas

O schema descreve **quais dados existem, o que significam, quais valores são válidos e como se relacionam**. Não é apenas uma lista de campos ou tipos primitivos.

Uma propriedade semanticamente tipada pode declarar:

- Identidade nominal do tipo e versão.
- Tipo de representação e domínio de valores.
- Obrigatoriedade, nulabilidade explícita e valores padrão.
- Restrições/refinamentos, como intervalo, formato ou conjunto permitido.
- Invariantes e relações entre propriedades.
- Sensibilidade, classificação de dados e regras de exposição.
- Proveniência, evidência de validação e condições de confiança.
- Regras de serialização, compatibilidade e evolução do schema.

Exemplo conceitual:

```text
Property SaleId
  nominal_type = SaleId
  representation = UUID
  required = true

Property Quantity
  nominal_type = PositiveQuantity
  representation = UInt32
  constraint = value > 0

Property PaymentAuthorization
  nominal_type = VerifiedPaymentAuthorization
  evidence_required = signed_authorization
  replay_protection = required
```

Esse trecho ilustra os campos conceituais do schema; a sintaxe acima não deve ser considerada uma gramática oficial da 2flow sem implementação correspondente.

A diferença entre `String`, `SaleId` e `SessionId` é semântica: mesmo que compartilhem a representação física, não devem ser intercambiáveis automaticamente. Tipos refinados restringem valores possíveis; validação de entrada constrói esses tipos a partir de dados externos.

**Limite importante:** um nome de tipo não comprova por si só uma propriedade do mundo real. `VerifiedPaymentAuthorization` só é confiável se sua construção exigir e verificar evidências adequadas. O schema expressa o contrato; código confiável, protocolo e verificações precisam fazer cumprir o contrato.

## 8. Configuração: política sem alterar a semântica

A configuração declara escolhas de implantação e operação que não precisam estar codificadas no grafo de negócio. Dependendo do runtime, pode incluir:

- Backend e target de compilação.
- Topologia: processo local, workers, cluster ou distribuição.
- Transporte e serialização.
- Limites de concorrência, timeout, retry e backpressure.
- Endereços de serviços e descoberta.
- Políticas de autenticação, autorização e certificados.
- Persistência, retenção e observabilidade.
- Recursos e limites por Action.
- Flags de funcionalidades e parâmetros operacionais.

A configuração deve ser validada por schema próprio e separada de segredos. Chaves privadas, tokens e credenciais não devem ser gravados em arquivos de configuração versionados ou embutidos em artefatos públicos; use um mecanismo apropriado de gestão de segredos.

A configuração pode escolher **como** um contrato será executado, mas não deve silenciosamente enfraquecer **o que** o contrato exige. Por exemplo, selecionar transporte local ou remoto pode mudar a implementação física, mas não deve remover uma autorização obrigatória.

## 9. A composição: fluxo + schema + configuração

A proposta pode ser expressa como:

[
SystemDefinition = Flow_{2flow} + SemanticSchema + Configuration
]

Para virar um sistema executável, a definição também precisa de um conjunto de capacidades implementadas:

[
ExecutableSystem = Compile(SystemDefinition, Backend, RuntimeCapabilities)
]

- **2flow:** descreve causalidade, bifurcações, falhas, gates e wiring dos comportamentos.
- **Schema semântico:** descreve propriedades, eventos, entradas, saídas, contratos e invariantes de dados.
- **Configuração:** seleciona parâmetros e políticas operacionais válidas.
- **Backend/Runtime:** implementa transporte, persistência, concorrência, segurança, relógio, recursos e integrações concretas.

A tese é que os três artefatos podem funcionar como **uma especificação declarativa suficiente do sistema específico**, desde que a infraestrutura de execução já forneça as primitivas exigidas. Não é necessário redesenhar a lógica de negócio para cada topologia quando o grafo, os contratos e as capacidades do backend continuam compatíveis.

Essa separação ajuda a evitar dois extremos: codificar toda a arquitetura manualmente em cada implementação, ou tentar colocar a lógica de negócio inteira em uma configuração genérica sem semântica executável.

## 10. Da DSL à execução: AST, IR e compilação

Um pipeline de compilação possível é:

```text
2flow + Semantic Schema + Config
               |
               v
        Tokenizer / Parser
               |
               v
             AST
               |
               v
   Name, Type, Ownership, Contract Checks
               |
               v
       Canonical Semantic IR
               |
       +-------+--------+
       |                |
       v                v
 Local/Native Backend   Distributed Backend
       |                |
       v                v
 Native Calls       Actors / NATS / QUIC / RPC / WASM
```

A IR canônica deve conter, no mínimo, identidade e versão dos nós, proprietário, eventos aceitos, comportamentos invocados, resultados possíveis, arestas topológicas, contratos, efeitos relevantes e requisitos de execução.

Se a implementação Zig processa a DSL em `comptime`, o parsing e parte das verificações podem acontecer na compilação, produzindo chamadas nativas e erros pedagógicos com `@compileError`. Isso depende de como o parser, a validação e a geração foram implementados. Uma validação feita somente em runtime não deve ser descrita como garantia estática de compilação.

O compilador deve falhar de modo explícito quando faltar um backend para uma capacidade requerida, quando tipos não forem compatíveis ou quando ownership e contratos forem violados.

## 11. Como a mesma definição projeta arquiteturas diferentes

Uma definição semântica pode ser projetada para diferentes topologias físicas:

| Projeção | Mecanismo possível | Requisito que não desaparece |
|---|---|---|
| Monolito modular | Chamadas locais e módulos | Contratos e isolamento de domínio |
| Concorrência local | Threads, tasks ou event loop | Sincronização e segurança de dados |
| Actor model | Mailboxes e mensagens | Semântica de entrega e estado |
| Mensageria distribuída | NATS ou outro broker | Duplicação, ordenação e idempotência |
| RPC/QUIC | Chamadas entre processos | Autenticação, timeout e falha parcial |
| WebAssembly | Módulos WASM | Host capabilities e limites do sandbox |
| Cluster horizontal | Múltiplas instâncias | Coordenação, descoberta e persistência |

A topologia lógica pode permanecer estável; a projeção física muda. Mas nem todas as arquiteturas são equivalentes para qualquer fluxo. Um fluxo que exige transação local atômica não pode ser convertido automaticamente em vários serviços remotos sem definir a semântica para falhas parciais e compensações.

A promessa correta é **portabilidade semântica com backends conformes**, não ausência de diferenças operacionais entre arquiteturas.

## 12. Por que isso pode reduzir código, tokens e alucinações

Quando um modelo gera código diretamente de linguagem natural, precisa inferir estruturas, tipos, nomes, bibliotecas, convenções e tratamento de erros. Modelos diferentes podem escolher implementações diferentes porque o pedido normalmente não define uma única solução.

Com 2flow, schema e configuração canônicos, as decisões estáveis são declaradas uma vez e validadas. O modelo pode ajudar a converter intenção em especificação, mas a geração de código a partir de uma IR validada pode ser determinística.

[
Prompt\rightarrow ProposedDefinition\rightarrow ValidatedIR\rightarrow GeneratedCode
]

A economia de tokens é mais provável quando a definição é reutilizada por muitas Actions, linguagens, projetos ou versões. A redução de alucinações é mais provável quando o compilador usa contratos explícitos e não pede ao modelo para inventar detalhes de implementação.

Isso não elimina bugs nem erros semânticos: uma definição pode estar incorreta, um backend pode conter defeitos e um compilador pode gerar código errado. A garantia vem de validação, testes de conformidade, versões controladas e evidências reproduzíveis.

## 13. Concorrência, falhas, Saga e intervenção humana

A topologia declara a intenção concorrente; o runtime implementa a execução real. O compilador deve distinguir paralelismo permitido de dependências causais obrigatórias.

Para cada fork-join, a especificação deve definir:

- Critério de conclusão da barreira.
- Política quando um ramo retorna `Error<E>`.
- Timeout e cancelamento.
- Efeitos parciais já confirmados.
- Compensações necessárias e seus próprios erros.
- Condições para retry seguro.

Uma Saga é uma sequência de operações de domínio com compensações explícitas, não uma garantia de rollback atômico global. Um gate HITL também exige estado durável, identidade do aprovador, escopo da aprovação, expiração e proteção contra replay.

## 14. Segurança, 2FV e limites das garantias

A arquitetura pode exigir mTLS para canais entre componentes, autorização por Action, validação de mensagens e criptografia adequada para dados em trânsito e em repouso. O compilador pode verificar que uma configuração declara os requisitos; o runtime e as bibliotecas precisam aplicá-los de verdade.

A ideia de 2FV (*Two-Factor Validation*) pode separar:

1. **Pre-flight no cliente:** feedback antecipado para dados inválidos e redução de requisições desnecessárias.
2. **Validação autoritativa no servidor:** repetição das verificações relevantes antes de autorizar efeitos e persistência.

A validação no navegador não é uma fronteira de segurança: o cliente está sob controle de quem o executa e pode ser modificado ou ignorado. O servidor precisa revalidar entradas, autorização e invariantes. “Dois fatores” neste nome não deve ser confundido com autenticação multifator.

Tamanhos específicos de WASM, latências, percentuais de economia ou redução de tráfego só devem ser publicados como fatos depois de medições reproduzíveis no ambiente e nas cargas declaradas.

## 15. Exemplo integrado

O exemplo seguinte é esquemático. Ele mostra a separação conceitual; a sintaxe exata de schema e config precisa corresponder às gramáticas e aos validadores realmente implementados.

### Fluxo

```2flow
flow PurchaseProducts

ProcurementAgent.RegisterPurchase
  :--: StockAgent.IncreaseStock
  :--: FinancialAgent.RegisterFinancialEntry

execution ProcurementAgent.RegisterPurchase
  -> PurchaseEvidenceReceived
  ->> RegisterPurchase
  <<- RegisterPurchase
  <- Ok<PurchaseRegistered>
  <- Error<PurchaseRegistrationError>

execution StockAgent.IncreaseStock
  -> Ok<PurchaseRegistered>
  ->> IncreaseStock
  <<- IncreaseStock
  <- Ok<StockEntryCommitted>
  <- Error<StockEntryError>
```

### Schema semântico (conceitual)

```text
PurchaseEvidenceReceived:
  purchase_id: PurchaseId [required]
  supplier_id: SupplierId [required]
  items: NonEmptyList<PurchaseItem> [required]

PurchaseItem:
  product_id: ProductId [required]
  quantity: PositiveQuantity [required]

PurchaseRegistered:
  purchase_id: PurchaseId
  evidence_id: EvidenceId
```

### Configuração (conceitual)

```text
target = "distributed"
transport = "mtls"
delivery = "at-least-once"
idempotency = "required"
stock_agent.replicas = 3
financial_agent.replicas = 2
observability.correlation_id = "required"
```

A configuração acima não cria por si só os serviços, certificados, banco, broker ou política de idempotência. Ela seleciona requisitos e parâmetros que o runtime precisa implementar e validar. Se a configuração pede `at-least-once`, os consumidores devem estar preparados para mensagens duplicadas; se exige mTLS, o provisionamento e a validação dos certificados precisam estar operacionais.

## 16. O que significa “criar qualquer sistema com qualquer arquitetura”

A afirmação deve ser entendida como uma hipótese de **completude declarativa dentro do conjunto de capacidades suportadas**:

> Para um sistema cujo comportamento, dados, contratos e políticas possam ser expressos na 2flow, no schema semântico e na configuração, um backend compatível deve conseguir materializar uma implementação sem exigir que a lógica de negócio seja reescrita para cada topologia.

Isso se aproxima de um modelo de plataforma orientado por especificação. Para sustentar a promessa, a plataforma precisa demonstrar:

1. **Expressividade:** os três artefatos conseguem representar os comportamentos e requisitos necessários.
2. **Completude de backend:** existem adaptadores para todas as capacidades físicas exigidas pelo alvo.
3. **Preservação semântica:** a projeção mantém os contratos e efeitos observáveis relevantes.
4. **Verificação:** incompatibilidades e capacidades ausentes causam erro explícito, não comportamento silenciosamente degradado.
5. **Operabilidade:** deployment, observabilidade, segredos, migrações, recuperação e ciclo de vida também são tratados.
6. **Conformidade:** cada backend passa por uma suíte compartilhada de testes e, quando aplicável, provas formais.

“Qualquer arquitetura” em sentido literal é amplo demais para ser garantido por uma DSL finita. Arquiteturas novas podem exigir novas primitivas, novos adaptadores, modelos de consistência ou integrações externas. A 2flow pode continuar sendo a linguagem comum de orquestração enquanto o catálogo de capacidades evolui.

## 17. Invariantes canônicos

1. `:--:` avança somente quando o resultado satisfaz o contrato causal de sucesso.
2. `!->` é uma aresta de falha, não um terceiro tipo de retorno.
3. Cada fronteira executável representa sucesso ou falha explicitamente tipados.
4. `->` e `<-` representam entrada e saída em relação ao escopo declarado.
5. `->>` e `<<-` representam invocação e recepção de comportamento próprio.
6. Um agente não chama diretamente comportamentos internos pertencentes a outro agente.
7. Comunicação entre agentes respeita contratos de fronteira explícitos.
8. `[A, B]` expressa concorrência lógica, não garante paralelismo físico em qualquer runtime.
9. `[? Gate]` exige evidência humana autenticada, autorizada e vinculada à operação.
10. Normalização pertence às etapas de validação ou auto-cura definidas pelo contrato, não deve ser um efeito oculto do comportamento de domínio.
11. Schema, configuração e versões são validados antes da implantação.
12. Uma projeção de runtime deve declarar capacidades ausentes e preservar os contratos que afirma suportar.
13. Falhas parciais, retries e compensações precisam ter semântica explícita.
14. A validação no cliente nunca substitui a validação autoritativa no servidor.

## 18. Validação e critérios de prova

A implementação da 2flow deve testar não apenas se a sintaxe é aceita, mas também se sua semântica é preservada.

- **Parser:** todos os oito operadores, precedência, agrupamento e erros de sintaxe.
- **Type checker:** entradas/saídas incompatíveis, tipos nominais, erros permitidos e contratos.
- **Ownership:** chamadas diretas inválidas entre contextos devem falhar na validação.
- **Fork-join:** sucesso, falha de ramo, timeout, cancelamento e política de barreira.
- **Saga:** compensação acionada apenas nas condições declaradas, idempotência e falha da própria compensação.
- **HITL:** aprovação, rejeição, expiração, identidade, autorização e replay.
- **Backends:** uma mesma IR executada em runtimes distintos deve passar pela mesma suíte de conformidade.
- **Reprodutibilidade:** fixar versões, configuração e dependências; comparar hashes e contratos gerados.
- **Segurança:** testar configurações inválidas, canais, permissões, segredos e validação autoritativa.
- **Desempenho:** medir separadamente custo de compilação, latência de execução, serialização, rede, CPU, RAM e I/O.

Uma compilação bem-sucedida comprova que certas verificações estáticas passaram; não prova, por si só, a correção do comportamento, a segurança de ponta a ponta ou a equivalência entre todos os runtimes.

## Conclusão

A 2flow descreve a causalidade e o protocolo de execução; o schema semântico tipa os dados, contratos e invariantes; a configuração escolhe políticas e parâmetros de implantação. Juntos, esses artefatos podem se tornar uma especificação canônica a partir da qual compiladores e runtimes geram sistemas em diferentes linguagens e topologias.

O princípio arquitetural é **definir o significado uma vez e projetá-lo para diferentes mecanismos de execução**, em vez de reimplementar a lógica de negócio para cada arquitetura. A promessa é mais forte quando acompanhada por uma IR canônica, backends conformes, verificação de contratos, testes compartilhados e evidências reproduzíveis. Assim, a portabilidade não depende de supor que todas as arquiteturas são iguais, mas de definir exatamente o que deve permanecer invariável entre elas.
