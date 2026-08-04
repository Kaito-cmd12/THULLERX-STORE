-- ======================================================================
-- THULLERX STORE - HUB OFICIAL (AUTO FARM + AUTO QUEST + AUTO CHEST)
-- VERSÃO CORRIGIDA: AUTO CHEST COM NOCLIP E BUSCA GLOBAL
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Blox fruits",
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

-- Estado da máquina de Auto Quest
_G.QuestState = "IDLE"
_G.QuestRetryCount = 0
_G.QuestRetryTime = 0

-- Alvo atual do Auto Chest
_G.ChestTarget = nil

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")

local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local RedeemRemote = Remotes:WaitForChild("Redeem")
local CommF_ = Remotes:FindFirstChild("CommF_") or Remotes:FindFirstChild("CommF")

-- LISTA DE CÓDIGOS DE EXP
local PromoCodes = {
    "EASTEREXP", "fudd10", "fudd10_V2", "Chandler", "BIGNEWS",
    "KITT_RESET", "Sub2UncleKizaru", "SUB2GAMERROBOT_RESET1",
    "Sub2Fer999", "Enyu_is_Pro", "JCWK", "StarcodeHEO", "MagicBUS",
    "KittGaming", "Sub2CaptainMaui", "Sub2OfficialNoobie", "TheGreatAce",
    "Sub2NoobMaster123", "Sub2Daigrock", "Axiore", "StrawHatMaine",
    "TantaiGaming", "Bluxxy", "SUB2GAMERROBOT_EXP1"
}

-- DETECÇÃO PRECISA DE LEVEL
local function GetPlayerLevel()
    local lvl = 1
    pcall(function()
        if LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") then
            lvl = LocalPlayer.Data.Level.Value
        elseif LocalPlayer:FindFirstChild("leaderstats") and LocalPlayer.leaderstats:FindFirstChild("Level") then
            lvl = LocalPlayer.leaderstats.Level.Value
        elseif LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.PlayerGui:FindFirstChild("Main") and LocalPlayer.PlayerGui.Main:FindFirstChild("Level") then
            local lvlText = LocalPlayer.PlayerGui.Main.Level.Text
            lvl = tonumber(string.match(lvlText, "%d+")) or 1
        end
    end)
    return lvl
end

-- TABELA DETALHADA DE QUESTS (SEA 1)
local LevelData = {
    {Min = 1,   Max = 9,   Quest = "BanditQuest1",       ID = 1, Mob = "Bandit",          QuestPos = CFrame.new(1059.3, 16.4, 1548.6)},
    {Min = 10,  Max = 14,  Quest = "JungleQuest",        ID = 1, Mob = "Monkey",          QuestPos = CFrame.new(-1598.4, 36.8, 153.8)},
    {Min = 15,  Max = 19,  Quest = "JungleQuest",        ID = 2, Mob = "Gorilla",         QuestPos = CFrame.new(-1598.4, 36.8, 153.8)},
    {Min = 20,  Max = 29,  Quest = "JungleQuest",        ID = 3, Mob = "Gorilla King",    QuestPos = CFrame.new(-1598.4, 36.8, 153.8)},
    {Min = 30,  Max = 39,  Quest = "BuggyQuest1",       ID = 1, Mob = "Pirate",          QuestPos = CFrame.new(-1141.0, 4.7, 3856.2)},
    {Min = 40,  Max = 59,  Quest = "BuggyQuest1",       ID = 2, Mob = "Brute",           QuestPos = CFrame.new(-1141.0, 4.7, 3856.2)},
    {Min = 60,  Max = 74,  Quest = "DesertQuest",       ID = 1, Mob = "Desert Bandit",   QuestPos = CFrame.new(897.0, 6.4, 4388.0)},
    {Min = 75,  Max = 89,  Quest = "DesertQuest",       ID = 2, Mob = "Desert Officer",  QuestPos = CFrame.new(897.0, 6.4, 4388.0)},
    {Min = 90,  Max = 99,  Quest = "SnowQuest",         ID = 1, Mob = "Snow Bandit",     QuestPos = CFrame.new(1385.8, 87.2, -1298.6)},
    {Min = 100, Max = 119, Quest = "SnowQuest",         ID = 2, Mob = "Snowman",         QuestPos = CFrame.new(1385.8, 87.2, -1298.6)},
    {Min = 120, Max = 149, Quest = "MarineQuest2",      ID = 1, Mob = "Chief Petty Officer", QuestPos = CFrame.new(-5036.0, 28.6, 4324.7)},
    {Min = 150, Max = 174, Quest = "SkyQuest",          ID = 1, Mob = "Sky Bandit",      QuestPos = CFrame.new(-4839.5, 717.5, -2620.5)},
    {Min = 175, Max = 189, Quest = "SkyQuest",          ID = 2, Mob = "Dark Master",     QuestPos = CFrame.new(-4839.5, 717.5, -2620.5)},
    {Min = 190, Max = 209, Quest = "PrisonerQuest",     ID = 1, Mob = "Prisoner",        QuestPos = CFrame.new(485.6, 4.4, 735.6)},
    {Min = 210, Max = 249, Quest = "PrisonerQuest",     ID = 2, Mob = "Dangerous Prisoner", QuestPos = CFrame.new(485.6, 4.4, 735.6)},
    {Min = 250, Max = 274, Quest = "ColosseumQuest",    ID = 1, Mob = "Toga Warrior",    QuestPos = CFrame.new(-1820.2, 7.2, -2745.8)},
    {Min = 275, Max = 299, Quest = "ColosseumQuest",    ID = 2, Mob = "Gladiator",       QuestPos = CFrame.new(-1820.2, 7.2, -2745.8)},
    {Min = 300, Max = 324, Quest = "MagmaQuest",        ID = 1, Mob = "Military Soldier", QuestPos = CFrame.new(-5315.8, 12.2, 8515.2)},
    {Min = 325, Max = 374, Quest = "MagmaQuest",        ID = 2, Mob = "Military Spy",    QuestPos = CFrame.new(-5315.8, 12.2, 8515.2)},
    {Min = 375, Max = 399, Quest = "FishmanQuest",      ID = 1, Mob = "Fishman Warrior", QuestPos = CFrame.new(61122.5, 18.4, 1569.3)},
    {Min = 400, Max = 449, Quest = "FishmanQuest",      ID = 2, Mob = "Fishman Commando", QuestPos = CFrame.new(61122.5, 18.4, 1569.3)},
    {Min = 450, Max = 699, Quest = "SkyQuest2",         ID = 1, Mob = "God's Guard",     QuestPos = CFrame.new(-4720.4, 845.2, -1950.5)}
}

