---
name: analise-backend
description: Analisa uma rota de backend seguindo sua implementação por routes, controller, service e repository, verifica as operações GET, POST, PUT/PATCH e DELETE relacionadas e registra evidências sem alterar código.
---

# Análise de rotas de backend

Investigue a rota indicada pelo usuário e documente, com evidências, como a requisição atravessa as camadas do backend, quais dependências e efeitos ela aciona e quais operações CRUD relacionadas estão implementadas. **A análise não altera código nem configuração.**

## Limites e segurança dos dados

- Comece pela leitura do código e dos artefatos existentes. Prefira rastreamento estático; use logs, tracing, documentação ou execução somente quando disponíveis e autorizados. Não invente camadas: algumas arquiteturas não usam repository ou service separados.
- Não rode migrations, seeders, scripts, suites de teste ou chamadas HTTP por padrão. Não use credenciais ou dados reais no relatório.
- Antes de qualquer chamada que possa gravar, atualizar ou excluir dados, confirme que a base é descartável e isolada, e que o usuário autorizou testes comportamentais nesse ambiente. Produção, ambiente compartilhado ou ambiente cuja natureza seja incerta não são ambientes seguros para mutações; interrompa os testes de escrita nesses casos e siga com a análise estática.
- Se testes comportamentais forem autorizados, crie apenas dados sintéticos identificáveis e registre operação, identificador, valores iniciais e alterados, além da forma de limpeza. Não use dados de outras pessoas nem faça ações que disparem efeitos externos (por exemplo, pagamentos, e-mails ou publicação). Se não for possível identificar e rastrear com segurança cada registro de teste, não faça mutações.
- **Não exclua automaticamente os registros criados nos testes.** Ao concluir a análise, mostre os identificadores dos registros de teste e pergunte se a análise está correta e se o usuário autoriza excluí-los. Faça a limpeza somente após uma resposta afirmativa e inequívoca; limite-a aos registros que a própria sessão criou e confira que foram removidos. Se a resposta for negativa, ambígua ou não chegar, preserve os dados, informe que a limpeza está pendente e não diga que o banco está limpo. Se não houve escrita, informe que não há registros de teste para limpar e ainda pergunte se a análise está correta.
- Pare e informe qualquer falha ou efeito inesperado antes de continuar. Não tente corrigir o backend durante a investigação.
- Mascare tokens, cookies, credenciais, dados pessoais e valores sensíveis em rotas, parâmetros, cabeçalhos, payloads, logs e relatórios.
- Separe fatos observados de hipóteses. Uma resposta HTTP comprova o que o cliente recebeu, não quais camadas internas foram executadas.

## Fluxo de análise

1. **Entenda a rota e o relato.** Registre método, caminho, recurso, problema esperado e exemplos fornecidos. Peça somente os dados faltantes que impedem localizar a implementação, como stack, base do projeto, versão da API ou ambiente; nunca peça segredos. Não assuma que uma rota encontrada pelo nome é a rota pretendida.
2. **Localize a entrada.** Leia a definição efetiva da rota e confirme método, caminho, parâmetros, middleware, autenticação/autorização, validação e handler associado. Considere prefixos, agrupamentos, versão da API, registro dinâmico e convenções do framework. Se houver várias rotas candidatas ou a resolução não puder ser confirmada, apresente essa limitação.
3. **Siga o fluxo de ponta a ponta.** A partir do handler, acompanhe chamadas e evidências pelo controller, service/caso de uso, repository/DAO e acesso ao banco ou serviços externos. Registre a sequência real, funções/arquivos relevantes, bifurcações, transações, tratamento de erros e retornos. Marque camada ausente ou não aplicável como tal; não deduza que ela existe só pela arquitetura esperada.
4. **Inspecione o comportamento da rota.** Identifique validação de entrada, regras de negócio, consulta/escrita, serialização da resposta, códigos HTTP, tratamento de exceções e efeitos colaterais. Correlacione logs ou traces com a requisição apenas quando houver identificador e evidência suficientes. Não atribua causalidade apenas pela proximidade temporal.
5. **Verifique a cobertura CRUD do recurso.** Procure e registre as rotas relacionadas de leitura (`GET`), criação (`POST`), atualização (`PUT` e/ou `PATCH`) e remoção (`DELETE`). Diferencie o que foi encontrado no código do que foi exercitado em runtime. `UPDATE` normalmente corresponde a `PUT` ou `PATCH`; reporte ambos separadamente quando existirem. Não presuma que toda coleção precisa ter os quatro métodos: registre ausências como lacunas observadas e não como defeito sem requisito que as exija. Inclua outras rotas e métodos pertinentes, como `OPTIONS`, sem classificá-los como CRUD.
6. **Decida se testes de runtime cabem.** A análise estática pode confirmar a presença e o encadeamento das rotas sem tocar no banco. Só execute requisições de escrita quando as condições de ambiente e autorização da seção de segurança estiverem atendidas. Nunca use registros preexistentes para testar `PUT`, `PATCH` ou `DELETE`; não use a rota de limpeza em dados que não foram criados nesta sessão. Registre os testes feitos e os não feitos, com motivo.
7. **Feche o diagnóstico.** Resuma o caminho confirmado da requisição e compare o resultado observado ao relato. Classifique cada conclusão como confirmada, hipótese ou inconclusiva. Liste arquivos ou camadas não localizados, instrumentação indisponível e riscos/efeitos não testados.
8. **Crie o relatório.** Salve Markdown no projeto analisado, preferindo `docs/analises/backend/analise-<slug>-AAAA-MM-DD.md`; crie somente as pastas necessárias. Se não puder gravar no projeto, entregue o Markdown na resposta e informe a limitação. Não altere arquivos de código/configuração: o relatório é o único artefato persistente da análise.
9. **Peça a confirmação final de limpeza.** Depois de apresentar o resultado da análise, pergunte em uma única mensagem se está tudo certo com a análise e se o usuário autoriza excluir os registros de teste identificados. Não apague nada antes dessa resposta. Com autorização, remova somente os registros sintéticos criados nesta sessão, verifique a ausência deles e informe o resultado; pare se o estado ou o escopo da exclusão for inesperado.

