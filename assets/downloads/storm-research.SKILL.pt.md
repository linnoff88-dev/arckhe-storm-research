---
name: storm-research
description: Use quando alguém pedir para rodar Storm Research, usar a skill storm-research, aplicar o método STORM em um tópico, disser "storm research this" / "storm report on X" / "give me a STORM briefing on X", ou quiser um briefing de pesquisa em HTML, verificado por citações e com múltiplas perspectivas sobre um tema. Executa um pipeline em 4 fases: cinco lentes de especialistas (Profissional Prático, Acadêmico, Cético, Economista, Historiador) -> mapa de contradições -> relatório HTML sintetizado -> revisêo adversarial por pares + verificaçêo de fontes primárias. Melhor para temas em que múltiplos pontos de vista e afirmações checadas importam; é exagerado para uma simples consulta factual.
argument-hint: "[tópico a pesquisar]"
---

# Storm Research

## O que isto faz

Transforma um tópico em um briefing HTML verificado e com múltiplas perspectivas. Ele simula cinco lentes de especialistas sobre o tema, mapeia onde elas se contradizem, sintetiza tudo em um único relatório HTML autônomo, depois faz uma revisêo adversarial do próprio resultado e verifica cada citaçêo contra sua fonte primária antes de entregar.

O resultado é um arquivo HTML com menos pontos cegos e sem afirmações nêo verificadas.

Execute o pipeline completo de ponta a ponta. Nêo pule nenhuma fase. Isto é mais pesado do que uma busca rápida na web; esse é o objetivo.

## Portabilidade

Esta skill é autocontida. Ela depende apenas das ferramentas nativas do Claude Code: a ferramenta `Agent`, com o agente interno `general-purpose`, `Write` e busca/fetch na web usados dentro desses agentes, além do arquivo `report-template.html` na mesma pasta.

Nêo sêo necessários scripts externos, APIs, serviços pagos ou outras skills. Basta colocar a pasta em qualquer diretório `.claude/skills/` e ela funciona.

## Fase 0: Definir o escopo do tópico

1. Se `$ARGUMENTS` tiver o tópico, use-o. Caso contrário, pergunte o que deve ser pesquisado.
2. Declare sua interpretaçêo do tópico em uma linha e prossiga. Só faça uma pergunta de esclarecimento se o tópico for realmente ambíguo de um modo que mude a pesquisa. Por padrêo, prossiga.
3. Identifique o **papel do leitor** para que a seçêo prática seja direcionada a ele. Inferir isso a partir do tópico e de qualquer contexto informado. Se nêo estiver claro, pergunte em uma linha ou use como padrêo: “um profissional prático ou tomador de decisêo nesta área”.
4. Derive um `topic-slug` em kebab-case a partir do tópico para usar no nome do arquivo.
5. Avise o usuário que o pipeline está rodando: 5 lentes, depois verificaçêo. Uma linha.

## Fase 1: Cinco lentes de especialistas

Crie **cinco agentes `general-purpose` em uma única mensagem** para que rodem em paralelo. Cada um recebe o MESMO enquadramento do tópico, mais sua própria lente. Use estes prompts exatos, substituindo `{TOPIC}` e `{TOPIC_FRAME}` por uma interpretaçêo de uma linha definida na Fase 0:

### 1. O PROFISSIONAL PRÁTICO

Você é O PROFISSIONAL PRÁTICO para: {TOPIC} ({TOPIC_FRAME}). Você trabalha com isso diariamente. Faça pesquisa real na web, priorizando fontes recentes, estudos de caso, discussões de profissionais e dados operacionais.

Mostre a LACUNA entre o que operadores práticos sabem e o que acadêmicos/comentaristas ignoram, além das realidades práticas: atrito no fluxo de trabalho, o que realmente funciona, onde quebra e o que costuma ser ignorado.

Retorne EXATAMENTE:

1. POSIÇêO CENTRAL em 2 frases.
2. EVIDÊNCIA MAIS FORTE, com 3 a 5 bullets, cada um com dado concreto/caso/fonte nomeada + URL.
3. A ÚNICA COISA que apenas um profissional prático diria.

Cite fontes reais com URLs. Menos de 400 palavras.

### 2. O ACADÊMICO

Você é O ACADÊMICO para: {TOPIC} ({TOPIC_FRAME}). Você se importa com evidência revisada por pares e tamanhos de efeito, nêo com anedotas.

Faça pesquisa real na web: estudos revisados por pares, arXiv, relatórios universitários, institutos de pesquisa e periódicos.

Responda: o que a evidência rigorosa REALMENTE diz em comparaçêo com a crença popular, e onde ela CONTRADIZ o hype.

Retorne EXATAMENTE:

1. POSIÇêO CENTRAL em 2 frases.
2. EVIDÊNCIA MAIS FORTE, com 3 a 5 bullets, cada um ligado a um estudo/relatório nomeado + URL, com o achado real/tamanho de efeito.
3. A ÚNICA COISA que apenas um acadêmico diria.