local function GetQuestData()
    local lvl = GetPlayerLevel()
    for _, data in ipairs(LevelData) do
        if lvl >= data.Min and lvl <= data.Max then
            return data
        end
    end
    return LevelData[1]
end

local function HasQuest()
    local pGui = LocalPlayer:FindFirstChild("PlayerGui")
    if pGui and pGui:FindFirstChild("Main") and pGui.Main:FindFirstChild("Quest") then
        return pGui.Main.Quest.Visible and pGui.Main.Quest.Container.QuestTitle.Title.Text ~= ""
    end
    return false
end

local function GetCurrentQuestTitle()
    local pGui = LocalPlayer:FindFirstChild("PlayerGui")
    if pGui and pGui:FindFirstChild("Main") and pGui.Main:FindFirstChild("Quest") and pGui.Main.Quest.Visible then
        return pGui.Main.Quest.Container.QuestTitle.Title.Text
    end
    return ""
end

local function CloseAnyDialogue()
    pcall(function()
        local pGui = LocalPlayer:FindFirstChild("PlayerGui")
        if pGui and pGui:FindFirstChild("Main") and pGui.Main:FindFirstChild("Dialogue") and pGui.Main.Dialogue.Visible then
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
            task.wait(0.05)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
        end
    end)
end

local function StopMovement()
    if _G.CurrentTween then
        _G.CurrentTween:Cancel()
        _G.CurrentTween = nil
    end
end

local function TweenTo(targetCFrame)
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    local hrp = char.HumanoidRootPart
    local dist = (hrp.Position - targetCFrame.Position).Magnitude
    local time = dist / _G.TweenSpeed

    local tweenInfo = TweenInfo.new(time, Enum.EasingStyle.Linear)
    _G.CurrentTween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
    _G.CurrentTween:Play()
end

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

        if currentTool then currentTool:Activate() end

        local viewportSize = workspace.CurrentCamera.ViewportSize
        VirtualInputManager:SendMouseButtonEvent(viewportSize.X / 2, viewportSize.Y / 2, 0, true, game, 1)
        VirtualInputManager:SendMouseButtonEvent(viewportSize.X / 2, viewportSize.Y / 2, 0, false, game, 1)
    end)
end

-- CRIAÇÃO DAS ABAS
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "sword" }),
    Chest = Window:AddTab({ Title = "Baús", Icon = "box" }),
    Movement = Window:AddTab({ Title = "Movimento", Icon = "move" }),
    Fruit = Window:AddTab({ Title = "Frutas", Icon = "apple" }),
    Settings = Window:AddTab({ Title = "Aparência & Temas", Icon = "settings" })
}

