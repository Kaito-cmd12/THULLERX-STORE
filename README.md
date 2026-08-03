-- ======================================================================
-- THULLERX STORE - HUB OFICIAL (DLC CODES & COMBAT FIX)
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
local VirtualUser = game:GetService("VirtualUser")

-- REMOTE ENCONTRADO NA SUA IMAGEM: ReplicatedStorage.Remotes.Redeem
local Remotes = ReplicatedStorage:WaitForChild("Remotes")
local RedeemRemote = Remotes:WaitForChild("Redeem")

-- LISTA DE CÓDIGOS DA SUA FOTO
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

-- SISTEMA DE ATAQUE (FORÇA O CLIQUE NO NPC)
local function DoFastAttack()
    pcall(function()
        local char = LocalPlayer.Character
        if not char then return end
        
        -- Equipa a ferramenta
        local tool = char:FindFirstChildOfClass("Tool")
        if not tool then
            for _, item in pairs(LocalPlayer.Backpack:GetChildren()) do
                if item:IsA("Tool") then
                    char.Humanoid:EquipTool(item)
                    break
                end
            end
        else
            tool:Activate()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton1(Vector2.new(500, 500))
        end
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

-- 1. AUTO FARM & CODES
Tabs.Main:AddSection("Farm de Nível Automático")

Tabs.Main:AddToggle("AutoFarmLevelToggle", {
    Title = "Ativar Auto Level / Ataque NPC",
    Default = false,
    Callback = function(Value)
        _G.AutoFarmLevel = Value
        if not Value then StopMovement() end
    end
})

Tabs.Main:AddSection("Recompensas & DLC Codes")

Tabs.Main:AddButton({
    Title = "Resgatar Todos os Códigos (DLC)",
    Description = "Envia os códigos direto para ReplicatedStorage.Remotes.Redeem",
    Callback = function()
        Fluent:Notify({ Title = "THULLERX STORE", Content = "Iniciando resgate no remote Redeem...", Duration = 3 })
        task.spawn(function()
            for _, code in ipairs(PromoCodes) do
                pcall(function()
                    -- Tenta chamar tanto InvokeServer quanto FireServer por garantia
                    if RedeemRemote:IsA("RemoteFunction") then
                        RedeemRemote:InvokeServer(code)
                    elseif RedeemRemote:IsA("RemoteEvent") then
                        RedeemRemote:FireServer(code)
                    end
                end)
                task.wait(0.4)
            end
            Fluent:Notify({ Title = "THULLERX STORE", Content = "Todos os códigos enviados!", Duration = 4 })
        end)
    end
})

-- 2. BAÚS
Tabs.Chest:AddSection("Coleta Automática de Baús")
Tabs.Chest:AddToggle("AutoChestToggle", {
    Title = "Ativar Auto Farm Baús",
    Default = false,
    Callback = function(Value)
        _G.AutoChest = Value
        if not Value then StopMovement() end
    end
})

-- 3. MOVIMENTO
Tabs.Movement:AddSection("Ajustes de Posição & Voo")
Tabs.Movement:AddSlider("HeightSlider", {
    Title = "Altura do Farm (Distância do NPC)",
    Description = "Deixe em 1 ou 2 para o soco/espada acertar",
    Default = 2, Min = 0, Max = 8, Rounding = 0,
    Callback = function(Value) _G.FarmHeight = Value end
})

Tabs.Movement:AddSlider("SpeedSlider", {
    Title = "Velocidade (Tween Speed)",
    Default = 250, Min = 50, Max = 350, Rounding = 0,
    Callback = function(Value) _G.TweenSpeed = Value end
})

-- 4. APARÊNCIA
Tabs.Settings:AddSection("Temas e Cores do Menu")
Tabs.Settings:AddDropdown("ThemeDropdown", {
    Title = "Tema da Interface",
    Values = {"Darker", "Dark", "Midnight", "Aqua", "Amethyst", "Rose"},
    Default = "Darker",
    Callback = function(Value) Fluent:SetTheme(Value) end
})

local ColorPicker = Tabs.Settings:AddColorpicker("WatermarkColor", {
    Title = "Cor da Marca d'Água & Borda",
    Default = Color3.fromRGB(255, 50, 50)
})

ColorPicker:OnChanged(function()
    TextLabel.TextColor3 = ColorPicker.Value
    MainFrame.BorderColor3 = ColorPicker.Value
end)

-- LOOP PRINCIPAL DO FARM & ATAQUE
task.spawn(function()
    while true do
        task.wait(0.05)
        if _G.AutoFarmLevel then
            pcall(function()
                local enemies = workspace:FindFirstChild("Enemies") or workspace
                local targetEnemy = nil

                -- Procura qualquer NPC vivo perto no jogo
                for _, enemy in pairs(enemies:GetChildren()) do
                    if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                        targetEnemy = enemy
                        break
                    end
                end

                if targetEnemy then
                    -- Cola direto no NPC na altura definida
                    LocalPlayer.Character.HumanoidRootPart.CFrame = targetEnemy.HumanoidRootPart.CFrame * CFrame.new(0, _G.FarmHeight, 0)
                    
                    -- Ataca constantemente
                    DoFastAttack()
                end
            end)
        end
    end
end)

Window:SelectTab(1)
