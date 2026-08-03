-- ======================================================================
-- THULLERX STORE - HUB OFICIAL BLOX FRUITS (WITH AUTO CODES V7)
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Blox Fruits Quest Hub",
    TabWidth = 160,
    Size = UDim2.fromOffset(630, 460),
    Acrylic = true,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.K
})

-- Marca d'água Customizável
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
StatusLabel.Text = "Teclar [K] para Ocultar"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatusLabel.TextSize = 13.0
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Variáveis Globais
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

-- Lista de Códigos do Blox Fruits
local PromoCodes = {
    "KITT_RESET",
    "Sub2Fer999",
    "Enyu_is_Pro",
    "Magicbus",
    "JCWK",
    "Starcodeheo",
    "Bluxxy",
    "fudd10_v2",
    "FUDD10",
    "BIGNEWS",
    "THEGREATACE",
    "SUB2GAMERROBOT_RESET1",
    "SUB2GAMERROBOT_EXP1",
    "Sub2OfficialNoobie",
    "Sub2UncleKizaru",
    "Sub2NoobMaster123",
    "Sub2Daigrock",
    "Axiore",
    "TantaiGaming",
    "STRAHATMAINE"
}

-- TABELA DE QUESTS DO BLOX FRUITS (First Sea)
local LevelData = {
    {Min = 1,   Max = 9,   Quest = "BanditQuest1",       ID = 1, Mob = "Bandit",          QuestPos = CFrame.new(1059.3, 16.4, 1548.6)},
    {Min = 10,  Max = 29,  Quest = "JungleQuest",        ID = 1, Mob = "Monkey",          QuestPos = CFrame.new(-1598.4, 36.8, 153.8)},
    {Min = 30,  Max = 59,  Quest = "BuggyQuest1",        ID = 1, Mob = "Pirate",          QuestPos = CFrame.new(-1141.0, 4.7, 3856.2)},
    {Min = 60,  Max = 89,  Quest = "DesertQuest",        ID = 1, Mob = "Desert Bandit",   QuestPos = CFrame.new(897.0, 6.4, 4388.0)},
    {Min = 90,  Max = 119, Quest = "SnowQuest",          ID = 1, Mob = "Snow Bandit",     QuestPos = CFrame.new(1385.8, 87.2, -1298.6)},
    {Min = 120, Max = 149, Quest = "MarineQuest2",       ID = 1, Mob = "Chief Petty Officer", QuestPos = CFrame.new(-5036.0, 28.6, 4324.7)},
    {Min = 150, Max = 189, Quest = "SkyQuest",           ID = 1, Mob = "Sky Bandit",      QuestPos = CFrame.new(-4839.5, 717.5, -2620.5)},
    {Min = 190, Max = 249, Quest = "PrisonerQuest",      ID = 1, Mob = "Prisoner",        QuestPos = CFrame.new(485.6, 4.4, 735.6)},
    {Min = 250, Max = 299, Quest = "ColosseumQuest",     ID = 1, Mob = "Toga Warrior",    QuestPos = CFrame.new(-1820.2, 7.2, -2745.8)},
    {Min = 300, Max = 374, Quest = "MagmaQuest",         ID = 1, Mob = "Military Soldier", QuestPos = CFrame.new(-5315.8, 12.2, 8515.2)},
    {Min = 375, Max = 449, Quest = "FishmanQuest",       ID = 1, Mob = "Fishman Warrior", QuestPos = CFrame.new(61122.5, 18.4, 1569.3)},
    {Min = 450, Max = 699, Quest = "SkyQuest2",          ID = 1, Mob = "God's Guard",     QuestPos = CFrame.new(-4720.4, 845.2, -1950.5)}
}

-- Parar Movimento
local function StopMovement()
    if _G.CurrentTween then
        _G.CurrentTween:Cancel()
        _G.CurrentTween = nil
    end
end

-- Voo Suave (Tween)
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

-- Pegar Nível
local function GetPlayerLevel()
    pcall(function()
        if LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") then
            return LocalPlayer.Data.Level.Value
        end
    end)
    return 1
end

-- Pegar Dados da Quest
local function GetQuestData()
    local lvl = GetPlayerLevel()
    for _, data in ipairs(LevelData) do
        if lvl >= data.Min and lvl <= data.Max then
            return data
        end
    end
    return LevelData[#LevelData]
end

-- Verificação de Quest Ativa
local function HasQuest()
    local pGui = LocalPlayer:FindFirstChild("PlayerGui")
    if pGui and pGui:FindFirstChild("Main") and pGui.Main:FindFirstChild("Quest") then
        return pGui.Main.Quest.Visible and pGui.Main.Quest.Container.QuestTitle.Title.Text ~= ""
    end
    return false
end

-- Pega Quest
local function TakeQuest(qData)
    pcall(function()
        ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", qData.Quest, qData.ID)
    end)
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
-- ABAS DA INTERFACE
-- ======================================================================
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "sword" }),
    Chest = Window:AddTab({ Title = "Baús", Icon = "box" }),
    Movement = Window:AddTab({ Title = "Movimento", Icon = "move" }),
    Fruit = Window:AddTab({ Title = "Frutas", Icon = "apple" }),
    Settings = Window:AddTab({ Title = "Aparência & Temas", Icon = "settings" })
}