-- 1. AUTO FARM & STATS & CODES
Tabs.Main:AddSection("Farm de Nível Automático")
Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Level / Ataque NPC",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then
            StopMovement()
            _G.QuestState = "IDLE"
        end
    end
})

Tabs.Main:AddSection("Distribuição de Pontos (Auto Stats)")
Tabs.Main:AddToggle("AutoStatsToggle", { Title = "Ativar Auto Stats", Default = false, Callback = function(V) _G.AutoStats = V end })
Tabs.Main:AddDropdown("StatDropdown", { Title = "Atributo", Values = {"Melee", "Defense", "Sword", "Demon Fruit"}, Default = "Melee", Callback = function(V) _G.SelectedStat = V end })
Tabs.Main:AddSlider("StatPointsSlider", { Title = "Pontos por Vez", Default = 1, Min = 1, Max = 10, Rounding = 0, Callback = function(V) _G.StatPoints = V end })

Tabs.Main:AddSection("Resgate de Códigos (Redeem Codes)")
Tabs.Main:AddButton({
    Title = "Resgatar Todos os Códigos (EXP)",
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
                task.wait(0.3)
            end
            Fluent:Notify({ Title = "THULLERX STORE", Content = "Códigos resgatados!", Duration = 3 })
        end)
    end
})

-- 2. AUTO CHEST
Tabs.Chest:AddSection("Coleta Automática de Baús")
Tabs.Chest:AddToggle("AutoChestToggle", {
    Title = "Ativar Auto Farm Baús",
    Default = false,
    Callback = function(V)
        _G.AutoChest = V
        if not V then
            StopMovement()
            _G.ChestTarget = nil
        end
    end
})

-- 3. MOVIMENTO
Tabs.Movement:AddSection("Ajustes de Posição & Voo")
Tabs.Movement:AddSlider("HeightSlider", { Title = "Altura do Farm", Default = 2, Min = 0, Max = 8, Rounding = 0, Callback = function(V) _G.FarmHeight = V end })
Tabs.Movement:AddSlider("SpeedSlider", { Title = "Velocidade (Tween Speed)", Default = 250, Min = 50, Max = 350, Rounding = 0, Callback = function(V) _G.TweenSpeed = V end })

-- 4. FRUTAS
Tabs.Fruit:AddSection("Ações de Frutas")
Tabs.Fruit:AddToggle("AutoSpinToggle", { Title = "Auto Girar Fruta", Default = false, Callback = function(V) _G.AutoSpinFruit = V end })
Tabs.Fruit:AddToggle("AutoStoreToggle", { Title = "Auto Guardar Frutas", Default = false, Callback = function(V) _G.AutoStoreFruit = V end })

-- 5. APARÊNCIA DA INTERFACE
Tabs.Settings:AddSection("Aparência do Menu")
Tabs.Settings:AddDropdown("ThemeDropdown", { Title = "Tema do Menu", Values = {"Darker", "Dark", "Midnight", "Aqua", "Amethyst", "Rose"}, Default = "Darker", Callback = function(V) Fluent:SetTheme(V) end })
Tabs.Settings:AddDropdown("FontDropdown", { Title = "Fonte da Marca d'Água", Values = {"SourceSansBold", "FredokaOne", "GothamBold", "Arcade"}, Default = "SourceSansBold", Callback = function(V) TextLabel.Font = Enum.Font[V] end })

local ColorPicker = Tabs.Settings:AddColorpicker("WatermarkColor", { Title = "Cor da Borda e Texto", Default = Color3.fromRGB(255, 50, 50) })
ColorPicker:OnChanged(function()
    TextLabel.TextColor3 = ColorPicker.Value
    MainFrame.BorderColor3 = ColorPicker.Value
end)

Tabs.Settings:AddToggle("WatermarkVisibleToggle", { Title = "Exibir Marca d'Água", Default = true, Callback = function(V) MainFrame.Visible = V end })

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

-- BUSCA NPC/MOB
local function GetTargetEnemy(mobName)
    local closest = nil
    local shortestDistance = math.huge
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end

    local enemies = workspace:FindFirstChild("Enemies") or workspace
    for _, enemy in pairs(enemies:GetChildren()) do
        if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
            if string.find(string.lower(enemy.Name), string.lower(mobName)) then
                local dist = (enemy.HumanoidRootPart.Position - hrp.Position).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    closest = enemy
                end
            end
        end
    end
    return closest
end

