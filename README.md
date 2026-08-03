-- ======================================================================
-- THULLERX STORE - HUB OFICIAL (TARGET BY NPC NAME)
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
_G.TargetNPCName = ""
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

-- LISTA DE CÓDIGOS
local PromoCodes = {
    "EASTEREXP", "fudd10", "fudd10_V2", "Chandler", "BIGNEWS", 
    "KITT_RESET", "Sub2UncleKizaru", "SUB2GAMERROBOT_RESET1", 
    "Sub2Fer999", "Enyu_is_Pro", "JCWK", "StarcodeHEO", "MagicBUS", 
    "KittGaming", "Sub2CaptainMaui", "Sub2OfficialNoobie", "TheGreatAce", 
    "Sub2NoobMaster123", "Sub2Daigrock", "Axiore", "StrawHatMaine", 
    "TantaiGaming", "Bluxxy", "SUB2GAMERROBOT_EXP1"
}

local function StopMovement()
    if _G.CurrentTween then
        _G.CurrentTween:Cancel()
        _G.CurrentTween = nil
    end
end

-- LÓGICA DE AUTO CLICKER
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

-- 1. SEÇÃO AUTO FARM E NPC TARGET
Tabs.Main:AddSection("Farm de Nível Automático")

Tabs.Main:AddInput("NPCNameInput", {
    Title = "Nome Exato do NPC da Missão",
    Default = "",
    Placeholder = "Ex: Bandit, Monkey, Pirate...",
    Callback = function(Text)
        _G.TargetNPCName = Text
    end
})

Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Level / Ataque NPC",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then StopMovement() end
    end
})

-- 2. SEÇÃO AUTO STATS
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

-- 3. SEÇÃO CÓDIGOS
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
        if _G.AutoStats then
            pcall(function()
                Remotes:WaitForChild("CommF_"):InvokeServer("AddPoint", _G.SelectedStat, _G.StatPoints)
            end)
        end
    end
end)

-- BUSCA NPC POR NOME FIEL
local function GetTargetEnemy()
    local closest = nil
    local shortestDistance = math.huge
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    
    if not hrp then return nil end
    
    local enemies = workspace:FindFirstChild("Enemies") or workspace
    for _, enemy in pairs(enemies:GetChildren()) do
        if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
            -- Se o usuário colocou o nome do NPC, filtra por ele
            if _G.TargetNPCName == "" or string.find(string.lower(enemy.Name), string.lower(_G.TargetNPCName)) then
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

-- LOOP PRINCIPAL DO FARM & ATAQUE
local currentTarget = nil
task.spawn(function()
    while true do
        task.wait(0.05)
        if _G.AutoFarmLevel then
            pcall(function()
                if not currentTarget or not currentTarget:FindFirstChild("Humanoid") or currentTarget.Humanoid.Health <= 0 or not currentTarget:FindFirstChild("HumanoidRootPart") then
                    currentTarget = GetTargetEnemy()
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