## Formato obrigatório do relatório

Use os títulos abaixo. Marque em **Checagens feitas** somente os itens realmente verificados.

```markdown
# Análise de backend - <recurso/funcionalidade>

- Data: <AAAA-MM-DD>
- Projeto/stack: <projeto e framework, se conhecidos>
- Ambiente: <somente identificação não sensível; não inclua credenciais>
- Rota solicitada: `<MÉTODO> <caminho sem valores sensíveis>`
- Resultado: <reproduzido | não reproduzido | análise estática | inconclusivo>
- Estado da limpeza: <não houve escrita | aguardando autorização | concluída | falhou>

## Visão geral

- Problema ou pergunta: <descrição>
- Exemplos fornecidos: <exemplos ou "nenhum informado">
- Resumo: <comportamento observado e certeza>

## Fluxo da requisição

| Ordem | Camada | Arquivo/função | Entrada e saída observadas | Evidência/limitação |
|---|---|---|---|---|
| 1 | Route | <arquivo e handler> | <método/caminho encaminhado> | <evidência> |
| 2 | Middleware/validação | <arquivo/função ou não aplicável> | <o que verifica> | <evidência> |
| 3 | Controller | <arquivo/função> | <como delega e responde> | <evidência> |
| 4 | Service/caso de uso | <arquivo/função ou não aplicável> | <regra de negócio> | <evidência> |
| 5 | Repository/DAO | <arquivo/função ou não aplicável> | <consulta/gravação> | <evidência> |
| 6 | Banco/serviço externo | <arquivo/operação ou não aplicável> | <efeito observado ou inferido> | <evidência> |

## Rotas CRUD relacionadas

| Operação | Método e rota | Handler/camadas | Encontrada? | Exercitada em runtime? | Observação |
|---|---|---|---|---|---|
| Leitura | `GET ...` | <referências> | <sim/não/inconclusivo> | <sim/não; motivo> | <observação> |
| Criação | `POST ...` | <referências> | <sim/não/inconclusivo> | <sim/não; motivo> | <observação> |
| Atualização completa | `PUT ...` | <referências> | <sim/não/inconclusivo> | <sim/não; motivo> | <observação> |
| Atualização parcial | `PATCH ...` | <referências> | <sim/não/inconclusivo> | <sim/não; motivo> | <observação> |
| Remoção | `DELETE ...` | <referências> | <sim/não/inconclusivo> | <sim/não; motivo> | <observação> |

> Separe presença no código de execução real. Quando não houver uma rota para um método, escreva "não encontrada"; a ausência só é defeito se o requisito do recurso exigir essa operação.

## Comportamento e evidências

- Validação/autorização: <observado ou não localizado>
- Dados consultados ou alterados: <tipo de dado; não inclua dados sensíveis>
- Resposta/status/erros: <observado ou inconclusivo>
- Efeitos colaterais: <observados, possíveis ou não identificados>
- Logs/traces: <evidências correlacionadas ou indisponíveis>

## Checagens feitas

- [ ] Método, caminho e handler confirmados
- [ ] Middleware, autenticação e validação inspecionados
- [ ] Camadas percorridas e arquivos/funções registrados
- [ ] Acesso a dados e dependências externas inspecionados
- [ ] Rotas GET, POST, PUT, PATCH e DELETE relacionadas verificadas
- [ ] Presença no código separada de teste em runtime
- [ ] Segredos e dados pessoais removidos
- [ ] Testes de escrita limitados a ambiente isolado autorizado
- [ ] Identificadores e valores de todos os dados de teste registrados
- [ ] Hipóteses, limitações e efeitos não verificados declarados

Marque somente o que foi realmente verificado; explique itens desmarcados ou indisponíveis.

## Achados

1. **<achado>** - certeza: <confirmado | hipótese | inconclusivo>.
   - Evidência: <arquivo/linha, resposta, log ou trace sem conteúdo sensível>.
   - Impacto/limitação: <efeito demonstrado ou o que falta comprovar>.

## Testes e dados de teste

| Operação | Ambiente confirmado | Identificador sintético | Resultado | Limpeza planejada |
|---|---|---|---|---|
| <operação ou "nenhuma"> | <identificação> | <ID ou "não se aplica"> | <resultado> | <ação precisa ou "não se aplica"> |

- Limpeza: <não houve escrita | aguardando autorização do usuário | concluída e verificada | falhou, com motivo>

## Limitações

- <limitação ou "nenhuma conhecida">
```

## Kaizen aplicado à análise

- **Gemba:** rastreie a implementação real e, quando autorizado e seguro, evidência de execução; não pare no nome da camada ou no rótulo de um teste.
- **Check/PDCA:** confronte o relato com as evidências e separe implementação presente de comportamento realmente exercitado.
- **Jidoka/andon:** sinalize falhas ou efeitos inesperados e interrompa a sequência até entendê-los; não corrija durante a análise.
- **Poka-yoke:** faça o fluxo de dados de teste favorecer registros sintéticos identificáveis e limpeza verificável, sem enfraquecer a confirmação necessária antes de excluir.
- **No-blame:** descreva comportamento e condições, não culpe pessoas.

Kaizen orienta como observar e verificar; não amplia o escopo para corrigir código, implementar contramedidas ou registrar itens em `KAIZEN_LOG.md`. O único artefato persistente é o relatório solicitado.
