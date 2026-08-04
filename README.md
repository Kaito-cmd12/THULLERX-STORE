-- ======================================================================
-- THULLERX STORE - CHALICE FINDER (DETECTOR DE CÁLICE SAGRADO)
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Chalice Finder",
    TabWidth = 160,
    Size = UDim2.fromOffset(500, 320),
    Acrylic = true,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.K
})

-- MARCA D'ÁGUA (THULLERX STORE)
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local TextLabel = Instance.new("TextLabel")
local StatusLabel = Instance.new("TextLabel")

ScreenGui.Parent = game:GetService("CoreGui")

MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.BorderColor3 = Color3.fromRGB(255, 50, 50)
MainFrame.BorderSizePixel = 2
MainFrame.Position = UDim2.new(0.01, 0, 0.02, 0)
MainFrame.Size = UDim2.new(0, 210, 0, 50)

TextLabel.Parent = MainFrame
TextLabel.BackgroundTransparency = 1
TextLabel.Position = UDim2.new(0.05, 0, 0.1, 0)
TextLabel.Size = UDim2.new(0, 190, 0, 20)
TextLabel.Font = Enum.Font.SourceSansBold
TextLabel.Text = "THULLERX STORE"
TextLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
TextLabel.TextSize = 17.0
TextLabel.TextXAlignment = Enum.TextXAlignment.Left

StatusLabel.Parent = MainFrame
StatusLabel.BackgroundTransparency = 1
StatusLabel.Position = UDim2.new(0.05, 0, 0.55, 0)
StatusLabel.Size = UDim2.new(0, 190, 0, 18)
StatusLabel.Font = Enum.Font.SourceSansItalic
StatusLabel.Text = "Status: Aguardando Checagem"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatusLabel.TextSize = 13.0
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

local Tabs = {
    Main = Window:AddTab({ Title = "Detector", Icon = "search" })
}

-- FUNÇÃO DE VERIFICAÇÃO DO CÁLICE
local function CheckForChalice()
    local found = false
    local chaliceObject = nil

    -- 1. Procura em itens jogados no chão pelo mapa
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Tool") or obj:IsA("Model") then
            if string.find(string.lower(obj.Name), "chalice") or string.find(string.lower(obj.Name), "god's chalice") then
                found = true
                chaliceObject = obj
                break
            end
        end
    end

    -- 2. Procura no inventário dos jogadores no servidor
    if not found then
        for _, player in pairs(game:GetService("Players"):GetPlayers()) do
            local backpack = player:FindFirstChild("Backpack")
            local char = player.Character
            
            if backpack and (backpack:FindFirstChild("God's Chalice") or backpack:FindFirstChild("Chalice")) then
                found = true
                break
            elseif char and (char:FindFirstChild("God's Chalice") or char:FindFirstChild("Chalice")) then
                found = true
                break
            end
        end
    end

    -- Atualiza a interface e dispara o aviso
    if found then
        StatusLabel.Text = "CÁLICE ENCONTRADO!"
        StatusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
        
        Fluent:Notify({
            Title = "THULLERX STORE",
            Content = "O CÁLICE SAGRADO ESTÁ NO SERVIDO!",
            Duration = 10
        })
    else
        StatusLabel.Text = "Cálice: Não encontrado"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
        
        Fluent:Notify({
            Title = "THULLERX STORE",
            Content = "Nenhum Cálice encontrado neste servidor.",
            Duration = 5
        })
    end
end

-- INTERFACE
Tabs.Main:AddSection("Verificação do Cálice Sagrado")

Tabs.Main:AddButton({
    Title = "Checar Cálice no Servidor",
    Callback = function()
        CheckForChalice()
    end
})

Tabs.Main:AddToggle("AutoCheckToggle", {
    Title = "Aviso Automático em Tempo Real",
    Default = false,
    Callback = function(Value)
        _G.AutoCheckChalice = Value
    end
})

-- LOOP DE CHECAGEM AUTOMÁTICA
task.spawn(function()
    while true do
        task.wait(5)
        if _G.AutoCheckChalice then
            CheckForChalice()
        end
    end
end)

Window:SelectTab(1)
