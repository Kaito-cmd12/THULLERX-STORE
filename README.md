
-- ======================================================================
-- THULLERX STORE - HUB OFICIAL BLOX FRUITS
-- Otimizado para Solara Executor
-- ======================================================================

-- 1. Carregamento da Interface Gráfica (UI Library)
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "THULLERX STORE | Blox Fruits Hub",
   LoadingTitle = "Iniciando Script...",
   LoadingSubtitle = "por ThullerX Store",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "ThullerXStore",
      FileName = "BloxFruitsConfig"
   },
   Discord = {
      Enabled = false
   },
   KeySystem = false
})

-- Marca d'água no canto da tela
local ScreenGui = Instance.new("ScreenGui")
local TextLabel = Instance.new("TextLabel")
ScreenGui.Parent = game.CoreGui
TextLabel.Parent = ScreenGui
TextLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
TextLabel.BackgroundTransparency = 0.5
TextLabel.Position = UDim2.new(0.01, 0, 0.01, 0)
TextLabel.Size = UDim2.new(0, 180, 0, 30)
TextLabel.Font = Enum.Font.SourceSansBold
TextLabel.Text = "THULLERX STORE"
TextLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
TextLabel.TextSize = 18.0

-- 2. Variáveis de Controle (Bools para os Loops)
local Flags = {
   AutoFarm = false,
   AutoChest = false,
   AutoSeaEvents = false,
   AutoSpinFruit = false,
   AutoStoreFruit = false,
   AutoFruitFinder = false,
   AutoBoss = false,
   AutoRaid = false,
   SelectedBoss = ""
}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local VirtualUser = game:GetService("VirtualUser")

-- Prevenção de AFK Kick (Anti-AFK)
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new(0, 0))
end)

-- ======================================================================
-- CATEGORIA 1: FARM GERAL & MOBS
-- ======================================================================
local MainTab = Window:CreateTab("Auto Farm", 4483362458)

MainTab:CreateToggle({
   Name = "Auto Farm Level / NPCs Próximos",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoFarm = Value
   end,
})

-- Função de Ataque e Farm Contínuo
task.spawn(function()
   while task.wait(0.1) do
      if Flags.AutoFarm then
         pcall(function()
            -- Busca o NPC mais próximo no workspace
            local closestEnemy = nil
            local shortestDistance = math.huge
            
            for _, enemy in pairs(game.Workspace.Enemies:GetChildren()) do
               if enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
                  local distance = (LocalPlayer.Character.HumanoidRootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
                  if distance < shortestDistance then
                     shortestDistance = distance
                     closestEnemy = enemy
                  end
               end
            end

            -- Teleporta suavemente acima do mob e ataca
            if closestEnemy then
               LocalPlayer.Character.HumanoidRootPart.CFrame = closestEnemy.HumanoidRootPart.CFrame * CFrame.new(0, 10, 0)
               
               -- Simula o clique do mouse (ataque principal da ferramenta equipada)
               VirtualUser:CaptureController()
               VirtualUser:ClickButton1(Vector2.new(50, 50))
            end
         end)
      end
   end
end)

MainTab:CreateToggle({
   Name = "Auto Farm Baús (Chests)",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoChest = Value
   end,
})

-- Loop de Auto Chest
task.spawn(function()
   while task.wait(0.5) do
      if Flags.AutoChest then
         pcall(function()
            for _, v in pairs(game.Workspace:GetChildren()) do
               if string.find(v.Name, "Chest") and v:FindFirstChild("TouchInterest") then
                  LocalPlayer.Character.HumanoidRootPart.CFrame = v.CFrame
                  task.wait(0.2)
               end
            end
         end)
      end
   end
end)

-- ======================================================================
-- CATEGORIA 2: SEA EVENTS & BEASTS
-- ======================================================================
local SeaTab = Window:CreateTab("Sea Events", 4483362458)

SeaTab:CreateToggle({
   Name = "Auto Sea Events (Sea Beasts & Terror Sharks)",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoSeaEvents = Value
   end,
})

task.spawn(function()
   while task.wait(0.2) do
      if Flags.AutoSeaEvents then
         pcall(function()
            -- Procura por entidades de eventos de mar no Workspace
            for _, seaEnemy in pairs(game.Workspace.SeaBeasts:GetChildren()) do
               if seaEnemy:FindFirstChild("Humanoid") and seaEnemy.Humanoid.Health > 0 then
                  -- Posiciona o jogador a uma distância segura acima do Sea Beast
                  LocalPlayer.Character.HumanoidRootPart.CFrame = seaEnemy.HumanoidRootPart.CFrame * CFrame.new(0, 50, 0)
                  VirtualUser:ClickButton1(Vector2.new(50, 50))
               end
            end
         end)
      end
   end
end)

-- ======================================================================
-- CATEGORIA 3: FRUTAS (SPIN, FIND & STORE)
-- ======================================================================
local FruitTab = Window:CreateTab("Frutas", 4483362458)

FruitTab:CreateToggle({
   Name = "Auto Girar Fruta (Random Fruit)",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoSpinFruit = Value
   end,
})

task.spawn(function()
   while task.wait(5) do
      if Flags.AutoSpinFruit then
         pcall(function()
            -- Invoca a compra de fruta aleatória no NPC Cousin
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Cousin", "Buy")
         end)
      end
   end
end)

FruitTab:CreateToggle({
   Name = "Auto Guardar Fruta (Store Fruit)",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoStoreFruit = Value
   end,
})

task.spawn(function()
   while task.wait(2) do
      if Flags.AutoStoreFruit then
         pcall(function()
            for _, tool in pairs(LocalPlayer.Backpack:GetChildren()) do
               if string.find(tool.Name, "Fruit") then
                  game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("StoreFruit", tool.Name, tool)
               end
            end
         end)
      end
   end
end)

FruitTab:CreateToggle({
   Name = "Auto Coletar Fruta Spawnada",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoFruitFinder = Value
   end,
})

task.spawn(function()
   while task.wait(1) do
      if Flags.AutoFruitFinder then
         pcall(function()
            for _, obj in pairs(game.Workspace:GetChildren()) do
               if string.find(obj.Name, "Fruit") and obj:FindFirstChild("Handle") then
                  LocalPlayer.Character.HumanoidRootPart.CFrame = obj.Handle.CFrame
               end
            end
         end)
      end
   end
end)

-- ======================================================================
-- CATEGORIA 4: BOSSES & RAIDS
-- ======================================================================
local BossTab = Window:CreateTab("Bosses & Raids", 4483362458)

BossTab:CreateDropdown({
   Name = "Selecionar Boss",
   Options = {"Boss 1", "Boss 2", "Doflamingo", "Rip Indra"},
   CurrentOption = {"Boss 1"},
   Callback = function(Option)
      Flags.SelectedBoss = Option[1]
   end,
})

BossTab:CreateToggle({
   Name = "Auto Farm Boss",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoBoss = Value
   end,
})

BossTab:CreateToggle({
   Name = "Auto Raid (Iniciar e Completar)",
   CurrentValue = false,
   Callback = function(Value)
      Flags.AutoRaid = Value
   end,
})

-- Notificação Inicial da UI
Rayfield:Notify({
   Title = "THULLERX STORE Loaded",
   Content = "Script carregado com sucesso no Solara!",
   Duration = 5,
   Image = 4483362458,
})