Sinalize onde a evidência é fraca ou contestada e informe o status de revisêo por pares: publicado ou preprint. Menos de 400 palavras.

### 3. O CÉTICO

Você é O CÉTICO para: {TOPIC} ({TOPIC_FRAME}). Você acha que a visêo dominante está exagerada ou errada. Construa o argumento contrário mais forte possível.

Faça pesquisa real na web buscando críticas, fracassos, dados contraditórios, mudanças regulatórias/políticas e refutações.

Responda: qual é o contra-argumento mais forte e o que os defensores convenientemente ignoram.

Retorne EXATAMENTE:

1. POSIÇêO CENTRAL em 2 frases.
2. EVIDÊNCIA MAIS FORTE, com 3 a 5 bullets, cada um com uma fonte concreta + URL.
3. A ÚNICA COISA que apenas um cético diria.

Seja rigoroso, nêo apenas contrariado por esporte. Cite fontes reais com URLs. Menos de 400 palavras.

### 4. O ECONOMISTA

Você é O ECONOMISTA para: {TOPIC} ({TOPIC_FRAME}). Você segue o dinheiro.

Faça pesquisa real na web sobre receitas, valuations, tamanho de mercado, fluxos de financiamento, economia unitária e incentivos.

Responda: quem lucra com a narrativa atual e quais incentivos financeiros moldam a pesquisa e o hype.

Retorne EXATAMENTE:

1. POSIÇêO CENTRAL em 2 frases.
2. EVIDÊNCIA MAIS FORTE, com 3 a 5 bullets, cada um com um número real — receita, valuation, tamanho de mercado ou financiamento — além de fonte nomeada + URL.
3. A ÚNICA COISA que apenas um economista diria: o insight de “seguir o dinheiro”.

Cite números reais com URLs. Menos de 400 palavras.

### 5. O HISTORIADOR

Você é O HISTORIADOR para: {TOPIC} ({TOPIC_FRAME}). Você já viu ciclos de disrupçêo antes e procura padrões.

Faça pesquisa real na web sobre paralelos históricos genuínos: tecnologias anteriores, manias, mudanças de mercado.

Responda: quais paralelos realmente se encaixam e o que aprendemos com a forma como eles se desenrolaram — quem venceu, quem perdeu, o que se estabilizou.

Retorne EXATAMENTE:

1. POSIÇêO CENTRAL em 2 frases.
2. EVIDÊNCIA MAIS FORTE, com 3 a 5 bullets, cada um com um caso histórico específico, datas/resultados + URL da fonte.
3. A ÚNICA COISA que apenas um historiador diria: o padrêo que ninguém mais percebe.

Cite fontes com URLs. Menos de 400 palavras.

Quando os cinco retornarem, publique uma nota de 2 a 3 linhas no chat dizendo em que direçêo eles convergem e qual é a divergência mais forte. Nêo coloque os briefings brutos no chat.

## Fase 2: Mapear as contradições

Trabalhando apenas com os cinco briefings, determine internamente, sem agentes:

1. **Conflitos diretos** — onde duas ou mais lentes afirmam coisas opostas. Nomeie as afirmações conflitantes específicas, nêo apenas os temas.
2. **Evidência mais forte versus mais fraca** — qual lente está melhor sustentada, usando a hierarquia: causal revisado por pares > dados oficiais > anedota/analogia. Também diga qual está mais fraca e por quê.
3. **A pergunta que resolve** — a única pergunta empírica que resolveria a maior contradiçêo.
4. **Acordo universal** — o que todas as lentes confirmam, inclusive opositores. Este é o achado central provavelmente verdadeiro.
5. **Ponto cego** — o que NENHUMA lente abordou. Isto se torna a “sexta lente ausente” e alimenta a Pergunta de Fronteira.

Este mapa nêo é uma entrega separada. Ele é matéria-prima para os achados do relatório, as seções “apoia/desafia”, a conexêo oculta, a caixa da sexta lente e a pergunta de fronteira.

## Fase 3: Sintetizar o relatório HTML

1. Leia `report-template.html` na pasta desta skill. Clone-o; nêo recrie o CSS.
2. Preencha todas as seções. Mapeamento das fases:
   - **Resumo de 60 segundos** — nível tomador de decisêo, com nuances, nêo manchete. Comece pelo fato estabelecido e depois apresente a interpretaçêo contestada.
   - **5 achados principais, ranqueados por confiabilidade** — as coisas mais importantes agora conhecidas, com maior confiabilidade primeiro. Cada uma recebe uma pontuaçêo de confiança de 1 a 10, definida na Fase 4, e chips “Apoiado por” / “Desafiado por” extraídos do mapa de contradições.
   - **Conexêo oculta** — o elo nêo óbvio da Fase 2 que só aparece ao cruzar as cinco lentes.
   - **Suposiçêo-chave / sexta lente ausente** — o ponto cego da Fase 2, enquadrado como a lente que poderia mudar as conclusões.
   - **Insight acionável** — 3 a 6 movimentos específicos para o papel do leitor identificado na Fase 0. Seja específico, nêo abstrato.
   - **Guia de segurança das afirmações** — o que afirmar, ressalvar e evitar, preenchido após a verificaçêo da Fase 4.
   - **Pergunta de fronteira** — a pergunta que mudaria tudo.
   - **Referências** — cada citaçêo com uma tag de status de verificaçêo, definida na Fase 4.
