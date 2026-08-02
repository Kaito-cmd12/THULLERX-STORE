-- ======================================================================
-- THULLERX STORE - HUB OFICIAL BLOX FRUITS (AUTO LEVEL & SAFE TWEEN)
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Blox Fruits Level Hub",
    TabWidth = 160,
    Size = UDim2.fromOffset(600, 420),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.K -- Pressione K para Abrir/Fechar
})

-- Marca d'água + Instrução da Tecla K
local ScreenGui = Instance.new("ScreenGui")
local TextLabel = Instance.new("TextLabel")
local InfoLabel = Instance.new("TextLabel")

ScreenGui.Parent = game:GetService("CoreGui")

TextLabel.Parent = ScreenGui
TextLabel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
TextLabel.BackgroundTransparency = 0.3
TextLabel.Position = UDim2.new(0.01, 0, 0.01, 0)
TextLabel.Size = UDim2.new(0, 180, 0, 30)
TextLabel.Font = Enum.Font.SourceSansBold
TextLabel.Text = "THULLERX STORE"
TextLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
TextLabel.TextSize = 18.0

InfoLabel.Parent = ScreenGui
InfoLabel.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
InfoLabel.BackgroundTransparency = 0.5
InfoLabel.Position = UDim2.new(0.01, 0, 0.05, 0)
InfoLabel.Size = UDim2.new(0, 180, 0, 20)
InfoLabel.Font = Enum.Font.SourceSans
InfoLabel.Text = "Pressione [K] para Abrir/Fechar"
InfoLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
InfoLabel.TextSize = 13.0

-- Variáveis Globais de Controle
_G.AutoFarmLevel = false
_G.AutoChest = false
_G.AutoSeaEvents = false
_G.AutoSpinFruit = false
_G.AutoStoreFruit = false
_G.TweenSpeed = 250 -- Velocidade do deslocamento ajustável

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ======================================================================
-- MA PEAMENTO DE NÍVEIS E ILHAS (FIRST SEA)
-- ======================================================================
local LevelData = {
    {MinLevel = 1,   MaxLevel = 9,   MobName = "Bandit",          Position = CFrame.new(1059, 16, 1548)},
    {MinLevel = 10,  MaxLevel = 29,  MobName = "Monkey",          Position = CFrame.new(-1610, 37, 147)},
    {MinLevel = 30,  MaxLevel = 59,  MobName = "Pirate",          Position = CFrame.new(-1141, 4, 3856)},
    {MinLevel = 60,  MaxLevel = 89,  MobName = "Desert Bandit",   Position = CFrame.new(897, 6, 4388)},
    {MinLevel = 90,  MaxLevel = 119, MobName = "Snow Bandit",     Position = CFrame.new(1353, 87, -1328)},
    {MinLevel = 120, MaxLevel = 149, MobName = "Chief Petty Officer", Position = CFrame.new(-5036, 20, 4324)},
    {MinLevel = 150, MaxLevel = 174, MobName = "Sky Bandit",      Position = CFrame.new(-4839, 717, -2620)},
    {MinLevel = 175, MaxLevel = 189, MobName = "Dark Master",      Position = CFrame.new(-5250, 388, -2250)},
    {MinLevel = 190, MaxLevel = 249, MobName = "Prisoner",        Position = CFrame.new(485, 4, 735)},
    {MinLevel = 250, MaxLevel = 299, MobName = "Toga Warrior",    Position = CFrame.new(-1820, 7, -2745)},
    {MinLevel = 300, MaxLevel = 374, MobName = "Military Soldier", Position = CFrame.new(-5315, 8, 8515)},
    {MinLevel = 375, MaxLevel = 449, MobName = "Fishman Warrior", Position = CFrame.new(61122, 18, 1569)},
    {MinLevel = 450, MaxLevel = 524, MobName = "God's Guard",     Position = CFrame.new(-4720, 845, -1950)},
    {MinLevel = 525, MaxLevel = 624, MobName = "Shandora Warrior", Position = CFrame.new(-7927, 5544, -380)},
    {MinLevel = 625, MaxLevel = 699, MobName = "Galley Pirate",   Position = CFrame.new(5230, 38, 4850)}
}

-- Obtém o nível atual do jogador no Blox Fruits
local function GetPlayerLevel()
    pcall(function()
        if LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") then
            return LocalPlayer.Data.Level.Value
        end
    end)
    return 1
end

-- Busca a ilha e NPC correspondentes ao nível atual
local function GetCurrentTarget()
    local level = GetPlayerLevel()
    for _, info in ipairs(LevelData) do
        if level >= info.MinLevel and level <= info.MaxLevel then
            return info
        end
    end
    -- Padrão se o nível ultrapassar a lista
    return LevelData[#LevelData]
end

-- Função de Deslocamento Suave (Anti-Kick Tween)
local function TweenTo(targetCFrame)
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local hrp = character.HumanoidRootPart
        local distance = (hrp.Position - targetCFrame.Position).Magnitude
        local duration = distance / _G.TweenSpeed
        
        local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear)
        local tween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
        tween:Play()
        return tween
    end
