-- ======================================================================
-- THULLERX STORE - HUB OFICIAL (AUTO FARM + AUTO QUEST)
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Quest Hub",
    TabWidth = 160,
    Size = UDim2.fromOffset(630, 480),
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
StatusLabel.Text = "Teclar [K] para Ocultar"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatusLabel.TextSize = 13.0
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Variáveis Globais
_G.AutoFarmLevel = false
_G.AutoChest = false
_G.AutoSpinFruit = false
_G.AutoStoreFruit = false
_G.AutoStats = false
_G.SelectedStat = "Melee"
_G.StatPoints = 1
_G.TweenSpeed = 250
_G.FarmHeight = 2
_G.CurrentTween = nil

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local VirtualUser = game:GetService("VirtualUser")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local RedeemRemote = Remotes:WaitForChild("Redeem")
local CommF_ = Remotes:FindFirstChild("CommF_") or Remotes:FindFirstChild("CommF")

-- LISTA DE CÓDIGOS
local PromoCodes = {
    "EASTEREXP", "fudd10", "fudd10_V2", "Chandler", "BIGNEWS", 
    "KITT_RESET", "Sub2UncleKizaru", "SUB2GAMERROBOT_RESET1", 
    "Sub2Fer999", "Enyu_is_Pro", "JCWK", "StarcodeHEO", "MagicBUS", 
    "KittGaming", "Sub2CaptainMaui", "Sub2OfficialNoobie", "TheGreatAce", 
    "Sub2NoobMaster123", "Sub2Daigrock", "Axiore", "StrawHatMaine", 
    "TantaiGaming", "Bluxxy", "SUB2GAMERROBOT_EXP1"
}

-- TABELA DE QUESTS DO SEA 1
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

local function GetPlayerLevel()
    pcall(function()
        if LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") then
            return LocalPlayer.Data.Level.Value
        end
    end)
    return 1
end

local function GetQuestData()
    local lvl = GetPlayerLevel()
    for _, data in ipairs(LevelData) do
        if lvl >= data.Min and lvl <= data.Max then
            return data
        end
    end
    return LevelData[#LevelData]
end

local function HasQuest()
    local pGui = LocalPlayer:FindFirstChild("PlayerGui")
    if pGui and pGui:FindFirstChild("Main") and pGui.Main:FindFirstChild("Quest") then
        return pGui.Main.Quest.Visible and pGui.Main.Quest.Container.QuestTitle.Title.Text ~= ""
    end
    return false
end

local function StopMovement()
    if _G.CurrentTween then
        _G.CurrentTween:Cancel()
        _G.CurrentTween = nil
    end
end

-- LÓGICA DE AUTO CLICKER E EQUIPAR ARMA
local function DoRealAutoClick()
    pcall(function()
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("Humanoid") then return end
        
        local currentTool = char:FindFirstChildOfClass("Tool")
        if not currentTool then
            for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
                if item:IsA("Tool") then
                    char.Humanoid:EquipTool(item)
                    currentTool = item
                    break
                end
            end
        end
        
        if currentTool then
            currentTool:Activate()
        end

        local viewportSize = workspace.CurrentCamera.ViewportSize
        local centerX = viewportSize.X / 2
        local centerY = viewportSize.Y / 2
        
        VirtualInputManager:SendMouseButtonEvent(centerX, centerY, 0, true, game, 1)
        VirtualInputManager:SendMouseButtonEvent(centerX, centerY, 0, false, game, 1)
        
        VirtualUser:CaptureController()
        VirtualUser:ClickButton1(Vector2.new(centerX, centerY))
    end)
end

-- ABAS DA INTERFACE
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "sword" }),
    Chest = Window:AddTab({ Title = "Baús", Icon = "box" }),
    Movement = Window:AddTab({ Title = "Movimento", Icon = "move" }),
    Fruit = Window:AddTab({ Title = "Frutas", Icon = "apple" }),
    Settings = Window:AddTab({ Title = "Aparência & Temas", Icon = "settings" })
}

-- 1. AUTO FARM
Tabs.Main:AddSection("Farm de Nível Automático")

Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Level / Ataque NPC",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then StopMovement() end
    end
})

-- 2. AUTO STATS
Tabs.Main:AddSection("Distribuição de Pontos (Auto Stats)")

Tabs.Main:AddToggle("AutoStatsToggle", {
    Title = "Ativar Auto Stats",
    Default = false,
    Callback = function(Value)
        _G.AutoStats = Value
    end
})

Tabs.Main:AddDropdown("StatDropdown", {
    Title = "Atributo",
    Values = {"Melee", "Defense", "Sword", "Demon Fruit"},
    Default = "Melee",
    Callback = function(Value)
        _G.SelectedStat = Value
    end
})

Tabs.Main:AddSlider("StatPointsSlider", {
    Title = "Pontos por Vez",
    Default = 1, Min = 1, Max = 10, Rounding = 0,
    Callback = function(Value)
        _G.StatPoints = Value
    end
})

