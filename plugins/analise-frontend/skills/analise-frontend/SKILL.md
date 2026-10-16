---
name: analise-frontend
description: Analisa defeitos em frontends React componentizados acompanhando os controles usados, seus identificadores, Console e Network no navegador, e registra achados em Markdown sem alterar código.
---

# Análise de erros de frontend React

Investigue o comportamento relatado em um frontend React pela interface em execução no navegador, acompanhando os componentes e controles usados no fluxo. O resultado é um relatório Markdown baseado em evidências. **Não altere código em nenhuma etapa.**

## Limites

- Interaja com a aplicação pelo navegador: siga os passos do usuário, clique nos controles relevantes e observe o resultado visual.
- Use as ferramentas de inspeção do navegador para examinar mensagens no **Console**, requisições na aba **Network** e, quando disponível, a árvore do **React DevTools > Components** durante cada ação.
- Registre os controles/componentes realmente utilizados e seus identificadores observáveis (`id`, `data-testid`, `name`, `aria-label` ou texto acessível). Só chame algo de componente React pelo nome se esse nome aparecer na árvore do React DevTools.
- Não leia nem edite arquivos-fonte para diagnosticar, não rode comandos, testes, builds, scripts, chamadas diretas à API ou ferramentas de backend. Não corrija nem contorne o defeito.
- Não envie ações que possam criar, excluir, pagar, publicar ou alterar dados reais sem autorização clara do usuário para essa ação. Se o fluxo envolver uma ação desse tipo e não houver autorização, pare antes dela, registre a limitação e documente as evidências já coletadas.
- Não exponha tokens, cookies, dados pessoais ou outros segredos no relatório. Remova ou mascare valores sensíveis de URLs, cabeçalhos, payloads e mensagens.
- Distinga observação de hipótese. Não atribua uma causa que o navegador não comprova. O escopo é frontend; uma resposta de rede é evidência do que o browser recebeu, não diagnóstico da implementação do servidor.

## Fluxo de análise

1. **Entenda o relato.** Registre o problema em uma frase e preserve os exemplos, texto de erro e passos fornecidos pelo usuário. Se URL, credenciais, estado de teste ou passos forem necessários para alcançar a tela, peça apenas o dado ausente; nunca solicite senha ou segredo. Não invente um ambiente ou comportamento esperado.
2. **Prepare a observação.** Abra a aplicação autorizada no navegador. Registre a URL da tela e o estado inicial relevante. Limpe ou marque o início das mensagens do Console e da atividade de Network, se a ferramenta permitir, para associar as evidências às ações executadas.
3. **Reproduza pela interface e mapeie os componentes usados.** Siga os passos informados, um a um. Após cada clique, preenchimento ou seleção, registre em ordem o tipo de controle/componente, o identificador e a ação. Prefira o `id`; se não houver, use `data-testid`, `name`, `aria-label` ou texto acessível e diga qual atributo usou. Se React DevTools estiver disponível, registre o nome e caminho na árvore de componentes React que ajudem a localizar o controle; não altere props, estado ou dados por essa ferramenta. Não deduza nomes internos do React pela aparência do DOM. Anote também a mudança visual, navegação, carregamento, bloqueio ou ausência de resposta. Se não reproduzir, anote exatamente o caminho tentado e o resultado.
4. **Inspecione o Console.** Examine erros, warnings e logs relacionados ao horário e à ação. Registre texto relevante, tipo e momento; associe a evidência ao passo e componente correspondente. Não trate qualquer `console.log` como erro e não conclua causalidade só pela proximidade temporal.
5. **Inspecione o Network.** Para as ações observadas, registre as requisições relevantes: método, URL/rota sem parâmetros sensíveis, tipo (documento, fetch/XHR, asset), status ou falha, e a associação com o passo/componente da interface. Capture também redirecionamentos ou chamadas repetidas que ajudem a explicar o fluxo.
6. **Separe navegação e API.** Relate rotas de frontend (path/hash da aplicação, incluindo transições sem recarga) separadamente das rotas de rede/API. Informe método e status para chamadas de rede. Se nada relevante aparecer, declare que não foi observado; não deduza uma rota a partir do nome de um botão.
7. **Feche o diagnóstico.** Resuma o problema reproduzido e os passos mínimos para chegar a ele. Marque explicitamente conclusões confirmadas, hipóteses e itens inconclusivos. Se uma ação, identificador, aba ou árvore React não estava acessível, declare isso em vez de preencher lacunas.
8. **Crie o relatório.** Salve um arquivo Markdown no repositório/projeto analisado. Prefira `docs/analises/frontend/analise-<slug>-AAAA-MM-DD.md`; se essa pasta não existir, crie somente as pastas necessárias para o relatório. Use um slug curto da funcionalidade e a data local. Se não houver acesso de escrita ao projeto, entregue o Markdown no resultado e informe que não foi possível salvar o arquivo. Criar o relatório é a única escrita permitida: não altere arquivos de código ou configuração.

