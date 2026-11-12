# Versão do mod

A versão atual é **0.6.0**. Ela pode ser lida de três formas, e as três sempre
concordam, porque o número nasce num lugar só no código e é carimbado nos três
na hora de compilar.

Voltar ao [índice](../README.md).

---

## Com o servidor no ar

Pela porta Source RCON, ou pela porta de linha, ou pelo chat:

```
version
```

```
bsbr_scum 0.6.0
portas source=28015 linha=27100
opcoes elevate=1 split_replies=0 echo=0 utf8=1 perf_log=0
```

É a resposta mais confiável, porque vem do binário que está carregado naquele
momento. A senha nunca aparece.

Para um gerenciador externo, é uma chamada igual a qualquer outra:

```python
versao = client.send_command("version").splitlines()[0]   # "bsbr_scum 0.6.0"
```

## Com o servidor parado

**Pelo arquivo**, ao lado do `config.ini`:

```
ue4ss\Mods\bsbr_scum\version.txt
```

```
bsbr_scum 0.6.0
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
  SteamID, arma e posição. Uma chamada por morte, nada por tick. Ver
  [comandos/mod.md](../comandos/mod.md#logsnpc_killslog--npc-armado-morto-por-jogador).
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
