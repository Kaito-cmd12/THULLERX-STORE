-- ======================================================================
-- THULLERX STORE - HUB OFICIAL BLOX FRUITS (V4 PROFESSIONAL)
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Blox Fruits Premium Hub",
    TabWidth = 160,
    Size = UDim2.fromOffset(620, 440),
    Acrylic = true,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.K
})

-- Marca d'água Profissional com Status
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local TextLabel = Instance.new("TextLabel")
local StatusLabel = Instance.new("TextLabel")

ScreenGui.Parent = game:GetService("CoreGui")

MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderColor3 = Color3.fromRGB(255, 50, 50)
MainFrame.BorderSizePixel = 1
MainFrame.Position = UDim2.new(0.01, 0, 0.02, 0)
MainFrame.Size = UDim2.new(0, 200, 0, 45)

TextLabel.Parent = MainFrame
TextLabel.BackgroundTransparency = 1
TextLabel.Position = UDim2.new(0.05, 0, 0.1, 0)
TextLabel.Size = UDim2.new(0, 190, 0, 20)
TextLabel.Font = Enum.Font.SourceSansBold
TextLabel.Text = "THULLERX STORE"
TextLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
TextLabel.TextSize = 16.0
TextLabel.TextXAlignment = Enum.TextXAlignment.Left

StatusLabel.Parent = MainFrame
StatusLabel.BackgroundTransparency = 1
StatusLabel.Position = UDim2.new(0.05, 0, 0.55, 0)
StatusLabel.Size = UDim2.new(0, 190, 0, 15)
StatusLabel.Font = Enum.Font.SourceSansItalic
StatusLabel.Text = "Teclar [K] para Ocultar Menu"
StatusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
StatusLabel.TextSize = 12.0
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Variáveis de Estado
_G.AutoFarmLevel = false
_G.AutoChest = false
_G.AutoSpinFruit = false
_G.AutoStoreFruit = false
_G.TweenSpeed = 250
_G.CurrentTween = nil

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Tabela Mapeada de Ilhas/Mobs
local LevelData = {
    {MinLevel = 1,   MaxLevel = 14,  MobName = "Bandit",          Position = CFrame.new(1059, 16, 1548)},
    {MinLevel = 15,  MaxLevel = 29,  MobName = "Monkey",          Position = CFrame.new(-1610, 37, 147)},
    {MinLevel = 30,  MaxLevel = 59,  MobName = "Pirate",          Position = CFrame.new(-1141, 4, 3856)},
    {MinLevel = 60,  MaxLevel = 89,  MobName = "Desert Bandit",   Position = CFrame.new(897, 6, 4388)},
    {MinLevel = 90,  MaxLevel = 119, MobName = "Snow Bandit",     Position = CFrame.new(1353, 87, -1328)},
    {MinLevel = 120, MaxLevel = 149, MobName = "Chief Petty Officer", Position = CFrame.new(-5036, 20, 4324)},
    {MinLevel = 150, MaxLevel = 189, MobName = "Sky Bandit",      Position = CFrame.new(-4839, 717, -2620)},
    {MinLevel = 190, MaxLevel = 249, MobName = "Prisoner",        Position = CFrame.new(485, 4, 735)},
    {MinLevel = 250, MaxLevel = 299, MobName = "Toga Warrior",    Position = CFrame.new(-1820, 7, -2745)},
    {MinLevel = 300, MaxLevel = 374, MobName = "Military Soldier", Position = CFrame.new(-5315, 8, 8515)},
    {MinLevel = 375, MaxLevel = 449, MobName = "Fishman Warrior", Position = CFrame.new(61122, 18, 1569)},
    {MinLevel = 450, MaxLevel = 699, MobName = "God's Guard",     Position = CFrame.new(-4720, 845, -1950)}
}

-- Função para Parar Movimento Imediatamente
local function StopMovement()
    if _G.CurrentTween then
        _G.CurrentTween:Cancel()
        _G.CurrentTween = nil
    end
end

-- Função de Voo/Tween Seguro
local function TweenTo(targetCFrame)
    local character = LocalPlayer.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local hrp = character.HumanoidRootPart
        local distance = (hrp.Position - targetCFrame.Position).Magnitude
        local duration = distance / _G.TweenSpeed
        
        StopMovement()
        
        local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear)
        _G.CurrentTween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
        _G.CurrentTween:Play()
        return _G.CurrentTween
    end
end

