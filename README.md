# Defusadores de Problema 🎯

> **Mais do que descobrir por que você perdeu um round, queremos ajudar você a reconhecer o que observar e como decidir melhor no próximo.**

**Projeto:** Hackathon de VALORANT — SEMCOMP / UNIVAVÁ 2026  
**Equipe:** Os Defusadores de Problema  
**Estado:** desenvolvimento de um protótipo de análise estratégica pós-partida.  
**Nome do produto:** em definição.

## 1. Visão geral

Nossa proposta é uma plataforma de **aprendizado de inteligência de jogo em VALORANT**, voltada a jogadores e equipes que desejam desenvolver tomada de decisão, leitura de round e coordenação coletiva.

Partimos da seguinte observação: um jogador pode saber que perdeu um round e até reconhecer que morreu, mas não necessariamente entender **quais informações estavam disponíveis, quais alternativas existiam e como reconhecer situações semelhantes no futuro**. Estatísticas comuns (KDA, headshots, dano) descrevem resultados; nossa proposta busca transformar acontecimentos da partida em **hipóteses estratégicas fundamentadas e feedbacks acionáveis**.

Em vez de uma recomendação genérica como “melhore a mira” ou uma ordem pouco explicativa como “você deveria ter rotacionado”, queremos ensinar *por que* uma alternativa mereceria consideração e *quais sinais* poderiam levar o jogador a identificá-la: vantagem numérica, eventos no kill feed, informação confirmada por aliados, contexto do plant, posicionamento e tempo do round.

A ferramenta não pretende substituir a análise humana ou afirmar que existe uma jogada perfeita para toda situação. Seu objetivo é apoiar a reflexão sobre escolhas, inclusive reconhecendo quando **não há evidência suficiente para concluir que houve um erro**.

## 2. O problema que queremos resolver

Jogadores intermediários e times enfrentam dificuldades para:

- Identificar **momentos-chave** nos quais o estado de um round mudou de maneira relevante.
- Entender como **decisões individuais se relacionam ao funcionamento do time**.
- Diferenciar uma decisão potencialmente ruim de uma boa decisão com resultado desfavorável.
- Transformar uma análise pós-partida em uma habilidade aplicável durante futuras partidas.
- Aprender a perceber sinais antes de decidir — não apenas receber uma crítica depois do resultado.

Nossa hipótese de produto é que um coaching guiado, baseado em dados reais e explicações contextualizadas, pode estimular essa leitura estratégica.

## 3. Nossa solução

A plataforma analisará dados oficiais de partidas disponibilizados pelo hackathon para oferecer uma experiência de **revisão estratégica pós-partida**, individual e coletiva.

O fluxo pretendido é:

1. **Ler a partida:** processar os arquivos JSON e organizar jogadores, rounds, eliminações, economia, plant/defuse e posições observadas.
2. **Reconstruir estados:** representar o estado conhecido do round em instantes relevantes, como após uma eliminação ou um plant.
3. **Estimar probabilidades ao longo dos rounds:** analisar individualmente os estados observados de cada round, utilizando uma rede bayesiana para estimar a probabilidade de vitória da equipe em diferentes instantes.
4. **Identificar momentos-chave:** localizar situações em que a equipe apresentou uma probabilidade elevada de vitória em um round perdido ou uma probabilidade baixa de vitória em um round vencido.
5. **Explorar cenários alternativos:** modificar variáveis de entrada da rede para investigar como diferentes condições se relacionam com a probabilidade estimada de vitória, respeitando as limitações das evidências disponíveis.
6. **Consolidar a análise da partida:** reunir os resultados de todos os rounds para identificar padrões recorrentes, pontos fortes, oportunidades de melhoria e comportamentos estratégicos individuais e coletivos.
7. **Explicar e ensinar:** transformar as análises em feedbacks contextualizados que mostrem quais decisões poderiam ser consideradas e como reconhecer situações semelhantes em partidas futuras.
8. **Engajar o jogador:** propor desafios de leitura de round e acompanhar seu progresso de aprendizado.

No hackathon, o objetivo é demonstrar esse fluxo com **casos reais extraídos da base**, processamento verificável e uma interface prototipada. A experiência visual poderá ser desenvolvida no **Axure RP**; ela não precisa estar conectada a um backend completo para demonstrar o conceito, mas qualquer análise apresentada como real deve ter sido efetivamente calculada e conferida.

