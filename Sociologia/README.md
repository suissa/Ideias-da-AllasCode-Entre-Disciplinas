# Sociologia da Arquitetura AllasCode

> Como a sociologia ajuda a compreender uma arquitetura de software baseada em agentes autônomos, mensagens, eventos, coreografia, contratos explícitos e tipos semânticos.

## Sumário

- [1. Introdução: software como organização social](#1-introdução-software-como-organização-social)
- [2. Durkheim: divisão social do trabalho e solidariedade orgânica](#2-durkheim-divisão-social-do-trabalho-e-solidariedade-orgânica)
- [3. Weber: autoridade hierárquica versus coordenação distribuída](#3-weber-autoridade-hierárquica-versus-coordenação-distribuída)
- [4. Luhmann: comunicação, autonomia e sistemas](#4-luhmann-comunicação-autonomia-e-sistemas)
- [5. Castells: arquitetura como sociedade em rede](#5-castells-arquitetura-como-sociedade-em-rede)
- [6. Blau: troca social, contribuição e interdependência](#6-blau-troca-social-contribuição-e-interdependência)
- [7. Tipos semânticos: CPF não é apenas uma string](#7-tipos-semânticos-cpf-não-é-apenas-uma-string)
- [8. Berger e Luckmann: classificações e significados compartilhados](#8-berger-e-luckmann-classificações-e-significados-compartilhados)
- [9. Comparação entre os modelos arquiteturais](#9-comparação-entre-os-modelos-arquiteturais)
- [10. Implicações para a AllasCode](#10-implicações-para-a-allascode)
- [11. Limites das analogias sociológicas](#11-limites-das-analogias-sociológicas)
- [12. Síntese: princípios sociológicos da arquitetura](#12-síntese-princípios-sociológicos-da-arquitetura)
- [Referências conceituais](#referências-conceituais)

## 1. Introdução: software como organização social

Uma pergunta central da sociologia é: como indivíduos e grupos relativamente autônomos conseguem cooperar, coordenar suas ações e produzir resultados coletivos sem depender de uma autoridade central que controle cada movimento?

Essa pergunta oferece uma lente útil para compreender a arquitetura AllasCode, organizada em torno de agentes especializados, mensagens, eventos, contratos e fluxos coreografados.

A analogia não afirma que agentes de software sejam pessoas ou sociedades humanas. Ela compara estruturas de coordenação: quem decide, como as responsabilidades são distribuídas, como as contribuições se conectam e como o comportamento coletivo emerge da interação entre partes.

A arquitetura baseada em chamadas de funções pode concentrar em um componente o conhecimento sobre a sequência de execução. Em uma arquitetura coreografada, agentes podem reagir a eventos relevantes, realizar seu trabalho e publicar resultados sem conhecer todos os consumidores ou o fluxo global.

A tese central desta explicação é:

**A AllasCode pode ser entendida como uma organização de capacidades especializadas: cada agente domina uma responsabilidade delimitada, comunica resultados por contratos explícitos e participa de um comportamento coletivo sem depender de um controlador universal.**

Essa autonomia exige coordenação. Contratos, políticas de segurança, observabilidade, tratamento de falhas e mecanismos de recuperação são equivalentes funcionais — não literais — às regras e instituições que tornam possível a cooperação organizada.

## 2. Durkheim: divisão social do trabalho e solidariedade orgânica

Émile Durkheim, em *Da divisão do trabalho social*, distinguiu duas formas de solidariedade:

- **Solidariedade mecânica:** a coesão social depende principalmente das semelhanças entre os membros.
- **Solidariedade orgânica:** a coesão decorre da diferenciação e da interdependência entre membros que desempenham funções distintas.

A solidariedade orgânica é uma analogia especialmente útil para sistemas compostos por agentes especializados.

Em uma sociedade complexa, um profissional não precisa dominar o trabalho de todos os demais. Precisa executar sua função, respeitar normas compartilhadas e entregar contribuições das quais outras atividades possam depender. De forma semelhante, um agente de cobrança não precisa conhecer a implementação de um agente de notificação. Precisa processar corretamente a responsabilidade que lhe cabe e publicar o resultado previsto no contrato.

### 2.1 Especialização sem conhecimento total

Um agente pode:

1. Receber os dados de que necessita.
2. Validar a mensagem e o contexto de acordo com seu domínio.
3. Executar seu comportamento especializado.
4. Publicar um resultado ou evento contratualmente definido.
5. Encerrar sua participação sem precisar conhecer todos os consumidores futuros.

O sistema coletivo passa a depender da conexão entre contribuições especializadas, não de uma única unidade que implemente cada detalhe.

### 2.2 Interdependência não é independência absoluta

A solidariedade orgânica não significa que cada parte seja autossuficiente. As partes são diferentes precisamente porque dependem de contribuições complementares.

Na arquitetura, contratos incompatíveis, eventos perdidos, consumidores ausentes ou estados inconsistentes podem interromper essa cooperação. Por isso, a autonomia precisa vir acompanhada de contratos versionados, garantias de entrega apropriadas, idempotência, observabilidade e estratégias de recuperação.

**Implicação para a AllasCode:** a especialização dos agentes pode reduzir acoplamento de implementação, enquanto a interdependência explícita permite que o sistema produza resultados coletivos.

## 3. Weber: autoridade hierárquica versus coordenação distribuída

Max Weber estudou autoridade, dominação legítima e burocracia. Seus conceitos ajudam a analisar arquiteturas em que um componente concentra decisões e controla a sequência de execução.

### 3.1 Fluxo centralmente coordenado

Em um fluxo imperativo típico:

1. O componente A chama B.
2. B retorna um resultado.
3. A interpreta o resultado e chama C.
4. A decide a próxima etapa e mantém o controle da sequência.

Esse desenho pode ser apropriado quando a sequência precisa ser explícita e local. A analogia com hierarquia fica mais forte quando um coordenador concentra decisões e os componentes subordinados não podem escolher suas próprias reações.

### 3.2 Coreografia orientada a eventos

Em um fluxo coreografado:

1. A publica um evento sobre um fato ocorrido.
2. B reage porque aquele evento pertence à sua responsabilidade.
3. B realiza seu trabalho e publica outro evento.
4. C reage ao novo evento se o contrato e as regras do domínio determinarem que deve fazê-lo.

A sequência global não precisa estar codificada em um único componente. A coordenação pode surgir das regras e dos contratos que conectam as operações.

**Diferença fundamental:** coordenar por invocações explícitas não é o mesmo que coordenar por eventos, contratos e regras compartilhadas.

Isso não torna toda chamada de função uma forma de dominação, nem torna todo sistema orientado a eventos descentralizado. Um orquestrador também pode emitir todos os comandos e controlar integralmente os agentes. A descentralização depende da distribuição efetiva de decisões e responsabilidades.

### 3.3 Autonomia com governança

A autonomia não exige que todos os agentes tenham os mesmos poderes. Um agente que autoriza pagamentos deve ter competências diferentes de um agente que envia notificações. O segundo não deve poder contornar a autorização do primeiro.

A igualdade relevante é a autonomia dentro de uma responsabilidade delimitada, não a igualdade irrestrita de autoridade.

**Implicação para a AllasCode:** descentralizar a execução não significa remover políticas, permissões ou pontos legítimos de decisão. Significa evitar que um controlador universal precise decidir cada passo do comportamento do sistema.

## 4. Luhmann: comunicação, autonomia e sistemas

Niklas Luhmann colocou a comunicação no centro de sua teoria dos sistemas sociais. Em sua abordagem, um sistema social se reproduz por conexões entre comunicações, sem depender de uma única consciência que conheça e controle todas as partes.

A transposição para software é uma analogia, não uma equivalência literal: agentes não são sistemas sociais humanos. Ainda assim, a ideia de que operações podem se conectar sem conhecimento total oferece uma perspectiva útil.

### 4.1 Um agente não precisa conhecer o todo

Considere um processo de entrega:

- Um agente recebe a solicitação.
- Outro valida os dados necessários.
- Outro identifica um entregador elegível.
- Outro acompanha o estado da entrega.
- Outro processa o pagamento segundo as regras aplicáveis.

O agente que identifica o entregador não precisa implementar o pagamento nem conhecer o funcionamento interno da notificação. Precisa reconhecer a entrada pertinente, aplicar suas regras e produzir uma saída contratualmente válida.

### 4.2 Acoplamento por contratos

Sistemas relativamente independentes podem interagir por interfaces, mensagens e eventos definidos. Esse acoplamento permite mudanças internas sem obrigar todos os participantes a conhecer a implementação uns dos outros.

A autonomia, porém, depende de condições concretas: contratos compreensíveis, compatibilidade de versões, correlação de eventos, tratamento de duplicatas, recuperação de falhas e rastreabilidade do fluxo.

### 4.3 Emergência do comportamento coletivo

O comportamento do conjunto pode resultar da conexão entre operações especializadas, em vez de estar inteiramente programado em um agente central.

Isso não significa que o sistema dispense desenho ou regras globais. Significa que nem toda regra de coordenação precisa ser transformada em uma sequência central de chamadas.

**Implicação para a AllasCode:** a inteligência operacional pode ser distribuída entre agentes, contratos e mecanismos de coordenação, preservando responsabilidades locais bem delimitadas.

## 5. Castells: arquitetura como sociedade em rede

Manuel Castells estudou redes de informação e a transformação das organizações por fluxos de comunicação. A ideia de sociedade em rede ajuda a pensar como os nós se conectam e como a estrutura das conexões afeta a autonomia.

Em uma arquitetura de agentes, cada participante pode conhecer apenas os canais, contratos e eventos pertinentes ao seu domínio, em vez de manter referências diretas a todos os demais componentes.

Isso pode favorecer:

- **Evolução local:** mudanças internas não exigem alterações em todos os consumidores, desde que os contratos continuem compatíveis.
- **Extensibilidade:** novos consumidores podem reagir a eventos existentes sem exigir que o produtor conheça todos eles.
- **Separação de responsabilidades:** cada agente mantém foco em seu domínio.
- **Flexibilidade topológica:** as relações entre participantes podem evoluir por meio de contratos e infraestrutura de mensagens.

Esses benefícios não são automáticos. Uma rede pode manter pontos de concentração, dependências rígidas ou componentes críticos. Um sistema centralizado também pode ser resiliente e modular.

**Implicação para a AllasCode:** a topologia das relações é uma decisão arquitetural relevante porque influencia o custo de mudança, o conhecimento compartilhado e os pontos de falha.

## 6. Blau: troca social, contribuição e interdependência

Peter Blau estudou troca social, poder e estrutura organizacional. Sua perspectiva permite analisar a cooperação como uma relação entre contribuições, expectativas e dependências.

Em uma organização, um participante entrega algo que outros podem utilizar para realizar suas próprias atividades. No software, a analogia aparece quando:

1. Um agente disponibiliza um resultado validado.
2. Outro utiliza esse resultado para cumprir sua responsabilidade.
3. Um terceiro produz uma nova contribuição.
4. O conjunto alcança um resultado que nenhuma parte precisaria produzir isoladamente.

A reciprocidade humana envolve interesses, normas e relações sociais que não devem ser atribuídos automaticamente a agentes de software. A comparação diz respeito à estrutura das contribuições, não a motivações humanas dos agentes.

A cooperação exige também regras sobre quem pode produzir, consumir ou autorizar determinados resultados. Nem todos os participantes precisam possuir as mesmas permissões.

**Implicação para a AllasCode:** a coordenação deve ser modelada em torno de contribuições verificáveis e responsabilidades claras, em vez de pressupor que um agente central precisa conhecer e controlar cada consumidor.

## 7. Tipos semânticos: CPF não é apenas uma string

Um dos pontos centrais da filosofia da AllasCode é que os dados devem representar conceitos do domínio, e não apenas o formato primitivo usado para armazená-los.

Um CPF pode ser representado como uma sequência de caracteres. Mas a representação não define, sozinha, as regras que tornam o valor válido para uma finalidade específica.

Uma sequência arbitrária de caracteres não se torna um CPF válido apenas por ser aceita como `string`.

### 7.1 Representação, significado e contrato

| Camada | Pergunta | Exemplo |
|---|---|---|
| Representação primitiva | Como o valor é armazenado? | `string` |
| Tipo semântico | Que conceito ele representa? | `CPF` |
| Regra de validação | Quais condições definem sua validade? | Formato e dígitos verificadores |
| Contrato de domínio | Em qual contexto e sob quais condições pode ser usado? | CPF validado para determinada operação |

O tipo primitivo responde: **como o valor é representado?**

O tipo semântico responde: **que conceito o valor representa e quais operações fazem sentido para ele?**

O contrato de domínio especifica as condições necessárias para o uso pretendido. Validade sintática, validade segundo dígitos verificadores, situação cadastral e autorização para uma operação são condições diferentes e não devem ser confundidas.

### 7.2 Exemplo em TypeScript

```typescript
type CPF = string & {
  readonly __brand: "CPF";
};

function parseCPF(input: string): CPF {
  if (!isValidCPF(input)) {
    throw new Error("CPF inválido");
  }

  return input as CPF;
}

function findCustomer(cpf: CPF) {
  // Recebe um valor previamente validado.
}
```

Nesse exemplo, `isValidCPF` é uma função ilustrativa que precisa ser implementada. O tipo marcado ajuda a impedir que uma `string` arbitrária seja passada diretamente onde se espera um CPF.

Para maior rigor, a criação do valor deve ser controlada por um módulo e a validação deve ocorrer nas fronteiras apropriadas. O tipo marcado do TypeScript é uma convenção de compilação, não uma garantia de segurança em tempo de execução.

### 7.3 Outros conceitos semânticos

- **Idade:** pode influenciar elegibilidade e restrições, não sendo apenas um número.
- **Endereço:** pode ter componentes, regras de normalização e critérios de interpretação.
- **Dinheiro:** envolve valor, moeda e regras de precisão; um número isolado pode ser insuficiente.
- **Identidade autenticada:** não é apenas um identificador; depende de evidências e procedimentos de verificação.
- **Evento de domínio:** não é apenas um objeto serializado; declara que algo relevante ocorreu segundo um contrato.

Tipos semânticos podem ser implementados sobre tipos primitivos. O objetivo não é abolir `string`, `number` ou `boolean`, mas impedir que sua generalidade apague as distinções importantes do domínio.

**Implicação para a AllasCode:** os contratos devem preservar significado e regras de uso, em vez de aceitar valores apenas porque são representáveis no formato primitivo esperado.

## 8. Berger e Luckmann: classificações e significados compartilhados

Peter L. Berger e Thomas Luckmann exploraram como classificações, convenções e instituições participam da construção de uma realidade social compartilhada.

A analogia para software é que as categorias de um domínio carregam significados que vão além da representação bruta. Dinheiro, identidade, endereço e autorização são reconhecidos por meio de convenções, regras e práticas específicas.

No software, a semântica precisa ser expressa de maneira verificável: nomes de tipos, invariantes, validadores, contratos e políticas. Ao contrário de uma convenção informal, uma regra codificada pode ser aplicada e testada automaticamente.

A analogia tem limites. Os tipos de software não são construídos socialmente da mesma forma que instituições humanas. O ponto comum é que a classificação define distinções importantes e orienta o uso dos valores.

**Implicação para a AllasCode:** a semântica compartilhada reduz ambiguidades entre agentes e torna os contratos mais expressivos, desde que as regras sejam explícitas e testáveis.

## 9. Comparação entre os modelos arquiteturais

| Dimensão | Coordenação centralizada | Coordenação coreografada |
|---|---|---|
| Fluxo de trabalho | Sequência controlada por um componente | Sequência distribuída por eventos e regras |
| Conhecimento | O coordenador conhece muitas etapas | Cada agente conhece o necessário para sua responsabilidade |
| Comunicação | Chamadas e retornos explícitos são comuns | Mensagens e eventos conectam operações |
| Mudança | Alterações no fluxo central podem afetar várias etapas | Mudanças internas podem ser isoladas por contratos compatíveis |
| Extensão | Novas capacidades podem exigir alterações no coordenador | Novos consumidores podem reagir a eventos existentes |
| Depuração | A sequência pode ser mais direta de seguir | É importante ter correlação, rastreamento e observabilidade distribuída |
| Falhas | O coordenador pode ser um ponto crítico, conforme o desenho | Mensagens, consumidores e estados distribuídos exigem recuperação explícita |
| Autoridade | Pode ficar concentrada no coordenador | Pode ser distribuída, mantendo autoridades específicas por domínio |

A tabela descreve tendências, não garantias. Chamadas de função não implicam necessariamente acoplamento excessivo, e eventos não garantem descentralização. A arquitetura precisa ser avaliada pela distribuição real das decisões, pelos contratos e pelas propriedades operacionais.

## 10. Implicações para a AllasCode

As analogias sociológicas ajudam a formular decisões de arquitetura concretas.

### 10.1 Agentes com responsabilidades delimitadas

Cada agente deve ter uma função compreensível e um conjunto explícito de entradas, saídas, permissões e invariantes. Isso reduz a necessidade de conhecimento global e facilita testes isolados.

### 10.2 Contratos como regras de cooperação

Mensagens e eventos precisam ter esquemas, semântica e regras de compatibilidade. Um evento deve expressar um fato ou uma intenção com significado definido, não apenas transportar dados sem contexto.

### 10.3 Coreografia sem ingenuidade operacional

A coreografia reduz a necessidade de um controlador universal, mas torna importantes questões como entrega de mensagens, duplicatas, ordenação, idempotência, timeouts, compensações, reprocessamento e observabilidade.

### 10.4 Autoridade distribuída conforme o domínio

Autonomia não significa que qualquer agente possa executar qualquer ação. Autorização, identidade, integridade e limites de domínio precisam ser impostos por mecanismos verificáveis.

### 10.5 Semântica como parte do contrato

Tipos de domínio e validadores devem tornar explícito o que os dados significam e quais condições precisam satisfazer. Isso permite que agentes diferentes interpretem os contratos de forma consistente.

### 10.6 O sistema coletivo como resultado da colaboração

A arquitetura deve tornar possível adicionar ou substituir capacidades sem exigir que um componente central conheça todas as implementações. Essa propriedade depende de contratos estáveis e de dependências bem delimitadas.

## 11. Limites das analogias sociológicas

As teorias sociológicas oferecem modelos conceituais, não provas de que uma arquitetura seja automaticamente mais rápida, segura ou escalável.

- Uma organização descentralizada pode ter coordenação ruim; o mesmo vale para um sistema distribuído.
- Uma hierarquia pode ser eficiente quando decisões sequenciais e autoridade explícita são necessárias.
- Uma arquitetura orientada a eventos pode manter um orquestrador central e, portanto, continuar centralizada.
- Agentes de software não possuem necessariamente consciência, interesses, direitos ou motivações como pessoas.
- Tipos semânticos melhoram a expressão do domínio, mas não substituem validação em tempo de execução, autorização, testes ou controles de segurança.
- Mais componentes não significam automaticamente maior modularidade; fronteiras mal definidas podem aumentar a complexidade.

O critério útil não é eliminar centralização a qualquer custo. É colocar coordenação, autoridade e conhecimento nos lugares em que sejam necessários, evitando dependências globais desnecessárias.

## 12. Síntese: princípios sociológicos da arquitetura

A perspectiva sociológica permite sintetizar a filosofia em quatro princípios.

### 12.1 Autonomia funcional

Cada agente deve ser responsável por uma capacidade delimitada, sem precisar conhecer a implementação de todos os demais.

### 12.2 Interdependência organizada

A autonomia não elimina dependências. Ela as torna explícitas por meio de mensagens, eventos, contratos e regras compartilhadas.

### 12.3 Coordenação sem controlador universal

O comportamento coletivo pode emergir da interação entre agentes especializados, sem exigir que um componente determine cada etapa. Isso não elimina governança, políticas de segurança ou autoridades específicas.

### 12.4 Semântica explícita

Os dados devem representar conceitos do domínio e respeitar suas invariantes, em vez de serem tratados como valores primitivos intercambiáveis.

A combinação é mais importante do que qualquer princípio isolado. Uma rede de agentes autônomos com contratos frágeis pode se tornar caótica. Tipos semânticos rigorosos em um sistema excessivamente acoplado não resolvem, sozinhos, os problemas de coordenação. O valor está em combinar autonomia, interdependência, governança e significado explícito.

**Conclusão:** a analogia sociológica mais forte para a AllasCode é a solidariedade orgânica de Durkheim, complementada pela teoria dos sistemas de Luhmann e pela sociedade em rede de Castells. Para os tipos semânticos, a analogia mais útil está nas classificações institucionais e nos significados compartilhados.

> **A AllasCode busca organizar o software como uma sociedade de capacidades especializadas: cada participante domina sua responsabilidade, comunica resultados segundo contratos explícitos e contribui para um comportamento coletivo sem depender de um controlador universal; os tipos semânticos definem o significado e os limites dos dados que circulam nessa rede.**

## Referências conceituais

- **Durkheim, Émile.** *De la division du travail social* (1893). Divisão do trabalho e solidariedade mecânica e orgânica.
- **Weber, Max.** *Wirtschaft und Gesellschaft* (publicado postumamente em 1922). Autoridade, dominação legítima e burocracia.
- **Luhmann, Niklas.** *Soziale Systeme: Grundriß einer allgemeinen Theorie* (1984). Teoria dos sistemas sociais, comunicação e reprodução operacional.
- **Castells, Manuel.** *The Rise of the Network Society* (1996). Redes, fluxos de informação e transformação organizacional.
- **Blau, Peter M.** *Exchange and Power in Social Life* (1964). Troca social, poder e estrutura social.
- **Berger, Peter L.; Luckmann, Thomas.** *The Social Construction of Reality* (1966). Conhecimento, classificação e construção social da realidade.

As referências são pontos de partida teóricos para a analogia. Elas não afirmam que esses autores tenham proposto diretamente arquiteturas de software orientadas a agentes ou eventos.