end

-- Função de Ataque
local function DoAttack()
    pcall(function()
        local char = LocalPlayer.Character
        if char then
            local tool = char:FindFirstChildOfClass("Tool")
            if not tool then
                local bTool = LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
                if bTool then char.Humanoid:EquipTool(bTool) end
            else
                tool:Activate()
            end
        end
    end)
end

-- ======================================================================
-- INTERFACE E CATEGORIAS
-- ======================================================================
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "rbxassetid://4483362458" }),
    Movement = Window:AddTab({ Title = "Movimentação", Icon = "rbxassetid://4483362458" }),
    Fruit = Window:AddTab({ Title = "Frutas", Icon = "rbxassetid://4483362458" }),
    Settings = Window:AddTab({ Title = "Aparência", Icon = "rbxassetid://4483362458" })
}

-- ======================================================================
-- 1. AUTO FARM POR NÍVEL
-- ======================================================================
Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Auto Farm Level (Detecta Ilha Automaticamente)",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
    end
})

Tabs.Main:AddToggle("AutoChestToggle", {
    Title = "Auto Farm Baús (Navegação)",
    Default = false,
    Callback = function(Value)
        _G.AutoChest = Value
    end
})

-- Loop Inteligente de Farm por Nível
task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoFarmLevel then
            pcall(function()
                local targetInfo = GetCurrentTarget()
                local enemies = workspace:FindFirstChild("Enemies")
                local targetEnemy = nil

                -- Procura o mob específico da ilha atual no workspace
                if enemies then
                    for _, enemy in pairs(enemies:GetChildren()) do
                        if enemy.Name == targetInfo.MobName and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                            targetEnemy = enemy
                            break
                        end
                    end
                end

                -- Se encontrar o mob da ilha, vai até ele e ataca
                if targetEnemy then
                    while _G.AutoFarmLevel and targetEnemy and targetEnemy:FindFirstChild("Humanoid") and targetEnemy.Humanoid.Health > 0 do
                        task.wait(0.05)
                        local mobPos = targetEnemy.HumanoidRootPart.CFrame * CFrame.new(0, 8, 0)
                        local tween = TweenTo(mobPos)
                        if tween then tween.Completed:Wait() end
                        DoAttack()
                    end
                else
                    -- Caso o mob ainda não tenha dado respawn, navega até a ilha dele
                    local islandTween = TweenTo(targetInfo.Position * CFrame.new(0, 20, 0))
                    if islandTween then islandTween.Completed:Wait() end
                end
            end)
        end
    end
end)

-- Loop de Coleta de Baús
task.spawn(function()
    while true do
        task.wait(0.5)
        if _G.AutoChest then
            pcall(function()
                for _, obj in pairs(workspace:GetChildren()) do
                    if string.find(obj.Name, "Chest") and obj:IsA("BasePart") then
                        if _G.AutoChest then
                            local tween = TweenTo(obj.CFrame)
                            if tween then tween.Completed:Wait() end
                            task.wait(0.3)
                        end
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- 2. MOVIMENTAÇÃO
-- ======================================================================
Tabs.Movement:AddSlider("SpeedSlider", {
    Title = "Velocidade de Voo (Tween Speed)",
    Description = "Ajuste a velocidade para viajar entre ilhas sem tomar kick",
    Default = 250,
    Min = 50,
    Max = 350,
    Rounding = 0,
    Callback = function(Value)
        _G.TweenSpeed = Value
    end
})

-- ======================================================================
-- 3. FRUTAS
-- ======================================================================
Tabs.Fruit:AddToggle("AutoSpinToggle", { Title = "Auto Girar Fruta", Default = false, Callback = function(V) _G.AutoSpinFruit = V end })
Tabs.Fruit:AddToggle("AutoStoreToggle", { Title = "Auto Guardar Frutas", Default = false, Callback = function(V) _G.AutoStoreFruit = V end })

task.spawn(function()
    while true do
        task.wait(2)
        if _G.AutoSpinFruit then
            pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("Cousin", "Buy") end)
        end
    end
end)

task.spawn(function()
    while true do
        task.wait(1)
        if _G.AutoStoreFruit then
            pcall(function()
                for _, tool in pairs(LocalPlayer.Backpack:GetChildren()) do
                    if string.find(tool.Name, "Fruit") then
                        ReplicatedStorage.Remotes.CommF_:InvokeServer("StoreFruit", tool.Name, tool)
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- 4. APARÊNCIA & COLOR PICKER
-- ======================================================================
local ColorPicker = Tabs.Settings:AddColorpicker("ColorPicker", {
    Title = "Cor da Marca d'Água",
    Default = Color3.fromRGB(255, 50, 50)
})

ColorPicker:OnChanged(function()
    TextLabel.TextColor3 = ColorPicker.Value
end)

Window:SelectTab(1)
