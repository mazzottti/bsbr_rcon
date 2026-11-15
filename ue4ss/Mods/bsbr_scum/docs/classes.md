# Classes do jogo mapeadas pelo mod

Classe do SCUM → o que o mod lê ou chama nela → comando para usar.

Voltar ao [índice](README.md). Sintaxe completa dos comandos:
[comandos/mod.md](comandos/mod.md).

Os exemplos são a linha enviada pela porta Source (28015) ou de linha (27100).
Para ver todas as propriedades e funções de qualquer classe:

```
inspect /Script/SCUM.<Classe>
```

---

## Resumo

| Classe | Para que serve | Comando |
|---|---|---|
| [`ConZPlayerController`](#conzplayercontroller) | jogador conectado: nome, admin, fama, dinheiro | `players`, `ListPlayers`, `as` |
| [`BP_Prisoner_C`](#bp_prisoner_c) | personagem: SteamID, posição, veículo montado | `get player`, `Location` |
| [`Vehicle`](#vehicle) | veículos do mapa: id, posição, dono, tranca, nome | `vehicles`, `veiculos_listar`, `get`/`set`/`call vehicle` |
| [`AdminCommand`](#admincommand) | os 233 comandos do jogo | `commands`, `sudo` |
| [`PlayerRpcChannel`](#playerrpcchannel) | canal do jogador: comandos do jogo e zonas | comandos do jogo, `zona_criar`, `zona_remover` |
| [`MiscStatics`](#miscstatics) | mensagem no chat | `say` |
| [`ArmedNPCBase`](#armednpcbase) | NPC armado (guarda) | evento `npc_morto` (`eventos`) |
| [`CustomZoneRegistry`](#customzoneregistry) | zonas personalizadas | `zona_listar` |
| [`BP_Brenner_C`](#bp_brenner_c) | Brenner | `DestroyAllBrennersWithinRadius`, ameaça do `jogador_morto` |
| [`Sentry2`, `Razor`, `Dropship`, `HuntingManager`, `Zombie2`](#mortes-sentry2-razor-dropship-huntingmanager-zombie2) | sentinela, Razor, nave, caça, zumbi | eventos de morte (`eventos`, `eventos_capturar`) |
| [qualquer outra](#qualquer-outra-classe) | leitura e escrita genérica | `find`, `inspect`, `get`, `set`, `call`, `spawn` |

---

## ConZPlayerController

`/Script/SCUM.ConZPlayerController` (instância: `BP_ConZPlayerController_C`).
Um por jogador conectado.

| Membro | Tipo | O que é |
|---|---|---|
| `Pawn` | objeto | o personagem ([`BP_Prisoner_C`](#bp_prisoner_c)) |
| `_repFamePoints` | int32 | fama |
| `_moneyBalanceRep` | int64 | dinheiro |
| `_goldBalanceRep` | int64 | ouro |
| `GetPlayerName()` | função | nome do jogador |
| `GetUserProfileName()` | função | nome do perfil |
| `IsUserAdmin()` | função | é admin |
| `GetPlayerRpcChannel()` | função | o [`PlayerRpcChannel`](#playerrpcchannel) dele |
| `Server_RequestRespawnWaitTime` | função | gancho do `jogador_morto` (0.7.0): o cliente pede o tempo de renascimento ao abrir a tela de morte; vítima = `Pawn`. Os parâmetros não são lidos |

```
players
ListPlayers
as 76561190000000000 <comando>
```

`as <SteamID>` executa o comando no contexto daquele jogador.

Validado em 15/11/2026 em 5 mortes (zumbi, javali, queda, zumbi kamikaze):
5 eventos, no mesmo segundo do log do jogo, sem repetição. O cliente chama
`Server_RequestRespawnWaitTime` uma vez na morte e outra logo depois de
renascer. A proteção é por estado, por personagem: depois do `jogador_morto`
o personagem fica marcado e os disparos seguintes são ignorados até o
renascimento; um disparo até 5 s depois do renascimento é o eco dele e é
ignorado. A proteção antiga (só 5 s) deixava sair um `jogador_morto` falso
quando o jogador esperava mais de 5 s na tela de morte: visto e corrigido em
15/11/2026, correção validada no servidor (30 s na tela de morte: uma
morte e um renascimento; a morte seguinte saiu normalmente). Limitações: mod subindo com o
jogador já na tela de morte perde essa morte; morrer de novo menos de 5 s
depois de renascer não gera evento.

## BP_Prisoner_C

`BP_Prisoner_C` (base `/Script/SCUM.ConZCharacter`). O personagem do jogador.

| Membro | Tipo | O que é |
|---|---|---|
| `_userId` | string | SteamID |
| `_userProfileName` | string | nome do perfil |
| `_repMount.MountedSlot` | objeto | assento ocupado; o `Outer` dele é o veículo |
| `K2_GetActorLocation()` | função | posição em cm |
| `LastHitBy`, `_weaponClassLastHitBy` | objeto, classe | atacante e arma do último dano com autor, lidos no `jogador_morto` |
| `Server_RespawnAtCommonSpawnLocation` | função | gancho do `jogador_renasceu` (0.7.0), declarada em `/Script/SCUM.Prisoner`; vítima = o próprio personagem |
| `Server_Respawn*` | funções | na subida o mod procura outras funções do `Prisoner` com esse começo de nome para pôr o mesmo gancho; não há nenhuma (visto no `UE4SS.log`) |

```
get player Fulano
Location 76561190000000000
```

O personagem é o **mesmo objeto** antes e depois da morte e do renascimento
(medido em 15/11/2026); um segundo depois da morte ele ainda tem `LastHitBy`
e os efeitos do corpo (`BleedingInjury`, `Bleeding`, `Infection_Chest`,
`HealthLow`, `Pain`...). `Server_RespawnAtCommonSpawnLocation` foi validada
em 3 renascimentos: chega 2 a 4 s depois da morte, com o personagem ainda no
local da morte (o jogo registra `prisoner revived` ~2 s depois).

`get` grava em `bsbr_scum_target.txt`, na pasta `Win64` do servidor.

## Vehicle

`/Script/SCUM.Vehicle` (em versões antigas do jogo, `VehicleBase`).

| Membro | O que é |
|---|---|
| `GetMovementComponent()` | componente com motor, câmbio e massa; é o alvo real de `get`/`set`/`call vehicle` |
| `_itemContainer` | objeto: o contêiner do veículo. Dono, tranca e nome moram nele, não no ator |
| `_itemContainer._owningUserProfileId.Value` | int64: id do perfil do dono; `0` = sem dono |
| `_itemContainer._lockData` | lista: uma entrada por tranca instalada; tamanho `0` = destrancado |
| `_itemContainer._nameableItemComponent._nameData.Value` | string: nome dado pelo jogador |

```
vehicles
veiculos_listar
get vehicle 590005
set vehicle 590005 <caminho> <valor>
call vehicle 590005 StartEngine
```

O id é a primeira coluna de `vehicles`. `set` aceita float, int e bool; física
(massa, marchas, motor) não responde, porque quem simula é o cliente. `call`
só chama função sem parâmetros.

`veiculos_listar` (0.7.0, validado em 15/11/2026) lê os três campos do contêiner de cada
veículo carregado:

```
veiculos: 1
veiculo;510019;BPC_RIS;309403;368051;21249;1;76561190000000000;1;
```

Colunas: id, classe, x, y, z, perfil do dono, SteamID do dono (`0` se
offline), trancas, nome. O dono continua preenchido depois de a tranca sair;
o veículo conta como do jogador só com trancas > 0.

## AdminCommand

`/Script/SCUM.AdminCommand`. Cada comando do jogo é uma classe em
`/Game/ConZ_Files/AdminCommands/`.

| Membro | Tipo | O que é |
|---|---|---|
| `_verb` | string | o nome digitado |
| `_description` | texto | descrição |
| `_argumentDescriptions` | lista | descrição de cada argumento |
| `_numberOfRequiredArguments` | int | argumentos obrigatórios |
| `_numberOfRepeatingArguments` | int | argumentos que se repetem |
| `_requiredExecutorLevel` | byte | nível exigido; 3 e 4 são Developer |
| `_isEnabledInShippingBuild` | bool | habilitado no servidor |

```
commands
<Verbo> <argumentos>
sudo <Verbo> <argumentos>
as 76561190000000000 sudo SpawnBrenner
```

`sudo` baixa o `_requiredExecutorLevel`, executa e restaura no mesmo tick.

## PlayerRpcChannel

`/Script/SCUM.PlayerRpcChannel`. Um por jogador; obtido com
`ConZPlayerController.GetPlayerRpcChannel()`.

| Função | Uso no mod |
|---|---|
| `Chat_Server_ProcessAdminCommand` | entrega ao jogo os comandos dele (`ListPlayers`, `SpawnItem`...); o mod chama, não escuta |
| `Chat_Client_SendMessageToChat` | resposta do mod no chat de um jogador |
| `CustomZones_Server_UpdateCustomZoneData(globalConfiguration, configurations, Regions)` | grava a lista inteira de zonas |

```
zona_criar evento_42 -113084 -70633 60000 "Evento PVE" "Arena do Brenner"
zona_remover evento_42
```

`zona_criar` e `zona_remover` precisam de um jogador conectado, para emprestar
o canal.

## MiscStatics

`/Script/SCUM.Default__MiscStatics`.

| Função | Uso no mod |
|---|---|
| `BroadcastChatLine` | mensagem para todos |
| `SendChatLineToPlayer` | mensagem para um jogador |

```
say Servidor reiniciando em 5 minutos
```

## ArmedNPCBase

`/Script/SCUM.ArmedNPCBase`. Os guardas (`BP_Guard_Lvl_1` e afins).

| Membro | Tipo | O que é |
|---|---|---|
| `NetMulticast_OnConZCharacterKilled` | função | gancho: dispara uma vez por NPC morto |
| `LastHitBy` | objeto | controller de quem deu o último dano |
| `_weaponClassLastHitBy` | classe | arma do último dano |

Não tem comando próprio. Desde a 0.7.0 o mod põe um evento `npc_morto` na
fila em memória, buscada com `eventos` (o `logs\npc_kills.log` deixou de
existir), um por NPC morto por jogador:

```
42;2026.11.10-21.57.50;npc_morto;BP_Guard_Lvl_1;2147482345;76561190000000000;1;Fulano;Weapon_M9;140508.703;-440821.125;9946.081
```

Ver [comandos/mod.md](comandos/mod.md#evento-npc_morto-substitui-o-logsnpc_killslog).

## CustomZoneRegistry

`/Script/SCUM.CustomZoneRegistry`.

| Membro | Uso no mod |
|---|---|
| `NetMulticast_ReceiveCustomZoneData` | gancho: a lista de zonas que o servidor aplicou |

Estruturas da lista:

| Estrutura | Campo | Tipo | O que é |
|---|---|---|---|
| `CustomZoneConfiguration` | `Name` | string | nome da configuração (`Evento PVE`) |
| | `Color` | cor | cor no mapa |
| `CustomZoneRegion` | `Name` | string | nome que o jogador lê ao entrar |
| | `Location` | Vector2D | centro, em cm |
| | `Size` | Vector2D | círculo: X = Y = raio em cm |
| | `Shape` | byte | forma |
| | `ConfigurationIndex` | int | posição da configuração na lista, a partir de 0 |

```
zona_listar
```

```
zonas: update=2 multicast=2 ultima=2026.11.11-20.18.42 origem=multicast
config;0;Outposts;0.319;0.784;0.080
config;1;Evento PVE;0.061;0.631;0.776
regiao;Outpost B4;570708.000;-226174.000;30000.000;0.000;0;0;editor
regiao;Arena do Brenner;-113084.000;-70633.000;60000.000;60000.000;0;1;mod;evento_42
```

## BP_Brenner_C

`/Game/ConZ_Files/Characters/NPCs/Creature/Brenner/BP_Brenner.BP_Brenner_C`.

| Função | Uso no mod |
|---|---|
| `K2_GetActorLocation()` | distância até o jogador do `as` |
| `K2_DestroyActor()` | remoção |

```
as 76561190000000000 sudo SpawnBrenner
as 76561190000000000 DestroyAllBrennersWithinRadius 100
```

Raio em metros, padrão 50. Os dois exigem `as <SteamID>`.

## Mortes: Sentry2, Razor, Dropship, HuntingManager, Zombie2

Ganchos dos eventos de morte da 0.7.0. Os daqui são candidatos (15/11/2026):
os três primeiros instalam e nunca foram exercitados; o do `HuntingManager`
teve 0 disparos com 8 animais criados por `SpawnRandomAnimal` e mortos pelo
jogador, e não foi testado com animal do próprio jogo.

| Classe | Função com gancho | Vítima | Evento |
|---|---|---|---|
| `/Script/SCUM.Sentry2` | `NetMulticast_OnSentryDeath` | o próprio objeto | `sentinela_destruida` |
| `/Script/SCUM.Razor` | `NetMulticast_OnRazorKilled` | o próprio objeto | `razor_morto` |
| `/Script/SCUM.Dropship` | `NetMulticast_OnDropshipDeath` | o próprio objeto | `nave_destruida` |
| `/Script/SCUM.HuntingManager` | `OnAnimalKilled(ConZCharacter)` | o parâmetro | `animal_morto` (não dispara para animal criado por comando) |
| `/Script/SCUM.Zombie2` | nenhuma | — | sem evento: o gancho em `GoToRagdoll` e o `zumbi_morto` foram removidos (0 disparos). Sondadas `Zombie2`, `ConZPlayerController`, `Prisoner` e `ConZCharacter` com 10 abates cada: nenhuma função dispara quando o jogador mata zumbi. Contagem só pelo `SCUM.db` (`survival_stats.puppets_killed`) |

O `jogador_morto` e o `jogador_renasceu` têm gancho no
[`ConZPlayerController`](#conzplayercontroller) e no
[`BP_Prisoner_C`](#bp_prisoner_c). Nos eventos, a classe da vítima sai com o
nome real da classe Blueprint (sem `_C`); para sentinela, Razor e nave ele
ainda não foi visto, e os exemplos usam a classe nativa (`Sentry2`, `Razor`,
`Dropship`). Nomes reais já vistos: zumbi `BP_Zombie_Hospital_Female`, urso
`BP_BTBear_Mutant_ApexHunt` (controller `BP_BTBear_Mutant_Apex_HuntController`),
NPC armado `BP_Guard_Lvl_1`, arma `Weapon_AK47`.

Da vítima o mod lê `LastHitBy` (controller do último atacante, padrão do
`ConZCharacter`) e `_weaponClassLastHitBy`. Sem o campo `LastHitBy`, o evento
sai sem matador (`0;0;-;-`).

`Sentry2`, `Razor`, o Brenner e `Vehicle` também servem de "ameaça próxima"
no `jogador_morto` (a mais perto a até 150 m; veículo de id `0` não conta).
`Zombie2`, `Animal`, `ArmedNPCBase`, `Razor` e `Prisoner` decidem a `causa`
por `IsA` no atacante.

```
eventos_capturar tudo 120
eventos 0
sonda_funcoes /Script/SCUM.Zombie2 60
```

## Qualquer outra classe

| Comando | O que faz |
|---|---|
| `find <texto>` | classes e objetos carregados com esse texto no nome |
| `inspect <caminho da classe>` | propriedades, funções e herança |
| `get <alvo>` | todas as propriedades do alvo, em `bsbr_scum_target.txt` |
| `set <alvo> <caminho> <valor>` | escreve float, int ou bool |
| `call <alvo> <Função>` | chama função sem parâmetros |
| `search <alvo> <valor>` | acha a propriedade que tem esse valor |
| `spawn <caminho da classe> <x> <y> <z>` | cria um ator |

Alvos:

| Alvo | Resolve para |
|---|---|
| `player <nome>` | o personagem do jogador |
| `vehicle <id>` | o veículo com esse id |
| `class <caminho>` | o molde da classe; vale para instâncias criadas depois |
| `object <nome ou caminho>` | qualquer objeto carregado |

```
find Rager
inspect /Script/SCUM.ConZGameState
get class /Game/ConZ_Files/Vehicles/Car/Rager/BPC_Rager.BPC_Rager_C
set class /Game/ConZ_Files/Vehicles/Car/Rager/BPC_Rager.BPC_Rager_C _driveComponent.ChassisMass 300
get object /Game/ConZ_Files/Skills/DrivingSkill.DrivingSkill
spawn /Game/ConZ_Files/Vehicles/Car/Rager/BPC_Rager.BPC_Rager_C 0 0 5000
```

Caminho: ponto entra em struct, colchete indexa array
(`GearboxData.ForwardGears[7].Ratio`). Só enxergam o que está carregado em
memória. `find`, `search` e `inspect` varrem todos os objetos do servidor:
use para investigar, não em automação.
