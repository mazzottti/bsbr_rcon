# Comandos do mod

Estes **não são do SCUM**. São do `bsbr_scum`, e funcionam por reflexão da Unreal:
leem e escrevem propriedades dos objetos vivos do servidor, sem passar pelo
interpretador de comandos do jogo.

Voltar ao [índice](../README.md).

---

## Como chamar

Pelas portas do mod: o painel (porta Source) ou o terminal do servidor (porta de
linha), com a função `scum` do PowerShell descrita em
[conectar.md](../orientacoes/conectar.md). Exemplo: `scum vehicles`. A resposta
volta pela própria conexão; sem `as`, o alvo é o primeiro jogador vivo.

Os comandos do mod não entram pelo chat do jogo desde a 0.7.0 (o motivo está em
[versao.md](../orientacoes/versao.md)).

**Não são sensíveis a maiúsculas.** `scum VEHICLES` e `scum vehicles` são a
mesma coisa. Vale para o nome do comando — o resto da linha (nome de jogador,
caminho de propriedade, classe) mantém a grafia.

### Abreviação

Basta digitar o suficiente para não ser ambíguo:

```
scum veh           → vehicles
scum pl            → players
scum s             → 's' e ambiguo: say, search, set, spawn, sudo
```

A abreviação só vale para comandos do mod. Verbos do jogo passam intactos.

---

## Resumo

| Comando | O que faz |
|---|---|
| `help [comando]` | A lista, ou o detalhe de um comando |
| `players` | Jogadores conectados, com ID e SteamID |
| `vehicles` | Veículos com o ID que a interface mostra |
| `veiculos_listar` | Veículos carregados com dono, SteamID, trancas e nome; não grava em log, feito para consulta em laço (validado em 15/11/2026) |
| `eventos [ultimo]` | Busca a fila de acontecimentos em memória (`npc_morto`...) depois de `ultimo` e descarta até ele; não grava em log (validado em 15/11/2026) |
| `eventos_teste <n>` | Diagnóstico: põe `n` eventos falsos (1 a 20000) na fila, para teste de carga |
| `eventos_capturar [padrao\|tudo\|tipo,tipo,...] [minutos]` | Quais mortes entram na fila; fora de `padrao` vence sozinho (1 a 720 min, padrão 120) (compilado, a validar) |
| `morte_teste <alvo>` | Diagnóstico: põe na fila um `jogador_morto` de um jogador vivo, para conferir os campos (compilado, a validar) |
| `sonda_funcoes <classe> <segundos>\|ler` | Diagnóstico: quais funções declaradas na classe rodam numa janela de 1 a 120 s; não é para produção (compilado, a validar) |
| `get <alvo> [caminho]` | Despeja as propriedades do alvo num arquivo |
| `set <alvo> <caminho> <valor>` | Escreve uma propriedade |
| `call <alvo> <função>` | Chama uma função sem parâmetros |
| `spawn <classe> <x> <y> <z>` | Cria um ator direto pela engine |
| `sudo <comando>` | Executa um comando de nível Developer |
| `DestroyAllBrennersWithinRadius [Radius [m]]` | Destrói os Brenners num raio do jogador do `as` (padrão 50 m) |
| `zona_listar` | Zonas personalizadas que o mod viu passar, com `mod` ou `editor` por região (em teste) |
| `zona_criar <id> <x> <y> <raio_cm> <configuracao> <titulo>` | Cria ou atualiza a zona circular do `id`; o título é o nome que o jogador lê; aceita zona sobre outra (validado em 15/11/2026) |
| `zona_remover <id>` | Remove a zona do `id`, e só zona criada pelo mod (validado em 15/11/2026) |
| `zona_esquecer` | Solta a lista de zonas carregada do `zonas.cache` e apaga o arquivo (os ids ficam) |
| `find <texto>` | Procura classes e objetos carregados |
| `search <alvo> <valor>` | Procura uma propriedade com esse valor |
| `inspect <classe>` | Reflexão completa de uma classe |
| `say <texto>` | Anuncia no chat |
| `anuncio [segundos] [cor:<nome\|#RRGGBB>] <mensagem>` | Aviso no centro da tela para todos; segundos de 1 a 60, sem eles 3 s; cor padrão laranja (cor validada em 15/11/2026) |
| `aviso <alvo> [segundos] [cor:<nome\|#RRGGBB>] <mensagem>` | O mesmo aviso do centro só para um jogador conectado; sem segundos, 5 s; cor padrão amarelo (cor validada em 15/11/2026) |
| `sonda_aviso` | Diagnóstico: registra no `bsbr_scum.log` o formato de cada aviso do jogo; só lê |
| `sonda_morte <segundos>\|ler\|estado <alvo>` | Diagnóstico: quais funções do jogo rodam na morte de um jogador; desligada por padrão (`[debug] sonda_morte`) |
| `corpo <alvo>` | Efeitos do corpo, último dano e arma de um jogador: a causa provável da morte; só lê (em teste) |
| `commands` | Gera o catálogo dos comandos do jogo |
| `capture on\|off` | Liga a captura da saída dos comandos |

E um prefixo, que não é comando:

| Prefixo | O que faz |
|---|---|
| `as <jogador> <comando>` | Executa o comando **como** aquele jogador (forma curta: `@<jogador>`) |

---

## `as` e `@` — mirar em outro jogador

```powershell
scum as Fulano SpawnItem Weapon_M9
```

**`@Fulano` é a mesma coisa que `as Fulano`.** No PowerShell, `@` no início de um
argumento é *splatting* e some antes da função receber — e `-` seria lido como
nome de parâmetro. Por isso o terminal usa `as`. O `@` funciona desde que entre
aspas: `scum '@Fulano' ...`; um programa que fala direto com a porta pode
mandar `@Fulano ...` sem aspas.

Comandos de admin rodam **no contexto de quem os envia**. É isso que define onde
o item cai, onde o veículo nasce, de quem é a posição consultada. Trocar o
executor redireciona os 233 comandos de uma vez, sem precisar de argumento de
posição em cada um.

Aceita três identificadores, todos **exatos** — sem abreviar:

| Forma | Exemplo |
|---|---|
| ID do perfil | `as 1` |
| SteamID | `as 76561190000000000` |
| Nome completo | `as Fulano` |

Use `players` para vê-los. Só a caixa é ignorada: `as fulano` funciona.

> Casamento por trecho seria perigoso: `1` acertaria o jogador `17`, e `Ma`
> acertaria dois jogadores diferentes — em silêncio, e o erro só apareceria pela
> reclamação de quem levou.

**Sem `as`**, o alvo é o primeiro jogador vivo (a porta não tem remetente).

Combina com tudo:

```
scum as Fulano sudo SpawnBrenner
scum as Fulano Teleport 559735 -215880 1172
scum as Fulano get player
```

---

## `help`

```powershell
scum help
scum help set
```

O detalhe de `get`, `set`, `call` e `search` inclui os alvos aceitos; o de `set`
inclui a sintaxe do caminho; o de `sudo`, o aviso dos comandos destrutivos.

---

## `players`

```powershell
scum players
```

```
1 jogador(es):
    1 | Fulano             | 76561190000000000
alvo: '@<id|steamid|nome> <comando>' - nome completo, sem abreviar
```

As três colunas são ID do perfil, nome e SteamID — os mesmos que o SCUM registra
no log como `'76561190000000000:Fulano(1)'`. Todos servem de alvo para o `@`.

---

## `vehicles`

```powershell
scum vehicles
```

```
2 veiculo(s):
    590005 |        2 m | BPC_Rager_C_2147337983  <- voce esta neste
    140073 |      374 m | BPC_Tractor_C_2147338019
```

Ordenado por distância. A primeira coluna é o **ID que a interface do jogo
mostra** — use ele em `get` e `set`.

A marcação `<- voce esta neste` sai da cadeia de montaria, que é determinística:
o slot onde você senta mora dentro do veículo, então subir pelo `Outer` chega
nele em um salto.

> Veículos aquáticos aparecem com ID `0`. O identificador deles não fica no mesmo
> lugar; ainda não mapeado.

---

## `veiculos_listar` — dono, tranca e nome dos veículos

**Validado em 15/11/2026** no servidor de desenvolvimento, com a 0.7.0:
devolveu o dono (perfil), o SteamID do dono online e o número de trancas;
tranca tirada às 11:28:55 e recolocada às 11:29:41 apareceu na leitura
seguinte; o dono permaneceu ao tirar a tranca.

