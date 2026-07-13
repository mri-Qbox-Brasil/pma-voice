# 🎤 pma-voice - Manual de Funcionalidades

Sistema de voz de alta performance para FiveM/RedM usando servidor mumble interno com áudio nativo, rádio e chamadas.

**Versão:** Latest | **Framework:** Todos | **Licença:** MIT

---

## 🎯 O que o pma-voice faz

O pma-voice é uma solução completa de VOIP para servidores FiveM/RedM. Ele gerencia chat de voz por proximidade, comunicação via rádio, chamadas telefônicas e aplica efeitos de áudio 3D com suporte a submix (eco, reverberação). O sistema é altamente otimizado com 0.00ms de uso idle.

---

## ⚙️ Como funciona

O pma-voice utiliza o servidor mumble interno do FiveM/RedM para transmissão de áudio. O sistema gerencia canais de voz, aplica efeitos de áudio baseados em distância e sincroniza o estado de voz entre todos os jogadores conectados.

### Sistema de Proximidade
- Voz transmitida baseada na distância entre jogadores
- Múltiplos modos de proximidade (sussurro, normal, grito)
- Ciclo entre modos via tecla configurável (padrão: F11)

### Rádio
- Canais de rádio com restrição de acesso
- Efeito de submix aplicado (som de rádio)
- Suporte a múltiplos canais simultâneos

### Chamadas
- Chamadas ponto-a-ponto entre jogadores
- Integração com sistemas de telefone (como NPWD)
- Volume independente

---

## 🔧 Configuração

Configure tudo via convars no `server.cfg`:

```cfg
# Configurações de Áudio
setr voice_useNativeAudio false      # Áudio nativo do jogo (efeitos 3D)
setr voice_use2dAudio false          # Áudio 2D (volume constante)
setr voice_use3dAudio false          # Áudio posicional 3D
setr voice_useSendingRangeOnly false # Apenas alcance de envio

# Interface e Controles
setr voice_enableUi 1                # Habilitar UI de voz (0/1)
setr voice_enableProximityCycle 1    # Ciclo de proximidade (0/1)
setr voice_defaultCycle "F11"        # Tecla para ciclar modos
setr voice_defaultRadio "LMENU"      # Tecla para usar rádio (ALT)

# Rádio e Chamadas
setr voice_enableRadios 1            # Sistema de rádio (0/1)
setr voice_enableCalls 1             # Sistema de chamadas (0/1)
setr voice_enableSubmix 1            # Efeitos de submix (0/1)
setr voice_defaultRadioVolume 30     # Volume padrão rádio (1-100)
setr voice_defaultCallVolume 60      # Volume padrão chamadas (1-100)

# Servidor Externo (Opcional)
setr voice_externalAddress ""        # IP do servidor mumble externo
setr voice_externalPort 0            # Porta do servidor externo
```

### Referência de ConVars
| ConVar | Padrão | Descrição | Tipo |
|--------|---------|-----------|------|
| `voice_useNativeAudio` | false | Usar áudio nativo com efeitos 3D | boolean |
| `voice_enableUi` | 1 | Mostrar UI de voz | int (0/1) |
| `voice_enableProximityCycle` | 1 | Habilitar tecla F11 | int (0/1) |
| `voice_defaultRadio` | "LMENU" | Tecla para rádio (LMENU=ALT) | string |
| `voice_defaultRadioVolume` | 30 | Volume do rádio | int (1-100) |
| `voice_enableRadios` | 1 | Sistema de rádio | int (0/1) |
| `voice_enableCalls` | 1 | Sistema de chamadas | int (0/1) |
| `voice_enableSubmix` | 1 | Efeitos de submix | int (0/1) |
| `voice_allowSetIntent` | 1 | Permitir definir intent (speech/music) | int (0/1) |
| `voice_debugMode` | 0 | Log de debug (1=básico, 4=verboso) | int |

---

## 📤 Exports

### Exports do Cliente
| Export | Descrição | Parâmetros |
|--------|-----------|-------------|
| `setVoiceProperty` | Define configuração de voz | `property` (string), `value` (any) |
| `setRadioChannel` | Define canal de rádio | `channel` (int) |
| `setCallChannel` | Define canal de chamada | `channel` (int) |
| `setRadioVolume` | Define volume do rádio | `volume` (int, 1-100) |
| `setCallVolume` | Define volume de chamada | `volume` (int, 1-100) |
| `addPlayerToRadio` | Entra no canal de rádio | `channel` (int) |
| `addPlayerToCall` | Entra no canal de chamada | `channel` (int) |
| `removePlayerFromRadio` | Sai do rádio | None |
| `removePlayerFromCall` | Sai da chamada | None |
| `toggleMutePlayer` | Alterna mudo de jogador | `serverId` (int) |

### Exports do Servidor
| Export | Descrição | Parâmetros |
|--------|-----------|-------------|
| `setPlayerRadio` | Define rádio do jogador | `source` (int), `channel` (int) |
| `setPlayerCall` | Define chamada do jogador | `source` (int), `channel` (int) |
| `addChannelCheck` | Adiciona verificação de acesso | `channel` (int), `callback` (function) |
| `getPlayersInRadioChannel` | Lista jogadores no rádio | `channel` (int) |

---

## 📡 Eventos

### Eventos do Cliente
| Evento | Descrição | Parâmetros |
|--------|-----------|-------------|
| `pma-voice:settingsCallback` | Retorna configurações atuais | `cb(voiceSettings)` |
| `pma-voice:radioActive` | Rádio ativado/desativado | `boolean` |
| `pma-voice:setTalkingMode` | Mudança de modo de proximidade | `modeId` (int) |