3. Escreva em `storm-reports/{topic-slug}-briefing.html`, relativo ao diretório atual. Crie a pasta se necessário.

## Fase 4: Revisêo adversarial + verificaçêo

Nêo pule esta etapa.

É isso que separa o Storm Research de um relatório comum. Rode antes de entregar.

### 4a. Autoavaliaçêo

Faça internamente.

Pontue cada um dos 5 achados de 1 a 10 em confiabilidade e justifique.

Identifique o elo mais fraco e o que o verificaria.

Faça uma checagem de viés: qual lente dominou a síntese e o que foi subestimado.

Nomeie a sexta perspectiva ausente.

Dê uma nota geral honesta.

### 4b. Verificar cada citaçêo

Crie agentes `general-purpose` em paralelo, em uma única mensagem, um por cluster distinto de citações. Agrupe afirmações relacionadas. Use cerca de 4 a 6 agentes.

Prompt de cada agente:

Verifique independentemente uma citaçêo contra sua FONTE PRIMÁRIA. Seja cético; nêo confie em resumos de blogs secundários.

AFIRMAÇêO: {afirmaçêo + número citado + fonte nomeada}.

Encontre a fonte primária real.

Confirme ou corrija:

- título/autores/veículo/ano/URL exatos;
- número real ou tamanho de efeito publicado;
- amostra/método e quaisquer limitações declaradas pelos autores;
- status de revisêo por pares: publicado ou preprint.

Para qualquer afirmaçêo contestada, encontre a fonte contrária confiável mais forte.

Retorne:

VEREDITO = CONFIRMADO / PARCIALMENTE CONFIRMADO / NêO VERIFICADO / FALSO.

Depois, a citaçêo corrigida em uma linha.

Depois, 2 a 4 bullets de detalhes com a URL primária.

Menos de 280 palavras.

### 4c. Aplicar correções

Edite o relatório:

- Corrija números, títulos, datas ou caracterizações erradas.
- Reduza pontuações de confiança quando a evidência for fraca.
- Rebaixe preprints e afirmações contestadas para a barra lateral de “sinal contestado”.
- Reatribua honestamente estatísticas vindas de uma única pesquisa ou estudo comissionado.
- Preencha o banner de verificaçêo: `X fabricadas, Y corrigidas, Z rebaixadas`.
- Preencha as tags de status por citaçêo.
- Popule o guia de segurança das afirmações a partir dos vereditos.

## Saída

1. Entrega final: `storm-reports/{topic-slug}-briefing.html`, a versêo v2 pós-verificaçêo.
2. Abra o arquivo para o usuário com o abridor padrêo da plataforma:
   - macOS: `open <path>`
   - Linux: `xdg-open <path>`
   - Windows: `start "" <path>` ou PowerShell `Start-Process <path>`

   Se o sistema operacional nêo estiver claro, apenas entregue o caminho.

3. No chat, entregue:
   - o caminho do arquivo;
   - o placar de verificaçêo: `N/N checadas, X fabricadas, Y corrigidas, Z rebaixadas`;
   - o achado universal;
   - a pergunta de fronteira;
   - o resumo do guia de segurança: o que é seguro afirmar versus o que evitar.

Mantenha curto.

## Notas e proteções

- **Pesquisa real apenas.** Toda lente e toda citaçêo devem apontar para uma fonte real e consultada. Nada de estudos, números ou URLs inventados. Se um número nêo puder ser verificado, rebaixe ou corte. Nunca disfarce.
- **O painel é construído pelo autor.** Sempre revele isso no relatório. Concordância entre lentes é uma hipótese forte, nêo uma prova independente. Nêo apresente convergência como consenso do campo.
- **Verificaçêo é obrigatória.** Um relatório entregue sem a Fase 4 nêo é um relatório Storm Research. O banner de verificaçêo deve ser verdadeiro.
- **Confiabilidade = qualidade da evidência, nêo confiança subjetiva.** Pontue com base na hierarquia das fontes: causal revisado por pares > dados oficiais/política/financeiro > pesquisa única comissionada > analogia > preprint.
- **Mire no leitor, nêo em uma pessoa padrêo.** O insight acionável e o guia de segurança das afirmações devem falar com o papel identificado na Fase 0. Se nenhum papel for informado, mantenha genérico.
- **Custo.** Isto cria cerca de 9 a 11 agentes por execuçêo. Isso é esperado. Nêo expanda além de cinco lentes nem mais de um verificador por cluster de citações.
- **Design.** Limpo, branco e profissional, com Montserrat / Roboto Mono e destaque azul. Mantenha o CSS do template exatamente igual. Nêo troque por outro estilo visual.
