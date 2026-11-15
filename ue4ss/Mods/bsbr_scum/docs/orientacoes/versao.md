# Versão do mod

A versão atual é **0.8.1**. Ela pode ser lida de três formas, e as três sempre
concordam, porque o número nasce num lugar só no código e é carimbado nos três
na hora de compilar.

Voltar ao [índice](../README.md).

---

## Com o servidor no ar

Pela porta Source RCON ou pela porta de linha:

```
version
```

```
bsbr_scum 0.8.1
portas source=28015 linha=27100
opcoes elevate=1 split_replies=0 echo=0 utf8=1 perf_log=0 executor=jogo
```

É a resposta mais confiável, porque vem do binário que está carregado naquele
momento. A senha nunca aparece.

Para um gerenciador externo, é uma chamada igual a qualquer outra:

```python
versao = client.send_command("version").splitlines()[0]   # "bsbr_scum 0.8.1"
```

## Com o servidor parado

**Pelo arquivo**, ao lado do `config.ini`:

```
ue4ss\Mods\bsbr_scum\version.txt
```

```
bsbr_scum 0.8.1
carregado 2026-10-15 00:31:04
```

A primeira linha é a versão. A segunda é quando o mod subiu pela última vez, e
serve de aviso: se você trocou a DLL e ainda não reiniciou o servidor, a data
estará velha e o número pode ser o da versão anterior.

**Ou pela própria DLL**, sem depender de arquivo nosso:

```powershell
(Get-Item "...\ue4ss\Mods\bsbr_scum\dlls\main.dll").VersionInfo.FileVersion
```

Esse é o caminho mais direto para um instalador ou painel: é o recurso padrão
de versão do Windows, igual ao que aparece nas propriedades do arquivo.

## Por que não fica no `config.ini`

O `config.ini` é **entrada**: você edita, e um gerenciador externo também
escreve nele. Uma versão ali envelheceria sozinha a cada troca de DLL, ou
sumiria quando o arquivo fosse regravado. Um número que mente sobre qual
binário está rodando atrapalha mais do que ajuda, então a versão fica fora dele.

---

## O que mudou na 0.8.1

**A medir no servidor** (ver "Desempenho" em `comandos/mod.md`).

- **Otimização, sem comando novo e sem mudança de resposta.** Os comandos que
  leem os mesmos campos de muitos veículos ou jogadores passaram a achar cada
  campo **uma vez** depois da subida do servidor e a guardar o endereço dele,
  em vez de procurá-lo pelo nome em cada veículo e em cada jogador. A posição
  é lida direto do ator quando ele não está preso a outro (jogador montado
  continua pelo caminho antigo).
- Comandos que ficaram mais leves: `veiculos_listar`, `vehicles`,
  `ListPlayers`, `players`, a recontagem e a busca do jogador mais perto do
  `spawn_area`, e todo comando que procura um jogador pelo SteamID, perfil ou
  nome (`as`, `SendChat`, `SendNotification`, `aviso`, `Location <SteamID>` e
  os comandos do jogo).
- **Respostas idênticas, byte a byte**: mesmo formato, mesma ordem, mesmo
  arredondamento.
- **Auto-verificação**: nas primeiras leituras depois da subida, cada campo é
  lido pelos dois caminhos e comparado. Se algum divergir (por exemplo, depois
  de um patch do jogo), o mod registra um erro no `UE4SS.log`
  (`[bsbr_scum][campos]`) e aquele campo volta ao caminho antigo até o
  reinício. As respostas continuam certas; só o ganho se perde.

## O que mudou na 0.8.0

**Validado no servidor de dev em 15/11/2026:** 100 NPCs a 10/s em 12,5 s, todos em pé, 30 FPS; cancelamento e evento final conferidos.

- **`spawn_area <tipo> <classe|random> <quantidade> <x> <y> <raio_cm> [<por_segundo>]`**,
  comando novo: espalha criaturas num círculo, em ritmo controlado, pelos
  comandos do próprio jogo, um por comando, com ponto sorteado no círculo.
  Tipos `npc` (exige a classe, ex. `BP_Guard_Lvl_1`), `zumbi` e `animal`
  (classe ou `random`). Robô (sentinela) e drone não têm comando de spawn no
  jogo: `erro: tipo sem spawn`. Exemplo, 100 guardas numa cidade:
  `spawn_area npc BP_Guard_Lvl_1 100 -113084 -70633 30000 10`.