## 4. Por que uma rede bayesiana — e não somente uma LLM?

Uma LLM é adequada para **gerar linguagem natural, organizar explicações e adaptar o feedback ao usuário**. Mas, se receber dados desestruturados e for solicitada a “inventar” probabilidades ou inferir causalidade sem um modelo estatístico, poderá produzir números e interpretações sem sustentação.

Escolhemos estudar uma **rede bayesiana** como núcleo probabilístico porque ela permite:

- Representar explicitamente variáveis do jogo como **nós** (por exemplo, vantagem numérica, lado de ataque/defesa, tempo e spike plantada).
- Descrever dependências entre variáveis em um **grafo**.
- Estimar e consultar probabilidades condicionais usando **tabelas de probabilidade condicional (CPTs)**.
- Deixar rastreável quais **evidências** participaram de uma estimativa.
- Atualizar inferências quando muda o estado observado ou o conjunto de evidências.
- Comunicar incerteza e trabalhar com conhecimento inicial explicitamente justificado quando os dados forem escassos.

**Importante:** a rede bayesiana **não elimina automaticamente erros, vieses ou alucinações**. Sua confiabilidade depende da qualidade dos dados, da escolha dos nós, da estrutura do grafo, da estimativa das CPTs e da validação. Também **não prova causalidade por padrão**.

### Papel de cada componente

| Componente | Responsabilidade | O que não deve fazer |
|---|---|---|
| Processamento determinístico (Python) | Extrair eventos, calcular contagens e montar estados verificáveis | Inventar eventos ausentes |
| Rede bayesiana | Estimar probabilidades condicionais para estados definidos | Tratar associações como efeitos causais comprovados |
| Regras e revisão tática | Filtrar cenários plausíveis, validar hipóteses e limitações | Julgar decisões sem contexto suficiente |
| LLM (opcional) | Traduzir os resultados estruturados em feedback claro e pedagógico | Criar números, eventos ou certezas não fornecidos pelo modelo |
| Protótipo | Demonstrar a jornada e os resultados ao jogador | Apresentar funcionalidades futuras como já implementadas |

Em resumo: **os dados e o modelo fornecem evidências e estimativas; a camada de linguagem explica sem inventar a análise.**

## 5. Como funciona a comparação de cenários

Queremos localizar situações em que o jogador poderia investigar alternativas e visualizar como certas condições se relacionam à chance estimada de vitória.

Exemplo *hipotético*:

- **Estado observado:** defesa, 3 aliados vivos contra 4 adversários, spike não plantada, determinado tempo restante.
- **Pergunta:** como a probabilidade estimada de vitória se compara à de estados semelhantes com 4 aliados contra 4 adversários, mantendo evidências comparáveis?
- **Saída:** o modelo consulta `P(vitória | evidências)` em cada cenário e apresenta a diferença **em pontos percentuais**, junto de sua fonte, limitações e confiabilidade.

Não vamos simplesmente alterar “pesos” até conseguir uma porcentagem desejada. Os parâmetros da rede devem ser **estimados a partir de dados ou definidos como hipóteses iniciais justificadas**, e então testados/validados. O usuário altera **condições de um cenário para consulta**, não as probabilidades aprendidas arbitrariamente.

### Identificação de momentos-chave pela probabilidade de vitória

A rede bayesiana será utilizada para estimar a probabilidade de vitória da equipe em diferentes estados de cada round.

Nosso objetivo não é analisar apenas o resultado final, mas identificar situações em que a equipe esteve em uma condição favorável e perdeu, ou em uma condição desfavorável e venceu.

Inicialmente, utilizaremos dois critérios de identificação:

**1. Rounds perdidos — oportunidades de vitória desperdiçadas**

Para cada round perdido, identificaremos o instante observado em que a equipe apresentou a maior probabilidade estimada de vitória.

Esse instante representa uma oportunidade para investigar como a partida evoluiu de uma situação favorável até a derrota.

A plataforma poderá explorar cenários alternativos para investigar quais fatores estiveram associados à redução das chances de vitória e quais decisões estratégicas merecem revisão.

**2. Rounds vencidos — situações de risco superadas**

Para cada round vencido, identificaremos o instante observado em que a equipe apresentou a menor probabilidade estimada de vitória.

Esse instante permite investigar situações em que a equipe esteve próxima de perder o controle do round, mesmo que tenha conseguido vencer posteriormente.

