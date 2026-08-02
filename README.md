-- ======================================================================
-- THULLERX STORE - HUB OFFICIAL BLOX FRUITS (SOLARA FIX)
-- ======================================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "THULLERX STORE",
    SubTitle = "Blox Fruits Hub",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 400),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

-- Marca d'água fixa
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

-- Variáveis Globais de Controle
_G.AutoFarm = false
_G.AutoChest = false
_G.AutoSeaEvents = false
_G.AutoSpinFruit = false
_G.AutoStoreFruit = false
_G.AutoFruitFinder = false

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- Sistema de Dano/Ataque direto do Blox Fruits
local function DoFastAttack()
    pcall(function()
        local char = LocalPlayer.Character
        if char and char:FindFirstChildOfClass("Tool") then
            -- Ativa a animação e aciona o Remote de combate
            char:FindFirstChildOfClass("Tool"):Activate()
            local combatRemote = ReplicatedStorage:FindFirstChild("RigControllerEvent", true) or ReplicatedStorage.Remotes:FindFirstChild("Validator")
            if combatRemote then
                combatRemote:FireServer("weapon")
            end
        end
    end)
end

-- ======================================================================
-- ABAS NO LADO ESQUERDO (SIDEBAR)
-- ======================================================================
local Tabs = {
    Main = Window:AddTab({ Title = "Auto Farm", Icon = "rbxassetid://4483362458" }),
    Sea = Window:AddTab({ Title = "Sea Events", Icon = "rbxassetid://4483362458" }),
    Fruit = Window:AddTab({ Title = "Frutas", Icon = "rbxassetid://4483362458" }),
    Boss = Window:AddTab({ Title = "Bosses & Raids", Icon = "rbxassetid://4483362458" }),
    Settings = Window:AddTab({ Title = "Aparência", Icon = "rbxassetid://4483362458" })
}

-- ======================================================================
-- 1. AUTO FARM
-- ======================================================================
Tabs.Main:AddToggle("AutoFarmToggle", {
    Title = "Auto Farm Mobs Próximos",
    Default = false,
    Callback = function(Value)
        _G.AutoFarm = Value
    end
})

Tabs.Main:AddToggle("AutoChestToggle", {
    Title = "Auto Coletar Baús",
    Default = false,
    Callback = function(Value)
        _G.AutoChest = Value
    end
})

-- Loop de Farm e Teleporte
task.spawn(function()
    while true do
        task.wait(0.05)
        if _G.AutoFarm then
            pcall(function()
                local enemies = workspace:FindFirstChild("Enemies")
                if enemies then
                    for _, enemy in pairs(enemies:GetChildren()) do
                        if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                            while _G.AutoFarm and enemy.Humanoid.Health > 0 do
                                task.wait(0.02)
                                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                                    -- Trava o jogador acima do Mob
                                    LocalPlayer.Character.HumanoidRootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 7, 0)
                                    DoFastAttack()
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- Loop de Coleta de Baús
task.spawn(function()
    while true do
        task.wait(0.3)
        if _G.AutoChest then
            pcall(function()
                for _, obj in pairs(workspace:GetChildren()) do
                    if string.find(obj.Name, "Chest") and obj:IsA("BasePart") then
                        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            LocalPlayer.Character.HumanoidRootPart.CFrame = obj.CFrame
                            task.wait(0.2)
                        end
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- 2. SEA EVENTS
-- ======================================================================
Tabs.Sea:AddToggle("AutoSeaToggle", {
    Title = "Auto Sea Beasts & Terror Sharks",
    Default = false,
    Callback = function(Value)
        _G.AutoSeaEvents = Value
    end
})

task.spawn(function()
    while true do
        task.wait(0.1)
        if _G.AutoSeaEvents then
            pcall(function()
                local seaBeasts = workspace:FindFirstChild("SeaBeasts")
                if seaBeasts then
                    for _, beast in pairs(seaBeasts:GetChildren()) do
                        if beast:FindFirstChild("Humanoid") and beast.Humanoid.Health > 0 then
                            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                                LocalPlayer.Character.HumanoidRootPart.CFrame = beast.HumanoidRootPart.CFrame * CFrame.new(0, 50, 0)
                                DoFastAttack()
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- ======================================================================
-- 3. FRUTAS
-- ======================================================================
Tabs.Fruit:AddToggle("AutoSpinToggle", {
    Title = "Auto Girar Fruta",
    Default = false,
    Callback = function(Value)
        _G.AutoSpinFruit = Value
    end
})

Tabs.Fruit:AddToggle("AutoStoreToggle", {
    Title = "Auto Guardar Frutas",
    Default = false,
    Callback = function(Value)
        _G.AutoStoreFruit = Value
    end
})

Tabs.Fruit:AddToggle("AutoFindToggle", {
    Title = "Auto Coletar Frutas do Chão",
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
