<div align="center">

# bsbr_scum

**Mod de administração remota para servidor dedicado de SCUM.**
Recebe comandos por TCP e os executa dentro do jogo, sem travar o servidor:
nada roda por tick além de esvaziar a fila de comandos.

![C++](https://img.shields.io/badge/C%2B%2B-23-00599C?logo=cplusplus&logoColor=white)
![UE4SS](https://img.shields.io/badge/UE4SS-C%2B%2B%20mod-7c3aed)
![Unreal Engine](https://img.shields.io/badge/Unreal%20Engine-4.27-0E1128?logo=unrealengine&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-x64-0078D4?logo=windows&logoColor=white)
![Source RCON](https://img.shields.io/badge/protocolo-Source%20RCON-f59e0b)
![SCUM](https://img.shields.io/badge/SCUM-1.3.5-1f2937)
![licença](https://img.shields.io/badge/licen%C3%A7a-propriet%C3%A1ria-6b7280)

</div>

Este repositório guarda **só o pacote pronto para instalar**: os binários e a
documentação dos comandos. Cada versão é uma tag (`v0.5.4`) que não muda
depois de publicada. O painel **bsbr_ssm** consulta este repositório para
saber se há versão nova e instalá-la.

## O que ele faz

| Área | O que entrega |
| --- | --- |
| **Comandos do jogo** | os 233 comandos de administração do SCUM pela rede, inclusive os de nível Developer |
| **Sem admin online** | entrega de item, veículo e kit com qualquer jogador conectado; não precisa de administrador no servidor |
| **Alvo por jogador** | `as <SteamID> <comando>` executa no contexto daquele jogador (posição, inventário, mira) |
| **Leitura do servidor** | `players`, `vehicles`, `ListPlayers` montado por leitura de propriedades, sem capturar a saída do jogo |
| **Zonas de evento** | cria e remove zona personalizada ao vivo, com o nome que o jogador lê ao entrar |
| **Log de NPC armado** | `logs/npc_kills.log`: quem matou cada guarda, com arma e posição |
| **Brenner** | `DestroyAllBrennersWithinRadius`, que o jogo não tem |

## Como é feito

| Camada | Tecnologia | Por quê |
| --- | --- | --- |
| Mod | C++23 sobre [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) | roda dentro do `SCUMServer.exe` e fala com o jogo por reflexão da Unreal, por nome, sem assinatura de bytes |
| Rede | duas portas TCP: Source RCON (28015) e linha de texto (27100) | compatível com clientes Source RCON existentes; a de linha serve a scripts |
| Segurança da porta | senha, limite de conexões, bloqueio após senhas erradas | a porta Source fica fechada enquanto não houver senha no `config.ini` |
| Desempenho | fila de comandos esvaziada no tick do motor; ganchos só em funções específicas | custo declarado por comando, zero por tick |

## Conteúdo

```
dwmapi.dll                         carregador do UE4SS (vai ao lado do SCUMServer.exe)
ue4ss/
  UE4SS.dll                        UE4SS compilado para este mod
  UE4SS-settings.ini
  LICENSE-UE4SS.txt
  Mods/
    mods.txt
    bsbr_scum/
      dlls/main.dll                o mod
      version.txt                  versão deste pacote
      config.exemplo.ini           modelo de configuração, sem senha
      docs/                        os 233 comandos do jogo e os do mod
```

`UE4SS.dll` e `main.dll` saem do mesmo build e **são trocados sempre
juntos**.

## Instalar à mão

Com o servidor **parado**:

1. Copie `dwmapi.dll` e a pasta `ue4ss` para
   `<servidor>\SCUM\Binaries\Win64\`, ao lado do `SCUMServer.exe`.
2. Na primeira instalação, copie
   `ue4ss\Mods\bsbr_scum\config.exemplo.ini` para `config.ini` na mesma
   pasta e preencha `password` na seção `[source]`. Em atualização, **não
   toque no `config.ini` que já existe**.
3. Inicie o servidor com `-nobattleye`.

Para conferir: o comando `version` pela porta Source responde
`bsbr_scum <versão>`.

## Versão

A versão pode ser lida de três formas, e as três sempre concordam:

| Quando | Como |
| --- | --- |
| Servidor no ar | comando `version` |
| Servidor parado | primeira linha de `ue4ss\Mods\bsbr_scum\version.txt` |
| Servidor parado, pelo Windows | `(Get-Item main.dll).VersionInfo.FileVersion` |

O histórico do que mudou em cada versão está em
[`ue4ss/Mods/bsbr_scum/docs/orientacoes/versao.md`](ue4ss/Mods/bsbr_scum/docs/orientacoes/versao.md),
e a lista dos comandos em
[`ue4ss/Mods/bsbr_scum/docs/README.md`](ue4ss/Mods/bsbr_scum/docs/README.md).

## Licenças

O mod `bsbr_scum` é proprietário. O UE4SS é distribuído sob a licença MIT,
reproduzida em `ue4ss/LICENSE-UE4SS.txt`. Nenhum arquivo do SCUM faz parte
deste pacote.
