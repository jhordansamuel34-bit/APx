-- Colocar este Script em ServerScriptService

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

-- Criar RemoteEvents (se já existirem, reaproveita)
local clickEvent = ReplicatedStorage:FindFirstChild("ClickEvent")
if not clickEvent then
    clickEvent = Instance.new("RemoteEvent")
    clickEvent.Name = "ClickEvent"
    clickEvent.Parent = ReplicatedStorage
end

local multiplierUpdateEvent = ReplicatedStorage:FindFirstChild("MultiplierUpdateEvent")
if not multiplierUpdateEvent then
    multiplierUpdateEvent = Instance.new("RemoteEvent")
    multiplierUpdateEvent.Name = "MultiplierUpdateEvent"
    multiplierUpdateEvent.Parent = ReplicatedStorage
end

-- Tabela para armazenar estado do multiplicador por jogador no servidor
local playerMultipliers = {} -- [player] = {enabled = false, value = 1}

-- Configurações de segurança / limites
local MAX_MULTIPLIER = 10
local MIN_MULTIPLIER = 1
local BASE_CLICK_AMOUNT = 1

-- Cria leaderstats "Clicks"
local function setupLeaderstats(player)
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player

    local clicks = Instance.new("IntValue")
    clicks.Name = "Clicks"
    clicks.Value = 0
    clicks.Parent = leaderstats
end

Players.PlayerAdded:Connect(function(player)
    setupLeaderstats(player)
    -- estado inicial do multiplicador
    playerMultipliers[player] = { enabled = false, value = 1 }
end)

Players.PlayerRemoving:Connect(function(player)
    playerMultipliers[player] = nil
end)

-- Atualiza estado do multiplicador a pedido do cliente (o servidor faz clamp)
multiplierUpdateEvent.OnServerEvent:Connect(function(player, enabled, value)
    if not player then return end
    value = tonumber(value) or 1
    if value < MIN_MULTIPLIER then value = MIN_MULTIPLIER end
    if value > MAX_MULTIPLIER then value = MAX_MULTIPLIER end
    playerMultipliers[player] = { enabled = enabled and true or false, value = math.floor(value) }
    -- Opcional: enviar confirmação de volta ao cliente (poderia usar outro RemoteEvent ou Bindable)
end)

-- Processa cliques pedidos pelo cliente
clickEvent.OnServerEvent:Connect(function(player, extraAmount)
    if not player then return end
    local multState = playerMultipliers[player] or { enabled = false, value = 1 }
    local extra = tonumber(extraAmount) or 0
    local increment = BASE_CLICK_AMOUNT + extra
    if multState.enabled then
        increment = math.floor(increment * multState.value)
    end

    -- Segurança: limitar ganho por clique para evitar exploração
    local MAX_GAIN_PER_CLICK = 1000
    if increment > MAX_GAIN_PER_CLICK then
        increment = MAX_GAIN_PER_CLICK
    end

    local clicks = player:FindFirstChild("leaderstats") and player.leaderstats:FindFirstChild("Clicks")
    if clicks and clicks:IsA("IntValue") then
        clicks.Value = clicks.Value + increment
    end
end)
