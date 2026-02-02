```
═══════════════════════════════════════════════════════════════════════════════
🐺 LXRCore-AI-Seek - Framework Integration Guide
   The Land of Wolves 🐺 | მგლების მიწა
═══════════════════════════════════════════════════════════════════════════════
```

# Framework Integration Guide

## 🎯 Overview

LXRCore-AI-Seek is designed to integrate seamlessly with multiple RedM/FiveM frameworks, providing AI-powered capabilities to your gaming server.

```
═══════════════════════════════════════════════════════════════════════════════
█████ SUPPORTED FRAMEWORKS
═══════════════════════════════════════════════════════════════════════════════
```

### Primary Frameworks (Priority Support)

#### 1. LXR-Core 🐺
- **Status**: Primary Framework
- **Priority**: Highest
- **Support Level**: Full
- **Developer**: iBoss21 / The Lux Empire
- **Description**: The flagship framework for The Land of Wolves ecosystem

#### 2. RSG-Core
- **Status**: Primary Framework
- **Priority**: High
- **Support Level**: Full
- **Description**: Popular RedM framework with extensive community support

### Supported Frameworks

#### 3. VORP Core
- **Status**: Supported/Legacy
- **Priority**: Medium
- **Support Level**: Compatible
- **Description**: Established RedM framework with large user base

---

## 🔧 Integration Methods

```
═══════════════════════════════════════════════════════════════════════════════
█████ INTEGRATION PATTERNS
═══════════════════════════════════════════════════════════════════════════════
```

### Auto-Detection System

LXRCore-AI-Seek automatically detects the active framework:

```lua
-- Framework Priority (in order)
1. LXR-Core    (Primary)
2. RSG-Core    (Primary)
3. VORP Core   (Supported)
4. Standalone  (Fallback)
```

### Framework Adapter Layer

The adapter provides unified APIs across frameworks:

```lua
-- Unified function calls
Framework:Notify(source, message, type)
Framework:GetPlayer(source)
Framework:AddMoney(source, account, amount)
Framework:RemoveItem(source, item, count)
```

Internal mapping ensures compatibility:
- LXR-Core → `LXR:Client:Notify`, `LXR:Server:GetPlayer`
- RSG-Core → `RSGCore:Client:Notify`, `RSGCore.Functions.GetPlayer`
- VORP → `VORPcore.NotifyRightTip`, `VORPcore.getUser`

---

## 📦 Installation by Framework

```
═══════════════════════════════════════════════════════════════════════════════
█████ FRAMEWORK-SPECIFIC SETUP
═══════════════════════════════════════════════════════════════════════════════
```

### LXR-Core Integration

```lua
-- resources/[lxr]/lxr-ai-seek/fxmanifest.lua
fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM...'

description 'LXRCore-AI-Seek Integration'
version '1.0.0'
author 'iBoss21 / The Lux Empire'

dependencies {
    'lxr-core'  -- LXR-Core dependency
}

shared_scripts {
    'config.lua',
    'shared/*.lua'
}

client_scripts {
    'client/*.lua'
}

server_scripts {
    'server/*.lua'
}
```

### RSG-Core Integration

```lua
-- resources/[rsg]/rsg-ai-seek/fxmanifest.lua
fx_version 'cerulean'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM...'

description 'LXRCore-AI-Seek for RSG-Core'
version '1.0.0'

dependencies {
    'rsg-core'  -- RSG-Core dependency
}

shared_scripts {
    '@rsg-core/shared/locale.lua',
    'config.lua'
}

client_scripts {
    'client/*.lua'
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/*.lua'
}
```

### VORP Integration

```lua
-- resources/vorp-ai-seek/fxmanifest.lua
fx_version 'adamant'
game 'rdr3'
rdr3_warning 'I acknowledge that this is a prerelease build of RedM...'

description 'LXRCore-AI-Seek for VORP'
version '1.0.0'

dependencies {
    'vorp_core',
    'vorp_inventory'
}

server_scripts {
    'config.lua',
    'server/*.lua'
}

client_scripts {
    'client/*.lua'
}
```

---

## 🔌 Framework Adapter Implementation

```
═══════════════════════════════════════════════════════════════════════════════
█████ ADAPTER LAYER CODE
═══════════════════════════════════════════════════════════════════════════════
```

### Example: shared/framework.lua

