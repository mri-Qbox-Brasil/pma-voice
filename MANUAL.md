# pma-voice — Manual

VOIP para FiveM/RedM construído sobre o servidor Mumble embutido: voz por proximidade, rádio e chamadas, configurado inteiramente por convars.

---

## Sumário

1. [Dependências](#dependências)
2. [Instalação](#instalação)
3. [Permissões (ACE)](#permissões-ace)
4. [Configuração](#configuração)
5. [Proximidade](#proximidade)
6. [Rádio](#rádio)
7. [Chamadas](#chamadas)
8. [Comandos](#comandos)
9. [State bags](#state-bags)
10. [Integrações](#integrações)
11. [Entrypoints para outros recursos](#entrypoints-para-outros-recursos)
12. [Estrutura de arquivos](#estrutura-de-arquivos)

---

## Dependências

| Recurso | Obrigatório | Observação |
|---|---|---|
| OneSync | Sim | Declarado como `/onesync` em `dependencies`. O recurso não inicia sem ele |

Não depende de framework: funciona em QBox, QBCore, ESX ou servidor sem framework. Não usa banco de dados nem `ox_lib`.

**Não pode rodar junto com outro sistema de voz.** O manifest declara `provides` para `mumble-voip`, `tokovoip`, `toko-voip` e `tokovoip_script` — rodar qualquer um deles em paralelo quebra o áudio. Se usar vMenu, desligue o chat de voz dele.

Também não sobrescreva estas natives em outros recursos, ou o pma-voice perde o controle da proximidade:

- `NetworkSetTalkerProximity`
- `MumbleSetTalkerProximity`
- `MumbleSetAudioInputDistance`
- `MumbleSetAudioOutputDistance`
- `NetworkSetVoiceActive`

---

## Instalação

1. Copie a pasta `pma-voice` para `resources/`.
2. Adicione ao `server.cfg`:
   ```
   ensure pma-voice
   ```
3. Remova qualquer outro recurso de voz (`mumble-voip`, `tokovoip`, voz do vMenu).
4. Configure as convars (ver [Configuração](#configuração)). O mínimo recomendado:
   ```
   setr voice_useNativeAudio true
   setr voice_useSendingRangeOnly true
   ```
   Se **nenhuma** convar de modo de áudio estiver definida, o recurso liga essas duas automaticamente na inicialização e avisa no console.

Não há SQL para importar.

---

## Permissões (ACE)

Apenas o `/muteply` é restrito (registrado em `server/mute.js` como comando restrito):

```
add_ace group.admin command.muteply allow
```

Sem essa ACE, o comando não faz nada. Nenhuma outra funcionalidade do recurso usa ACE.

---

## Configuração

Toda a configuração é por convar, no `server.cfg`. Use `setr` (replicated) — o client precisa ler os valores. Nenhuma é obrigatória; todas têm padrão.

### Modo de áudio

| Convar | Tipo | Padrão | Descrição |
|---|---|---|---|
| `voice_useNativeAudio` | bool | `false` | Usa o áudio nativo do jogo, com oclusão e posicionamento 3D. Muda as distâncias dos modos de proximidade (whisper/normal/shout passam de 3/7/15 para 1.5/3/6 unidades) |
| `voice_useSendingRangeOnly` | bool | `false` | Impede que quem entra direto no servidor Mumble transmita para todo mundo. **Recomendado ligar** — o recurso avisa no console se estiver desligado |
| `voice_use2dAudio` | bool | — | Usado apenas na detecção de "nenhum modo de áudio configurado", que dispara os padrões automáticos |
| `voice_use3dAudio` | bool | — | Idem `voice_use2dAudio` |
| `voice_enableSubmix` | int | `1` | Aplica o efeito de submix (som de rádio/telefone) na voz de quem fala no rádio ou em chamada |

### Interface e teclas

| Convar | Tipo | Padrão | Descrição |
|---|---|---|---|
| `voice_enableUi` | int | `1` | Mostra a UI de voz (indicador de modo e de fala) |
| `voice_refreshRate` | int | `200` | Intervalo, em ms, do loop que recalcula os alvos de voz e atualiza a UI |
| `voice_enableProximityCycle` | int | `1` | Permite ao jogador ciclar entre os modos de proximidade |
| `voice_defaultCycle` | string | `F11` | Tecla padrão do ciclo de proximidade (só FiveM; no RedM é `]`) |
| `voice_defaultRadio` | string | `LMENU` | Tecla padrão de falar no rádio, ALT esquerdo (só FiveM; no RedM é `[`) |
| `voice_defaultVoiceMode` | int | `2` | Índice do modo de proximidade inicial (`1` = Whisper, `2` = Normal, `3` = Shouting) |
| `voice_disableAutomaticListenerOnCamera` | int | `0` | Com `1`, desliga o modo "ouvinte" automático quando o jogador está em uma câmera renderizada ou em espectador |

### Rádio

| Convar | Tipo | Padrão | Descrição |
|---|---|---|---|
| `voice_enableRadios` | int | `1` | Liga o sistema de rádio. Com `0`, todos os exports e comandos de rádio viram no-op |
| `voice_defaultRadioVolume` | int | `30` | Volume inicial do rádio, de 1 a 100. Valores `0` ou `1` são rejeitados e resetados para o padrão com aviso no console |
| `voice_enableRadioAnim` | int | `0` | Toca a animação de falar no rádio |
| `voice_disableVehicleRadioAnim` | int | `0` | Com `1`, não toca a animação de rádio quando o jogador está em veículo |
| `voice_useEmoteMenuAnim` | int | `1` | Usa um comando de emote em vez da animação nativa (`random@arrests`) |
| `voice_emoteMenuAnim` | string | `e wt2` | Comando executado ao começar a falar no rádio, se `voice_useEmoteMenuAnim = 1` |
| `voice_emoteMenuStopAnim` | string | `e c` | Comando executado ao parar de falar no rádio |
| `voice_syncPlayerNames` | int | `0` | Sincroniza os nomes dos jogadores do canal de rádio para o client |

### Chamadas

| Convar | Tipo | Padrão | Descrição |
|---|---|---|---|
| `voice_enableCalls` | int | `1` | Liga o sistema de chamadas. Com `0`, os exports de chamada viram no-op |
| `voice_defaultCallVolume` | int | `60` | Volume inicial das chamadas, de 1 a 100. Mesma regra de rejeição de `0`/`1` do rádio |

### Cliques de microfone

| Convar | Tipo | Padrão | Descrição |
|---|---|---|---|
| `voice_onClickVolume` | int | `10` | Volume do clique ao **abrir** o rádio (0 a 100) |
| `voice_offClickVolume` | int | `3` | Volume do clique ao **fechar** o rádio (0 a 100) |

O jogador pode ligar/desligar os próprios cliques; a preferência fica em KVP local (`pma-voice_enableMicClicks`), alterada via export `setVoiceProperty('micClicks', ...)`.

### Servidor Mumble externo e diagnóstico

| Convar | Tipo | Padrão | Descrição |
|---|---|---|---|
| `voice_externalAddress` | string | `""` | Endereço de um servidor Mumble externo. Mudanças em runtime reconectam os clients automaticamente |
| `voice_externalPort` | int | `0` | Porta do servidor Mumble externo |
| `voice_externalDisallowJoin` | int | `0` | Com `1`, o servidor **recusa todas as conexões** de jogadores. Use só em servidores dedicados a hospedar o Mumble |
| `voice_hideEndpoints` | int | `1` | Oculta o endereço do servidor Mumble nos logs |
| `voice_debugMode` | int | `0` | `1` liga os logs de info, `4` liga os logs verbosos (inclui dump das tabelas de rádio) |
| `voice_allowSetIntent` | int | `1` | Permite ao jogador usar o `/setvoiceintent` |

> O `fxmanifest.lua` declara uma convar `voice_uiRefreshRate` no painel de configuração do FiveM, mas o código lê **`voice_refreshRate`**. Use `voice_refreshRate`.

---

## Proximidade

Três modos, definidos em `shared.lua`, com distâncias que dependem de `voice_useNativeAudio`:

| Modo | Distância (padrão) | Distância (`voice_useNativeAudio = true`) |
|---|---|---|
| Whisper | 3.0 | 1.5 |
| Normal | 7.0 | 3.0 |
| Shouting | 15.0 | 6.0 |

O jogador cicla entre eles com a tecla de `voice_defaultCycle` (F11). Outros recursos podem adicionar e remover modos (`addVoiceMode` / `removeVoiceMode`), forçar um alcance fixo (`overrideProximityRange`) ou substituir inteiramente a lógica de quem entra no alcance (`overrideProximityCheck`).

Quando o jogador está em modo espectador ou com uma câmera renderizada, o recurso liga automaticamente o modo "ouvinte", em que ele escuta todos os jogadores independentemente da distância.

---

## Rádio

Um jogador entra em um canal numérico (`setRadioChannel`) e fala segurando a tecla de `voice_defaultRadio` (ALT). Enquanto fala, os outros do canal ouvem sua voz com o submix de rádio e o clique de microfone.

O rádio é bloqueado quando:

- `voice_enableRadios` está em `0`;
- o jogador está morto (`IsPlayerDead` ou state bag `isDead`);
- `radioEnabled` foi desligado via `setVoiceProperty('radioEnabled', false)`;
- a state bag `disableRadio` tem **qualquer** bit ligado.

### `disableRadio` (bitwise)

Outros recursos ligam e desligam bits para bloquear o rádio por motivos diferentes, sem pisar uns nos outros. Use os exports `addRadioDisableBit` / `removeRadioDisableBit`. O rádio só volta a funcionar quando o valor chega a `0`.

```lua
-- exemplo: bloquear rádio enquanto algemado
exports['pma-voice']:addRadioDisableBit(2)
exports['pma-voice']:removeRadioDisableBit(2)
```

Os bits não são impostos pelo recurso — a convenção fica a cargo dos recursos que os usam (ex.: 1 = morto, 2 = algemado, 8 = submerso, 16 = sem item de rádio).

### Restringir canais

O servidor pode registrar uma checagem por canal com `addChannelCheck`. Se a função retornar `false`, o jogador é removido do canal e o client recebe `pma-voice:radioChangeRejected`. As checagens são apagadas automaticamente quando o recurso que as registrou para.

---

## Chamadas

Canais de chamada funcionam como salas: todos no mesmo `callChannel` se ouvem, com o submix de telefone, independentemente da distância. Não há tecla — a voz de proximidade normal é o que entra na chamada.

`setCallChannel(0)` (ou o export `removePlayerFromCall`) tira o jogador da chamada. É o mecanismo usado por recursos de telefone.

---

## Comandos

| Comando | Permissão | Descrição |
|---|---|---|
| `/muteply <id> [duração]` | ACE `command.muteply` | Alterna o mute de um jogador no Mumble. A duração é em segundos e o padrão é 900 (15 min); passado o tempo, o mute cai sozinho. Rodar o comando de novo antes disso desfaz o mute |
| `/setvoiceintent <speech\|music>` | Qualquer jogador | Define o perfil de captura do microfone. `speech` (padrão) liga supressão de ruído e filtro passa-alta; `music` desliga os dois. Só funciona com `voice_allowSetIntent = 1` |
| `/vol <1-100>` | Qualquer jogador | Ajusta o volume do rádio **e** das chamadas de uma vez |
| `/cycleproximity` | Qualquer jogador | Alterna entre os modos de proximidade. Mapeado em F11 por padrão |
| `+radiotalk` / `-radiotalk` | Qualquer jogador | Comandos de keybind (segurar/soltar) para falar no rádio. Mapeados em ALT esquerdo por padrão |

---

## State bags

O estado de voz de cada jogador é exposto em state bags, legíveis do servidor com `Player(source).state` e do client com `Player(serverId).state`.

| State bag | Tipo | Descrição |
|---|---|---|
| `proximity` | tabela | `{ index, distance, mode }` — modo de proximidade atual do jogador |
| `radioChannel` | int | Canal de rádio atual. `0` = fora do rádio |
| `callChannel` | int | Canal de chamada atual. `0` = fora de chamada |
| `radioActive` | bool | `true` enquanto o jogador está com a tecla de rádio pressionada |
| `voiceIntent` | string | `speech` ou `music` |
| `disableRadio` | int | Bitfield de bloqueios do rádio. `0` = rádio liberado |
| `radio` / `call` | int | Volume atual de rádio e de chamada do jogador (1 a 100) |
| `submix` | string | Submix aplicado à voz do jogador. Mudanças são aplicadas automaticamente nos outros clients |
| `assignedChannel` | int | Canal Mumble interno do jogador. Uso interno |
| `muted` | bool | Definido pelo `/muteply` |
| `isDead` | bool | **Lido, não escrito** pelo pma-voice. Seu recurso de morte deve setar esta state bag para bloquear o rádio de quem está morto |
| `disableProximity` | bool | **Lida** pelo recurso: quando `true`, o jogador não transmite nem recebe voz por proximidade |

Exemplo, do lado do servidor:

```lua
local canal = Player(source).state.radioChannel
if canal and canal > 0 then
    print(('%s está no canal %s'):format(GetPlayerName(source), canal))
end
```

---

## Integrações

### Recursos de telefone

Qualquer recurso de telefone integra chamadas colocando os dois lados no mesmo canal:

```lua
-- client, ao atender
exports['pma-voice']:setCallChannel(idDaLigacao)
-- ao desligar
exports['pma-voice']:setCallChannel(0)
```

Do lado do servidor, o equivalente é `exports['pma-voice']:setPlayerCall(source, canal)`.

### Recursos de rádio com restrição de job

O servidor pode barrar canais por job, gangue ou item registrando uma checagem:

```lua
exports['pma-voice']:addChannelCheck(1, function(source)
    local player = exports.qbx_core:GetPlayer(source)
    return player and player.PlayerData.job.name == 'police'
end)
```

### Recursos de morte e de algemas

O pma-voice não sabe quando o jogador morre ou é algemado — quem informa é o outro recurso, pela state bag `isDead` ou pelos bits de `disableRadio`:

```lua
-- client, no seu recurso de morte
LocalPlayer.state:set('isDead', true, false)
```

### Compatibilidade com mumble-voip / TokoVoip

O manifest declara `provides` para `mumble-voip`, `tokovoip`, `toko-voip` e `tokovoip_script`, e o recurso expõe os aliases `SetRadioChannel`, `SetCallChannel`, `SetMumbleProperty` e `SetTokoProperty`. Recursos escritos para esses sistemas funcionam sem alteração — mas os recursos originais **não** podem estar rodando.

---

## Entrypoints para outros recursos

### Exports de client

```lua
-- Rádio
exports['pma-voice']:setRadioChannel(canal)      -- 0 remove do rádio
exports['pma-voice']:addPlayerToRadio(canal)
exports['pma-voice']:removePlayerFromRadio()
exports['pma-voice']:addRadioDisableBit(bit)
exports['pma-voice']:removeRadioDisableBit(bit)
exports['pma-voice']:toggleRadioAnim()           -- alterna a animação
exports['pma-voice']:setDisableRadioAnim(bool)
exports['pma-voice']:getRadioAnimState()

-- Chamadas
exports['pma-voice']:setCallChannel(canal)       -- 0 encerra a chamada
exports['pma-voice']:addPlayerToCall(canal)
exports['pma-voice']:removePlayerFromCall()

-- Volume
exports['pma-voice']:setRadioVolume(vol)         -- 1 a 100
exports['pma-voice']:getRadioVolume()
exports['pma-voice']:setCallVolume(vol)
exports['pma-voice']:getCallVolume()

-- Mute local
exports['pma-voice']:toggleMutePlayer(serverId)
exports['pma-voice']:isPlayerMuted(serverId)
exports['pma-voice']:getMutedPlayers()

-- Propriedades
exports['pma-voice']:setVoiceProperty('radioEnabled', true)  -- ou 'micClicks'

-- Proximidade
exports['pma-voice']:addVoiceMode(distancia, nome)
exports['pma-voice']:removeVoiceMode(nome)
exports['pma-voice']:overrideProximityRange(alcance, desativarCiclo)
exports['pma-voice']:clearProximityOverride()
exports['pma-voice']:overrideProximityCheck(fn)  -- fn(ply) -> bool, distancia
exports['pma-voice']:resetProximityCheck()
exports['pma-voice']:setAllowProximityCycleState(bool)
exports['pma-voice']:setListenerOverride(bool)   -- ouve todos, ignorando distância
exports['pma-voice']:setVoiceState('proximity')  -- ou 'channel', com um canal

-- Submix
exports['pma-voice']:registerCustomSubmix(callback)  -- callback -> { nome, submixId }
exports['pma-voice']:setEffectSubmix('radio', submixId)  -- ou 'call'
```

Os overrides de proximidade e o proximity check são resetados automaticamente se o recurso que os registrou for parado.

### Exports de servidor

```lua
exports['pma-voice']:setPlayerRadio(source, canal)
exports['pma-voice']:setPlayerCall(source, canal)
exports['pma-voice']:getPlayersInRadioChannel(canal)   -- retorna { [serverId] = estaFalando }
exports['pma-voice']:addChannelCheck(canal, fn)        -- fn(source) -> bool
exports['pma-voice']:overrideRadioNameGetter(canal, fn) -- fn(source) -> string
```

### Eventos de client

```lua
-- disparado ao começar/parar de falar no rádio
AddEventHandler('pma-voice:radioActive', function(ativo) end)

-- disparado ao trocar de modo de proximidade
AddEventHandler('pma-voice:setTalkingMode', function(modo) end)

-- disparado quando a animação de rádio é ligada/desligada
AddEventHandler('pma-voice:toggleRadioAnim', function(desativada) end)

-- momento certo para registrar submixes customizados
AddEventHandler('pma-voice:registerCustomSubmixes', function() end)

-- retorna a Cfg do recurso (voiceModes)
TriggerEvent('pma-voice:settingsCallback', function(cfg) end)
```

### Evento de servidor

```lua
-- emitido pelo /muteply
AddEventHandler('pma-voice:playerMuted', function(alvo, autor, estaMutado, duracao) end)
```

---

## Estrutura de arquivos

```
pma-voice/
├── client/
│   ├── commands.lua          — /setvoiceintent, /vol, /cycleproximity e overrides de proximidade
│   ├── events.lua            — conexão/desconexão do Mumble e estado inicial dos alvos de voz
│   ├── init/
│   │   ├── init.lua          — inicialização do KVP de mic clicks e reentrada em rádio/chamada
│   │   ├── main.lua          — volumes, submixes de rádio e chamada, mute local, setVoiceProperty
│   │   ├── proximity.lua     — loop de alvos de voz, modo espectador/ouvinte, modos de voz
│   │   └── submix.lua        — aplica a state bag `submix` na voz dos outros jogadores
│   ├── module/
│   │   ├── phone.lua         — canais de chamada
│   │   └── radio.lua         — canais de rádio, +radiotalk/-radiotalk, animação, bits de disableRadio
│   └── utils/
│       └── Nui.lua           — fila de mensagens para a UI (só envia depois do uiReady)
├── server/
│   ├── main.lua              — state bags, canais Mumble por jogador, defaults de convar, getters
│   ├── mute.js               — comando /muteply (em JS por causa do clearTimeout)
│   └── module/
│       ├── phone.lua         — setPlayerCall e sincronização das chamadas
│       └── radio.lua         — setPlayerRadio, addChannelCheck, overrideRadioNameGetter
├── ui/                       — UI compilada (indicador de voz) e os sons de clique de microfone
├── voice-ui/                 — código-fonte Vue da UI
├── docs/                     — documentação por export, do upstream
├── shared.lua                — Cfg.voiceModes, logger e type_check
└── fxmanifest.lua
```