-- ======================================================================
-- 1. AUTO FARM LEVEL & CODES
-- ======================================================================
Tabs.Main:AddSection("Farm de Nível Automático")

Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Level (Com Quests)",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then StopMovement() end
    end
})

Tabs.Main:AddSection("Recompensas & Promocodes")

-- Botão de Resgatar Todos os Códigos
Tabs.Main:AddButton({
    Title = "Resgatar Todos os Códigos (Redeem All Codes)",
    Description = "Aplica automaticamente todos os códigos de XP 2x e Beli ativas",
    Callback = function()
        Fluent:Notify({
            Title = "THULLERX STORE",
            Content = "Resgatando códigos... Aguarde!",
            Duration = 3
        })
        task.spawn(function()
            for _, code in ipairs(PromoCodes) do
                pcall(function()
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("RedeemCode", code)
                end)
                task.wait(0.2)
            end
            Fluent:Notify({
                Title = "THULLERX STORE",
                Content = "Todos os códigos foram resgatados com sucesso!",
                Duration = 5
            })
        end)
    end
})

-- Loop do Auto Farm
task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoFarmLevel then
            pcall(function()
                local qData = GetQuestData()
                
                if not HasQuest() then
                    local dist = (LocalPlayer.Character.HumanoidRootPart.Position - qData.QuestPos.Position).Magnitude
                    if dist > 15 then
                        local tween = TweenTo(qData.QuestPos * CFrame.new(0, 3, 0))
                        if tween then tween.Completed:Wait() end
                    else
                        LocalPlayer.Character.HumanoidRootPart.CFrame = qData.QuestPos * CFrame.new(0, 3, 0)
                        task.wait(0.2)
                        TakeQuest(qData)
                        task.wait(0.5)
                    end
                else
                    local enemies = workspace:FindFirstChild("Enemies")
                    local targetEnemy = nil

                    if enemies then
                        for _, enemy in pairs(enemies:GetChildren()) do
                            if enemy.Name == qData.Mob and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                                targetEnemy = enemy
                                break
                            end
                        end
                    end

                    if targetEnemy then
                        while _G.AutoFarmLevel and HasQuest() and targetEnemy and targetEnemy:FindFirstChild("Humanoid") and targetEnemy.Humanoid.Health > 0 do
                            task.wait(0.05)
                            local mobPos = targetEnemy.HumanoidRootPart.CFrame * CFrame.new(0, 6, 0)
                            local tween = TweenTo(mobPos)
                            if tween then tween.Completed:Wait() end
                            DoAttack()
                        end
                    else
                        local islandPos = qData.QuestPos * CFrame.new(0, 20, 100)
                        local tween = TweenTo(islandPos)
                        if tween then tween.Completed:Wait() end
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- 2. AUTO CHEST
-- ======================================================================
Tabs.Chest:AddSection("Coleta Automática de Baús")

Tabs.Chest:AddToggle("AutoChestToggle", {
    Title = "Ativar Auto Farm Baús",
    Default = false,
    Callback = function(Value)
        _G.AutoChest = Value
        if not Value then StopMovement() end
    end
})

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
Tabs.Movement:AddSection("Velocidade do Voo")

Tabs.Movement:AddSlider("SpeedSlider", {
    Title = "Velocidade (Tween Speed)",
    Description = "Ajuste a velocidade entre 50 e 350",
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
Tabs.Fruit:AddSection("Ações de Frutas")
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
-- 5. PERSONALIZAÇÃO & APARÊNCIA AVANÇADA
-- ======================================================================
Tabs.Settings:AddSection("Temas e Cores do Menu")

Tabs.Settings:AddDropdown("ThemeDropdown", {
    Title = "Tema da Interface",
    Values = {"Darker", "Dark", "Midnight", "Aqua", "Amethyst", "Rose"},
    Default = "Darker",
    Callback = function(Value)
        Fluent:SetTheme(Value)
    end
})

Tabs.Settings:AddToggle("AcrylicToggle", {
    Title = "Efeito Transparência (Acrylic)",
    Default = true,
    Callback = function(Value)
        Window:SetAcrylic(Value)
    end
})

Tabs.Settings:AddSection("Marca d'Água (THULLERX STORE)")

local ColorPicker = Tabs.Settings:AddColorpicker("WatermarkColor", {
    Title = "Cor da Marca d'Água & Borda",
    Default = Color3.fromRGB(255, 50, 50)
})

ColorPicker:OnChanged(function()
    TextLabel.TextColor3 = ColorPicker.Value
    MainFrame.BorderColor3 = ColorPicker.Value
end)

Tabs.Settings:AddToggle("ShowWatermark", {
    Title = "Exibir Caixa THULLERX STORE na Tela",
    Default = true,
    Callback = function(Value)
        MainFrame.Visible = Value
    end
})

Window:SelectTab(1)