- Responde na hora `ok spawn_area <id> agendado: ...` e vai criando, no máximo
  `por_segundo` por segundo (1 a 20, padrão 10), só nos ticks em que nenhum
  comando das portas espera. Um trabalho por vez.
- O jogo só cria perto de jogador (até 1 km) e com altura perto do terreno, e
  recusa em silêncio. Por isso o mod exige um jogador a até 1 km do centro,
  acha o chão de cada ponto com um traço vertical (`LineTraceSingle`) e cria
  3 m acima dele, e confere por contagem: `criados`
  é **estimativa** (criaturas morrem e andam). Se nada nascer depois de 20
  spawns, para sozinho (`nada_nasceu`).
- **`spawn_area_estado`** e **`spawn_area_cancelar`**, comandos novos; o fim
  vira o evento `spawn_area` na fila `eventos` (entra sempre, não depende de
  `eventos_capturar`).
- Abreviação: `spa`/`spaw` deixaram de ser `spawn` (agora são ambíguos com os
  três `spawn_area*`). `spawn` por extenso não mudou.
- Os outros comandos e respostas não mudaram.
- Ver [comandos/mod.md](../comandos/mod.md#spawn_area--criaturas-espalhadas-num-círculo-em-teste).

## O que mudou na 0.7.0

Estado final depois dos testes da build final no servidor de desenvolvimento
em 15/11/2026. Publicada em 15/11/2026.

**Validado:**

- `veiculos_listar`: dono (perfil), SteamID do dono online e trancas; tranca
  tirada e recolocada apareceu na leitura seguinte; 0,38 ms com 3 veículos e
  0,91 ms com 26 (o `vehicles` deu 1,1 ms e 2,6 ms nas mesmas condições).
- `eventos` e `npc_morto`: 41 NPCs armados mortos em 27 s, 41 recebidos,
  0 perdidos; a confirmação esvaziou a fila. Na build final, 3 guardas mortos
  na faca deram 3 eventos com arma `1H_Military_Survival_Knife` (o campo da
  arma vale para arma de fogo e arma branca). Estresse: 161 milhões de
  eventos de teste em 14,4 min, sem queda, sem erro e sem crescimento de
  memória.
- `jogador_morto` com as causas `zumbi` (4 mortes), `animal` (javali
  mutante) e `queda` (duas quedas de 320 m, sem atacante); `jogador_renasceu`.
- **Morte duplicada corrigida e validada**: com o jogador 30 s na tela de
  morte, saiu um `jogador_morto` (16:43:41, `queda`) e o `jogador_renasceu`
  (16:44:11), sem `jogador_morto` extra (o gatilho disparou 2 vezes, 1
  evento); a morte seguinte do mesmo jogador (16:44:55) saiu normalmente.
- `zona_criar` sem recusa de invasão: zona criada exatamente sobre o Outpost
  B4 respondeu `ok ... criada`; atualizar, remover e remover de novo
  responderam `ok ... atualizada`, `ok ... removida` e `ok ... nao existia`;
  título com acento funcionou. Custo: `zona_criar` 7,6 ms, `zona_remover`
  4,2 ms, `zona_listar` 0,06 ms.
- Comandos de mundo na thread do jogo: `version` mostra `executor=jogo`;
  comandos do jogo de níveis 2, 3 e 4 (`SetGodMode`, `SetImmortality`,
  `SendNotification`) funcionaram pela porta, conferidos pelo efeito no
  personagem; `SpawnBrenner` com `as` e `DestroyAllBrennersWithinRadius`
  funcionaram. `erro: thread do jogo nao respondeu` nunca apareceu em uso
  normal; apareceu uma vez com o servidor a ~5 FPS por causa de 87 veículos
  inválidos.
- `anuncio`, `aviso` e as cores.

**Não validado** (instalado, nunca exercitado): `sentinela_destruida`,
`razor_morto`, `nave_destruida`; `animal_morto` (não dispara para animal
criado por comando: 0 disparos com 8 abates; não testado com animal do
próprio jogo); a causa do zumbi kamikaze depois do ajuste (hoje
`desconhecida`); as causas por robô (sentinela), Razor, Brenner, NPC, outro
jogador, fome, sede e frio; `aviso` com dois jogadores online; o `nome` do
`veiculos_listar` com veículo nomeado; veículo aquático; qual regra o jogo
aplica dentro de zonas sobrepostas. Sem validação registrada:
`eventos_capturar`, `morte_teste`, `sonda_funcoes` e `corpo`.

- **Mudança incompatível: o `logs/npc_kills.log` deixou de existir.** O mod
  não cria mais a pasta `logs` nem escreve o arquivo. A morte de NPC armado
  por jogador sai como evento `npc_morto` na fila em memória, com os mesmos
  dados. Quem lia o arquivo tem de passar a chamar `eventos`.
- **`eventos [<ultimo>]`**, comando novo: busca a fila de acontecimentos em
  memória (4096 registros) depois de `<ultimo>` e descarta até ele.
  Cabeçalho `eventos: <n> epoca=<epoca> ultimo=<seq> perdidos=<p> mais=<0|1>`
  e uma linha `<seq>;<hora>;<tipo>;<campos...>` por evento, até 500 por
  resposta. Feito para o painel chamar a cada 1 ou 2 s pela porta Source; não
  grava nada em arquivo.
- **`eventos_teste <n>`**, diagnóstico: põe `n` eventos falsos (1 a 20000) na
  fila, para teste de carga.
- **Mais mortes na fila**: `jogador_morto` com a causa deduzida pelo mod e
  as classes cruas (atacante, pawn, arma, ameaça a até 150 m, efeitos do
  corpo), `jogador_renasceu` (SteamID, perfil, nome e posição),
  `sentinela_destruida`, `razor_morto`, `nave_destruida` e `animal_morto`,
  estes na mesma linha do `npc_morto`. Em 15/11/2026 o `jogador_morto`
  (`ConZPlayerController:Server_RequestRespawnWaitTime`) e o
  `jogador_renasceu` (`Prisoner:Server_RespawnAtCommonSpawnLocation`) foram
  validados; a posição do renascimento é a do local da morte. O cliente chama
  o gatilho do `jogador_morto` uma vez na morte e outra no renascimento; a
  proteção contra repetição é por estado (uma morte por vida; o renascimento
  libera a próxima; aviso de morte até 5 s depois do renascimento é
  ignorado), não mais só 5 s, que deixava sair um `jogador_morto` falso
  quando o jogador esperava na tela de morte (corrigido e **validado** em
  15/11/2026). Limitações: mod subindo com o jogador já na tela de morte
  perde essa morte; morrer de novo menos de 5 s depois de renascer não gera
  evento. Causas validadas: `zumbi`, `animal`, `queda`. Causas sem atacante:
  `explosao`/`sentinela` (queimadura), `queda` (desmaio; medida em duas
  quedas), `sangramento` (só o efeito `Bleeding` exato), `frio`, `calor`,
  senão `desconhecida` (o zumbi kamikaze sai assim). `sentinela_destruida`,
  `razor_morto` e `nave_destruida` são candidatos nunca exercitados;
  `animal_morto` é candidato que não disparou com animal criado por comando e
  não foi testado com animal do próprio jogo. **`zumbi_morto` foi
  removido**: o jogo não tem função que dispare no abate; zumbi e animal
  mortos por jogador se contam pelo `SCUM.db`. Ver
  [comandos/mod.md](../comandos/mod.md#tipos-novos-de-morte).
- **`eventos_capturar [padrao|tudo|tipo,tipo,...] [minutos]`**, comando novo
  (local): escolhe quais mortes entram na fila; fora de `padrao`, volta
  sozinho ao `padrao` depois dos minutos (1 a 720, padrão 120).
- **`morte_teste <alvo>`** e **`sonda_funcoes <classe> <segundos>|ler`**,
  diagnóstico: um `jogador_morto` de um jogador vivo para conferir os campos,
  e uma sonda de funções de uma classe com ganchos removíveis (não é para
  produção).
- **Comandos da porta Source não vão mais para o `bsbr_scum.log`**: nem a
  linha do comando, nem a resposta, nem o eco no UE4SS.log. O painel registra
  os dele. A porta de linha continua registrada como antes; a linha
  `[perf]` do `[debug] perf_log` continua para todos.
- **`veiculos_listar`**, sem argumentos: uma linha por veículo carregado, com
  id, classe, posição, perfil do dono, SteamID do dono, trancas e nome.
  `veiculo;<id>;<classe>;<x>;<y>;<z>;<perfil_dono>;<steamid_dono>;<trancas>;<nome>`
- Só enxerga veículo carregado em memória (perto de algum jogador). O SteamID
  vem `0` com o dono offline; o veículo conta como do jogador só com
  `trancas` > 0.
- Feito para consulta em laço (30 s; mínimo 10 s): não grava nada em arquivo,
  nem (pela porta Source) a chamada no `bsbr_scum.log`.
- **`zona_criar` não recusa mais a zona que entra em outra.** A conferência
  de invasão da 0.5.5 saiu: o mod só vê círculos e não sabe qual
  sobreposição é problema (uma zona de "proibido construir" não é; um posto
  avançado pode ser). A resposta `erro: a zona invadiria "..."` deixou de
  existir. Quem chama decide, com a lista do `zona_listar`. **Validado em
  15/11/2026** com uma zona exatamente sobre o Outpost B4; qual regra o jogo
  aplica dentro da área sobreposta não foi medido.
- **`anuncio [segundos] <mensagem>`**, comando novo: o aviso do centro da
  tela para todos. Sem segundos, repassa ao `Announce` do jogo (3 s); com
  segundos (1 a 60), fica esse tempo. **Validado em 15/11/2026**:
  `anuncio 15 <frase>` ficou 15 s na tela e o servidor seguiu estável.
- **`aviso <alvo> [segundos] <mensagem>`**, comando novo: o mesmo aviso só
  para um jogador conectado (SteamID, id do perfil ou nome); sem segundos,
  5 s. Resposta `ok aviso enviado a <perfil> (<n> s)`. **Validado em
  15/11/2026**: amarelo, só para o jogador, frase inteira; SteamID offline
  respondeu `erro: jogador "..." nao esta conectado`.
- **`cor:<nome|#RRGGBB>`** no `anuncio` e no `aviso`. **Validado em
  15/11/2026**: as oito cores e `#8000FF` apareceram na tela na cor pedida,
  no global e no individual. Opcional, antes da mensagem,
  em qualquer ordem com os segundos. Nomes `vermelho`, `laranja`, `amarelo`,
  `verde`, `azul`, `ciano`, `rosa`, `branco`; sem cor, laranja no `anuncio` e
  amarelo no `aviso`. A resposta só cita a cor quando pedida
  (`ok anuncio enviado (8 s, cor vermelho)`).
- **`sonda_aviso`**, diagnóstico: registra no `bsbr_scum.log` o formato de
  cada aviso do jogo. Só lê; não deixar ligado.
- **`sonda_morte <segundos>|ler|estado <alvo>`**, diagnóstico, **desligada
  por padrão**: mostra quais funções do jogo rodam quando um jogador morre.
  Só existe com `[debug] sonda_morte = 1` no `config.ini` e
  `HookUObjectProcessEvent = 1` no `UE4SS-settings.ini`; nunca em produção.
  `estado <alvo>` funciona sempre e é o mesmo que `corpo <alvo>`. Ver
  [comandos/mod.md](../comandos/mod.md#sonda_morte--o-que-roda-quando-um-jogador-morre-diagnóstico).
- **Comandos de mundo passam a rodar na thread do jogo.** Até a 0.6 todo
  comando rodava na thread do laço do UE4SS, em paralelo com o jogo (corrida
  de dados). Agora os que tocam o jogo (spawn, teleporte, `ListPlayers`,
  `veiculos_listar`, `zona_*`, `anuncio`, `aviso`, comandos do jogo...) rodam
  no tick do motor, **um por tick: no máximo ~30 por segundo no total**.
  `eventos`, `eventos_teste`, `eventos_capturar`, `version` e `sonda_morte ler` não tocam o jogo,
  rodam na hora fora dela e não entram nessa conta. Tick parado por mais de
  2 s com comando esperando: `erro: thread do jogo nao respondeu`.
  **Validado em 15/11/2026** (`executor=jogo`; o erro só apareceu com o
  servidor a ~5 FPS). Ver
  [comandos/mod.md](../comandos/mod.md#desempenho).
- **`version`**: a linha `opcoes` ganhou no fim `executor=jogo|ue4ss`, onde os
  comandos de mundo estão rodando. `ue4ss` quer dizer que o registro no tick do
  motor falhou (exige `HookEngineTick = 1`); o mod segue como na 0.6.
- **`corpo <alvo>`**, comando novo (em teste): efeitos do corpo
  (`Bleeding`, `Hunger`, `PrisonerBodyEffect_Death`...), último dano e arma
  de um jogador, para descobrir a causa da morte. Só lê. Ver
  [comandos/mod.md](../comandos/mod.md#corpo--causa-provável-da-morte-em-teste).
- **Mudança incompatível: os comandos do mod pelo chat do jogo foram
  retirados.** `#bsbr <comando>`, `#bsbr @Fulano <comando>` e `#bsbr` sozinho
  (a ajuda) deixaram de existir. Motivo: com o jogo na 1.3.6.1, o cliente
  passou a barrar verbos que não conhece; `#bsbr ...` mostra "comando não
  reconhecido" e nem chega ao servidor (conferido em 15/11/2026). O mod não
  instala mais o gancho que lia o chat. Os comandos do mod seguem pelas duas
  portas (painel e terminal), inclusive `as <jogador>` e `@<jogador>`; os
  comandos do **próprio jogo** pelo chat (`#ListPlayers` etc.) continuam
  funcionando normalmente.
- Abreviação: `sonda` sozinho deixou de ser `sonda_aviso` (agora é ambíguo
  com `sonda_morte`); `co` deixou de ser `commands` (agora é ambíguo com
  `corpo`).
- Os outros comandos e respostas não mudaram.
- Ver [comandos/mod.md](../comandos/mod.md#veiculos_listar--dono-tranca-e-nome-dos-veículos),
  [comandos/mod.md](../comandos/mod.md#eventos--fila-de-acontecimentos-em-memória)
  e [comandos/mod.md](../comandos/mod.md#anuncio-e-aviso--aviso-no-centro-da-tela).

## O que mudou na 0.6.0

- **As respostas do mod saem em UTF-8**, nas duas portas. Antes ia um byte
  por caractere (Latin-1), e nome com acento chegava quebrado em quem lê
  UTF-8. Agora os dois sentidos concordam: o mod lê UTF-8 (desde a 0.5.4) e
  responde UTF-8. Texto sem acento é idêntico ao de antes.
- **Quem precisar do formato antigo:** `[api] utf8 = 0` no `config.ini`.
- A linha `opcoes` do comando `version` ganhou `utf8=1`.
- A recusa de zona que invadiria outra (0.5.5) foi validada no servidor.

## O que mudou na 0.5.5

`zona_criar` **recusa a zona que invadiria outra**. Ainda **em teste**:
compila, a recusa não foi exercitada no servidor.

- Antes de enviar qualquer coisa, o círculo pedido é comparado com **todas as
  outras regiões** da lista (do editor ou de outro `id`). Se a distância entre
  os centros for menor que a soma dos raios, nada é criado:
  `erro: a zona invadiria "<nome da regiao>" (distancia entre centros <d> m, minimo <m> m); nada criado`
- Encostar exatamente é permitido. Com várias invadidas, a resposta cita a de
  maior invasão.
- Vale também para **atualizar** um `id` que já existe: mover ou aumentar a
  zona para dentro de outra é recusado e a zona antiga fica como estava. A
  região do próprio `id` não conta na comparação.
- Região com tamanho menor que 1 cm (sobra de teste) não cobre nada e não
  bloqueia. Região retangular entra com um raio que a cobre com folga.
- Custo: uma distância por região existente, só quando o comando roda.
- Os outros comandos e respostas não mudaram.
- Ver [comandos/mod.md](../comandos/mod.md#zona_listar-zona_criar-zona_remover--zonas-personalizadas-em-teste).

## O que mudou na 0.5.4

Zona com **título**, separado do `id`. **Validado no servidor de
desenvolvimento em 11/11/2026**: zona criada com título acentuado, o aviso de
entrada do jogo mostra o título, e a linha no `SCUM.db` saiu correta. O resto
da lista abaixo segue **em teste**.

- **`zona_criar <id> <x> <y> <raio_cm> <configuracao> <titulo>`**: o sexto
  dado é novo e obrigatório. O jogo mostra o nome da região na tela de quem
  entra na zona ("Você entrou na zona ..."), então esse nome passou a ser um
  título para o jogador ler, e não mais `bsbr_<nome>`. O `id` é só a chave de
  quem chama. **A forma antiga, de cinco dados, responde o uso.**
- **O prefixo `bsbr_` acabou.** O mod sabe quais zonas são dele por um
  registro `id → título` em `ue4ss\Mods\bsbr_scum\zonas.ids`. Zona criada
  por versão anterior (`bsbr_...`) passa a contar como do editor: apague-a no
  editor de zonas.
- **Respostas com o `id`**: `ok zona evento_42 criada (titulo "...", ...)`,
  `ok zona evento_42 removida`, `ok zona evento_42 nao existia`. Erros novos:
  `erro: titulo invalido: ...` e `erro: ja existe uma zona chamada "..."`.
- **`zona_listar`**: a região do mod traz o `id` no fim
  (`...;mod;evento_42`); a coluna do nome é o título.
- **`zona_remover <id>`** de um `id` que o mod não registrou responde
  `ok ... nao existia` mesmo sem lista de zonas.
- **Acento na entrada**: comando recebido pelas duas portas em UTF-8 válido é
  decodificado como UTF-8 (antes, cada byte virava um caractere). Linha que
  não é UTF-8 válido segue como antes. A **resposta** não mudou: um byte por
  caractere.
- **`zonas.cache`** não é mais recusado quando a versão do mod muda; só
  quando o layout das zonas do jogo muda. Atualizar o mod deixou de exigir um
  salvamento no editor.
- `zona_esquecer` não apaga o `zonas.ids`.
- Ver [comandos/mod.md](../comandos/mod.md#zona_listar-zona_criar-zona_remover--zonas-personalizadas-em-teste).

## O que mudou na 0.5.3

Arranque a frio das zonas: o mod deixa de exigir um salvamento no editor a
cada subida do servidor. **A cópia em arquivo compila, mas ainda não foi
exercitada no servidor.**

- **`zonas.cache`**: toda vez que o servidor aplica um salvamento de zonas, o
  mod grava a lista em `ue4ss\Mods\bsbr_scum\zonas.cache`, e a lê de volta
  quando o servidor sobe. `zona_criar` e `zona_remover` passam a funcionar
  logo depois de um reinício.
- **`zona_listar`** ganhou a origem `arquivo`: `zonas: update=0 multicast=0
  ultima=<data do arquivo> origem=arquivo`. Depois do primeiro salvamento visto
  ao vivo a origem volta a ser `multicast`.
- **`zona_esquecer`**, comando novo: solta a lista de origem `arquivo` e apaga
  o `zonas.cache`. É a válvula para arquivo velho (zonas mexidas com o mod
  fora do ar, banco apagado ou restaurado).
- **`[zonas] cache`** no `config.ini`: `1` (padrão) liga o arquivo, `0` desliga
  a gravação e a leitura.
- Salvar uma vez no editor de zonas só é preciso **na primeira vez**, depois de
  um `zona_esquecer`, e depois de atualizar o mod ou o jogo (o arquivo só vale
  para a mesma versão do mod e o mesmo layout de zonas).
- Os comandos e as respostas de `zona_criar` e `zona_remover` não mudaram.
- Ver [comandos/mod.md](../comandos/mod.md#zona_listar-zona_criar-zona_remover--zonas-personalizadas-em-teste).

## O que mudou na 0.5.2

Zonas personalizadas **em teste**: criar e remover ainda não foram exercitados
no servidor.

- **`zona_criar <nome> <x> <y> <raio_cm> <configuracao>`** e
  **`zona_remover <nome>`**: criam, atualizam e removem uma região circular
  ao vivo, pela mesma função que o editor de zonas do jogo usa. A região do
  mod é guardada como `bsbr_<nome>`, e só região com esse prefixo pode ser
  removida pelo mod. Só respondem `ok` quando o servidor distribui o conjunto
  de volta com a mudança.
- **`zona_listar`** passou a ler de uma cópia fiel do conjunto de zonas (não
  mais de texto) e ganhou o campo final `mod` ou `editor` em cada região.
- **Limitação**: a cada subida do servidor, um admin precisa abrir o editor de
  zonas e salvar uma vez antes de os três comandos funcionarem. O mod não
  guarda as zonas em disco.
- `logs/npc_kills.log`: sem mudança de comportamento. Conferido ao vivo em
  11/11/2026 que, se um zumbi dá o golpe fatal depois de um jogador ter
  acertado o NPC, nenhuma linha é gravada; dano sem autor (queda, afogamento)
  depois de um acerto de jogador ainda pode creditar a morte a ele.
- Ver [comandos/mod.md](../comandos/mod.md#zona_listar-zona_criar-zona_remover--zonas-personalizadas-em-teste).

## O que mudou na 0.5.1

- **`logs/npc_kills.log` validado** no servidor em 11/11/2026: dois guardas
  mortos, duas linhas, sem repetição. Nada mudou no código dele.
- **`zona_listar` virou sondagem passiva**, ainda só leitura. A captura da
  0.5.0 (pedir os dados pelo canal de um jogador) foi testada e não funciona
  num servidor dedicado; foi removida. Agora o mod só escuta as duas funções
  que rodam no servidor quando alguém salva no editor de zonas, guarda a última
  lista em texto e registra no `UE4SS.log` o layout das estruturas. O comando
  não precisa mais de jogador conectado e ganhou a cor na linha `config`. Ver
  [comandos/mod.md](../comandos/mod.md#zona_listar-zona_criar-zona_remover--zonas-personalizadas-em-teste).

## O que mudou na 0.5.0

Versão de **sondagem**.

- **`logs/npc_kills.log`**: uma linha por NPC armado morto por jogador, com
  SteamID, arma e posição. Uma chamada por morte, nada por tick. Substituído
  na 0.7.0 pelo evento `npc_morto`: ver
  [comandos/mod.md](../comandos/mod.md#evento-npc_morto-substitui-o-logsnpc_killslog).
- **`zona_listar`**, comando do mod, só leitura: primeira tentativa de listar
  as zonas personalizadas do jogo (substituída na 0.5.1).

## O que mudou na 0.4.0

- **`DestroyAllBrennersWithinRadius [Radius [m]]`**, comando do mod. O jogo
  não tem como remover o Brenner; este acha os `BP_Brenner_C` num raio do
  jogador do `as` (padrão 50 m) e os destrói. Ver
  [comandos/mod.md](../comandos/mod.md#destroyallbrennerswithinradius--remover-brenners).
- **`SpawnBrenner` passa a exigir `as <SteamID>`.** Sem alvo, o Brenner nascia
  na frente do primeiro jogador vivo. Agora responde o uso e não executa.

## O que mudou na 0.3.0

- **Comando do jogo sem nenhum admin conectado.** A entrega automática de item
  e veículo funciona de madrugada, quando não há administrador online. Ainda é
  preciso um jogador qualquer conectado. Ver
  [comandos/mod.md](../comandos/mod.md).
- **Porta Source endurecida**: limite de conexões, bloqueio após senhas
  erradas, resposta imediata enquanto o servidor ainda está subindo, e escolha
  entre uma resposta inteira ou picotada conforme o cliente.
- **Alvo por SteamID no fim do comando**, no mesmo formato que os
  gerenciadores já usam.
- **Verbo desconhecido** responde sem incomodar o jogo.
- **Chat limpo**: a porta de linha não ecoa mais nada no jogo. O registro
  completo continua em `bsbr_scum.log`, agora com quem digitou cada comando
  pelo chat.
- **`version`**, este comando.
