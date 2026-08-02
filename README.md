-- ======================================================================
-- THULLERX STORE - HUB OFICIAL BLOX FRUITS (SOLARA ULTRA FIX)
-- ======================================================================

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "THULLERX STORE | Blox Fruits",
   LoadingTitle = "Iniciando Script...",
   LoadingSubtitle = "por ThullerX Store",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false,
   -- Define a barra de abas na ESQUERDA (Sidebar) em vez do topo
   TabWidth = 160,
   SubTitle = "THULLERX STORE",
})

-- Marca d'água no canto da tela
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

-- Variáveis de Controle
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
-- ABAS / CATEGORIAS (SIDEBAR ESQUERDA)
-- ======================================================================
local MainTab = Window:CreateTab("Auto Farm", 4483362458)
local SeaTab = Window:CreateTab("Sea Events", 4483362458)
local FruitTab = Window:CreateTab("Frutas", 4483362458)
local BossTab = Window:CreateTab("Bosses & Raids", 4483362458)
local ConfigTab = Window:CreateTab("Aparência", 4483362458)

-- ======================================================================
-- 1. AUTO FARM
-- ======================================================================
MainTab:CreateToggle({
   Name = "Auto Farm Mobs Próximos",
   CurrentValue = false,
   Callback = function(Value)
      _G.AutoFarm = Value
   end,
})

MainTab:CreateToggle({
   Name = "Auto Farm Baús (Chests)",
   CurrentValue = false,
   Callback = function(Value)
      _G.AutoChest = Value
   end,
})

-- Loop Farm Mobs
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

-- Loop Baús
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
-- 2. SEA EVENTS
-- ======================================================================
SeaTab:CreateToggle({
   Name = "Auto Sea Events (Sea Beasts & Terror Sharks)",
   CurrentValue = false,
   Callback = function(Value)
      _G.AutoSeaEvents = Value
   end,
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
-- 3. FRUTAS
-- ======================================================================
FruitTab:CreateToggle({
   Name = "Auto Girar Fruta (Random Fruit)",
   CurrentValue = false,
   Callback = function(Value)
      _G.AutoSpinFruit = Value
   end,
})

FruitTab:CreateToggle({
   Name = "Auto Guardar Fruta no Inventário",
   CurrentValue = false,
   Callback = function(Value)
      _G.AutoStoreFruit = Value
   end,
})

FruitTab:CreateToggle({
   Name = "Auto Coletar Fruta do Chão",
   CurrentValue = false,
   Callback = function(Value)
      _G.AutoFruitFinder = Value
   end,
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
-- 4. BOSSES & RAIDS
-- ======================================================================
BossTab:CreateDropdown({
   Name = "Selecionar Boss",
   Options = {"Boss 1", "Boss 2", "Doflamingo", "Rip Indra"},
   CurrentOption = {"Boss 1"},
   Callback = function(Option)
      print("Boss selecionado: " .. Option[1])
   end,
})

-- ======================================================================
-- 5. PERSONALIZAÇÃO DE COR
-- ======================================================================
ConfigTab:CreateColorPicker({
    Name = "Cor do Texto 'THULLERX STORE'",
    Color = Color3.fromRGB(255, 50, 50),
    Callback = function(Value)
        TextLabel.TextColor3 = Value
    end
})

Rayfield:Notify({
   Title = "THULLERX STORE",
   Content = "Carregado com sucesso sem erros!",
   Duration = 3,
})