```
veiculos_listar
```

Sem argumentos. Não precisa de jogador conectado. É a consulta feita para um
painel chamar em laço, pela porta Source ou pela de linha.

```
veiculos: <n>
veiculo;<id>;<classe>;<x>;<y>;<z>;<perfil_dono>;<steamid_dono>;<trancas>;<nome>
```

```
veiculos: 2
veiculo;510019;BPC_RIS;309403;368051;21249;1;76561190000000000;1;
veiculo;500263;BPC_Rager;309410;368060;21250;0;0;0;
```

| Campo | Significado |
|---|---|
| `id` | o mesmo ID que a interface mostra e que `DestroyVehicle <id>` aceita |
| `classe` | classe do veículo, sem o `_C` |
| `x`, `y`, `z` | posição em cm, sem casas decimais |
| `perfil_dono` | ID do perfil do dono (`user_profile.id` no `SCUM.db`); `0` = sem dono |
| `steamid_dono` | SteamID do dono **só se ele estiver online**; senão `0` |
| `trancas` | quantidade de trancas instaladas; `0` = destrancado |
| `nome` | nome dado pelo jogador; vazio se não tiver. `;` e quebras de linha viram espaço |

- Separador `;`, uma linha por veículo, UTF-8. No máximo 10 colunas.
- A ordem das linhas não é garantida. Use o `id` como chave.
- **Só enxerga veículo carregado em memória**, ou seja, perto de algum
  jogador. O resto só existe no `SCUM.db`. Sem nenhum carregado, a resposta é
  só `veiculos: 0`. Veículo que some da resposta não foi destruído: o jogador
  se afastou.
- **SteamID `0` com dono offline.** Resolva o dono pelo `perfil_dono`, no
  banco; não dependa de `steamid_dono`.
- **Conta como do jogador só com `trancas` > 0.** O jogo mantém o
  `perfil_dono` depois de a tranca sair (validado: tirar a tranca deixou
  `perfil_dono = 1` e `trancas = 0`).
- **Não grava nada em arquivo**: nem a resposta, nem (pela porta Source) a
  chamada no `bsbr_scum.log`. Quem quiser histórico guarda do lado de lá.
- Se um patch do jogo renomear os campos do contêiner, `perfil_dono` e
  `trancas` passam a vir `0` em todos, sem erro.