## Formato obrigatório do relatório

Use estes títulos e mantenha caixas de seleção Markdown em **Checagens feitas**:

```markdown
# Análise de frontend — <funcionalidade>

- Data: <AAAA-MM-DD>
- Ambiente: <URL/base e navegador, se conhecido; omita segredos>
- Resultado: <reproduzido | não reproduzido | inconclusivo>

## Visão Geral

- Problema reportado: <descrição>
- Exemplos fornecidos pelo usuário: <exemplos ou “nenhum informado”>
- Estado inicial relevante: <descrição>

## Rotas Utilizadas

### Rotas de frontend
- `<rota>` — <como/ao executar qual passo foi acessada>

### Requisições observadas na Network
- `<MÉTODO> <rota>` — status/falha: `<valor>`; passo: <passo>; observação: <resumo>

> Registre “Nenhuma rota relevante observada” quando aplicável. Não inclua parâmetros ou valores sensíveis.

## Componentes utilizados no fluxo

Liste, na ordem de uso, os componentes/controles acionados. Registre o tipo de identificador observado. Não atribua nome interno de componente React se esse nome não apareceu no React DevTools.

| Passo | Controle/elemento | Identificador observado | Componente React (DevTools, se visível) | Ação | Resultado |
|---|---|---|---|---|---|
| 1 | Botão | `id=XPOT-1234` | <nome/caminho ou “não observado”> | Clique | <resultado observado> |
| 2 | Campo de texto | `id=serial_number_warehouse` | <nome/caminho ou “não observado”> | Preencheu | <resultado observado; não copie valor sensível> |
| 3 | Campo de texto | `id=descricao_warehouse` | <nome/caminho ou “não observado”> | Preencheu | <resultado observado; não copie valor sensível> |

> O exemplo é apenas de formato. Substitua pelos controles observados no fluxo real. Se não foi possível identificar um controle, descreva-o visualmente e marque o identificador como “não observado”. Se React DevTools não estiver disponível, informe isso; IDs de DOM identificam controles, não nomes de componentes React.

## Checagens feitas

- [ ] Fluxo e passos relatados pelo usuário percorridos
- [ ] Cliques/ações da interface observados
- [ ] Componentes/controles percorridos registrados em ordem, com identificadores e atributos observados
- [ ] Nome interno de componente React registrado somente se visível no React DevTools
- [ ] Resultado visual após cada ação registrado
- [ ] Console inspecionado e mensagens relevantes associadas aos passos
- [ ] Network inspecionada e rotas/métodos/status relevantes registrados
- [ ] Navegação de frontend separada de chamadas de rede/API
- [ ] Segredos e dados pessoais removidos do relatório
- [ ] Limitações e hipóteses identificadas

Marque somente o que foi realmente verificado. Explique itens desmarcados ou indisponíveis.

## Problema Encontrado

- Achado: <comportamento observado ou “não foi possível confirmar”>
- Evidências: <mensagens do Console, requisições/status e comportamento visual, sem dados sensíveis>
- Certeza: <confirmado | hipótese | inconclusivo>

### Passo a passo para chegar ao problema
1. <estado ou rota inicial>
2. <ação específica>
3. <resultado observável que demonstra o problema>

### Limitações
- <o que não foi possível verificar, ou “nenhuma”>
```

## Kaizen aplicado à análise

- **Gemba:** baseie o relato no fluxo real da interface, nos componentes/controles utilizados e nas evidências vistas no Console e Network.
- **Jidoka/andon:** ao encontrar falha ou bloqueio, registre-o visivelmente com o passo e a evidência; não tente corrigi-lo durante a investigação.
- **PDCA, fase Check:** compare o resultado observado com o problema e os exemplos do relato. Diga se reproduziu, não reproduziu ou permaneceu inconclusivo.
- **No-blame:** descreva comportamento, condições e evidências, sem atribuir culpa a pessoas.

Kaizen não amplia o escopo desta skill: não implemente contramedidas, não edite código e não crie entradas em `KAIZEN_LOG.md`. O único artefato escrito nesta tarefa é o relatório de análise solicitado.
