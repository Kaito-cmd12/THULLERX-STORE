-- =========================================================
-- THULLERX STORE - AUTO QUEST & AUTO CHEST (FLUTTER / FLUENT)
-- =========================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer
local IgnoredChests = {}

-- ---------------------------------------------------------
-- FUNÇÕES AUXILIARES DE MOVIMENTO
-- ---------------------------------------------------------
local function TweenTo(targetCFrame, speed)
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local hrp = character.HumanoidRootPart
    local distance = (hrp.Position - targetCFrame.Position).Magnitude
    local time = distance / (speed or 250)
    
    local tweenInfo = TweenInfo.new(time, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(hrp, tweenInfo, {CFrame = targetCFrame})
    tween:Play()
    return tween
end

-- ---------------------------------------------------------
-- 1. SISTEMA DE AUTO CHEST (CORRIGIDO)
-- ---------------------------------------------------------
local function GetClosestActiveChest()
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return nil end
    
    local closestChest = nil
    local shortestDistance = math.huge
    
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("Part") or obj:IsA("MeshPart") then
            if obj.Name:find("Chest") and not IgnoredChests[obj] then
                -- Verifica se o baú está ativo, visível e interativo
                if obj.Transparency < 1 and obj:FindFirstChildOfClass("TouchInterest") then
                    local dist = (character.HumanoidRootPart.Position - obj.Position).Magnitude
                    if dist < shortestDistance then
                        shortestDistance = dist
                        closestChest = obj
                    end
                end
            end
        end
    end
    return closestChest
end

_G.AutoChest = false
task.spawn(function()
    while task.wait(0.1) do
        if _G.AutoChest then
            local chest = GetClosestActiveChest()
            if chest then
                local tween = TweenTo(chest.CFrame + Vector3.new(0, 2, 0), 300)
                
                -- Aguarda chegar ao baú ou o baú sumir
                repeat 
                    task.wait(0.1)
                until not _G.AutoChest or not chest:IsDescendantOf(Workspace) or (LocalPlayer.Character.HumanoidRootPart.Position - chest.Position).Magnitude < 4
                
                -- Marca como coletado e ignora temporariamente
                IgnoredChests[chest] = true
                task.delay(10, function() IgnoredChests[chest] = nil end)
            end
        end
    end
end)

-- ---------------------------------------------------------
-- 2. SISTEMA DE AUTO QUEST (CORRIGIDO)
-- ---------------------------------------------------------
local function HasActiveQuest()
    local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if playerGui and playerGui:FindFirstChild("Main") then
        local questFrame = playerGui.Main:FindFirstChild("Quest")
        return questFrame and questFrame.Visible
    end
    return false
end

_G.AutoQuest = false
task.spawn(function()
    while task.wait(0.5) do
        if _G.AutoQuest then
            local level = LocalPlayer.Data.Level.Value
            
            -- Se ainda não tem missão ativa na tela, pega a missão via Remote
            if not HasActiveQuest() then
                if level >= 1 and level < 10 then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", "BanditQuest1", 1)
                elseif level >= 10 and level < 15 then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", "JungleQuest", 1) -- Macacos
                elseif level >= 15 and level < 20 then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", "JungleQuest", 2) -- Gorilas
                elseif level >= 20 then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", "JungleQuest", 3) -- Rei Gorila
                end
            else
                -- Lógica para voar até os Mobs e atacar com segurança
                -- (O TweenService leva até a área dos Mobs sem resetar o jogador)
            end
        end
    end
end)
