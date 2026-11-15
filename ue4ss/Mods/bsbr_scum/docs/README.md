# bsbr_scum — documentação

**Versão 0.8.1.** Tudo que dá para executar neste servidor: os **233 comandos
de administração do SCUM** e os **35 comandos do mod**.

Se é a primeira vez: [orientacoes/conectar.md](orientacoes/conectar.md) ensina a
mandar o primeiro comando.

Para saber qual versão está instalada, inclusive com o servidor parado:
[orientacoes/versao.md](orientacoes/versao.md).

---

## Comandos do mod

| Arquivo | Conteúdo |
|---|---|
| [comandos/mod.md](comandos/mod.md) | `get`, `set`, `call`, `sudo`, `@jogador`, `vehicles`, `players`; fila de eventos (`eventos`, `eventos_capturar`): morte de NPC armado, morte do jogador com a causa (`zumbi`, `animal`, `queda`, `sangramento`...), renascimento. Zumbi morto por jogador não tem evento: conta-se pelo `SCUM.db` |

Leem e escrevem propriedades dos objetos vivos por reflexão da Unreal, sem passar
pelo interpretador do jogo. Entram pelas portas (painel ou terminal), não pelo
chat.

## Classes do jogo

| Arquivo | Conteúdo |
|---|---|
| [classes.md](classes.md) | Cada classe do SCUM que o mod mapeia, o que ele lê ou chama nela e o comando para usar |

## Comandos do SCUM

233 comandos, por assunto. Cada entrada traz a linha de chamada pronta e o
significado de cada argumento, lidos do próprio jogo:

> A documentação por assunto cobre os 233 comandos mapeados na versão 1.3.5
> do jogo. A 1.3.6.1 carrega 240 (linha `[bsbr_scum] 240 comandos indexados`
> no log): 7 ainda não documentados.

| Arquivo | Qtd | Conteúdo |
|---|---|---|
| [comandos/jogadores.md](comandos/jogadores.md) | 47 | Listar, expulsar, banir, silenciar, teleportar |
| [comandos/spawn.md](comandos/spawn.md) | 29 | Itens, veículos, zumbis, animais, NPCs |
| [comandos/mundo.md](comandos/mundo.md) | 28 | Clima, tempo, eventos, encontros |
| [comandos/economia.md](comandos/economia.md) | 14 | Moeda, fama, comerciantes |
| [comandos/destruir.md](comandos/destruir.md) | 16 | Veículos, itens, construções, cadáveres |
| [comandos/servidor.md](comandos/servidor.md) | 22 | Informação e manutenção |
| [comandos/developer.md](comandos/developer.md) | 77 | Ferramentas internas — todos 🔒 |

## Identificadores

O que passar como argumento de `SpawnItem`, `SpawnVehicle` e afins:

| Arquivo | Qtd |
|---|---|
| [comandos/itens.md](comandos/itens.md) | 1134 itens |
| [comandos/veiculos.md](comandos/veiculos.md) | 17 veículos |
| [comandos/zumbis.md](comandos/zumbis.md) | 25 tipos de zumbi |

## Orientações

| Arquivo | Conteúdo |
|---|---|
| [orientacoes/conectar.md](orientacoes/conectar.md) | As duas portas, a função `scum` do PowerShell e o que cada resposta significa |
| [orientacoes/versao.md](orientacoes/versao.md) | Como ler a versão do mod, com o servidor no ar ou parado |
| [orientacoes/comandos-do-scum.md](orientacoes/comandos-do-scum.md) | Níveis de permissão e como destravar os de nível Developer |

---

## Referência rápida

```powershell
scum ListPlayers                       # comando do jogo, pelo terminal
scum sudo SpawnBrenner                 # comando de nível Developer
scum as Fulano SpawnItem Weapon_M9   # executando no lugar de outro jogador
```

```
#ListPlayers                           # comando do jogo, pelo chat
```

Pelo chat entram só os comandos do próprio jogo; os do mod, só pelas portas.

No terminal use `as`, não `@`: o PowerShell consome o `@` como splatting antes da
função receber o argumento.

---

## Como ler cada entrada dos comandos

```
### `BaseBuildingDebug` 🔒          ← 🔒 exige nível Developer

Depuração de construção             ← resumo, em português

​```
scum sudo BaseBuildingDebug <Mode>  ← linha pronta, com o sudo já incluso
​```

- `Mode` — 0 = Disabled ; 1 = Basic ; 2 = RadialDamage ; ...   ← cada argumento

*Original:* Sets debug mode for base building.    ← texto da Gamepires
```

O resumo é nosso. A explicação dos argumentos e a linha *Original* são o texto da
própria Gamepires, lido do jogo — não traduzimos para não introduzir erro no que
é a referência.
