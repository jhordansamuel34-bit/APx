local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local QAEvent = ReplicatedStorage:WaitForChild("QAControlEvent")

-- ========== GUI ==========
local gui = Instance.new("ScreenGui")
gui.Name = "QAGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.fromScale(0.2, 0.32)
frame.Position = UDim2.fromScale(0.75, 0.32)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 10)

local layout = Instance.new("UIListLayout", frame)
layout.Padding = UDim.new(0, 6)

local padding = Instance.new("UIPadding", frame)
padding.PaddingTop = UDim.new(0, 8)
padding.PaddingBottom = UDim.new(0, 8)
padding.PaddingLeft = UDim.new(0, 8)
padding.PaddingRight = UDim.new(0, 8)

-- ========== STATUS ==========
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 0, 36)
statusLabel.TextScaled = true
statusLabel.Font = Enum.Font.GothamBold
statusLabel.BackgroundTransparency = 1
statusLabel.TextColor3 = Color3.fromRGB(200, 80, 80)
statusLabel.Text = "QA: DESATIVADO"
statusLabel.Parent = frame

-- ========== MULTIPLIER LABEL ==========
local multiplierLabel = Instance.new("TextLabel")
multiplierLabel.Size = UDim2.new(1, 0, 0, 30)
multiplierLabel.TextScaled = true
multiplierLabel.Font = Enum.Font.Gotham
multiplierLabel.BackgroundTransparency = 1
multiplierLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
multiplierLabel.Text = "Multiplicador: x1"
multiplierLabel.Parent = frame

-- ========== BOTÕES ==========
local buttons = {}

local function createButton(text, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    btn.BackgroundColor3 = color
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Parent = frame

    local c = Instance.new("UICorner", btn)
    c.CornerRadius = UDim.new(0, 8)

    btn.MouseButton1Click:Connect(callback)
    table.insert(buttons, btn)
end

createButton("Ativar x100", Color3.fromRGB(60, 120, 255), function()
    QAEvent:FireServer("ENABLE", 100)
end)

createButton("Ativar x1000", Color3.fromRGB(80, 160, 255), function()
    QAEvent:FireServer("ENABLE", 1000)
end)

createButton("Ativar x1M", Color3.fromRGB(100, 200, 255), function()
    QAEvent:FireServer("ENABLE", 1000000)
end)

createButton("Desativar QA", Color3.fromRGB(180, 60, 60), function()
    QAEvent:FireServer("DISABLE")
end)

-- ========== ATUALIZAÇÃO VISUAL ==========
local function updateUI()
    local enabled = player:GetAttribute("QAEnabled")
    local multiplier = player:GetAttribute("QAMultiplier") or 1

    if enabled then
        statusLabel.Text = "QA: ATIVO"
        statusLabel.TextColor3 = Color3.fromRGB(80, 200, 120)
        multiplierLabel.Text = "Multiplicador: x" .. tostring(multiplier)
    else
        statusLabel.Text = "QA: DESATIVADO"
        statusLabel.TextColor3 = Color3.fromRGB(200, 80, 80)
        multiplierLabel.Text = "Multiplicador: x1"
    end
end

-- Atualiza quando servidor muda
player:GetAttributeChangedSignal("QAEnabled"):Connect(updateUI)
player:GetAttributeChangedSignal("QAMultiplier"):Connect(updateUI)

-- Inicial
task.delay(1, updateUI)# APx