Dono, tranca e nome não moram no ator do veículo, e sim no contêiner dele
(`_itemContainer`); ver [classes.md](../classes.md#vehicle).

**Intervalo:** 30 segundos em operação normal; 10 segundos no mínimo, e só
enquanto alguém estiver olhando. Nunca mais rápido: o comando roda na thread
do jogo. Uma chamada serve o painel inteiro; não chame por jogador nem por
tela.

**Custo:** uma passagem pela lista de atores do mundo, mais três leituras por
veículo carregado. Nada por tick. Cresce com o número de veículos carregados
(teto: a frota inteira). Medido em 15/11/2026: **0,38 ms com 3 veículos e
0,91 ms com 26** (cerca de 0,023 ms por veículo). O `vehicles`, que grava cada
linha em log, deu 1,1 ms e 2,6 ms nas mesmas condições.

Falta validar: o campo `nome` com um veículo nomeado, e veículo aquático (no
`vehicles` ele aparece com ID `0`).

---

## Alvos

`get`, `set`, `call` e `search` recebem um **alvo**. Cinco formas:

| Alvo | Resolve para |
|---|---|
| `player` | Seu prisioneiro (ou o do `@`, se houver) |
| `player <nome>` | O prisioneiro daquele jogador |
| `vehicle` | O veículo em que você está; se não estiver em nenhum, o mais próximo |
| `vehicle <id>` | O veículo com esse ID da interface |
| `vehicle <nome>` | Por trecho do nome da instância — `BPC_Rager_C_2147329237` |
| `class <caminho>` | O **molde** da classe: o que escrever vale para instâncias criadas **depois** |
| `object <nome ou caminho>` | Qualquer objeto carregado — aceita nome parcial |

Para `vehicle`, o alvo real é o componente de movimento (`GetMovementComponent`),
não o ator — é lá que moram motor, câmbio e massa.

> **`vehicle` sem filtro já acertou um trator e um avião** antes da cadeia de
> montaria funcionar. Na dúvida, confirme com `vehicles` e mire pelo ID.

---

## `get` — ler o estado

```powershell
scum get vehicle 590005
scum get player Fulano
scum get class /Game/ConZ_Files/Vehicles/Car/Rager/BPC_Rager.BPC_Rager_C
scum get object /Game/ConZ_Files/Skills/DrivingSkill.DrivingSkill
scum get object BP_Rager_MountSlot_Seat_FrontLeft_C_2147337373
```

Grava em `bsbr_scum_target.txt`, na pasta `Win64` do servidor. Uma linha por
propriedade, com tipo, caminho e valor:

```
float    EngineData.MaxRPM                    7000
struct   GearboxData
array    GearboxData.ForwardGears             [8]
float    GearboxData.ForwardGears[0].Ratio    4.7
byte     Role                                 3
enum     _repMount._mountFlags                3
name     SlotId.TagName                       MountSlot.Rager.SeatFrontLeft
string   _userId                              76561190000000000
object   _repMount.MountedSlot                BP_Rager_MountSlot_Seat_FrontLeft_C_2147337373
object   _networkPrediction                   <nulo>
```

Tipos lidos com valor: `float`, `int`, `int64`, `uint64`, `bool`, `byte`, `enum`,
`name`, `string` e `object` (mostra o **nome do objeto apontado**). Desce até 3
níveis em struct e abre array de struct. Os demais aparecem com `-` — o nome está
lá, o valor não.

**Enums saem como número.** `Role` e `RemoteRole` seguem `ENetRole`: `0` nenhum,
`1` proxy simulado, `2` proxy autônomo, `3` autoridade.

**Referências mostram o nome, não descem.** Descer viraria recursão sem fim. Para
abrir o objeto apontado, use o nome dele em `get object` — aceita nome parcial.

---

## `set` — escrever

```powershell
scum set vehicle 590005 EngineData.MaxRPM 9000
scum set vehicle 590005 GearboxData.FinalRatio 2.5
scum set vehicle 590005 GearboxData.ForwardGears[7].Ratio 0.4
scum set class /Game/ConZ_Files/Vehicles/Car/Rager/BPC_Rager.BPC_Rager_C _driveComponent.ChassisMass 300
```

O caminho aceita **ponto** para entrar em struct e **colchete** para indexar
array. Também atravessa referência de objeto — é assim que se chega no
`_driveComponent` a partir do molde do ator.

Aceita **float, int e bool**. Tipos compostos ficam de fora de propósito:
escrever tamanho errado num ponteiro é corrupção de memória, não erro tratável.

O alvo pode ter duas palavras (`vehicle 590005`), então caminho e valor são lidos
**do fim para o começo** — analisar do começo fazia o ID virar nome de
propriedade.

### O que `set` não consegue mudar

**Física de veículo.** Massa, marchas, motor, velocidade — nada disso responde.
Três vias testadas, três fechadas:

| Via | Resultado |
|---|---|
| Escrever na instância viva | O PhysX monta o veículo no spawn e nunca mais relê |
| Escrever no `class` e spawnar | A escrita **propaga** (verificado), mas o cliente simula com os valores dele |
| Console da Unreal via `Exec` | *"Command is disabled in shipping build."* |

Confirmado no ator: `bReplicateMovement = 0` — o servidor não replica o movimento
do veículo. Quem simula é o cliente do motorista. Mudar isso exigiria mod em cada
máquina.

Vale **só para física**. O resto do estado do veículo — combustível, bateria,
vida, tranca — é autoritativo no servidor e responde normalmente.

---

## `call` — chamar função

```powershell
scum call vehicle 590005 StartEngine
```

Só funções **sem parâmetros**. O retorno sai no log interpretado como float e
como int, já que o tipo não é conhecido de antemão.

Funções nativas de `AActor` como `GetOwner` **não são encontradas** — não são
UFunction refletida. Para essas, leia a propriedade equivalente com `get`; e
quando o objeto não é `Actor`, a posse está no `Outer`, não em `Owner`.

---

## `find`, `search` e `inspect` — descobrir

```powershell
scum find Rager
scum search vehicle 2450
scum inspect /Script/SCUM.VehicleBase
```

`find` lista classes e objetos carregados cujo nome contém o texto.
`search` procura uma propriedade com determinado valor dentro de um alvo.
`inspect` despeja a reflexão completa de uma classe: propriedades, funções, herança.

> Só enxergam o que está **carregado em memória**. Classe de asset nunca usado não
> aparece. Carregar sob demanda foi tentado e derrubava o servidor
> (`Package_LoadSummary has zero prerequisites`), então foi removido.

Estes três varrem todos os objetos do servidor — é o trabalho deles. São
ferramentas de investigação, não de automação; ver a seção de desempenho.

---

## `spawn` — criar ator pela engine

```powershell
scum spawn /Game/ConZ_Files/Vehicles/Car/Rager/BPC_Rager.BPC_Rager_C 0 0 5000
```

Vai direto no `BeginDeferredActorSpawnFromClass`, sem passar pelo comando
`SpawnVehicle` do jogo — logo, **sem verificação de permissão**. Só aceita classes
já carregadas.

Útil junto com `set class`: configure o molde, depois spawne.

Para spawnar perto de um jogador, prefira o comando do jogo com `as`, que já nasce
na posição dele:

```powershell
scum as Fulano SpawnVehicle BPC_Rager
```

---

## `sudo` — destravar nível Developer

```powershell
scum sudo SpawnBrenner
scum as Fulano sudo SpawnBrenner
```

O nível exigido é propriedade **do comando**, não do jogador. O mod rebaixa
`_requiredExecutorLevel` no molde da classe, executa e restaura no mesmo tick.

Destrava os 105 comandos de nível 3 e 4 — ver [developer.md](developer.md).

> **Cuidado.** Entre os destravados estão `CrashMajestically`, `CrashClient` e
> `DisableServer`. Um erro de digitação derruba o servidor, e o `DisableServer`
> bloqueia até a sua própria reconexão.

**`SpawnBrenner` exige o `as`.** O comando do jogo não tem parâmetro de jogador:
o Brenner nasce no contexto de quem invoca, e pela porta Source isso seria o
primeiro jogador vivo, que não pediu nada. Desde a 0.4.0, `sudo SpawnBrenner`
sem `as <SteamID>` responde o uso e não executa. Para remover, ver
[`DestroyAllBrennersWithinRadius`](#destroyallbrennerswithinradius--remover-brenners).

---

## `DestroyAllBrennersWithinRadius` — remover Brenners

```powershell
scum as 76561190000000000 DestroyAllBrennersWithinRadius
scum as 76561190000000000 DestroyAllBrennersWithinRadius 100
```

```
como Fulano:
2 Brenner(s) destruido(s) num raio de 50 m
```

O SCUM não tem comando que remova o Brenner: ele não é zumbi, animal nem NPC
armado, então `DestroyZombiesWithinRadius`, `DestroyAllAnimalsWithinRadius`,
`DestroyArmedNPCsWithinRadius` e `DestroyEntity` passam por ele. Este comando é
do mod e faz o que falta: acha os atores `BP_Brenner_C` num raio do jogador e os
destrói, tirando colisão e desenho antes para o cliente não ficar com um Brenner
fantasma.

Mesma forma do `DestroyAllRazorsWithinRadius` do jogo, com duas diferenças:

- **O jogador é obrigatório**, pelo `as <SteamID>`. Não existe
  `Location`: o centro é sempre esse jogador. Sem `as`, responde o uso e não faz
  nada.
- **`Radius [m]` é opcional**, padrão **50**, de 1 a 2000. Fora da faixa ou não
  numérico, responde o uso.

| Situação | Resposta |
|---|---|
| Sem `as` | `uso: as <SteamID> DestroyAllBrennersWithinRadius [Radius [m]]  (1 a 2000, padrao 50)` |
| SteamID de quem não está conectado | `jogador '…' nao encontrado ou desconectado` (o próprio `as` recusa) |
| Jogador morto ou carregando | `o jogador do 'as' nao tem prisioneiro no mundo agora (morto ou carregando)` |
| Nenhum no raio | `nenhum Brenner num raio de 50 m` |
| N no raio | `N Brenner(s) destruido(s) num raio de 50 m` |

Custo: uma passagem pela lista de atores do mundo por chamada, mais três
chamadas por Brenner destruído. Nada por tick. Serve para limpar depois de um
evento com `DestroyAllBrennersWithinRadius 2000`.

---

## `zona_listar`, `zona_criar`, `zona_remover` — zonas personalizadas (em teste)

**Em teste.** `zona_listar`, `zona_criar` e `zona_remover` foram exercitados
no servidor de desenvolvimento em 11/11/2026 (0.5.2), com um admin online, na
forma antiga (sem título). O **id separado do título** da 0.5.4 foi validado
no mesmo dia: zona criada com título acentuado, o aviso de entrada do jogo
mostra o título, e a linha no `SCUM.db` saiu correta. **Sem controle de
sobreposição (0.7.0): validado em 15/11/2026**, com uma zona criada
exatamente sobre o Outpost B4 (ver [`zona_criar`](#zona_criar)). O
`zona_esquecer` ainda **não foi exercitado no servidor**. Teste primeiro num
servidor de desenvolvimento, com cópia do `SCUM.db`.

```
zona_listar
zona_criar <id> <x> <y> <raio_cm> <configuracao> <titulo>
zona_remover <id>
zona_esquecer
```

O jogo não tem "criar uma região": o editor de zonas do admin reenvia o
**conjunto inteiro** (configuração global, configurações e regiões) a cada
salvamento. O mod faz o mesmo. Ele escuta, sem interferir, as duas funções que
carregam esse conjunto (o admin salvando no editor, e o servidor distribuindo
o que aplicou), guarda uma **cópia fiel** do último conjunto e, para criar ou
remover uma região, manda uma segunda cópia alterada pela mesma função que o
editor usa. Tudo o que o mod não entende (regras de dano, cor, dono da zona)
volta ao jogo exatamente como veio.

### De onde o mod tira a lista: o editor e o `zonas.cache`

O jogo só mostra o conjunto de zonas quando alguém **salva no editor de zonas**
(Esc › configurações do servidor › zonas). Nada passa no boot nem na entrada
de jogador. Para não depender disso a cada subida do servidor, o mod guarda a
lista em disco:

- Toda vez que o servidor **aplica** um salvamento de zonas (feito no editor
  ou pelo próprio mod), a lista vai para `ue4ss\Mods\bsbr_scum\zonas.cache`.
- Quando o servidor sobe, o mod lê o arquivo. O `zona_listar` mostra
  `origem=arquivo` e, em `ultima`, a data em que o arquivo foi gravado.
  `zona_criar` e `zona_remover` funcionam a partir dessa lista como
  funcionariam a partir de uma vista ao vivo.
- No primeiro salvamento visto ao vivo, a lista do arquivo é trocada pela
  nova e a origem volta a ser `multicast`.

Salvar uma vez no editor (sem mudar nada) **só é preciso**:

- na primeira vez que o mod roda no servidor;
- depois de um `zona_esquecer`;
- depois de uma atualização do **jogo** que mude o formato das zonas: o
  arquivo só vale para o mesmo layout e é recusado fora disso. Atualizar só o
  mod não recusa mais o arquivo (0.5.4);
- com `[zonas] cache = 0` no `config.ini`: aí é a cada subida, como antes.

Sem lista, os comandos respondem:

```
erro: o mod ainda nao viu a lista de zonas; abra o editor de zonas no jogo e salve uma vez
```

Arquivo que o mod não consegue ler por inteiro (layout
diferente, truncado, contagem que não fecha, região apontando para
configuração que não existe) é **recusado**: vira `zonas.cache.recusado`, sai
um aviso `[bsbr_scum][zona]` no `UE4SS.log`, e o mod segue como se nunca
tivesse visto a lista. Nada de um arquivo recusado é usado.

O que o arquivo guarda: tudo o que o jogo mandou, campo a campo, inclusive o
que o mod não entende (regras de dano, cor, textos, dono da zona). O dono
(`user_id` no `SCUM.db`) é gravado e remontado pela própria Unreal; se na
leitura ele não voltar idêntico, o arquivo inteiro é recusado em vez de a zona
voltar com outro dono.

### `zona_esquecer` — quando o arquivo pode estar velho

O arquivo é a lista **como o mod a viu pela última vez**. Se as zonas mudarem
sem o mod ver (o servidor rodou sem o mod, o `SCUM.db` foi apagado ou
restaurado), o arquivo fica velho. Como o jogo só aceita a lista inteira, um
`zona_criar` em cima de arquivo velho **trocaria as zonas do servidor pela
lista antiga**. O mod não tem como perceber sozinho: ele não consegue ler as
zonas do jogo.

```
zona_esquecer
```

| Situação | Resposta |
|---|---|
| A lista veio do arquivo (`origem=arquivo`), ou não há lista | `ok lista de zonas esquecida` — a lista sai da memória e o `zonas.cache` é apagado |
| A lista foi vista ao vivo (`origem=multicast` ou `update`) | `ok nada a esquecer (lista vista ao vivo)` — nada muda |
| A lista saiu da memória, mas o arquivo não pôde ser apagado | `erro: lista esquecida na memoria, mas o arquivo zonas.cache nao pode ser apagado; ...` |

Depois do `ok lista de zonas esquecida`, `zona_criar` e `zona_remover`
respondem `erro: o mod ainda nao viu a lista de zonas; ...` até alguém salvar
no editor.

O `zona_esquecer` **não mexe no `zonas.ids`**: o mod esquece a lista, não
quais zonas ele criou. Quando a lista voltar, elas continuam marcadas `mod` e
removíveis pelo `id`.

Uso esperado, pelo painel: depois de o servidor subir, se o `zona_listar`
vier com `origem=arquivo`, comparar as regiões e configurações com
`custom_zone_region` e `custom_zone_configuration` do `SCUM.db`. Se diferirem,
mandar `zona_esquecer` **antes** de qualquer `zona_criar` ou `zona_remover`.
À mão: na dúvida, apague o `zonas.cache` com o servidor parado.

A chave do `config.ini`:

```ini
[zonas]
cache = 1     ; 1 (padrão) grava e lê o zonas.cache; 0 não grava nem lê
```

Com `cache = 0`, um `zonas.cache` que tenha sobrado é apagado na subida.

### Id e título: quem é quem

O `zona_criar` recebe dois nomes, e eles não se misturam:

| | Exemplo | Para quê |
|---|---|---|
| `id` | `evento_42` | chave de quem chama (o painel). 1 a 48 caracteres de `[A-Za-z0-9_-]`. **O jogador nunca vê.** É o que se manda no `zona_remover`. |
| `titulo` | `"Arena do Brenner"` | o **nome da região dentro do jogo**, gravado exatamente como veio. |

O título existe porque o próprio jogo o mostra (medido em 11/11/2026): quando
o jogador cruza a borda aparece um aviso curto, no idioma do cliente dele,
"Você entrou na zona **<nome da região>**". Na saída o aviso é "Você saiu da
zona", sem nome. Até a 0.5.3 o nome era `bsbr_evento_42`, e era isso que o
jogador lia.

Regras do título:

- 1 a 48 caracteres, contados depois de tirar os espaços das pontas; entre
  aspas no comando. Aceita espaço e **acento**: mande o comando em **UTF-8**.
- Sem `;`, sem aspas dentro e sem caractere de controle. Título só de espaços
  é recusado.
- Não pode ser igual (sem diferenciar maiúsculas) ao nome de **outra** zona,
  seja do editor, seja de outro `id`. Dois eventos ao mesmo tempo precisam de
  títulos diferentes (`Arena do Brenner 2`).
- Para trocar o título de uma zona, repita o `zona_criar` com o mesmo `id`.

#### `zonas.ids`: como o mod sabe quais zonas são dele

Sem o prefixo `bsbr_`, o que marca uma zona como "criada pelo mod" é um
registro `id → título`, guardado em `ue4ss\Mods\bsbr_scum\zonas.ids`: texto
UTF-8, uma zona por linha, `id<TAB>titulo`.

```
evento_42	Arena do Brenner
evento_43	Caça ao Tesouro
```

- Uma região é **do mod** quando o nome dela é **igual** (maiúsculas e acentos
  contam) ao título de uma linha do registro. O `zona_remover` só chega a uma
  região por aí: zona do editor nunca é tocada.
- O registro só muda depois de o jogo confirmar a zona, e é regravado a cada
  mudança. É lido uma vez, quando o servidor sobe.
- Não depende do `zonas.cache` nem de `[zonas] cache`, e o `zona_esquecer`
  não o apaga.
- O `id` não diferencia maiúsculas (`Evento_42` e `evento_42` são a mesma
  zona); volta no `zona_listar` como foi escrito no último `zona_criar`.
- Arquivo ausente ou ilegível: um aviso `[bsbr_scum][zona]` no `UE4SS.log` e
  registro vazio (o ilegível vira `zonas.ids.recusado`).

**Se o `zonas.ids` se perder**, as zonas que o mod criou continuam no jogo,
mas o mod passa a tratá-las como do editor: `zona_remover` responde
`ok ... nao existia` e `zona_criar` com o mesmo título é recusado por título
repetido. Apague-as à mão no editor de zonas. O mesmo vale para a zona do mod
que um admin **renomear** no editor.

### `zona_listar`

```
zonas: update=<n> multicast=<n> ultima=<AAAA.MM.DD-HH.MM.SS> origem=<update|multicast|arquivo>
config;<índice>;<nome>;<r>;<g>;<b>
regiao;<nome visível>;<x>;<y>;<tam_x>;<tam_y>;<forma>;<índice da configuração>;<mod|editor>[;<id>]
```

```
zonas: update=2 multicast=2 ultima=2026.11.11-20.18.42 origem=multicast
config;0;Outposts;0.319;0.784;0.080
config;1;Evento PVE;0.061;0.631;0.776
regiao;Outpost B4;570708.000;-226174.000;30000.000;0.000;0;0;editor
regiao;Arena do Brenner;-113084.000;-70633.000;60000.000;60000.000;0;1;mod;evento_42
```

- `update` e `multicast` contam quantas vezes cada função disparou desde que o
  servidor subiu; `ultima` é a hora local da última cópia e `origem` diz de
  onde ela veio. Vale a do `multicast` (o que o servidor aplicou); a do
  `update` só é guardada enquanto não houver nem `multicast` nem lista lida do
  arquivo.
- Logo depois de um reinício, com a lista lida do `zonas.cache`:
  `zonas: update=0 multicast=0 ultima=<data do arquivo> origem=arquivo`.
- `<índice>` da configuração é a **posição** dela na lista (Outposts = 0), e é
  esse número que a região guarda em `<índice da configuração>`.
- Círculo feito no editor ou pelo mod: `tam_x` = `tam_y` = raio em cm, forma 0.
- No fim de cada região, `mod` ou `editor`; a do mod traz ainda o `id`.
- Acento na **resposta**: UTF-8 desde a 0.6.0 (antes, um byte por caractere,
  Latin-1). Para achar a zona, use o `id`, não o texto do título.
- Só lê. Não precisa de jogador conectado.

### `zona_criar`

```
zona_criar evento_42 -113084 -70633 60000 "Evento PVE" "Arena do Brenner"
zona_criar evento_43 250000 -15000 80000 Evento_PVP "Caça ao Tesouro"
```

Os **seis** dados são obrigatórios; faltando ou sobrando algum, o comando
responde o uso e não faz nada.

- `id` e `titulo`: ver [Id e título](#id-e-título-quem-é-quem).
- `id` que o mod já registrou: o comando **atualiza** centro, raio,
  configuração e título da mesma região. Se a região dele tiver sido apagada
  ou renomeada no editor, cria outra. Chamar duas vezes é seguro.
- `x`, `y`: centro em cm do mundo (os mesmos números do `Teleport`).
- `raio_cm`: de 100 a 1500000.
- `configuracao`: **nome** de uma configuração que já existe no jogo (criada
  no editor, com a cor e as regras). Com espaço, use aspas ou troque o espaço
  por `_`. O mod procura primeiro o nome exato e depois a forma com `_` → espaço,
  sem diferenciar maiúsculas. Nome com acento só casa se o comando chegar em
  UTF-8; para não depender disso, dê às configurações de evento um nome sem
  acento.
- Precisa de **um jogador conectado**, para emprestar o canal por onde o
  editor fala com o servidor (o do `as`, ou um admin conectado, ou qualquer
  jogador). O jogo pode recusar o salvamento vindo de quem não é admin.

**O mod não confere sobreposição (desde a 0.7.0).** O `zona_criar` aceita
um círculo que entra em outra região, do editor ou de outro `id`. Quem decide
se uma sobreposição é aceitável é quem chama, com a lista do `zona_listar`.
Da 0.5.5 à 0.6.0 o mod recusava com `erro: a zona invadiria "..."`; essa
resposta não existe mais.

Validado em 15/11/2026, exatamente sobre o Outpost B4 (mesmo centro; o
outpost tem raio 30000):

```
zona_criar teste 570708 -226174 20000 "Evento PVE" "Teste Sobreposição"   → ok zona teste criada (...)
zona_criar teste 570708 -226174 30000 "Evento PVE" "Teste Sobreposição"   → ok zona teste atualizada (...)
zona_remover teste                                                       → ok zona teste removida
zona_remover teste                                                       → ok zona teste nao existia
```

A zona apareceu no `zona_listar` junto com o outpost, e o título com acento
funcionou. Não foi verificado dentro do jogo qual regra vale na área
sobreposta.

```
ok zona evento_42 criada (titulo "Arena do Brenner", configuracao Evento PVE, indice 1, raio 60000 cm)
ok zona evento_42 atualizada (titulo "Arena do Brenner", configuracao Evento PVP, indice 2, raio 30000 cm)
```

A resposta traz o `id`, não o título, logo depois de `ok zona`. O `ok` só sai
se o servidor **distribuiu o conjunto durante a chamada** e a cópia nova traz
a região com o título e os valores pedidos; só então o `zonas.ids` é
regravado. Se essa gravação falhar, a linha continua começando com `ok` (a
zona está no jogo) e termina com `; aviso: zonas.ids nao gravado, ...`.

### `zona_remover`

```
zona_remover evento_42
```

```
ok zona evento_42 removida
ok zona evento_42 nao existia
```

- `id` que o mod não registrou: `ok ... nao existia`, sem enviar nada e sem
  precisar da lista de zonas nem de jogador conectado.
- `id` registrado cuja região já não está na lista (apagada ou renomeada no
  editor): a linha sai do `zonas.ids` e a resposta é a mesma, sem enviar nada.
- `id` registrado com a região lá: ela é removida (todas as de nome idêntico,
  se houver repetida), com as mesmas exigências do `zona_criar`.

Responder `ok` para o que não existe serve para limpar no fim do evento e ao
subir o painel, sem precisar saber se a zona ainda estava lá.

### Erros

Toda recusa é uma linha começando com `erro:`, e **nada é enviado ao jogo**
(a não ser na última linha da tabela).

| Situação | Resposta |
|---|---|
| O mod não tem a lista: sem `zonas.cache` válido e ninguém salvou no editor desde que o servidor subiu | `erro: o mod ainda nao viu a lista de zonas; abra o editor de zonas no jogo e salve uma vez` |
| Número errado de argumentos | `erro: uso: zona_criar <id> <x> <y> <raio_cm> <configuracao> <titulo> ...` / `erro: uso: zona_remover <id>` |
| `id` vazio, com mais de 48 ou com caractere fora de `[A-Za-z0-9_-]` | `erro: id invalido: de 1 a 48 caracteres, so letras, numeros, _ e -` |
| Título vazio, só de espaços, com mais de 48, com `;`, aspas ou caractere de controle | `erro: titulo invalido: <motivo> (de 1 a 48 caracteres, sem ';' e sem aspas)` |
| Título igual ao nome de outra zona (do editor ou de outro `id`) | `erro: ja existe uma zona chamada "<titulo>"` |
| `x`, `y` ou `raio_cm` que não é número | `erro: x, y e raio_cm tem de ser numeros (ex.: -113084 -70633 60000)` |
| `x` ou `y` além de 10000000 cm | `erro: x ou y fora do mapa (limite de 10000000 cm para cada lado)` |
| Raio fora de 100 a 1500000 | `erro: raio_cm fora da faixa de 100 a 1500000` |
| Configuração que não existe | `erro: configuracao "X" nao existe` |
| Nenhum jogador conectado | `erro: nenhum jogador conectado para emprestar o canal RPC; nada enviado` |
| Mais de 256 regiões no servidor | `erro: o servidor ja tem 256 regioes, o limite do mod; nada enviado` |
| O jogo mudou a assinatura das funções de zona (patch) | `erro: o gancho de zona disparou (...) mas o conjunto nao foi copiado (ver UE4SS.log)` |
| O conjunto foi enviado e o servidor não o distribuiu de volta | `erro: o jogo nao confirmou a zona (salvamento recusado ou adiado); nada garantido` |

No último caso o mod não sabe se a zona entrou: confira com `zona_listar` ou
no mapa do jogo. O motivo detalhado fica no `UE4SS.log`, em linhas
`[bsbr_scum][zona]`.

### Custo

Nada por tick. Por salvamento no editor, até duas cópias do conjunto de zonas
(uma, depois do primeiro `multicast`). Por `zona_criar` ou `zona_remover`, uma
cópia do conjunto e uma chamada ao jogo. `zona_listar` só percorre a cópia.
Medido em 15/11/2026: `zona_criar` 7,6 ms, `zona_remover` 4,2 ms,
`zona_listar` 0,06 ms.
O `zonas.cache` custa uma gravação pequena a cada salvamento de zonas que o
servidor aplica (raro) e uma leitura quando o servidor sobe. O `zonas.ids`,
uma gravação menor ainda por `zona_criar` ou `zona_remover` que mudou algo, e
uma leitura quando o servidor sobe.

---

## `eventos` — fila de acontecimentos em memória

**Validado em 15/11/2026** no servidor de desenvolvimento, com a 0.7.0: 41
NPCs armados mortos em 27 s (14:24:14 a 14:24:41), 41 recebidos, 0 perdidos,
com SteamID, perfil, nome, arma (`Weapon_AK47`) e posição; a confirmação
(`eventos <ultimo>`) esvaziou a fila. Estresse: 161 milhões de eventos de
teste em 14,4 min, sem queda, sem erro e sem crescimento de memória. Na build
final, `npc_morto` validado de novo: 3 guardas mortos na faca, 3 eventos, arma
`1H_Military_Survival_Knife`.

```
eventos [<ultimo>]
```

O mod guarda numa fila em memória os acontecimentos que o painel precisa
gravar (hoje, a morte de NPC armado). O painel busca a cada 1 ou 2 segundos
pela porta Source e grava no banco dele. Nada vai para arquivo.

`<ultimo>` é a maior sequência que quem chama **já gravou**; `0` ou ausente =
nada. O mod descarta da fila tudo até `<ultimo>` (confirmação de recebimento)
e devolve o que vem depois, em ordem. `<ultimo>` maior que a última sequência
desta subida (número de outra época): nada é descartado, vai tudo.

```
eventos: <n> epoca=<epoca> ultimo=<maior sequencia emitida> perdidos=<p> mais=<0|1>
<seq>;<hora>;<tipo>;<campos...>
```

```
eventos: 1 epoca=1763215790 ultimo=42 perdidos=0 mais=0
42;2026.11.15-14.02.11;npc_morto;BP_Guard_Lvl_1;2147482345;76561190000000000;1;Fulano;Weapon_M9;140508.703;-440821.125;9946.081
```

- `seq` começa em 1 a cada subida do mod; `epoca` (segundos Unix da subida)
  muda junto, para quem chama perceber o reinício e zerar o `ultimo`.
- `hora` é a hora **local** do servidor, `AAAA.MM.DD-HH.MM.SS`.
- Fila de 4096 registros. Cheia, sai o mais antigo e conta em `perdidos`, que
  zera depois de reportado.
- Até 500 linhas e 48 KB por resposta; o resto vem com `mais=1`, chamar de novo.
- Separador `;`, UTF-8. Texto livre (nome de jogador) chega com `;` e quebras
  de linha trocados por espaço.
- Sem novidade, só o cabeçalho com `eventos: 0`.
- Servidor que cai perde o que ainda não foi buscado: é memória.

### Evento `npc_morto` (substitui o `logs/npc_kills.log`)

**Mudança incompatível da 0.7.0:** o `logs/npc_kills.log` deixou de existir.
O mod não cria mais a pasta `logs` nem escreve o arquivo; a mesma informação
sai como evento na fila.

```
<seq>;<hora>;npc_morto;<classe do npc>;<npc_id>;<steamid>;<perfil>;<nome>;<arma>;<x>;<y>;<z>
```

- Um evento por NPC armado cujo último dano veio de um jogador. Morte sem
  jogador (outro NPC, queda, sentinela) não gera evento.
- `steamid`, `perfil` e `nome` são de quem matou; `arma` é a classe da arma,
  de fogo (`Weapon_AK47`) ou branca (`1H_Military_Survival_Knife`), ou
  `Unknown` quando o jogo não informa; `npc_id` é o número da instância do NPC, ou `0`;
  coordenadas em cm, três casas.
- Custo: uma chamada por morte de NPC armado, zero por tick.
- A regra de quando registrar foi **validada no servidor em 11/11/2026**, ainda
  gravando no arquivo; pela fila, **validada em 15/11/2026** (41 de 41 e, na
  build final, 3 de 3 na faca). O `npc_kills.log` não é mais escrito.

### `eventos_teste <n>` — carga de teste

Diagnóstico: põe `n` eventos falsos (1 a 20000) na fila, como se chegassem
juntos, e responde `ok <n> eventos de teste na fila (tamanho <t>, perdidos <p>)`.

```
<seq>;<hora>;teste;<indice>;carga-de-teste
```

Custo: `n` registros montados de uma vez, fora da thread do jogo (comando
local, ver [Desempenho](#desempenho)). O script
`scripts/estresse_eventos.py` do repositório do mod usa isto para medir.

### Tipos novos de morte

Além do `npc_morto`, a 0.7.0 tem um gancho por tipo, instalado na subida.
Estado final em 15/11/2026 (build final): `npc_morto`, `jogador_morto` e
`jogador_renasceu` **validados**; os outros são candidatos (a função do jogo
existe, ninguém a viu disparar). Função ausente ou com outra assinatura:
aviso no `UE4SS.log` e o tipo fica sem gancho.

| Tipo | Função do jogo |
|---|---|
| `jogador_morto` | `ConZPlayerController:Server_RequestRespawnWaitTime` (o cliente pede o tempo de renascimento ao abrir a tela de morte); vítima = `Pawn` do controller. Validado: um evento por morte, no mesmo segundo do log do jogo, sem repetição (morte duplicada corrigida e validada, abaixo) |
| `jogador_renasceu` | `Prisoner:Server_RespawnAtCommonSpawnLocation` (o mod procura outras `Prisoner:Server_Respawn*` na subida; não há); vítima = o personagem. Validado |
| `sentinela_destruida` | `Sentry2:NetMulticast_OnSentryDeath`; candidato, nunca exercitado |
| `razor_morto` | `Razor:NetMulticast_OnRazorKilled`; candidato, nunca exercitado |
| `nave_destruida` | `Dropship:NetMulticast_OnDropshipDeath`; candidato, nunca exercitado |
| `animal_morto` | `HuntingManager:OnAnimalKilled` (parâmetro `ConZCharacter`); candidato que **não dispara** para animal criado por comando (0 disparos com 8 animais de `SpawnRandomAnimal`); não testado com animal do próprio jogo |

**`zumbi_morto` foi removido.** Nenhuma função de `Zombie2`,
`ConZPlayerController`, `Prisoner` ou `ConZCharacter` dispara quando o
jogador mata um zumbi (10 abates por classe sondada), e os contadores do
jogador não são legíveis por reflexão. Zumbi e animal mortos por jogador se
contam pelo `SCUM.db` (`survival_stats.puppets_killed`, `animals_killed` e
por espécie), que o jogo grava de minuto em minuto.

Criaturas, mesma linha e regras do `npc_morto` (só autor jogador, sem
repetição):

```
<seq>;<hora>;<tipo>;<classe da vitima>;<id da instancia>;<steamid>;<perfil>;<nome>;<arma>;<x>;<y>;<z>
```

Vítima **sem** o campo `LastHitBy` (sentinela e nave podem não ter) é
registrada mesmo assim, com `0;0;-` no matador e `-` na arma.

Jogador:

```
<seq>;<hora>;jogador_morto;<steamid>;<perfil>;<nome>;<causa>;<classe do atacante|->;<classe do pawn do atacante|->;<arma|->;<ameaca proxima|->;<distancia em m|->;<efeitos|->;<x>;<y>;<z>
```

```
61;2026.11.15-18.50.33;jogador_morto;76561190000000000;1;Fulano;zumbi;BP_ZombieAIController2;BP_Zombie_Hospital_Female;-;-;-;BleedingInjury,Bleeding,Infection_Chest,HealthLow,Pain;-113084.125;-70633.500;3120.250
```

- O cliente chama a função duas vezes por vida: uma na morte (ao abrir a
  tela de morte) e outra logo depois de renascer (medido). Por isso a
  proteção é por estado, por personagem: emitido o `jogador_morto`, o
  personagem fica marcado e novos disparos para ele são ignorados, sem limite
  de tempo; o renascimento desmarca (mesmo com `jogador_renasceu` fora da
  captura); um disparo até 5 s depois do renascimento é o eco dele e é
  ignorado. O personagem é o mesmo objeto antes e depois da morte, então a
  morte seguinte dele sai normalmente. Limitações aceitas: com o mod subindo
  com o jogador já na tela de morte, essa morte não sai; morrer de novo menos
  de 5 s depois de renascer não gera evento. A proteção antiga (só 5 s)
  deixava sair um `jogador_morto` falso logo depois do renascimento quando o
  jogador esperava mais de 5 s na tela de morte. **Corrigido e validado em
  15/11/2026**: com o jogador 30 s na tela de morte, saiu `jogador_morto`
  (16:43:41, `queda`) e `jogador_renasceu` (16:44:11), sem `jogador_morto`
  extra (o gancho disparou 2 vezes, o mod emitiu 1 evento); a segunda morte
  do mesmo jogador (16:44:55) saiu normalmente. `morte_teste` não usa nem
  altera esse estado.

- `causa`, nesta ordem: atacante jogador → `jogador`; zumbi → `zumbi`;
  derivado de `Animal` → `animal`; NPC armado → `npc`; Razor → `razor`;
  Brenner → `brenner`; outro atacante → `desconhecida`. Sem atacante:
  `BurnInjury*` → `explosao` (`sentinela` com `Sentry2` a até 150 m),
  `KnockoutWithBlackout` ou `PrisonerBodySymptom_Blackout` → `queda`,
  exatamente `Bleeding` (não `BleedingInjury`) → `sangramento`,
  `Hypothermia*` → `frio`, `Hyperthermia*` → `calor`, o resto →
  `desconhecida`.
- Medido em 15/11/2026 (resumo final): zumbi comum (4 vezes; pawns
  `BP_Zombie_Civilian_Muscular_Female`, `BP_Zombie_Military_Armored`) →
  `zumbi`; javali mutante (`BP_Boar_Mutant`) → `animal`; duas quedas de
  320 m, sem atacante → `queda`. Impacto forte de outra origem pode dar o
  mesmo quadro de `queda`. O zumbi kamikaze some do mundo ao explodir e não
  deixa atacante: hoje sai `desconhecida` (não validado depois do ajuste;
  antes saía `sangramento`). Robô (sentinela), Razor, Brenner, NPC, outro
  jogador, fome, sede e frio: a regra existe, **não testada** com o evento. O jogo não tem
  função de dano escutável; a causa só sai do que o personagem mostra na
  hora da morte, e a linha leva os campos crus para quem quiser
  reclassificar.
- `ameaca proxima`: a mais perto a até 150 m entre sentinela, Razor, Brenner
  e veículo (veículo de id `0`, criado pelo `spawn` do mod, não conta).
- `efeitos`: do corpo, sem `_C`, sem repetição, até 24.
- Custo, só na morte: as leituras do `corpo`, quatro `GetAllActorsOfClass` e
  um registro na fila.

Renascimento:

```
<seq>;<hora>;jogador_renasceu;<steamid>;<perfil>;<nome>;<x>;<y>;<z>
```

```
63;2026.11.15-18.51.02;jogador_renasceu;76561190000000000;1;Fulano;-113084.125;-70633.500;3120.250
```

- Chega 2 a 4 s depois da morte, e a posição é a do **local da morte**: o
  jogo ainda não moveu o personagem (o `prisoner revived` do log vem ~2 s
  depois). Validado. Sem proteção contra repetição.
- Custo, só no renascimento: a identidade, uma posição e um registro na fila.

### `eventos_capturar` — o que entra na fila

```
eventos_capturar
eventos_capturar tudo 180
eventos_capturar npc_morto,animal_morto 60
eventos_capturar padrao
```

| Modo | Tipos |
|---|---|
| `padrao` | `npc_morto`, `jogador_morto`, `sentinela_destruida`, `razor_morto`, `nave_destruida`, `jogador_renasceu` |
| `tudo` | o padrão mais `animal_morto` (candidato) |
| lista | exatamente os citados, por vírgula, sem espaço |

- Respostas: `ok captura padrao (<tipos>)`, `ok captura <tudo|lista> (<tipos>) por <n> min`.
  Tipo desconhecido: `erro: tipo desconhecido '<t>'; tipos: ...`, nada muda.
- Fora do `padrao`, volta sozinho ao `padrao` depois dos minutos (1 a 720,
  padrão 120), conferido quando chega um evento ou alguém consulta: sem timer.
- Sem argumento: `captura <modo> (<tipos>) [restam <n> min]` e uma linha
  `gancho;<tipo>;<ativo|ausente>;<disparos>` por tipo, para validar os
  candidatos.
- Comando local (fora da thread do jogo), como `eventos`. Reinício volta ao
  `padrao`.

### `morte_teste <alvo>` — conferir o `jogador_morto`

Diagnóstico. Monta e põe na fila a linha `jogador_morto` de um jogador
**vivo** (SteamID, perfil ou nome), com o que o personagem tem agora, sem
esperar o gatilho. Ignora o modo de captura. Resposta:
`ok evento jogador_morto emitido (causa <causa>)`. Comando de mundo.

---

## `sonda_funcoes` — quais funções de uma classe rodam (diagnóstico)

**Não é para produção.** Serve para achar o gatilho que falta (morte de
jogador, de zumbi) sem o gancho global de `ProcessEvent`.

```
sonda_funcoes /Script/SCUM.Prisoner 60     → ok sonda_funcoes aberta por 60 s: classe=... ganchos=<n> recusados=<r> acima_do_teto=<t>
sonda_funcoes ler
```

- Ao abrir, põe um gancho em cada função **declarada na classe** (não as
  herdadas) cujo nome tenha `NetMulticast`, `Multicast`, `Server_`, `OnRep_`,
  `die`, `death`, `dead`, `kill`, `ragdoll`, `corpse`, `respawn`, `health`
  ou `damage` (sem diferenciar maiúsculas); até 150. Abrir de novo zera a
  coleta anterior.
- O `ler` fecha a janela, **remove os ganchos** e responde:

```
sonda_funcoes: classe=<c> ganchos=<n> duracao=<s>
<ms desde a abertura>;<contagem>;<Funcao>;<objeto da 1a chamada>
```

  só das que dispararam, pela ordem do primeiro disparo (até 48 KB).
- Passado o tempo, os ganchos ficam **inertes** (uma leitura por chamada) até
  o `ler` ou uma nova abertura removê-los.
- Exige `executor=jogo` no `version` (registra e remove gancho na thread do
  jogo); sem ele, `erro: sonda_funcoes exige o executor do jogo ...`.
- Custo com a janela aberta: por chamada de função sondada, um mutex e um
  contador.

---

## `say` — anunciar

```powershell
scum say Servidor reiniciando em 5 minutos
```

Usa `MiscStatics::BroadcastChatLine`. É também por onde as respostas do mod
chegam ao chat.

---

## `anuncio` e `aviso` — aviso no centro da tela

```powershell
scum anuncio 8 Evento iniciado, corram para a area
scum aviso 76561190000000000 6 Voce entrou no evento
scum aviso Fulano Voce saiu do evento
scum anuncio 8 cor:vermelho Servidor reinicia em 5 minutos
scum aviso 76561190000000000 cor:#00FF00 6 Premio entregue
```

- `anuncio <mensagem>` repassa ao `Announce` do jogo (3 s, para todos).
- `anuncio <segundos> <mensagem>` mostra o mesmo aviso, laranja, para todos,
  pelo tempo pedido (1 a 60). Validado em 15/11/2026: `anuncio 15` ficou 15 s.
- `aviso <alvo> [segundos] <mensagem>` mostra o aviso amarelo do
  `SendNotification 2` só para um jogador. `<alvo>` é SteamID, id do perfil ou
  nome. Sem segundos, 5 s. Validado em 15/11/2026: amarelo, só para o
  jogador, frase inteira.
- Nos dois, se a primeira palavra (depois do alvo, no `aviso`) for inteiro e
  houver texto depois, ela é o tempo. A frase vai inteira, sem aspas.
- `cor:<nome|#RRGGBB>` (**validado em 15/11/2026**: as oito cores e
  `#8000FF` apareceram na cor pedida, no `anuncio` e no `aviso`), opcional, antes da mensagem e em qualquer ordem com
  os segundos; só é reconhecida no começo, sem diferenciar maiúsculas. Nomes:
  `vermelho` (255,0,0), `laranja` (255,137,0), `amarelo` (255,255,0),
  `verde` (0,255,0), `azul` (0,150,255), `ciano` (0,255,255),
  `rosa` (255,0,255), `branco` (255,255,255). Sem cor, laranja no `anuncio` e
  amarelo no `aviso`. `anuncio` com cor e sem segundos dura 3 s pelo caminho
  do mod (o `Announce` não aceita cor). Cor desconhecida: `erro: cor "<x>"
  desconhecida; use ... ou #RRGGBB`, nada enviado.
- Respostas: `ok anuncio enviado (<n> s)`, `ok aviso enviado a <perfil> (<n> s)`
  (com cor pedida: `ok anuncio enviado (8 s, cor vermelho)`,
  `ok aviso enviado a 1 (6 s, cor #00FF00)`),
  `erro: jogador "<alvo>" nao esta conectado`, ou `erro: ... mudou ...; nada
  enviado` se um patch do jogo mexer na função ou nos campos do aviso.
- Dois avisos do centro em seguida entram em fila.
- Custo: um objeto criado e uma chamada, só quando o comando roda.

`sonda_aviso` é diagnóstico: liga um gancho que registra no `bsbr_scum.log` o
formato de cada aviso que o jogo distribui. Só lê, mas varre os objetos a cada
aviso; não deixar ligado.

---

## `sonda_morte` — o que roda quando um jogador morre (diagnóstico)

**Nunca ligar em produção.** Serve para descobrir qual função do jogo
acompanha a morte de um jogador (zumbi, animal, fome, queda, veículo...), que o
jogo não registra em log. Desligada por padrão: só existe com as **duas**
chaves abaixo e um reinício.

```ini
; config.ini do mod
[debug]
sonda_morte = 1

; UE4SS-settings.ini — o pacote vem com 0 porque esse desvio já derrubou o
; servidor na entrada de jogador; volte a 0 depois do teste
[Hooks]
HookUObjectProcessEvent = 1
```

Com `sonda_morte = 0` (padrão) nada é registrado no motor: custo zero. Com 1,
o mod põe um pré-callback em **todo** `ProcessEvent` do servidor; com a janela
fechada ele faz uma leitura e volta. Com a janela aberta, cada chamada passa
por um mutex, um mapa e até cinco `IsA` — custo de diagnóstico.

```
sonda_morte 60          → ok sonda aberta por 60 s   (1 a 120; zera a coleta anterior)
sonda_morte ler         → o que foi coletado; fecha a janela se ainda aberta
sonda_morte estado <alvo>  → o mesmo que corpo <alvo>; funciona com a chave em 0
```

Sem a chave, `<segundos>` e `ler` respondem
`erro: sonda desligada; ligue [debug] sonda_morte = 1 e reinicie`.

A janela anota as chamadas cujo objeto é `Prisoner`, `ConZPlayerController`,
`PlayerRpcChannel`, `PrisonerCorpse` ou `Bedroll` (ou derivado), ou cuja função
tenha no nome `die`, `death`, `dead`, `kill`, `corpse`, `respawn`, `revive` ou
`wakeup`. Destas últimas guarda também os parâmetros de entrada (objeto,
classe, texto, inteiro, bool, enum) das 5 primeiras chamadas. Até 400 funções
distintas; as chamadas de função além disso contam em `descartadas`. Tudo em
memória.

Resposta do `ler` (até 48 KB; passou disso, `cortado=1` no cabeçalho):

```
sonda: janela <aberta|fechada> duracao=<s> funcoes=<n> descartadas=<k>
<ms desde a abertura>;<contagem>;<Classe:Funcao>;<objeto da 1a chamada>
param;<Classe:Funcao>;<ocorrencia>;<parametro>=<valor>;...
```

`estado <alvo>` é, desde a 0.7.0, o próprio [`corpo <alvo>`](#corpo--causa-provável-da-morte-em-teste)
(mesma função, mesma resposta). A resposta antiga (`personagem=... classe=...`,
`_damageActorType`, posição) deixou de existir.

Limite: só aparece o que passa por `ProcessEvent` (RPC recebido, multicast,
evento de Blueprint). Chamada C++ direta entre funções nativas não aparece.
Os comandos do próprio mod também passam por ali e podem aparecer na lista.

---

## `corpo` — causa provável da morte (em teste)

**Em teste.** Entrou na 0.7.0. Compila, mas ainda **não foi exercitado no
servidor**.

```
corpo <alvo>
```

`<alvo>` é SteamID, id do perfil ou nome. Só lê; não precisa de chave nenhuma.
Responde uma linha por item:

```
corpo: perfil=<id> steamid=<id> personagem=<nome do objeto>
efeitos: <n>
efeito;<classe sem _C>;<nome do objeto>
...
ultimo_dano;<objeto LastHitBy|->;<classe|->;<classe do Pawn dele|->
arma;<classe _weaponClassLastHitBy|->
imortal=<0|1|-> modo_deus=<0|1|->
vida;<campo>=<0|1>
```

- `efeito` sai uma vez para cada elemento de `_bodyEffects` do
  `_bodySimulationComponent` do personagem (até 64): `Bleeding`, `Hunger`,
  `Thirst`, `Hypothermia`, `Sepsis`, `PrisonerBodyEffect_Death`,
  `Unconsciousness`... Elemento inválido sai `efeito;-;-`.
- Se o jogo mudar o formato desses campos, a linha vira
  `efeitos: layout desconhecido (<tipo encontrado>)` e nada é lido às cegas.
- `ultimo_dano` vem de `LastHitBy` (o controller de quem bateu por último);
  fica `-` quando esse atacante já sumiu do servidor.
- `-` em `imortal`/`modo_deus`: o campo não existe nesta versão do jogo.
- `vida;...` só aparece se o personagem tiver algum campo verdadeiro/falso com
  `dead`, `alive` ou `died` no nome (até 8 linhas). O nome desse campo ainda não
  foi confirmado no servidor.
- Erros: `erro: jogador '<alvo>' nao encontrado; use SteamID, perfil ou nome`,
  `erro: jogador sem personagem agora`.
- Custo: um `controller_named`, até 64 elementos e uma passada pelas
  propriedades do personagem, na thread do jogo. Nada por tick.

---

## `commands` — catálogo do jogo

```powershell
scum commands
```

Gera a reflexão dos comandos de admin do SCUM em `bsbr_scum_comandos.txt`. A
versão organizada e traduzida está nos outros arquivos desta pasta.

---

## `capture` — ler a saída dos comandos

```powershell
scum capture on
```

Os comandos do jogo respondem no chat do cliente, não no log do servidor. Para
ler essa resposta é preciso observar `Chat_Client_SendMessageToChat`.

**Custa caro e não deve ficar ligado.** Exige `HookUObjectProcessEvent = 1` no
`UE4SS-settings.ini`, que desvia **toda** chamada de função do jogo — e esse hook
já derrubou o servidor na entrada de jogador. Nasce desligado.

Prefira ler o estado por reflexão com `get`, que não depende de hook nenhum.

---

## Desempenho

### Onde cada comando roda (0.7.0)

**Validado em 15/11/2026** na build final: `version` mostra `executor=jogo`;
comandos do jogo de níveis 2, 3 e 4 (`SetGodMode`, `SetImmortality`,
`SendNotification`) funcionaram pela porta, conferidos pelo efeito no
personagem; `SpawnBrenner` com `as` e `DestroyAllBrennersWithinRadius`
funcionaram. O `erro: thread do jogo nao respondeu` nunca apareceu em uso
normal; apareceu uma vez, num teste com o servidor a ~5 FPS por causa de 87
veículos inválidos.

Até a 0.6 todo comando rodava na thread do laço de eventos do UE4SS, **em
paralelo com o jogo** (corrida de dados). Desde a 0.7.0:

- **Comandos de mundo** — todos, menos os cinco locais abaixo: `ListPlayers`,
  `players`, `veiculos_listar`, spawn, teleporte, `zona_*`, `anuncio`, `aviso`,
  comandos do jogo, `corpo`... — rodam **na thread do jogo**, **um por tick do
  motor**. O servidor do SCUM roda perto de 30 ticks por segundo, então o
  total é de no máximo **~30 comandos de mundo por segundo**, somando as duas
  portas. A porta de linha passa na frente da porta Source.
- **Comandos locais** — `eventos`, `eventos_teste`, `eventos_capturar`,
  `version` e `sonda_morte ler` — não tocam objeto do jogo e rodam fora da thread do jogo,
  na hora (a cada ~5 ms). **Não entram na conta dos ~30 por segundo.** Com
  `[api] echo = 1`, os da porta de linha vão para a thread do jogo também (o
  eco no chat é uma chamada ao jogo).
- Um comando de cada vez: o local espera o de mundo que estiver rodando
  terminar, e vice-versa.
- Se o tick do jogo parar com comando de mundo esperando há mais de 2 s, a
  resposta é `erro: thread do jogo nao respondeu`, em vez de o cliente esperar
  o tempo inteiro.
- O `version` diz onde os comandos de mundo estão rodando: `executor=jogo`
  (normal) ou `executor=ue4ss` (o registro no tick do motor falhou — exige
  `HookEngineTick = 1` no `UE4SS-settings.ini`; o mod segue funcionando como
  na 0.6 e o erro aparece no `UE4SS.log`).
- A linha `[perf]` do `[debug] perf_log` mede o tempo do comando na thread que
  o executou: a do jogo, para comando de mundo.

### Custo por comando

Buscas por jogador e veículo passam pela **lista de atores do mundo** (milhares
de objetos), não pelo array global de UObjects (centenas de milhares). O mundo, o
índice de verbos e as classes são resolvidos uma vez e guardados.

Isso vale também para os 240 comandos do jogo, que precisam localizar um
controller antes de injetar.

| Comando | Custo |
|---|---|
| Comandos do jogo, `sudo`, `players`, `vehicles`, `get`, `set`, `call`, `DestroyAllBrennersWithinRadius` | Lista de atores — barato, pode ir para automação |
| `veiculos_listar` | Lista de atores, sem gravar em arquivo — feito para laço de 30 s (mínimo 10 s); medido em 15/11/2026 por frota: 0,38 ms com 3 veículos, 0,91 ms com 26, 2,1 ms com 89, 4,0 ms com 199 |
| `eventos` | Comando local, fora da thread do jogo. Sem novidade, uma comparação e uma linha; com novidade, uma cópia por linha devolvida. Sem arquivo — feito para laço de 1 a 2 s |
| `eventos_teste <n>` | Comando local: `n` registros de uma vez; diagnóstico, não usar em operação |
| `corpo <alvo>` | Um `controller_named` e até 64 efeitos lidos |
| `zona_criar`, `zona_remover`, `zona_listar` | Uma cópia do conjunto de zonas e uma chamada ao jogo (`zona_listar` só lê a cópia); medido em 15/11/2026: 7,6 ms, 4,2 ms e 0,06 ms |
| `find`, `search`, `inspect`, `object <nome parcial>` | Varredura completa — um engasgo de frame por chamada. Medido em 15/11/2026: `find`, `inspect` e `get object <nome>` custam **~1 s** na thread do jogo (o servidor para esse tempo). **Não usar com jogadores online** |

Medido em 15/11/2026: a **primeira** execução de cada tipo de comando do jogo
depois da subida do servidor custa ~100 a 300 ms na thread do jogo (ex.: o
primeiro `Location` levou 111 ms; os seguintes, 0,5 ms). É uma vez por tipo
de comando; depois o custo cai para o normal.

O `vehicles` grava cada linha em log: medido em 15/11/2026, 0,08 ms por veículo
(2,8 ms com 29 veículos); na comparação com o `veiculos_listar` deu 1,1 ms
com 3 e 2,6 ms com 26, contra 0,38 ms e 0,91 ms. Para consulta em laço, use o
`veiculos_listar`.

Desde a 0.7.0 nenhum comando que chega pela **porta Source** é gravado em
arquivo: nem a linha do comando, nem a resposta (o painel registra os dele).
A porta de linha continua no `bsbr_scum.log`. A linha `[perf]` do
`[debug] perf_log` continua para todas as origens, quando ligada.

A varredura roda na thread do jogo (desde a 0.7.0, de fato), então o frame
para enquanto ela acontece.
Imperceptível de vez em quando; perceptível num laço a cada segundo com jogadores
online.
