-- ======================================================================
-- THULLERX STORE - CHALICE FINDER (VERSÃO SEM CRASH / OTIMIZADA)
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

-- MARCA D'ÁGUA
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
StatusLabel.Text = "Status: Pronto"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatusLabel.TextSize = 13.0
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

local Tabs = {
    Main = Window:AddTab({ Title = "Detector", Icon = "search" })
}

-- BUSCA OTIMIZADA (SEM TRAVAR O JOGO)
local function CheckForChalice()
    local found = false
    local itemName = ""

    -- 1. Checa apenas os itens caídos diretamente no workspace (sem descer a árvore inteira)
    for _, obj in pairs(workspace:GetChildren()) do
        if obj:IsA("Tool") or obj:IsA("Model") then
            local name = string.lower(obj.Name)
            if string.find(name, "chalice") or string.find(name, "god") then
                found = true
                itemName = obj.Name
                break
            end
        end
    end

    -- 2. Checa inventário e mão dos jogadores
    if not found then
        for _, player in pairs(game:GetService("Players"):GetPlayers()) do
            local backpack = player:FindFirstChild("Backpack")
            local char = player.Character

            if backpack then
                for _, item in pairs(backpack:GetChildren()) do
                    if string.find(string.lower(item.Name), "chalice") then
                        found = true
                        itemName = item.Name
                        break
                    end
                end
            end

            if not found and char then
                for _, item in pairs(char:GetChildren()) do
                    if item:IsA("Tool") and string.find(string.lower(item.Name), "chalice") then
                        found = true
                        itemName = item.Name
                        break
                    end
                end
            end

            if found then break end
            task.wait() -- Micro-pausa entre checagens de jogadores para não sobrecarregar
        end
    end

    -- Retorno na interface
    if found then
        StatusLabel.Text = "CÁLICE ENCONTRADO!"
        StatusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
        Fluent:Notify({ Title = "THULLERX STORE", Content = "Cálice detectado: " .. itemName, Duration = 8 })
    else
        StatusLabel.Text = "Cálice: Não encontrado"
        StatusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
        Fluent:Notify({ Title = "THULLERX STORE", Content = "Nenhum Cálice neste servidor.", Duration = 4 })
    end
end

Tabs.Main:AddSection("Detector de Cálice Sagrado")

Tabs.Main:AddButton({
    Title = "Checar Cálice Agora",
    Callback = function()
        CheckForChalice()
    end
})

Tabs.Main:AddToggle("AutoCheckToggle", {
    Title = "Verificação Automática (10s)",
    Default = false,
    Callback = function(Value)
        _G.AutoCheckChalice = Value
    end
})

-- Loop seguro com intervalo de 10 segundos
task.spawn(function()
    while true do
        task.wait(10)
        if _G.AutoCheckChalice then
            CheckForChalice()
        end
    end
end)

Window:SelectTab(1)