O objetivo é identificar comportamentos de risco ou dificuldades recorrentes que poderiam comprometer partidas futuras.

**Comparação de cenários**

Após identificar esses momentos, o sistema permitirá modificar condições do estado analisado, como vantagem numérica, situação econômica ou estado da spike, para consultar como a probabilidade estimada de vitória varia.

Essas comparações servirão para formular hipóteses estratégicas e orientar o aprendizado, não para afirmar automaticamente que uma alteração específica teria causado uma vitória.

As situações identificadas deverão ser acompanhadas do instante analisado, das evidências disponíveis e das limitações da estimativa.

### Associação não é causalidade

Se um cenário A apresenta 35% de vitória e um cenário B apresenta 55%, isso **não permite afirmar**, por si só, que uma ação faria a chance subir 20 pontos percentuais. Pode haver fatores não observados, e o cenário B talvez nem corresponda a uma decisão realmente disponível ao jogador.

Nossa linguagem preferida será:

- **Quando existe apenas associação:** “Em estados semelhantes, o modelo estima uma chance de vitória maior quando [...]. Isso sugere um ponto para revisão.”
- **Quando existe hipótese tática plausível:** “Se você tivesse preservado a oportunidade de troca, *poderia* ter mantido uma situação numericamente mais favorável; vale observar [...] em futuras partidas.”
- **Quando não há evidência suficiente:** “Os dados disponíveis não permitem afirmar que uma decisão diferente aumentaria suas chances de vitória.”

A formulação “**se você fizesse X, poderia aumentar a chance de vitória**” é uma **hipótese pedagógica**, que só deve incluir um percentual específico se o tipo de comparação e suas limitações forem explicitados. Para reivindicar efeitos causais, seria necessário um modelo causal com premissas adicionais e validação apropriada.

## 6. Como será o feedback ao jogador

### Feedback baseado na evolução da probabilidade de vitória

A plataforma não terá como objetivo apenas informar a chance estimada de vitória em um determinado instante. Queremos utilizar a evolução dessa probabilidade para orientar a investigação de decisões estratégicas.

Nos rounds perdidos, o feedback poderá destacar situações em que a equipe esteve em uma condição favorável, explicando quais acontecimentos posteriores contribuíram para a mudança do estado do jogo.

Nos rounds vencidos, o feedback poderá destacar situações de risco que foram superadas, permitindo que os jogadores aprendam mesmo após resultados positivos.

Após selecionar um momento relevante, a plataforma apresentará as informações observadas, as condições que influenciaram a estimativa e cenários alternativos que merecem consideração.

O feedback seguirá uma estrutura pedagógica:

1. **O que aconteceu?** Apresentar o instante e o contexto observado.
2. **Como estava sua chance de vitória?** Exibir a estimativa da rede e sua evolução.
3. **Qual condição merece atenção?** Identificar variáveis e acontecimentos associados à situação.
4. **O que poderia ser diferente?** Comparar cenários alternativos plausíveis.
5. **Como reconhecer isso no futuro?** Ensinar quais informações o jogador deve observar para orientar suas decisões.

Nosso objetivo é permitir feedbacks como:

"Neste momento, sua equipe estava em vantagem numérica e apresentava uma probabilidade estimada favorável de vitória. Após a perda dessa vantagem, o modelo passou a estimar uma condição menos favorável. Preservar a possibilidade de apoio entre companheiros poderia ter ajudado a manter essa vantagem. Em situações semelhantes, observe o posicionamento do time antes de buscar um confronto isolado."

A plataforma sempre distinguirá as evidências observadas, as estimativas probabilísticas e as hipóteses estratégicas. Uma variação na probabilidade estimada não será apresentada, isoladamente, como prova de que determinada ação teria causado um resultado diferente.

## 7. Dados disponíveis e limites da análise

A base oficial é fornecida em **JSON** e inclui partidas do UNIVAVÁ. O arquivo de Haven usado como referência nesta fase contém informações como:

- Identificadores dos jogadores, equipes e agentes.
- Resultado de cada round e lado vencedor.
- Eliminações com assassino, vítima, assistências e tempo relativo ao round.
- Posições e orientações registradas **em determinados eventos**, além de posições de plant e defuse.
- Dados de economia, equipamentos, dano e contadores de uso de habilidades.

