# Comandos do mod

Estes **não são do SCUM**. São do `bsbr_scum`, e funcionam por reflexão da Unreal:
leem e escrevem propriedades dos objetos vivos do servidor, sem passar pelo
interpretador de comandos do jogo.

Voltar ao [índice](../README.md).

---

## As duas formas de chamar

Todo comando funciona igual pelos dois caminhos. A única diferença é o prefixo.

| | Terminal do servidor | Chat do jogo |
|---|---|---|
| Prefixo | `scum ` | `#bsbr ` |
| Exemplo | `scum vehicles` | `#bsbr vehicles` |
| Resposta | Saída do terminal | Mensagens no chat |
| Quem é o alvo padrão | Primeiro jogador vivo | **Quem digitou** |

O `scum` é a função do PowerShell descrita em [conectar.md](../orientacoes/conectar.md).
O `#` é o marcador de chat do SCUM; `bsbr` é o nosso.

**Não são sensíveis a maiúsculas.** `#BSBR VEHICLES` e `#bsbr vehicles` são a
mesma coisa. Vale para o marcador e para o nome do comando — o resto da linha
(nome de jogador, caminho de propriedade, classe) mantém a grafia.

### Abreviação

Basta digitar o suficiente para não ser ambíguo:

```
#bsbr veh          → vehicles
#bsbr pl           → players
#bsbr s            → 's' e ambiguo: say, search, set, spawn, sudo
```

A abreviação só vale para comandos do mod. Verbos do jogo passam intactos.

---

## Resumo

| Comando | O que faz |
|---|---|
| `help [comando]` | A lista, ou o detalhe de um comando |
| `players` | Jogadores conectados, com ID e SteamID |
| `vehicles` | Veículos com o ID que a interface mostra |
| `get <alvo> [caminho]` | Despeja as propriedades do alvo num arquivo |
| `set <alvo> <caminho> <valor>` | Escreve uma propriedade |
| `call <alvo> <função>` | Chama uma função sem parâmetros |
| `spawn <classe> <x> <y> <z>` | Cria um ator direto pela engine |
| `sudo <comando>` | Executa um comando de nível Developer |
| `DestroyAllBrennersWithinRadius [Radius [m]]` | Destrói os Brenners num raio do jogador do `as` (padrão 50 m) |
| `zona_listar` | Zonas personalizadas que o mod viu passar, com `mod` ou `editor` por região (em teste) |
| `zona_criar <id> <x> <y> <raio_cm> <configuracao> <titulo>` | Cria ou atualiza a zona circular do `id`; o título é o nome que o jogador lê (em teste) |
| `zona_remover <id>` | Remove a zona do `id`, e só zona criada pelo mod (em teste) |
| `zona_esquecer` | Solta a lista de zonas carregada do `zonas.cache` e apaga o arquivo (os ids ficam) |
| `find <texto>` | Procura classes e objetos carregados |
| `search <alvo> <valor>` | Procura uma propriedade com esse valor |
| `inspect <classe>` | Reflexão completa de uma classe |
| `say <texto>` | Anuncia no chat |
| `commands` | Gera o catálogo dos comandos do jogo |
| `capture on\|off` | Liga a captura da saída dos comandos |

E um prefixo, que não é comando:

| Prefixo | O que faz |
|---|---|
| `as <jogador> <comando>` | Executa o comando **como** aquele jogador (no chat: `@<jogador>`) |

---

## `@` e `as` — mirar em outro jogador

```powershell
scum as Fulano SpawnItem Weapon_M9
```
```
#bsbr @Fulano SpawnItem Weapon_M9
```

**São a mesma coisa, e as duas existem por necessidade.** No PowerShell, `@` no
início de um argumento é *splatting* e some antes da função receber — e `-` seria
lido como nome de parâmetro. Por isso o terminal usa `as`. No chat, onde nada
disso acontece, o `@` é o natural.

O `@` também funciona no terminal, desde que entre aspas: `scum '@Fulano' ...`.

Comandos de admin rodam **no contexto de quem os envia**. É isso que define onde
o item cai, onde o veículo nasce, de quem é a posição consultada. Trocar o
executor redireciona os 233 comandos de uma vez, sem precisar de argumento de
posição em cada um.

Aceita três identificadores, todos **exatos** — sem abreviar:

| Forma | Terminal | Chat |
|---|---|---|
| ID do perfil | `as 1` | `@1` |
| SteamID | `as 76561190000000000` | `@76561190000000000` |
| Nome completo | `as Fulano` | `@Fulano` |

Use `players` para vê-los. Só a caixa é ignorada: `as fulano` funciona.

> Casamento por trecho seria perigoso: `1` acertaria o jogador `17`, e `Ma`
> acertaria dois jogadores diferentes — em silêncio, e o erro só apareceria pela
> reclamação de quem levou.

**Sem `@`**, o alvo é quem digitou (no chat) ou o primeiro jogador vivo (no
terminal, que não tem remetente).

Combina com tudo:

```
#bsbr @Fulano sudo SpawnBrenner
#bsbr @Fulano Teleport 559735 -215880 1172
#bsbr @Fulano get player
```

---

## `help`

```powershell
scum help
scum help set
```
```
#bsbr
#bsbr help set
```