---

## 💻 State Bags

Acesse o estado do jogador via `Player(source).state`:

| State Bag | Descrição | Tipo |
|-----------|-----------|------|
| `proximity` | Tabela com modo, distância e nome | table |
| `radioChannel` | Canal de rádio atual (0 = nenhum) | int |
| `callChannel` | Canal de chamada atual (0 = nenhum) | int |
| `voiceIntent` | Intent ('speech' ou 'music') | string |
| `disableRadio` | Estado de rádio desabilitado (bitwise) | int |

### Estados de Rádio Desabilitado (Bitwise)
```lua
enum DisabledRadioStates {
    Enabled = 0,
    IsDead = 1,
    IsCuffed = 2,
    IsPdCuffed = 4,
    IsUnderWater = 8,
    DoesntHaveItem = 16,
    PlayerDisabledRadio = 32,
}
```

---

## 🎮 Comandos

| Comando | Descrição | Permissão |
|---------|-----------|------------|
| `/muteply [target] [duration]` | Mutar jogador | ACE: `command.muteply` |

### Configuração ACE para Comando
```cfg
add_ace group.superadmin command.muteply allow
add_ace group.admin command.muteply allow
```

---

## 🔗 Integrações

### Compatibilidade
O pma-voice fornece compatibilidade com APIs:
- **mumble-voip** - Compatibilidade de exports
- **toko-voip** - Compatibilidade parcial

### Integração com NPWD (Telefone)
```lua
-- NPWD usa pma-voice para chamadas telefônicas
exports['pma-voice']:setCallChannel(phoneCallChannel)
```

### Verificação de Canal de Rádio (Servidor)
```lua
exports['pma-voice']:addChannelCheck(1, function(source)
    local Player = QBCore.Functions.GetPlayer(source)
    return Player.PlayerData.job.name == 'police'
end)
```

---

## 💡 Casos de Uso

### Entrar no Rádio da Polícia (Cliente)
```lua
exports['pma-voice']:setRadioChannel(1)
print("Entrou no canal de rádio 1")
```

### Verificar Canal de Rádio do Jogador (Servidor)
```lua
local radioChannel = Player(source).state.radioChannel
if radioChannel and radioChannel > 0 then
    print(("Jogador %s está no canal de rádio %s"):format(GetPlayerName(source), radioChannel))
end
```

### Sistema de Rádio com Restrição de Job
```lua
RegisterCommand('radio', function(source, args)
    local channel = tonumber(args[1])
    if not channel then return end
    
    local Player = QBCore.Functions.GetPlayer(source)
    local restrictedChannels = {
        [1] = 'police',
        [2] = 'ambulance',
        [3] = 'mechanic'
    }
    
    if restrictedChannels[channel] then
        if Player.PlayerData.job.name == restrictedChannels[channel] then
            exports['pma-voice']:setPlayerRadio(source, channel)
        else
            TriggerClientEvent('ox_lib:notify', source, {
                title = 'Erro',
                description = 'Sem permissão para este canal',
                type = 'error'
            })
        end
    else
        exports['pma-voice']:setPlayerRadio(source, channel)
    end
end)
```

### Desabilitar Rádio quando Morto
```lua
-- Cliente
AddEventHandler('esx:onPlayerDeath', function()
    local state = LocalPlayer.state
    state:set('disableRadio', 1, true)  -- IsDead = 1
end)

AddEventHandler('esx:onPlayerSpawn', function()
    local state = LocalPlayer.state
    state:set('disableRadio', 0, true)  -- Enabled = 0
end)
```

---

## ⚠️ Solução de Problemas

### Voz não funciona
- Verifique se o OneSync está habilitado no servidor
- Confirme que `ensure pma-voice` está no server.cfg
- Verifique se não há conflito com outros sistemas de voz

### Não compatível com outros sistemas de voz
**IMPORTANTE:** pma-voice NÃO é compatível com:
- vMenu voice
- toko-voip (simultâneo)
- mumble-voip (simultâneo)

Se usar vMenu, [desative o chat de voz](https://docs.vespura.com/vmenu/faq/#q-how-do-i-disable-voice-chat).

### Não sobrescreva estas natives:
- `NetworkSetTalkerProximity`
- `MumbleSetTalkerProximity`
- `MumbleSetAudioInputDistance`
- `MumbleSetAudioOutputDistance`
- `NetworkSetVoiceActive`

### Rádio não funciona
- Verifique se `voice_enableRadios` está definido como 1
- Confirme que o jogador tem a tecla de rádio configurada (padrão: ALT)
- Verifique as restrições de canal com `addChannelCheck`

### Chamadas não conectam
- Verifique se `voice_enableCalls` está habilitado
- Confirme que o NPWD ou sistema de telefone está integrado
- Verifique o volume das chamadas

### Áudio 3D não funciona (RedM)
O áudio nativo não é suportado no RedM. Use configurações padrão.

---

## ⚠️ Aviso de Compatibilidade
Este script **NÃO** é compatível com outros sistemas de voz simultâneos. Use apenas um sistema de voz por servidor.

---

## 📚 Links
- [Documentação](https://github.com/AvarianKnight/pma-voice/wiki)
- [Issue Tracker](https://github.com/AvarianKnight/pma-voice/issues)
- [Builds Recomendadas](https://runtime.fivem.net/artifacts/)