-- 3. REDEEM CODES
Tabs.Main:AddSection("Resgate de Códigos (Redeem Codes)")

Tabs.Main:AddButton({
    Title = "Resgatar Todos os Códigos (DLC)",
    Description = "Executa a lista enviada via Remote Redeem",
    Callback = function()
        Fluent:Notify({ Title = "THULLERX STORE", Content = "Resgatando códigos...", Duration = 3 })
        task.spawn(function()
            for _, code in ipairs(PromoCodes) do
                pcall(function()
                    if RedeemRemote:IsA("RemoteFunction") then
                        RedeemRemote:InvokeServer(code)
                    elseif RedeemRemote:IsA("RemoteEvent") then
                        RedeemRemote:FireServer(code)
                    end
                end)
                task.wait(0.4)
            end
            Fluent:Notify({ Title = "THULLERX STORE", Content = "Códigos resgatados!", Duration = 4 })
        end)
    end
})

-- DEMAIS ABAS
Tabs.Chest:AddSection("Coleta Automática de Baús")
Tabs.Chest:AddToggle("AutoChestToggle", { Title = "Ativar Auto Farm Baús", Default = false, Callback = function(V) _G.AutoChest = V end })

Tabs.Movement:AddSection("Ajustes de Posição & Voo")
Tabs.Movement:AddSlider("HeightSlider", { Title = "Altura do Farm", Default = 2, Min = 0, Max = 8, Rounding = 0, Callback = function(V) _G.FarmHeight = V end })
Tabs.Movement:AddSlider("SpeedSlider", { Title = "Velocidade (Tween Speed)", Default = 250, Min = 50, Max = 350, Rounding = 0, Callback = function(V) _G.TweenSpeed = V end })

Tabs.Fruit:AddSection("Ações de Frutas")
Tabs.Fruit:AddToggle("AutoSpinToggle", { Title = "Auto Girar Fruta", Default = false, Callback = function(V) _G.AutoSpinFruit = V end })
Tabs.Fruit:AddToggle("AutoStoreToggle", { Title = "Auto Guardar Frutas", Default = false, Callback = function(V) _G.AutoStoreFruit = V end })

Tabs.Settings:AddSection("Temas e Cores do Menu")
Tabs.Settings:AddDropdown("ThemeDropdown", { Title = "Tema da Interface", Values = {"Darker", "Dark", "Midnight", "Aqua", "Amethyst", "Rose"}, Default = "Darker", Callback = function(V) Fluent:SetTheme(V) end })

local ColorPicker = Tabs.Settings:AddColorpicker("WatermarkColor", { Title = "Cor da Marca d'Água & Borda", Default = Color3.fromRGB(255, 50, 50) })
ColorPicker:OnChanged(function()
    TextLabel.TextColor3 = ColorPicker.Value
    MainFrame.BorderColor3 = ColorPicker.Value
end)

-- LOOP AUTO STATS
task.spawn(function()
    while true do
        task.wait(0.5)
        if _G.AutoStats and CommF_ then
            pcall(function()
                CommF_:InvokeServer("AddPoint", _G.SelectedStat, _G.StatPoints)
            end)
        end
    end
end)

-- BUSCA O NPC MAIS PRÓXIMO
local function GetClosestEnemy()
    local closest = nil
    local shortestDistance = math.huge
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    
    if not hrp then return nil end
    
    local enemies = workspace:FindFirstChild("Enemies") or workspace
    for _, enemy in pairs(enemies:GetChildren()) do
        if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
            local dist = (enemy.HumanoidRootPart.Position - hrp.Position).Magnitude
            if dist < shortestDistance then
                shortestDistance = dist
                closest = enemy
            end
        end
    end
    return closest
end

-- LOOP PRINCIPAL DE AUTO FARM / AUTO QUEST
local currentTarget = nil
task.spawn(function()
    while true do
        task.wait(0.05)
        if _G.AutoFarmLevel then
            pcall(function()
                local qData = GetQuestData()
                
                -- Se não tem quest, tenta pegar
                if not HasQuest() and CommF_ then
                    CommF_:InvokeServer("StartQuest", qData.Quest, qData.ID)
                end

                -- Mantém o alvo atual até ele morrer
                if not currentTarget or not currentTarget:FindFirstChild("Humanoid") or currentTarget.Humanoid.Health <= 0 or not currentTarget:FindFirstChild("HumanoidRootPart") then
                    currentTarget = GetClosestEnemy()
                end

                if currentTarget and currentTarget:FindFirstChild("HumanoidRootPart") then
                    LocalPlayer.Character.HumanoidRootPart.CFrame = currentTarget.HumanoidRootPart.CFrame * CFrame.new(0, _G.FarmHeight, 0)
                    workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, currentTarget.HumanoidRootPart.Position)
                    DoRealAutoClick()
                end
            end)
        else
            currentTarget = nil
        end
    end
end)

Window:SelectTab(1)