-- Leitura de Nível
local function GetPlayerLevel()
    pcall(function()
        if LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") then
            return LocalPlayer.Data.Level.Value
        end
    end)
    return 1
end

-- Busca Mob/Ilha
local function GetCurrentTarget()
    local level = GetPlayerLevel()
    for _, info in ipairs(LevelData) do
        if level >= info.MinLevel and level <= info.MaxLevel then
            return info
        end
    end
    return LevelData[#LevelData]
end

-- Ataque
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
-- ABAS LATERAIS
-- ======================================================================
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "sword" }),
    Chest = Window:AddTab({ Title = "Baús", Icon = "box" }),
    Movement = Window:AddTab({ Title = "Movimento", Icon = "move" }),
    Fruit = Window:AddTab({ Title = "Frutas", Icon = "apple" }),
    Settings = Window:AddTab({ Title = "Personalizar", Icon = "settings" })
}

-- ======================================================================
-- 1. AUTO FARM LEVEL
-- ======================================================================
Tabs.Main:AddSection("Farm de Nível Automático")

Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Farm Level",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then
            StopMovement()
        end
    end
})

task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoFarmLevel then
            pcall(function()
                local targetInfo = GetCurrentTarget()
                local enemies = workspace:FindFirstChild("Enemies")
                local targetEnemy = nil

                if enemies then
                    for _, enemy in pairs(enemies:GetChildren()) do
                        if enemy.Name == targetInfo.MobName and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                            targetEnemy = enemy
                            break
                        end
                    end
                end

                if targetEnemy then
                    while _G.AutoFarmLevel and targetEnemy and targetEnemy:FindFirstChild("Humanoid") and targetEnemy.Humanoid.Health > 0 do
                        task.wait(0.05)
                        local mobPos = targetEnemy.HumanoidRootPart.CFrame * CFrame.new(0, 6, 0)
                        local tween = TweenTo(mobPos)
                        if tween then tween.Completed:Wait() end
                        DoAttack()
                    end
                else
                    local islandTween = TweenTo(targetInfo.Position * CFrame.new(0, 15, 0))
                    if islandTween then islandTween.Completed:Wait() end
                end
            end)
        end
    end
end)

-- ======================================================================
-- 2. AUTO CHEST
-- ======================================================================
Tabs.Chest:AddSection("Coleta de Baús")

Tabs.Chest:AddToggle("AutoChestToggle", {
    Title = "Ativar Auto Farm Baús",
    Default = false,
    Callback = function(Value)
        _G.AutoChest = Value
        if not Value then
            StopMovement()
        end
    end
})

-- Loop de Baús com Varredura Profunda no Workspace
task.spawn(function()
    while true do
        task.wait(0.5)
        if _G.AutoChest then
            pcall(function()
                local chestsFound = {}
                for _, obj in pairs(workspace:GetDescendants()) do
                    if string.find(obj.Name, "Chest") and (obj:IsA("Part") or obj:IsA("MeshPart")) then
                        table.insert(chestsFound, obj)
                    end
                end

                for _, chest in ipairs(chestsFound) do
                    if not _G.AutoChest then break end
                    if chest and chest.Parent then
                        local tween = TweenTo(chest.CFrame * CFrame.new(0, 2, 0))
                        if tween then tween.Completed:Wait() end
                        task.wait(0.3)
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- 3. MOVIMENTAÇÃO
-- ======================================================================
Tabs.Movement:AddSection("Ajuste de Velocidade")

Tabs.Movement:AddSlider("SpeedSlider", {
    Title = "Velocidade do Voo",
    Description = "Altere a velocidade de deslocamento",
    Default = 250,
    Min = 50,
    Max = 350,
    Rounding = 0,
    Callback = function(Value)
        _G.TweenSpeed = Value
    end
})

-- ======================================================================
-- 4. FRUTAS
-- ======================================================================
Tabs.Fruit:AddSection("Gerenciamento de Frutas")

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
-- 5. APARÊNCIA
-- ======================================================================
Tabs.Settings:AddSection("Cores e Estilo")

local ColorPicker = Tabs.Settings:AddColorpicker("ColorPicker", {
    Title = "Cor da Marca d'Água",
    Default = Color3.fromRGB(255, 50, 50)
})

ColorPicker:OnChanged(function()
    TextLabel.TextColor3 = ColorPicker.Value
    MainFrame.BorderColor3 = ColorPicker.Value
end)

Window:SelectTab(1)