-- LOOP AUTO FARM / AUTO QUEST
local currentTarget = nil
task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoFarmLevel then
            pcall(function()
                local qData = GetQuestData()
                local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if not hrp then return end

                CloseAnyDialogue()

                if HasQuest() and not string.find(string.lower(GetCurrentQuestTitle()), string.lower(qData.Mob)) then
                    if CommF_ then pcall(function() CommF_:InvokeServer("AbandonQuest") end) end
                    currentTarget = nil
                    _G.QuestState = "GOTO_NPC"
                    task.wait(0.3)
                    return
                end

                if HasQuest() then
                    _G.QuestState = "HUNT"
                elseif _G.QuestState ~= "REQUEST" and _G.QuestState ~= "CONFIRM" then
                    _G.QuestState = "GOTO_NPC"
                end

                if _G.QuestState == "GOTO_NPC" then
                    local distToNPC = (hrp.Position - qData.QuestPos.Position).Magnitude
                    if distToNPC > 12 then
                        TweenTo(qData.QuestPos)
                    else
                        StopMovement()
                        _G.QuestState = "REQUEST"
                    end

                elseif _G.QuestState == "REQUEST" then
                    StopMovement()
                    CloseAnyDialogue()
                    if CommF_ then
                        pcall(function() CommF_:InvokeServer("StartQuest", qData.Quest, qData.ID) end)
                    end
                    _G.QuestRetryCount = 0
                    _G.QuestRetryTime = tick()
                    _G.QuestState = "CONFIRM"

                elseif _G.QuestState == "CONFIRM" then
                    CloseAnyDialogue()
                    if HasQuest() and string.find(string.lower(GetCurrentQuestTitle()), string.lower(qData.Mob)) then
                        _G.QuestState = "HUNT"
                    elseif tick() - _G.QuestRetryTime > 1.5 then
                        _G.QuestRetryCount = _G.QuestRetryCount + 1
                        if _G.QuestRetryCount >= 5 then
                            _G.QuestState = "GOTO_NPC"
                            _G.QuestRetryCount = 0
                        else
                            _G.QuestState = "REQUEST"
                        end
                    end

                elseif _G.QuestState == "HUNT" then
                    if not currentTarget or not currentTarget:FindFirstChild("Humanoid") or currentTarget.Humanoid.Health <= 0 then
                        currentTarget = GetTargetEnemy(qData.Mob)
                    end

                    if currentTarget and currentTarget:FindFirstChild("HumanoidRootPart") then
                        local targetCFrame = currentTarget.HumanoidRootPart.CFrame * CFrame.new(0, _G.FarmHeight, 0)
                        local dist = (hrp.Position - currentTarget.HumanoidRootPart.Position).Magnitude

                        if dist > 15 then
                            TweenTo(targetCFrame)
                        else
                            StopMovement()
                            hrp.CFrame = targetCFrame
                            DoRealAutoClick()
                        end
                    end
                end
            end)
        else
            currentTarget = nil
        end
    end
end)

-- ======================================================================
-- LOOP AUTO CHEST (REFEITO DO ZERO - GARANTIDO PARA AMBOS OS SEAS)
-- ======================================================================
local IgnoreChests = {}

-- Função de Noclip para não enganchar em paredes indo até os baús
task.spawn(function()
    game:GetService("RunService").Stepped:Connect(function()
        if _G.AutoChest then
            local char = LocalPlayer.Character
            if char then
                for _, part in pairs(char:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end
    end)
end)

-- Busca todos os baús válidos do jogo inteiro
local function GetClosestChest()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end

    local closestChest = nil
    local shortestDistance = math.huge

    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and string.find(string.lower(v.Name), "chest") then
            -- Verifica se o baú está ativo (não invisível e dentro do jogo)
            if v.Parent and v.Transparency < 1 and not IgnoreChests[v] then
                local dist = (hrp.Position - v.Position).Magnitude
                if dist < shortestDistance then
                    shortestDistance = dist
                    closestChest = v
                end
            end
        end
    end
    return closestChest
end

task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoChest and not _G.AutoFarmLevel then
            pcall(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if not hrp then return end

                local chest = GetClosestChest()

                if chest and chest.Parent then
                    local chestCFrame = chest.CFrame
                    local dist = (hrp.Position - chest.Position).Magnitude

                    -- Se estiver longe, vai de Tween
                    if dist > 10 then
                        TweenTo(chestCFrame)
                    else
                        -- Se estiver perto, teleporta em cima e aciona toque
                        StopMovement()
                        hrp.CFrame = chestCFrame
                        
                        -- Dispara evento de toque no baú
                        if firetouchinterest then
                            firetouchinterest(hrp, chest, 0)
                            task.wait(0.1)
                            firetouchinterest(hrp, chest, 1)
                        end

                        IgnoreChests[chest] = true
                        task.wait(0.2)
                    end
                else
                    StopMovement()
                    -- Limpa a lista de baús ignorados caso não ache nenhum no mapa
                    IgnoreChests = {}
                    task.wait(1)
                end
            end)
        end
    end
end)

Window:SelectTab(1)
