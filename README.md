-- ======================================================================
-- THULLERX STORE - HUB OFICIAL BLOX FRUITS (SIDEBAR & COLOR PICKER)
-- ======================================================================

local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

local Window = OrionLib:MakeWindow({
    Name = "THULLERX STORE | Blox Fruits Hub",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "ThullerXStoreConfig",
    IntroEnabled = true,
    IntroText = "THULLERX STORE",
    IntroIcon = "rbxassetid://4483362458"
})

-- Marca d'água no canto superior esquerdo
local ScreenGui = Instance.new("ScreenGui")
local TextLabel = Instance.new("TextLabel")
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

-- Variáveis de estado
_G.AutoFarm = false
_G.AutoChest = false
_G.AutoSeaEvents = false
_G.AutoSpinFruit = false
_G.AutoStoreFruit = false
_G.AutoFruitFinder = false

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Prevenção Anti-AFK
LocalPlayer.Idled:Connect(function()
    local vu = game:GetService("VirtualUser")
    vu:CaptureController()
    vu:ClickButton2(Vector2.new(0,0))
end)

-- Função Auxiliar de Ataque Solara
local function AutoAttack()
    local char = LocalPlayer.Character
    if char then
        if not char:FindFirstChildOfClass("Tool") then
            local tool = LocalPlayer.Backpack:FindFirstChildOfClass("Tool")
            if tool then
                char.Humanoid:EquipTool(tool)
            end
        end
        local currentTool = char:FindFirstChildOfClass("Tool")
        if currentTool and currentTool:FindFirstChild("Activate") then
            currentTool:Activate()
        end
    end
end

-- ======================================================================
-- ABAS / CATEGORIAS (BARRA LATERAL À ESQUERDA)
-- ======================================================================

local FarmTab = Window:MakeTab({
    Name = "Auto Farm",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

local SeaTab = Window:MakeTab({
    Name = "Sea Events",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

local FruitTab = Window:MakeTab({
    Name = "Frutas",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

local BossTab = Window:MakeTab({
    Name = "Bosses & Raids",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

local ConfigTab = Window:MakeTab({
    Name = "Personalização",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

-- ======================================================================
-- ABA 1: AUTO FARM
-- ======================================================================
FarmTab:AddToggle({
    Name = "Auto Farm Mobs Próximos",
    Default = false,
    Callback = function(Value)
        _G.AutoFarm = Value
    end    
})

FarmTab:AddToggle({
    Name = "Auto Coletar Baús (Chests)",
    Default = false,
    Callback = function(Value)
        _G.AutoChest = Value
    end    
})

-- Loop de Farm Mobs
task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoFarm then
            pcall(function()
                local enemies = workspace:FindFirstChild("Enemies")
                if enemies then
                    for _, enemy in pairs(enemies:GetChildren()) do
                        if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                            while _G.AutoFarm and enemy.Humanoid.Health > 0 do
                                task.wait(0.05)
                                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                                    LocalPlayer.Character.HumanoidRootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 8, 0)
                                    AutoAttack()
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- Loop de Baús
task.spawn(function()
    while true do
        task.wait(0.5)
        if _G.AutoChest then
            pcall(function()
                for _, obj in pairs(workspace:GetChildren()) do
                    if string.find(obj.Name, "Chest") and obj:IsA("BasePart") then
                        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            LocalPlayer.Character.HumanoidRootPart.CFrame = obj.CFrame
                            task.wait(0.3)
                        end
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- ABA 2: SEA EVENTS
-- ======================================================================
SeaTab:AddToggle({
    Name = "Auto Atacar Sea Beasts / Terror Sharks",
    Default = false,
    Callback = function(Value)
        _G.AutoSeaEvents = Value
    end    
})

task.spawn(function()
    while true do
        task.wait(0.2)
        if _G.AutoSeaEvents then
            pcall(function()
                local seaBeasts = workspace:FindFirstChild("SeaBeasts")
                if seaBeasts then
                    for _, beast in pairs(seaBeasts:GetChildren()) do
                        if beast:FindFirstChild("Humanoid") and beast.Humanoid.Health > 0 then
                            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                                LocalPlayer.Character.HumanoidRootPart.CFrame = beast.HumanoidRootPart.CFrame * CFrame.new(0, 60, 0)
                                AutoAttack()
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- ABA 3: FRUTAS
-- ======================================================================
FruitTab:AddToggle({
    Name = "Auto Girar Fruta (Random Fruit)",
    Default = false,
    Callback = function(Value)
        _G.AutoSpinFruit = Value
    end    
})

FruitTab:AddToggle({
    Name = "Auto Guardar Frutas no Inventário",
    Default = false,
    Callback = function(Value)
        _G.AutoStoreFruit = Value
    end    
})

FruitTab:AddToggle({
    Name = "Auto Coletar Frutas Spawnadas",
    Default = false,
    Callback = function(Value)
        _G.AutoFruitFinder = Value
    end    
})

task.spawn(function()
    while true do
        task.wait(2)
        if _G.AutoSpinFruit then
            pcall(function()
                ReplicatedStorage.Remotes.CommF_:InvokeServer("Cousin", "Buy")
            end)
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

task.spawn(function()
    while true do
        task.wait(1)
        if _G.AutoFruitFinder then
            pcall(function()
                for _, obj in pairs(workspace:GetChildren()) do
                    if string.find(obj.Name, "Fruit") then
                        local handle = obj:FindFirstChild("Handle") or obj:FindFirstChildOfClass("Part")
                        if handle and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            LocalPlayer.Character.HumanoidRootPart.CFrame = handle.CFrame
                        end
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- ABA 4: BOSSES & RAIDS
-- ======================================================================
BossTab:AddDropdown({
    Name = "Selecionar Boss",
    Default = "Nenhum",
    Options = {"Boss 1", "Boss 2", "Doflamingo", "Rip Indra"},
    Callback = function(Value)
        print("Boss selecionado: " .. Value)
    end    
})

-- ======================================================================
-- ABA 5: PERSONALIZAÇÃO DE COR
-- ======================================================================
ConfigTab:AddColorpicker({
    Name = "Cor do Texto da Marca d'Água",
    Default = Color3.fromRGB(255, 50, 50),
    Callback = function(Value)
        TextLabel.TextColor3 = Value
    end	
})

ConfigTab:AddColorpicker({
    Name = "Cor de Destaque do Menu",
    Default = Color3.fromRGB(255, 50, 50),
    Callback = function(Value)
        -- Atualiza dinamicamente as cores dos elementos principais do menu
        OrionLib:ChangeTheme({
            Main = Color3.fromRGB(25, 25, 25),
            Second = Color3.fromRGB(35, 35, 35),
            Stroke = Color3.fromRGB(60, 60, 60),
            Divider = Color3.fromRGB(60, 60, 60),
            Text = Color3.fromRGB(240, 240, 240),
            TextDark = Color3.fromRGB(150, 150, 150),
            Accent = Value
        })
    end	
})

OrionLib:Init()
