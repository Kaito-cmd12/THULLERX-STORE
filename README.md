-- ======================================================================
-- THULLERX STORE - HUB OFICIAL (AUTO FARM + AUTO QUEST + AUTO CHEST)
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

-- DETECÇÃO UNIVERSAL DE NÍVEL
local function GetPlayerLevel()
    local lvl = 1
    pcall(function()
        if LocalPlayer:FindFirstChild("leaderstats") and LocalPlayer.leaderstats:FindFirstChild("Level") then
            lvl = LocalPlayer.leaderstats.Level.Value
        elseif LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Level") then
            lvl = LocalPlayer.Data.Level.Value
        elseif LocalPlayer:FindFirstChild("PlayerGui") and LocalPlayer.PlayerGui:FindFirstChild("Main") then
            local lvlText = LocalPlayer.PlayerGui.Main.Level.Text
            lvl = tonumber(string.match(lvlText, "%d+")) or 1
        end
    end)
    return lvl
end

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

local function StopMovement()
    if _G.CurrentTween then
        _G.CurrentTween:Cancel()
        _G.CurrentTween = nil
    end
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

-- INTERFACE FLUENT
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "sword" }),
    Chest = Window:AddTab({ Title = "Baús", Icon = "box" }),
    Movement = Window:AddTab({ Title = "Movimento", Icon = "move" }),
    Fruit = Window:AddTab({ Title = "Frutas", Icon = "apple" }),
    Settings = Window:AddTab({ Title = "Aparência & Temas", Icon = "settings" })
}

Tabs.Main:AddSection("Farm de Nível Automático")
Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Level / Ataque NPC",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then StopMovement() end
    end
})

Tabs.Chest:AddSection("Coleta Automática de Baús")
Tabs.Chest:AddToggle("AutoChestToggle", { 
    Title = "Ativar Auto Farm Baús", 
    Default = false, 
    Callback = function(V) 
        _G.AutoChest = V 
        if not V then StopMovement() end
    end 
})

Tabs.Settings:AddSection("Aparência do Menu")
Tabs.Settings:AddDropdown("ThemeDropdown", { Title = "Tema do Menu", Values = {"Darker", "Dark", "Midnight", "Aqua", "Amethyst"}, Default = "Darker", Callback = function(V) Fluent:SetTheme(V) end })

-- BUSCA INIMIGO DA MISSÃO
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
        task.wait(0.05)
        if _G.AutoFarmLevel then
            pcall(function()
                local qData = GetQuestData()
                local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if not hrp then return end

                -- Se tiver quest de outro NPC/ilha, abandona
                if HasQuest() and not string.find(string.lower(GetCurrentQuestTitle()), string.lower(qData.Mob)) then
                    if CommF_ then CommF_:InvokeServer("AbandonQuest") end
                end

                -- Se não tiver quest, move até o NPC e pega a missão
                if not HasQuest() then
                    local distToNPC = (hrp.Position - qData.QuestPos.Position).Magnitude
                    if distToNPC > 15 then
                        hrp.CFrame = qData.QuestPos
                    else
                        if CommF_ then CommF_:InvokeServer("StartQuest", qData.Quest, qData.ID) end
                    end
                    return
                end

                -- Ataca o NPC correspondente à quest
                if not currentTarget or not currentTarget:FindFirstChild("Humanoid") or currentTarget.Humanoid.Health <= 0 then
                    currentTarget = GetTargetEnemy(qData.Mob)
                end

                if currentTarget and currentTarget:FindFirstChild("HumanoidRootPart") then
                    hrp.CFrame = currentTarget.HumanoidRootPart.CFrame * CFrame.new(0, _G.FarmHeight, 0)
                    DoRealAutoClick()
                end
            end)
        else
            currentTarget = nil
        end
    end
end)

-- LOOP DEDICADO AUTO CHEST
task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoChest and not _G.AutoFarmLevel then
            pcall(function()
                local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                if not hrp then return end
                
                for _, obj in pairs(workspace:GetDescendants()) do
                    if string.find(string.lower(obj.Name), "chest") then
                        local part = obj:IsA("BasePart") and obj or obj:FindFirstChildOfClass("BasePart")
                        if part and part.Transparency < 1 then
                            hrp.CFrame = part.CFrame
                            task.wait(0.2)
                            if not _G.AutoChest then break end
                        end
                    end
                end
            end)
        end
    end
end)

Window:SelectTab(1)