No chat, `#bsbr` sozinho já mostra a lista — não precisa escrever `help`.

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
scum get class /Game/ConZ_Files/Vehicles/Rager/BPC_Rager.BPC_Rager_C
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
scum set class /Game/ConZ_Files/Vehicles/Rager/BPC_Rager.BPC_Rager_C _driveComponent.ChassisMass 300
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
scum spawn /Game/ConZ_Files/Vehicles/Rager/BPC_Rager.BPC_Rager_C 0 0 5000
```

Vai direto no `BeginDeferredActorSpawnFromClass`, sem passar pelo comando
`SpawnVehicle` do jogo — logo, **sem verificação de permissão**. Só aceita classes
já carregadas.

Útil junto com `set class`: configure o molde, depois spawne.

Para spawnar perto de um jogador, prefira o comando do jogo com `@`, que já nasce
na posição dele:

```
#bsbr @Fulano SpawnVehicle BPC_Rager
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
sem `as <SteamID>` responde o uso e não executa. Pelo chat, o `@` ou o próprio
remetente já resolvem o alvo. Para remover, ver
[`DestroyAllBrennersWithinRadius`](#destroyallbrennerswithinradius--remover-brenners).

---

## `DestroyAllBrennersWithinRadius` — remover Brenners

```powershell
scum as 76561198121427572 DestroyAllBrennersWithinRadius
scum as 76561198121427572 DestroyAllBrennersWithinRadius 100
```

```
como Mazzotti:
2 Brenner(s) destruido(s) num raio de 50 m
```

O SCUM não tem comando que remova o Brenner: ele não é zumbi, animal nem NPC
armado, então `DestroyZombiesWithinRadius`, `DestroyAllAnimalsWithinRadius`,
`DestroyArmedNPCsWithinRadius` e `DestroyEntity` passam por ele. Este comando é
do mod e faz o que falta: acha os atores `BP_Brenner_C` num raio do jogador e os
destrói, tirando colisão e desenho antes para o cliente não ficar com um Brenner
fantasma.

Mesma forma do `DestroyAllRazorsWithinRadius` do jogo, com duas diferenças:

- **O jogador é obrigatório**, pelo `as <SteamID>` (ou `@` no chat). Não existe
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
forma antiga (sem título). O que veio depois compila, mas ainda **não foi
exercitado no servidor**: a cópia em arquivo da 0.5.3 (`zonas.cache`,
`origem=arquivo`, `zona_esquecer`) e o **id separado do título** da 0.5.4
(`zonas.ids`, acento no título). Teste primeiro num servidor de
desenvolvimento, com cópia do `SCUM.db`.

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
- Acento na **resposta**: o mod ainda devolve um byte por caractere
  (Latin-1), não UTF-8. `Caça` volta com o `ç` como o byte `E7`. O painel não
  deve depender do texto do título na resposta, só do `id`.
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
O `zonas.cache` custa uma gravação pequena a cada salvamento de zonas que o
servidor aplica (raro) e uma leitura quando o servidor sobe. O `zonas.ids`,
uma gravação menor ainda por `zona_criar` ou `zona_remover` que mudou algo, e
uma leitura quando o servidor sobe.

---

## `logs/npc_kills.log` — NPC armado morto por jogador

Não é comando: é um arquivo que o mod escreve sozinho, em
`ue4ss\Mods\bsbr_scum\logs\npc_kills.log` (UTF-8, só acrescenta).

```
2026.11.10-21.57.50: [Killed] BP_Guard_Lvl_1. NpcId: 2147482345. Killer: 76561198121427572 (1, Mazzotti). Weapon: Weapon_M9. Location: X=140508.703 Y=-440821.125 Z=9946.081
```

- Uma linha por NPC armado cujo último dano veio de um jogador. Morte sem
  jogador (outro NPC, queda, sentinela) não gera linha.
- `Killer` é `<SteamID> (<id do perfil>, <nome>)`; `Weapon` é `Unknown` quando
  o jogo não informa; `NpcId` é o número da instância do NPC, ou `0`.
- A data é a hora **local** do servidor (os logs do jogo usam UTC).
- Custo: uma chamada por morte de NPC armado, zero por tick.
- **Validado no servidor em 11/11/2026** (dois guardas, duas linhas, sem repetição).

---

## `say` — anunciar

```powershell
scum say Servidor reiniciando em 5 minutos
```

Usa `MiscStatics::BroadcastChatLine`. É também por onde as respostas do mod
chegam ao chat.

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

Buscas por jogador e veículo passam pela **lista de atores do mundo** (milhares
de objetos), não pelo array global de UObjects (centenas de milhares). O mundo, o
índice de verbos e as classes são resolvidos uma vez e guardados.

Isso vale também para os 273 comandos do jogo, que precisam localizar um
controller antes de injetar.

| Comando | Custo |
|---|---|
| Comandos do jogo, `sudo`, `players`, `vehicles`, `get`, `set`, `call`, `DestroyAllBrennersWithinRadius` | Lista de atores — barato, pode ir para automação |
| `find`, `search`, `inspect`, `object <nome parcial>` | Varredura completa — um engasgo de frame por chamada |

A varredura roda na thread do jogo, então o frame para enquanto ela acontece.
Imperceptível de vez em quando; perceptível num laço a cada segundo com jogadores
online.

---

## Ruído no chat

Quando um comando `#bsbr` é reconhecido, o mod apaga o texto antes do jogo
processá-lo — senão o SCUM responderia *"unrecognized command"*, já que `bsbr`
não existe no catálogo dele.

O apagamento escreve o terminador no lugar, sem liberar memória: a string vive no
frame do jogo, e devolver ao allocator dele, dentro da chamada dele, quebraria
longe do ponto de origem.