```lua
Framework = {}
Framework.Type = nil

-- Auto-detect framework
if GetResourceState('lxr-core') == 'started' then
    Framework.Type = 'LXR-Core'
    Framework.Core = exports['lxr-core']:GetCoreObject()
elseif GetResourceState('rsg-core') == 'started' then
    Framework.Type = 'RSG-Core'
    Framework.Core = exports['rsg-core']:GetCoreObject()
elseif GetResourceState('vorp_core') == 'started' then
    Framework.Type = 'VORP'
    Framework.Core = exports.vorp_core:GetCore()
else
    Framework.Type = 'Standalone'
end

-- Unified notification function
function Framework:Notify(source, message, type)
    if Framework.Type == 'LXR-Core' then
        TriggerClientEvent('LXR:Client:Notify', source, message, type)
    elseif Framework.Type == 'RSG-Core' then
        TriggerClientEvent('RSGCore:Notify', source, message, type)
    elseif Framework.Type == 'VORP' then
        TriggerClientEvent('vorp:TipRight', source, message, 3000)
    else
        -- Standalone fallback
        TriggerClientEvent('chat:addMessage', source, {
            args = {message}
        })
    end
end

-- Unified player getter
function Framework:GetPlayer(source)
    if Framework.Type == 'LXR-Core' then
        return Framework.Core.Functions.GetPlayer(source)
    elseif Framework.Type == 'RSG-Core' then
        return Framework.Core.Functions.GetPlayer(source)
    elseif Framework.Type == 'VORP' then
        return Framework.Core.getUser(source)
    end
    return nil
end
```

---

## 🎮 Use Case Examples

```
═══════════════════════════════════════════════════════════════════════════════
█████ PRACTICAL EXAMPLES
═══════════════════════════════════════════════════════════════════════════════
```

### AI-Powered NPC Dialogue

```lua
-- server/npc_dialogue.lua
RegisterNetEvent('ai-seek:getNPCResponse', function(npcId, playerMessage)
    local src = source
    local Player = Framework:GetPlayer(src)
    
    if not Player then return end
    
    -- Call AI model for response
    local aiResponse = AISeek:GenerateResponse(npcId, playerMessage)
    
    -- Send response back to client
    TriggerClientEvent('ai-seek:receiveNPCResponse', src, aiResponse)
    
    -- Notify player
    Framework:Notify(src, 'NPC responded', 'success')
end)
```

### Dynamic Quest Generation

```lua
-- server/quest_generator.lua
function GenerateDynamicQuest(playerId)
    local Player = Framework:GetPlayer(playerId)
    local playerLevel = Player.PlayerData.metadata.level
    
    -- Use AI to generate appropriate quest
    local questData = AISeek:GenerateQuest({
        level = playerLevel,
        location = GetPlayerLocation(playerId),
        preferences = Player.PlayerData.metadata.questPreferences
    })
    
    -- Give quest to player
    Framework:GiveQuest(playerId, questData)
    Framework:Notify(playerId, 'New quest available!', 'info')
end
```

---

## 🔐 Security Considerations

```
═══════════════════════════════════════════════════════════════════════════════
█████ SECURITY GUIDELINES
═══════════════════════════════════════════════════════════════════════════════
```

### Server-Side Validation

Always validate AI-generated content server-side:

```lua
function ValidateAIResponse(response)
    -- Check for inappropriate content
    if ContainsBlacklistedWords(response) then
        return false, "Content filtered"
    end
    
    -- Check length limits
    if #response > Config.MaxResponseLength then
        return false, "Response too long"
    end
    
    -- Verify structure
    if not IsValidFormat(response) then
        return false, "Invalid format"
    end
    
    return true, "Valid"
end
```

### Rate Limiting

Implement per-player rate limits:

```lua
local RateLimits = {}

function CheckRateLimit(source)
    local identifier = GetPlayerIdentifier(source)
    local current = os.time()
    
    if RateLimits[identifier] then
        if current - RateLimits[identifier] < Config.RateLimitSeconds then
            return false
        end
    end
    
    RateLimits[identifier] = current
    return true
end
```

---

## 📚 Additional Resources

- [LXR-Core Documentation](https://wolves.land/docs/lxr-core)
- [RSG-Core Documentation](https://rsg-framework.com)
- [VORP Documentation](https://vorp-core.com)

---

## 🤝 Community Support

### Get Help
- **Discord**: https://discord.gg/CrKcWdfd3A
- **GitHub**: https://github.com/iboss21/TheSigma/issues

### Contributing
We welcome framework-specific improvements:
- Bug fixes for specific frameworks
- Enhanced adapter layer
- Additional framework support
- Documentation improvements

---

<div align="center">
  <p><strong>🐺 Framework Integration for The Land of Wolves 🐺</strong></p>
  <p>Made with ❤️ by iBoss21 & The Lux Empire</p>
</div>