**Limitações importantes:** os eventos não formam uma gravação contínua de tudo que ocorreu no round. Não há registro confiável de comunicação de voz, atenção ao minimapa ou todos os movimentos em tempo contínuo. O arquivo de referência não oferece uma linha do tempo detalhada de uso/efeito das habilidades. Portanto, não podemos afirmar que uma pessoa “não olhou o mapa”, “ignorou uma call” ou “deveria ter rotacionado naquele segundo” só por observar uma morte.

### Regras do hackathon

- Usar **exclusivamente os JSONs fornecidos pela organização** como fonte de dados de partidas.
- Não coletar dados adicionais de serviços externos nem utilizar scraping ou integrações não autorizadas com o cliente da Riot.
- Focar em **análise pós-partida**, e não assistência em tempo real durante o jogo.
- Respeitar direitos autorais de vídeos, imagens e recursos visuais utilizados.

Vídeos de análise tática (como os estudados pela equipe) servem para **desenvolver critérios e repertório estratégico**, não como fonte adicional de estatísticas de partidas para o modelo.

## 8. Experiência do usuário e gamificação caso sobre tempo

O protótipo deverá comunicar tanto **análise individual** quanto **aprendizado coletivo**:

- **Revisão de partida:** selecionar round e navegar pela linha do tempo de momentos-chave.
- **Mapa tático:** observar os snapshots registrados, com equipes, posições e acontecimentos.
- **Treinamento de decisão:** antes de receber a explicação, o jogador escolhe o que faria e justifica seu raciocínio.
- **Feedback transparente:** exibir evidências, probabilidades (quando suficientemente validadas), hipótese, sinais de leitura e limitações.
- **Perfil de evolução:** separar métricas **observadas de desempenho** de **progresso de aprendizado** (desafios concluídos, temas revisados, XP).
- **Times:** criar ou integrar grupos, acompanhar padrões coletivos e discutir situações em conjunto.

Uma pontuação de “inteligência estratégica” não pode ser inventada por uma LLM. Qualquer skill quantitativa precisa ter **fórmula e dados de origem explicáveis**. O XP por atividades de aprendizado é distinto de uma estimativa de habilidade real.

## 9. O que será demonstrado no hackathon

A entrega principal do evento é um **vídeo pitch de até 5 minutos**, com link para os slides; o protótipo navegável é complementar. Nosso objetivo é demonstrar:

1. Um problema real de leitura estratégica em VALORANT.
2. Um round de campeonato como estudo de caso, contextualizado visualmente.
3. O processamento que gera estados reais verificáveis a partir do JSON oficial.
4. Uma estimativa probabilística **quando houver base suficiente**, deixando claro seu grau de confiabilidade.
5. A mesma situação na interface prototipada, mostrando como o jogador aprende.
6. O plano de evolução para análises mais sofisticadas, rede melhor calibrada e funcionalidades coletivas.

O pitch deve comunicar **o que já funciona**, **o que é protótipo visual** e **o que é visão futura**. Não precisamos exibir longos blocos de código: podemos mostrar uma execução breve do processamento, um evento real visualizado e sua representação no protótipo.

### Critérios de julgamento a atender

Os seis critérios oficiais possuem **peso igual** (1 a 5 pontos cada): inovação; impacto em jogadores e equipes; qualidade da análise e uso dos dados; viabilidade; usabilidade e experiência; qualidade do protótipo e apresentação.

**Prazo oficial de entrega:** 23 de outubro de 2026, às 23h59 (horário de Brasília).

## 10. Princípios que não podemos abandonar

1. **Primeiro os dados, depois as hipóteses.** Nenhum feedback pode contradizer a evidência registrada.
2. **Não confundir resultado com qualidade da decisão.** É possível jogar bem e perder, ou jogar mal e ganhar.
3. **Não confundir associação com causalidade.** Cenários alternativos são estimativas condicionais até que haja justificativa causal adicional.
4. **Não inventar números.** Toda porcentagem apresentada como real deve vir de um cálculo rastreável e validado.
5. **Explicar como perceber, não apenas o que fazer.** O aprendizado precisa ser transferível para o próximo round.
6. **Admitir incerteza.** Quando faltam dados, o sistema deve dizer isso.
7. **Manter a análise pós-partida.** Nada de vantagem em tempo real.
8. **Ser útil ao indivíduo e ao time.** Coordenação importa tanto quanto estatística individual.
