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

## Para quem integra

### Codificação: UTF-8 a partir da 0.6.0

| Versão do mod | Comando que você envia | Resposta que você recebe |
| --- | --- | --- |
| até 0.5.3 | um byte por caractere | Latin-1, um byte por caractere |
| 0.5.4 e 0.5.5 | **UTF-8** | Latin-1, um byte por caractere |
| **0.6.0 em diante** | **UTF-8** | **UTF-8** |

Texto só com ASCII é idêntico em todas as versões; a diferença aparece
apenas onde há acento, por exemplo no nome de um jogador no `ListPlayers`.

**Se o seu cliente lê respostas de versões anteriores à 0.6.0**, ajuste
antes de atualizar:

- decida pela versão, não por tentativa. A primeira linha do comando
  `version` (`bsbr_scum 0.6.0`) é ASCII e legível nos dois formatos;
- versão 0.6.0 ou maior: trate os bytes como UTF-8. Menor: como Latin-1
  (ISO-8859-1);
- quem convertia de Latin-1 para UTF-8 precisa parar de converter na 0.6.0,
  ou o acento sai em dobro (`Ã§` no lugar de `ç`).

Para manter o formato antigo num servidor, `utf8 = 0` na seção `[api]` do
`config.ini`. A linha `opcoes` do comando `version` mostra o valor em
vigor (`utf8=1` ou `utf8=0`).

### Atualização automática

Versão mais recente, em texto puro (`bsbr_scum 0.6.0`):

```
https://raw.githubusercontent.com/mazzottti/bsbr_rcon/main/ue4ss/Mods/bsbr_scum/version.txt
```

Pacote de uma versão (troque `0.6.0` pela versão lida acima):

```
https://github.com/mazzottti/bsbr_rcon/archive/refs/tags/v0.6.0.zip
```

O zip traz tudo sob a pasta `bsbr_rcon-0.6.0/`. Cada tag é imutável: o
conteúdo de uma versão publicada nunca muda.

Regras para instalar:

- somente com o servidor **parado**;
- copie `dwmapi.dll` e a pasta `ue4ss\` para `<servidor>\SCUM\Binaries\Win64\`;
- **nunca sobrescreva** `ue4ss\Mods\bsbr_scum\config.ini` (tem a senha); o
  pacote traz só o `config.exemplo.ini`;
- não apague `zonas.cache`, `zonas.ids`, `logs\` nem `bsbr_scum.log`;
- `UE4SS.dll` e `main.dll` são trocadas sempre juntas;
- compare versões número a número (`0.5.10` é maior que `0.5.9`) e atualize
  só se a disponível for maior;
- consulte a versão no máximo uma vez por hora;
- guarde a versão anterior para poder voltar se o servidor não subir.

## Licenças

O mod `bsbr_scum` é proprietário. O UE4SS é distribuído sob a licença MIT,
reproduzida em `ue4ss/LICENSE-UE4SS.txt`. Nenhum arquivo do SCUM faz parte
deste pacote.
