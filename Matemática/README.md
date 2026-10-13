# AllasCode pela Matemática

Esta disciplina explica a arquitetura e a plataforma AllasCode por meio de conceitos matemáticos fundamentais e avançados: teoria dos conjuntos, funções, lógica formal, álgebra, teoria da informação, probabilidade, sistemas de tipos, criptografia, teoria dos grafos, otimização, teoria das filas e sistemas de transição.

A ideia central é tratar a AllasCode como uma arquitetura de transformações semânticas tipadas: intenções são interpretadas, comportamentos são selecionados, Actions executam transformações sob contratos e evidências permitem verificar se a trajetória alcançou o destino esperado.

> **Nota de rigor:** analogias matemáticas ajudam a explicar o projeto, mas não constituem automaticamente provas de segurança, desempenho ou optimalidade. Afirmações sobre redução de tokens, qualidade de classificação, determinismo e escalabilidade devem ser validadas com medições reproduzíveis.

## Sumário

- [1. Fundamentos: conjuntos, funções e domínios](#1-fundamentos-conjuntos-funções-e-domínios)
- [2. A arquitetura como composição de transformações](#2-a-arquitetura-como-composição-de-transformações)
- [3. Semantic Behavior-typed Algebra](#3-semantic-behavior-typed-algebra)
- [4. AtomicSkill, AtomicBehavior, Action, Actor e Runtime](#4-atomicskill-atomicbehavior-action-actor-e-runtime)
- [5. BehaviorID: por que uma janela de três mensagens](#5-behaviorid-por-que-uma-janela-de-três-mensagens)
- [6. Doze categorias: discretização e complexidade de decisão](#6-doze-categorias-discretização-e-complexidade-de-decisão)
- [7. Tipagem forte, estrita e semântico-nominal](#7-tipagem-forte-estrita-e-semântico-nominal)
- [8. DSL ubíqua e semântica: da intenção ao código específico](#8-dsl-ubíqua-e-semântica-da-intenção-ao-código-específico)
- [9. Economia de tokens, custo de modelo e redução de alucinações](#9-economia-de-tokens-custo-de-modelo-e-redução-de-alucinações)
- [10. Determinismo e reuso: por que o mesmo pedido gera códigos diferentes](#10-determinismo-e-reuso-por-que-o-mesmo-pedido-gera-códigos-diferentes)
- [11. Criptografia híbrida e separação de responsabilidades](#11-criptografia-híbrida-e-separação-de-responsabilidades)
- [12. mTLS uniforme e invariância topológica](#12-mtls-uniforme-e-invariância-topológica)
- [13. Escalabilidade orientada por identidade e benchmarks de Action](#13-escalabilidade-orientada-por-identidade-e-benchmarks-de-action)
- [14. Teoria das filas, concorrência e mensageria](#14-teoria-das-filas-concorrência-e-mensageria)
- [15. Erros estruturados, recuperação e idempotência](#15-erros-estruturados-recuperação-e-idempotência)
- [16. T-DD: trajetória, destino e evidências](#16-t-dd-trajetória-destino-e-evidências)
- [17. Otimização global sob invariantes](#17-otimização-global-sob-invariantes)
- [18. AllasCode como sistema de invariantes](#18-allascode-como-sistema-de-invariantes)
- [19. Como validar as hipóteses empiricamente](#19-como-validar-as-hipóteses-empiricamente)
- [Conclusão](#conclusão)

## 1. Fundamentos: conjuntos, funções e domínios

A teoria dos conjuntos oferece uma linguagem simples para definir os domínios da arquitetura. Podemos representar:

- `I`: intenções reconhecidas.
- `B`: comportamentos semanticamente definidos.
- `A`: Actions disponíveis.
- `R`: runtimes capazes de executar operações.
- `S`: estados possíveis do sistema.
- `E`: evidências e resultados observáveis.

Uma função de classificação pode ser escrita como:

\[
f:I\times C\rightarrow B
\]

em que `C` é o contexto relevante. A função associa uma intenção e seu contexto a um comportamento compatível.

Uma Action transforma um estado:

\[
a:S\rightarrow \operatorname{Result}(S',E)
\]

O resultado diferencia uma execução bem-sucedida de uma falha estruturada. Em um sistema real, uma Action também pode ter efeitos externos, depender de recursos e precisar lidar com falhas parciais.

A separação de domínios é útil porque intenção, comportamento, execução e resultado não são a mesma coisa. Cada nível pode evoluir sem obrigar os outros a conhecer seus detalhes internos, desde que os contratos permaneçam compatíveis.

## 2. A arquitetura como composição de transformações

Na álgebra, operações podem ser compostas. Se:

\[
a_1:S_0\rightarrow S_1,\qquad
a_2:S_1\rightarrow S_2
\]

então a composição \(a_2\circ a_1\) leva \(S_0\) a \(S_2\). Em uma arquitetura distribuída, essas transformações podem ser executadas por Actors diferentes, em processos ou máquinas distintos.

A composição só é válida quando a saída de uma etapa satisfaz os requisitos da próxima:

\[
\operatorname{Post}(a_i)\models\operatorname{Pre}(a_{i+1})
\]

Essa é a ideia por trás das precondições e pós-condições usadas na lógica de Hoare:

\[
\{P\}\ a\ \{Q\}
\]

`P` descreve a condição exigida antes da operação e `Q` descreve o que deve valer após uma execução bem-sucedida.

**Benefício:** incompatibilidades podem ser detectadas nas fronteiras contratuais, em vez de aparecerem como suposições implícitas espalhadas pelo sistema. A modularidade não remove a complexidade; torna suas interfaces mais explícitas e verificáveis.

## 3. Semantic Behavior-typed Algebra

Semantic Behavior-typed Algebra pode ser apresentada como a combinação de três ideias:

1. **Álgebra:** define como operações são compostas.
2. **Semântica formal:** define o significado das operações no domínio.
3. **Sistemas de tipos:** definem quais composições são admissíveis.

Uma álgebra pode ser representada abstratamente como:

\[
\mathcal{A}=(B,\Omega,\mathcal{R})
\]

em que `B` é o conjunto de comportamentos, `Ω` é o conjunto de operações de composição e `ℛ` é o conjunto de regras que governa sua aplicação.

Considere:

\[
a:\texttt{ValidatedPayment}\rightarrow\texttt{PaymentReceipt}
\]

\[
b:\texttt{PaymentReceipt}\rightarrow\texttt{OrderConfirmation}
\]

A composição \(b\circ a\) é compatível porque o tipo de saída de `a` corresponde ao tipo de entrada de `b`. Uma string genérica, por outro lado, não deveria poder substituir silenciosamente um pagamento validado, uma identidade verificada ou uma autorização.

A álgebra não deve verificar apenas compatibilidade estrutural. Deve preservar o significado e as condições de validade do domínio. Um agente saber como solicitar uma operação não lhe concede automaticamente autorização para executá-la.

## 4. AtomicSkill, AtomicBehavior, Action, Actor e Runtime

A ontologia A3 pode ser explicada como a separação entre capacidade, especificação semântica, operação executável, responsabilidade de execução e ambiente computacional.

| Elemento | Interpretação |
|---|---|
| **AtomicSkill** | Unidade definida de capacidade ou procedimento |
| **AtomicBehavior** | Comportamento semanticamente especificado |
| **Action** | Operação executável com entradas, saídas, contrato e efeitos definidos |
| **Actor** | Entidade responsável por executar ou coordenar operações |
| **Runtime** | Ambiente que fornece mecanismos e recursos de execução |

Esses níveis não são sinônimos. Uma Action pode ser implementada em linguagens diferentes, desde que cada implementação preserve o comportamento observável exigido pelo contrato.

\[
\operatorname{Obs}(a_{\text{TS}})=\operatorname{Obs}(a_{\text{Zig}})
\]

Essa igualdade representa equivalência observável no escopo do contrato, não igualdade de código, latência ou consumo de memória.

A separação permite trocar uma implementação sem necessariamente modificar o significado da operação, o fluxo de negócio ou os consumidores que dependem de sua interface.

## 5. BehaviorID: por que uma janela de três mensagens

Considere uma conversa como uma sequência \(M=(m_1,m_2,\ldots,m_t)\). O estado operacional atual não é observado diretamente: o sistema o estima com base nas mensagens e no contexto disponível.

Uma janela de três mensagens é:

\[
W_t=(m_{t-2},m_{t-1},m_t)
\]

A classificação pode ser representada como:

\[
\hat Z_t=f(W_t,C_t)
\]

As três mensagens podem fornecer três sinais complementares:

1. Contexto imediatamente anterior.
2. Transição produzida pela interação e pela resposta anterior.
3. Evidência mais recente sobre o objetivo ou a solicitação atual.

A analogia com a propriedade de Markov é útil. Em um processo markoviano, o estado atual contém informação suficiente para modelar o próximo estado, sem consultar toda a sequência histórica:

\[
P(Z_{t+1}\mid Z_t,Z_{t-1},\ldots)=P(Z_{t+1}\mid Z_t)
\]

No BehaviorID, três mensagens podem ser uma aproximação prática dessa ideia: uma janela curta captura o contexto local necessário para uma decisão frequente, reduzindo texto a processar e a influência de mensagens antigas irrelevantes.

Contudo, a propriedade de Markov **não prova** que três mensagens sejam sempre suficientes. Uma restrição antiga, uma autorização, um compromisso ainda válido ou um objetivo de longa duração pode não aparecer na janela recente. Esses dados devem permanecer no estado persistente quando forem necessários.

A hipótese informacional que se deseja verificar é:

\[
I(Z_t;H_t\mid W_t,C_t)\approx 0
\]

em que `H_t` é o histórico completo e `I` é a informação mútua condicional. Intuitivamente, depois de conhecer a janela e o contexto persistente, o restante do histórico acrescentaria pouca informação para a decisão atual.

Três mensagens devem ser entendidas como uma **janela de inferência local**, não como substituto de memória operacional, contratos, histórico de auditoria ou evidências de segurança. Sua adequação deve ser medida com dados representativos, comparando janelas de diferentes tamanhos.

## 6. Doze categorias: discretização e complexidade de decisão

O BehaviorID pode mapear sinais comportamentais para um conjunto finito de categorias operacionais:

\[
Z=\{z_1,z_2,\ldots,z_{12}\}
\]

Isso é uma forma de discretização: um espaço de comportamentos muito amplo é representado por classes relevantes para as decisões do sistema. Não significa que pessoas tenham somente doze estados cognitivos possíveis; significa que o classificador utiliza doze classes para um propósito operacional definido.

Se duas situações levam à mesma política, com os mesmos contratos e consequências, talvez não seja necessário distingui-las. Se uma distinção muda uma permissão, uma decisão crítica ou o resultado esperado, agrupá-las pode ser perigoso.

Para doze categorias equiprováveis, a entropia é:

\[
H(Z)=\log_2 12\approx 3{,}585\text{ bits}
\]

Esse número descreve a incerteza teórica de doze alternativas equiprováveis; não é o custo computacional do modelo nem o número de bits usados para processar mensagens.

A escolha do número de categorias pode ser tratada como um problema de otimização:

\[
N^*=\arg\min_N\left[
L_{\text{decisão}}(N)+\lambda C_{\text{modelo}}(N)+\mu C_{\text{operação}}(N)
\right]
\]

O objetivo é equilibrar erros de decisão, dificuldade de classificação e custo de manutenção. Doze categorias podem ser um compromisso eficiente, mas sua optimalidade exige comparação empírica com alternativas, usando precisão, recall, matriz de confusão, calibração, custo e qualidade das ações resultantes.

## 7. Tipagem forte, estrita e semântico-nominal

A tipagem restringe quais valores podem ser utilizados em cada operação. A proposta semântico-nominal vai além do formato físico: o tipo deve representar o significado do dado.

Por exemplo, `UserId` e `SessionId` podem ser strings na representação física, mas não são intercambiáveis:

\[
\texttt{UserId}\neq\texttt{SessionId}
\]

- **Tipagem forte:** procura impedir operações inválidas entre tipos incompatíveis e conversões implícitas perigosas.
- **Tipagem estrita:** restringe ambiguidades, coerções silenciosas e valores que não respeitam o contrato.
- **Tipagem nominal:** distingue tipos pela identidade de sua declaração, não apenas pela estrutura.
- **Tipagem semântica:** representa significado, domínio e condições de validade.

Um tipo validado pode ser modelado como um subconjunto de valores primitivos:

\[
\texttt{VerifiedUserId}=\{x\in\texttt{String}\mid P(x)\}
\]

`P(x)` representa os critérios de validade. Uma transformação explícita de parsing e validação constrói o tipo refinado a partir do valor bruto.

Isso reduz o espaço de estados inválidos que pode circular dentro do domínio. Porém, o compilador só garante aquilo que o sistema de tipos e as funções de construção realmente codificam. Um tipo chamado `VerifiedIdentity` não prova, sozinho, que uma pessoa foi autenticada: a garantia depende de evidências e protocolos apropriados.

A validação de entradas externas, a autorização, a integridade criptográfica e as invariantes distribuídas continuam necessárias.

## 8. DSL ubíqua e semântica: da intenção ao código específico

Uma DSL (Domain-Specific Language) define uma linguagem orientada a um domínio específico. Quando é ubíqua, os termos utilizados no código, nos contratos, nos testes e na comunicação do domínio mantêm significados consistentes.

Na AllasCode, a DSL semântica pode funcionar como uma representação intermediária entre a intenção de alto nível e a implementação concreta.

\[
\text{Intenção}
\rightarrow\text{Semântica/DSL}
\rightarrow\text{IR tipada}
\rightarrow\text{Backend específico}
\rightarrow\text{Código}
\]

A representação intermediária (IR) pode explicitar:

- Identidade da Action e versão do contrato.
- Tipos nominais de entradas e saídas.
- Pré-condições e pós-condições.
- Erros permitidos e regras de recuperação.
- Efeitos e permissões necessários.
- Regras de serialização e comunicação.
- Restrições de segurança.
- Critérios de aceitação e evidências exigidas.

O compilador ou gerador converte essa representação em código para uma linguagem ou runtime específico. TypeScript, Zig ou outra linguagem tornam-se backends de uma mesma definição semântica, em vez de interpretações independentes de um pedido em linguagem natural.

A propriedade desejada é a preservação do contrato:

\[
\operatorname{Sem}(\text{DSL})\approx
\operatorname{Obs}(\text{código gerado})
\]

O símbolo \(\approx\) significa equivalência no escopo das propriedades observáveis especificadas. Ela precisa ser apoiada por validação de esquema, testes, verificações estáticas, testes de conformidade e, quando viável, verificação formal.

## 9. Economia de tokens, custo de modelo e redução de alucinações

Pedir a vários modelos que traduzam diretamente uma descrição extensa para código mistura diversas tarefas: interpretar a intenção, inferir regras ausentes, escolher abstrações, inventar nomes, definir tipos, selecionar bibliotecas e produzir sintaxe válida.

Cada decisão não especificada abre espaço para variações e suposições. Ao utilizar uma DSL semântica explícita, parte dessas decisões passa a ser definida uma única vez e reutilizada pelo compilador.

O modelo pode ser usado para interpretar ou propor a representação semântica, enquanto transformações determinísticas e verificações mecânicas cuidam das partes que não precisam de criatividade.

### 9.1 Onde os tokens podem ser economizados

Sem uma representação intermediária reutilizável, cada solicitação pode precisar reenviar ao modelo explicações de arquitetura, regras de tipos, contratos, convenções, padrões de erro e exemplos de implementação.

Com uma DSL, os elementos estáveis são representados uma vez e referenciados por identificadores ou estruturas compactas. O gerador reutiliza templates, definições e regras existentes.

O custo aproximado pode ser expresso como:

\[
C_{\text{direto}}=N(C_{\text{contexto}}+C_{\text{geração}}+C_{\text{correção}})
\]

\[
C_{\text{DSL}}=C_{\text{definição}}+
N(C_{\text{referência}}+C_{\text{geração determinística}}+C_{\text{validação}})
\]

em que `N` é o número de implementações ou evoluções que reutilizam a mesma definição.

A economia aparece quando o custo inicial de definir e validar a DSL é amortizado ao longo de muitas gerações. Ela não é automática: uma DSL excessivamente verbosa, um compilador complexo ou uma geração que continue exigindo grandes prompts pode reduzir ou anular o benefício.

A medição deve separar tokens de entrada, tokens de saída, chamadas ao modelo, custo monetário, latência, retrabalho e taxa de defeitos.

### 9.2 Menos decisões livres, menos oportunidades de alucinação

Uma alucinação de código pode aparecer como uma API inexistente, uma dependência inventada, um campo incompatível, um tipo incorreto ou uma regra de negócio presumida.

A DSL reduz a superfície de inferência quando determina explicitamente esses elementos e restringe as saídas possíveis. O compilador não precisa imaginar o que significa uma Action: ele lê sua definição formal.

Isso não elimina todos os erros. A DSL pode estar errada, a interpretação inicial pode estar errada e o gerador pode conter bugs. O benefício é converter muitas suposições implícitas em dados explícitos, validáveis e testáveis.

Uma divisão robusta de responsabilidades é:

1. O modelo interpreta linguagem natural quando a tarefa exige interpretação.
2. O modelo produz uma representação estruturada sujeita a esquema e regras semânticas.
3. Validadores rejeitam campos, tipos ou combinações inválidas.
4. O compilador determinístico gera o código específico.
5. Compiladores nativos, testes e verificadores de contrato avaliam o resultado.
6. A evidência de execução confirma os comportamentos que só podem ser verificados em runtime.

Assim, o modelo deixa de ser o único responsável por produzir código correto. Ele participa de um pipeline que limita, verifica e corrige os resultados.

### 9.3 Como demonstrar a economia

A comparação deve utilizar as mesmas tarefas, contratos, linguagens-alvo e critérios de aceitação. Para cada abordagem, medir:

- Tokens de entrada e saída.
- Custo total das chamadas ao modelo.
- Quantidade de correções manuais.
- Compilações aprovadas na primeira tentativa.
- Testes e contratos aprovados.
- Defeitos semânticos e APIs inventadas.
- Tempo total até a implementação aceita.
- Quantidade de código reutilizado.

A afirmação mais defensável é que uma DSL semântica **pode** reduzir tokens, custo e alucinações ao retirar do modelo decisões que podem ser codificadas e verificadas. A magnitude do ganho depende do desenho da DSL, da qualidade do compilador e do padrão de reutilização.

## 10. Determinismo e reuso: por que o mesmo pedido gera códigos diferentes

Um pedido em linguagem natural não define uma única implementação. Ele normalmente permite muitas soluções que parecem plausíveis.

Modelos diferentes têm parâmetros, dados de treinamento, políticas de geração, contextos e vieses diferentes. Mesmo o mesmo modelo pode produzir saídas distintas em execuções diferentes, dependendo da amostragem, do contexto e de mudanças de versão.

Assim, para uma solicitação `p`, é comum observar:

\[
G_{M_1}(p)\neq G_{M_2}(p)
\]

em que \(G_{M_i}\) é a geração de código realizada pelo modelo \(M_i\). As diferenças podem aparecer em nomes, estrutura, bibliotecas, tratamento de erros, concorrência, validações e decisões de arquitetura.

Não é um teorema de que todos os modelos sempre produzirão códigos diferentes; eles podem coincidir. O ponto é que **a linguagem natural, sozinha, não fornece um contrato que force implementações equivalentes**.

### 10.1 Separar interpretação de implementação

A estratégia é transformar o pedido em uma definição semântica canônica e depois gerar código a partir dela:

\[
p\xrightarrow{\text{interpretação}}D
\xrightarrow{\text{validação}}D^*
\xrightarrow{\text{compilação}}C
\]

- `p`: pedido original.
- `D`: definição semântica proposta.
- `D*`: definição validada e canônica.
- `C`: código gerado.

A interpretação pode continuar probabilística. O objetivo é que, uma vez validada e congelada a mesma definição `D*`, o backend determinístico produza a mesma saída para as mesmas entradas, versões e configurações.

Formalmente:

\[
\operatorname{Compile}(D^*,V,K)=C
\]

Se `D*`, a versão do compilador `V` e a configuração `K` forem iguais, o compilador deve produzir a mesma saída observável, salvo fontes deliberadas de não determinismo que tenham sido documentadas.

### 10.2 Determinismo não significa que todo código precisa ser idêntico

Há pelo menos três níveis diferentes:

1. **Determinismo textual:** mesmos bytes de saída.
2. **Equivalência estrutural:** mesmos tipos, contratos e componentes semânticos.
3. **Equivalência comportamental:** mesmo comportamento observável dentro do escopo definido.

Dois backends podem gerar código textual diferente e ainda preservar o mesmo contrato. O objetivo principal é a equivalência semântica e comportamental; a reprodução byte a byte é especialmente útil para builds, cache, auditoria e revisão de diffs.

Para tornar a geração reproduzível, é preciso controlar versões, configurações, ordenação de elementos, formatação, dependências e metadados voláteis. Se a saída incluir timestamps, caminhos locais ou identificadores aleatórios, esses valores precisam ser removidos, fixados ou explicitamente tratados.

### 10.3 Reuso como redução de graus de liberdade

Cada implementação independente pode inventar uma nova maneira de representar o mesmo conceito. Isso multiplica variantes, aumenta o custo de manutenção e dificulta comparar resultados.

Uma DSL canônica reduz esses graus de liberdade ao reutilizar:

- Tipos nominais compartilhados.
- Definições de contratos.
- Catálogos de erros.
- Primitivas de segurança.
- Regras de serialização.
- Templates de implementação.
- Testes de conformidade.
- Identificadores semânticos estáveis.

O código deixa de ser uma interpretação original de cada pedido e passa a ser uma materialização de definições que já possuem significado explícito.

Essa abordagem favorece consistência e reuso sem exigir que todos os backends compartilhem a mesma implementação interna.

### 10.4 Um único contrato, múltiplos backends

Uma Action pode ser definida uma vez e implementada para diferentes linguagens ou ambientes. Cada backend deve cumprir o mesmo conjunto de testes de conformidade.

\[
\operatorname{Contract}(A)
\models
\operatorname{Implementation}_i(A)
\]

A expressão representa a obrigação de cada implementação respeitar o contrato da Action. Não é suficiente compilar: é necessário verificar entradas, saídas, erros, efeitos e invariantes.

A arquitetura ganha previsibilidade porque as decisões semânticas são centralizadas, enquanto os detalhes específicos de linguagem ficam restritos ao backend.

## 11. Criptografia híbrida e separação de responsabilidades

A criptografia híbrida combina mecanismos assimétricos para estabelecimento ou encapsulamento de chaves com criptografia simétrica para proteger dados em volume.

Um pipeline típico é:

1. Autenticar as identidades conforme o protocolo.
2. Estabelecer ou encapsular o segredo de sessão.
3. Derivar chaves com uma função apropriada, como HKDF.
4. Proteger mensagens com uma cifra autenticada, como ChaCha20-Poly1305.
5. Gerenciar nonces, rotação, expiração e descarte de chaves.

De forma simplificada:

\[
C_{\text{total}}=C_{\text{estabelecimento}}+
n\cdot C_{\text{simétrico por byte}}
\]

A combinação evita utilizar operações assimétricas pesadas para proteger diretamente todo o conteúdo. O custo do estabelecimento é amortizado pelo processamento simétrico eficiente.

X25519 pode ser utilizado para acordo de segredo; HKDF deriva chaves; ChaCha20-Poly1305 protege conteúdo com confidencialidade e integridade autenticada. XChaCha20-Poly1305 utiliza um nonce maior e pode simplificar determinados desenhos de gerenciamento de nonces.

Esses mecanismos não são intercambiáveis: acordo de segredo não é autenticação, e autenticação não é autorização. A associação entre chave, identidade, permissão e contexto precisa ser estabelecida pelo protocolo.

A combinação X25519 com ML-KEM-768 (frequentemente representada em perfis híbridos como X25519MLKEM768) pode apoiar uma transição pós-quântica quando implementada por um protocolo compatível e corretamente especificado. Não torna automaticamente todo o sistema pós-quântico: certificados, negociação, composição de segredos, bibliotecas e protocolos também precisam ser avaliados.

## 12. mTLS uniforme e invariância topológica

A rede pode ser modelada como um grafo:

\[
G=(V,E)
\]

`V` representa serviços, agentes e componentes; `E` representa seus canais de comunicação. A política pretendida é que toda comunicação externa-interna e interna-interna respeite um contrato de canal autenticado e protegido.

\[
\forall e\in E,\quad \operatorname{SecureChannel}(e)
\]

O mTLS permite que cliente e servidor autentiquem suas identidades criptográficas durante o estabelecimento do canal. Isso evita tratar a simples localização em uma rede interna como prova suficiente de confiança.

Autenticação não é autorização. Mesmo após autenticar um serviço, o sistema precisa verificar se aquela identidade pode solicitar aquela Action nas condições presentes.

A vantagem de um contrato uniforme é a invariância topológica: quando um Actor sai de um processo e passa a executar em outra máquina, o significado da comunicação não precisa mudar.

\[
\text{um nó}\rightarrow\text{múltiplos nós}
\]

enquanto permanecem os requisitos de identidade, canal protegido e autorização.

A uniformidade reduz exceções de configuração e ajuda a escalar sem criar um modelo de confiança diferente para cada topologia. Para ser efetiva, precisa incluir gestão de certificados, renovação, revogação, descoberta de serviços, validação de identidade e políticas de autorização. Também não substitui criptografia em repouso nem a proteção dos dados dentro de filas e bancos.

## 13. Escalabilidade orientada por identidade e benchmarks de Action

Cada Action pode ter um perfil operacional distinto. Uma operação pode ser limitada por CPU; outra, por RAM; outra, por disco, rede ou latência de uma dependência externa.

Podemos representar o perfil observado de uma Action por:

\[
\mathbf{c}_i=
\begin{bmatrix}
C_{\text{CPU}}\\
C_{\text{RAM}}\\
C_{\text{Disco}}\\
C_{\text{Rede}}
\end{bmatrix}
\]

Um benchmark útil inclui CPU por execução, RAM de pico, I/O, latência p50/p95/p99, throughput, erros, timeouts e características da carga.

### Escala vertical

Aumenta os recursos do nó em que a Action executa: CPU, RAM ou capacidade de armazenamento. É adequada quando o gargalo é local e o software consegue aproveitar os recursos adicionais.

### Escala horizontal

Aumenta o número de instâncias que executam a Action. Pode ampliar a capacidade quando a carga é distribuível e as dependências compartilhadas não se tornam o gargalo.

### Escala distribuída

Distribui diferentes cargas por múltiplos nós, respeitando capacidade, latência, comunicação e limites de recursos.

A identidade semântica da Action deve ser separada de seu perfil de benchmark. Uma identidade descreve o que a operação faz; o benchmark descreve como determinada implementação se comportou em uma versão, hardware e carga específicos.

\[
B_i=g(a_i,v_i,h_i,d_i)
\]

em que `a_i` é a Action, `v_i` a versão da implementação, `h_i` o ambiente e `d_i` as características da carga.

A alocação pode ser formulada como otimização sujeita a limites:

\[
\sum_i x_{ij}r_{i,\text{RAM}}\le R_{j,\text{RAM}}
\]

e condições semelhantes para CPU e I/O. `x_{ij}` representa a carga da Action `i` atribuída ao nó `j`.

Um controlador pode então selecionar escala vertical, horizontal ou distribuição com base na pressão real. Os benchmarks orientam a decisão, mas não definem sozinhos a política: é preciso observar filas, capacidade compartilhada, custo, limites, estabilidade e oscilações de escala.

## 14. Teoria das filas, concorrência e mensageria

Em um sistema com taxa de chegada de trabalho \(\lambda\), capacidade de processamento por instância \(\mu\) e `n` instâncias equivalentes, a utilização aproximada é:

\[
\rho=\frac{\lambda}{n\mu}
\]

Para um modelo estável, a capacidade de processamento precisa superar a taxa de chegada. Quando a utilização se aproxima de 1, variações pequenas podem aumentar muito o tempo de espera.

A lei de Little relaciona o número médio de trabalhos no sistema ao tempo médio de permanência:

\[
L=\lambda W
\]

Isso explica por que throughput isolado não basta: filas crescentes e latência de cauda podem revelar saturação antes que o sistema pare completamente.

A mensageria assíncrona reduz dependências temporais. Um Actor publica uma mensagem e o consumidor a processa segundo o contrato, sem exigir que o produtor mantenha uma chamada síncrona aberta até a conclusão de todo o fluxo.

Quando duas Actions são independentes, podem ser executadas em paralelo:

\[
a_1\parallel a_2\rightarrow a_3
\]

O paralelismo pode reduzir latência, mas só é válido quando as dependências e os efeitos são compatíveis. Mensageria também exige regras para duplicação, ordenação, retry, idempotência, correlação e falhas parciais.

## 15. Erros estruturados, recuperação e idempotência

Uma operação pode retornar sucesso ou falha tipada:

\[
\operatorname{Result}(T,E)=\operatorname{Ok}(T)\mid\operatorname{Err}(E)
\]

Um erro estruturado pode conter código estável, categoria semântica, gravidade, recuperabilidade, ação recomendada e identificador de correlação.

Uma política de tratamento pode ser representada por:

\[
h:E\rightarrow\{\text{retry},\text{fail},\text{escalate},\text{compensate}\}
\]

A escolha depende do significado do erro e do estado da operação. Um timeout não prova que uma operação remota não aconteceu; a resposta pode ter sido perdida depois de a operação ser confirmada.

Por isso, retries precisam considerar idempotência, chaves de deduplicação e consulta do estado quando necessário. Um erro estruturado só é útil quando a política preserva os invariantes e evita transformar incerteza em falsa confirmação.

## 16. T-DD: trajetória, destino e evidências

O Trajectory-Driven Development pode ser entendido como um sistema de transições entre estados. Uma execução produz uma trajetória:

\[
S_0\xrightarrow{a_1}S_1\xrightarrow{a_2}S_2\xrightarrow{a_3}S_3
\]

A intenção define o objetivo inicial; a trajetória descreve a evolução; o destino define as condições esperadas no estado terminal.

A aceitação deve depender de evidência:

\[
\operatorname{Accept}(S_f,D)=\text{true}
\]

Uma sequência sem exceções não é necessariamente uma execução correta. Se o destino exige uma ordem registrada, uma mensagem de confirmação não prova que o registro ocorreu. A evidência precisa demonstrar o estado esperado.

Quando uma transição falha, a política pode repetir uma operação segura, selecionar outra trajetória, solicitar informação, encerrar com erro estruturado ou encaminhar o caso para intervenção humana.

O princípio é que a execução deve convergir para um destino verificável, não apenas percorrer uma lista de passos.

## 17. Otimização global sob invariantes

A eficiência do sistema completo não pode ser reduzida à velocidade de uma função isolada. Uma trajetória precisa equilibrar CPU, RAM, I/O, rede, latência e custo de falhas.

\[
J(\tau)=\alpha C_{\text{CPU}}+
\beta C_{\text{RAM}}+
\gamma C_{\text{I/O}}+
\delta C_{\text{rede}}+
\epsilon C_{\text{latência}}+
\zeta C_{\text{falhas}}
\]

O objetivo é selecionar uma trajetória válida que minimize o custo:

\[
\tau^*=\arg\min_{\tau\in\mathcal{T}_{\text{válida}}}J(\tau)
\]

A trajetória mais barata não é aceitável se viola um contrato, enfraquece a segurança ou deixa de alcançar o destino. A otimização deve acontecer dentro do conjunto de soluções válidas, e não pela remoção de verificações essenciais.

As janelas curtas de classificação, as categorias finitas, os tipos semântico-nominais, os contratos, a criptografia uniforme e os benchmarks podem contribuir para essa otimização em dimensões diferentes. Os ganhos precisam ser medidos para evitar tratar uma hipótese arquitetural como um resultado garantido.

## 18. AllasCode como sistema de invariantes

Um invariante é uma propriedade que precisa continuar verdadeira durante as transformações do sistema. Exemplos possíveis incluem:

- Uma Action recebe somente entradas aceitas pelo contrato.
- Uma operação protegida só ocorre quando a autorização é válida.
- Uma mensagem não é tratada como prova de identidade sem evidências adequadas.
- Uma trajetória só é aceita quando o destino é comprovado.
- Uma falha não é silenciosamente convertida em sucesso.
- A mudança de máquina não modifica o contrato semântico da Action.
- Backends diferentes preservam o comportamento observável especificado.
- A distribuição da carga respeita os limites de recursos e as exigências de latência.

Podemos representar o conjunto de invariantes como:

\[
\mathcal{I}=\{I_1,I_2,\ldots,I_n\}
\]

Uma transição válida deve preservar os invariantes aplicáveis:

\[
I(S_t)\land\operatorname{Valid}(a_t,S_t)
\Rightarrow I(S_{t+1})
\]

Em sistemas distribuídos, essa obrigação precisa considerar concorrência, efeitos externos, falhas parciais e consistência. Tipos estáticos ou contratos locais não provam automaticamente as propriedades do sistema inteiro; são partes de uma estratégia de verificação em camadas.

## 19. Como validar as hipóteses empiricamente

As escolhas arquiteturais tornam-se afirmações fortes quando podem ser medidas e reproduzidas.

### BehaviorID

Compare janelas de 1, 2, 3, 4 e mais mensagens, usando os mesmos conjuntos de avaliação. Meça qualidade da classificação, calibração, custo, latência e erros de decisão. Inclua casos em que uma restrição histórica precisa ser recuperada do estado persistente.

### Doze categorias

Compare conjuntos alternativos de categorias. Analise a matriz de confusão e verifique se categorias agrupadas realmente conduzem às mesmas decisões e consequências. Não use apenas a acurácia global: erros críticos podem ficar ocultos por classes frequentes.

### DSL e geração de código

Compare geração direta por modelo com geração a partir de uma DSL validada. Use as mesmas tarefas e contratos. Meça tokens, custo, tempo total, taxa de compilação inicial, conformidade dos contratos, defeitos semânticos, correções manuais e reuso.

### Determinismo

Congele a definição semântica, a versão do compilador, as dependências e a configuração. Execute a geração repetidamente e compare hashes dos artefatos. Em seguida, execute testes de conformidade para verificar equivalência comportamental entre backends.

### Escalabilidade

Execute benchmarks controlados por Action e por versão. Varie carga, concorrência, hardware e distribuição. Meça throughput, latência p95/p99, CPU, RAM, I/O, rede, filas, erros e custo. Verifique se o controlador escolhe a estratégia correta sem oscilar excessivamente.

### Segurança

Teste autenticação mútua, validação de certificados, autorização, rotação, revogação, proteção contra replay, gerenciamento de chaves e comportamento diante de configurações inválidas. A existência de mTLS ou criptografia híbrida, isoladamente, não prova a segurança integral do sistema.

O resultado desse processo é uma arquitetura cujas decisões não dependem apenas de argumentos conceituais: dependem também de contratos explícitos, medições reproduzíveis e evidências de conformidade.

## Conclusão

O AllasCode pode ser descrito matematicamente como uma arquitetura de transformações semânticas tipadas, coordenadas por contratos e evidências, cuja execução e distribuição procuram minimizar custos sem violar invariantes.

A teoria dos conjuntos delimita os domínios; a álgebra organiza composições; a lógica formal expressa contratos; a teoria da informação ajuda a analisar janelas e categorias; os sistemas de tipos restringem estados inválidos; a criptografia protege canais e dados; a teoria dos grafos representa a distribuição; a teoria das filas explica saturação; e a otimização orienta a alocação de recursos.

A DSL semântica acrescenta uma consequência central: em vez de pedir a cada modelo que reinvente uma implementação a partir de linguagem natural, o sistema pode validar uma definição canônica e compilá-la para backends específicos. Isso favorece determinismo, reuso e redução de decisões livres, além de poder diminuir tokens, custo e alucinações quando a definição é reutilizada e o compilador é confiável.

O objetivo não é tornar toda interpretação probabilística impossível. É delimitar onde a interpretação é necessária e onde a execução deve ser determinística, tipada, testável e reproduzível.
